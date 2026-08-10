---
layout: post
title: "ISA Extension in gem5 | Part 2 : Tag Manipulation Instructions"
description: Implementing ADDG, SUBG, and GMI, the arithmetic tag manipulation instructions of Arm MTE.
date: 2026-08-09 22:00:00
categories: [dev_log]
tags: [gem5, arm, arm_isa, arm_mte, mte, feat_mte, memory_tagging, architectural_simulation]
author: saber
---

With `IRG` working and tested in SE mode in [Part 1](2026-08-09-isa-extension-in-gem5-part1.md), it's time to tackle the rest of the **tag manipulation** group. These are the instructions that let you do arithmetic on tagged pointers, such as adjusting their address while also cycling through a set of candidate tags, or building exclusion masks for later `IRG` calls. None of these touch memory or CPU microarchitecture, which keeps them in the same comfortable territory as `IRG`.

The three instructions in this post are:

| Instruction | Operation |
|-------------|-----------|
| `ADDG Xd, Xn, uimm6, uimm4` | Add byte offset to address, select a new tag |
| `SUBG Xd, Xn, uimm6, uimm4` | Subtract byte offset from address, select a new tag |
| `GMI  Xd, Xn, Xm`           | Record the tag of a pointer into a bitmask |

`ADDG` and `SUBG` are closely related. They're essentially the same operation with the sign of the offset flipped. `GMI` is even simpler: it's a single bitwise `OR` with a shifted bit. What made this batch interesting was the decoder side, which forced me to learn a bit more about how gem5 handles instruction encodings across different format families.

## Reading the Encodings

### ADDG and SUBG

Looking at the ARM spec, both `ADDG` and `SUBG` are encoded in the **Add/subtract (immediate)** format family, the same family as plain `ADD Xd, Xn, #imm12`. The distinguishing bit is the `shift` field in bits [23:22]. For normal `ADD`/`SUB`, `shift` is `0b00` (no shift) or `0b01` (shift imm12 by 12). The value `0b10` is marked *reserved* in the base ISA, but MTE reuses it to signal an ADDG or SUBG variant. At that point, bits [21:16] become a 6-bit immediate for the byte offset (`uimm6`) and bits [13:10] become a 4-bit immediate for the tag offset (`uimm4`). The `op|S` field (bits [30:29]) then distinguishes `ADDG` (`0b00`) from `SUBG` (`0b10`).

![The ADDG instruction's encoding](/assets/img/figures/addg_instruction.svg){: .dark-invert}
<em>The ADDG instruction's encoding.</em>

In `decodeDataProcImm()` in `src/arch/arm/isa/formats/aarch64.isa`, the clean gem5 has the `case 0x2/0x3` block that checks `shift` and immediately returns `Unknown64` if it's `0b10`. Adding ADDG/SUBG means inserting that check first:

```cpp
uint8_t shift = bits(machInst, 23, 22);
if (shift == 0x2) {
    // shift==0b10 is the MTE slot for ADDG/SUBG
    uint64_t uimm6 = bits(machInst, 21, 16);
    uint64_t uimm4 = bits(machInst, 13, 10);
    switch (opc) {
      case 0x0: return new Addg(machInst, rdsp, rnsp, uimm6, uimm4);
      case 0x2: return new Subg(machInst, rdsp, rnsp, uimm6, uimm4);
      default:  return new Unknown64(machInst);
    }
}
// ... rest of the normal ADD/SUB handling
```
{: file='src/arch/arm/isa/formats/aarch64.isa'}

### GMI

`GMI` lives in the **Data processing (2 source)** family, alongside `IRG` and `PACGA`. The switch key is bits [15:10]; `IRG` is at `0x4`, and `GMI` sits right next to it at `0x5`. Adding it to `decodeDataProcTwoS()` is one line:

```cpp
case 0x4: return new Irg(machInst, makeSP(rd), makeSP(rn), rm);
case 0x5: return new Gmi(machInst, rdzr, rn, rm);   // <-- new
case 0xc: return new Pacga(machInst, rd, rn, makeSP(rm));
```
{: file='src/arch/arm/isa/formats/aarch64.isa'}

Note that `GMI` writes to the zero register alias (`rdzr`), not `makeSP`, because the result is a plain integer bitmask, not a stack-pointer-compatible address.

## A New ISA Template

`IRG` and `GMI` both use `DataX2RegOp` (one destination, two source registers), which already existed. But `ADDG` and `SUBG` need one register source **and two immediate operands**, so they need a different base class: `DataX1Reg2ImmOp`. The C++ class was already sitting in `src/arch/arm/insts/data64[.hh|.cc]` in gem5, but the ISA parser templates were missing. I added them to `src/arch/arm/isa/templates/data64.isa`:

{% raw %}
```python
def template DataX1Reg2ImmDeclare {{
    class %(class_name)s : public %(base_class)s
    {
      private:
        %(reg_idx_arr_decl)s;
      public:
        %(class_name)s(ExtMachInst machInst, RegIndex _dest,
                      RegIndex _op1, uint64_t _imm1, uint64_t _imm2);
        Fault execute(ExecContext *, trace::InstRecord *) const override;
    };
}};

def template DataX1Reg2ImmConstructor {{
    %(class_name)s::%(class_name)s(ExtMachInst machInst, RegIndex _dest,
                                   RegIndex _op1, uint64_t _imm1,
                                   uint64_t _imm2) :
        %(base_class)s("%(mnemonic)s", machInst, %(op_class)s,
                       _dest, _op1, _imm1, _imm2)
    {
        %(set_reg_idx_arr)s;
        %(constructor)s;
    }
}};
```
{: file='src/arch/arm/isa/templates/data64.isa'}
{% endraw %}

Same pattern as the existing templates, just wired differently. The DSL generates `imm1` and `imm2` as the two immediate operands.

## Helper Functions

All three instructions are simple enough that the main logic fits cleanly in `mte_helpers.cc`. `ADDG` and `SUBG` follow the same shape as `opIRG`: read the current tag from the source address, apply the exclusion policy via `chooseNonExcludedTag()`, then write the result back. The offset is `uimm6 << 4` (since MTE tag granules are 16 bytes, and `LOG2_TAG_GRANULE = 4`):

```cpp
Fault ArmISA::opADDG(ThreadContext *tc, uint64_t operand,
                     uint64_t uimm6, uint64_t uimm4, uint64_t *result)
{
    ExceptionLevel el = currEL(tc);
    GCR gcr_el1 = tc->readMiscReg(MISCREG_GCR_EL1);
    uint8_t startTag = allocationTagFromAddress(operand);
    uint8_t rtag = IsMTEEnabled(tc, el)
        ? chooseNonExcludedTag(tc, startTag, uimm4, gcr_el1.exclude)
        : 0;
    *result = addressWithAllocationTag(
        operand + (uimm6 << LOG2_TAG_GRANULE), rtag);
    return NoFault;
}
```
{: file='src/arch/arm/mte_helpers.cc'}

Now, `opSUBG`. My first version was literally a copy of `opADDG` with a `-` in the address arithmetic, and I moved on feeling pretty good about it. That was wrong, and it took me a while to figure it out.

Here's the catch: `chooseNonExcludedTag()` doesn't do arithmetic. It *walks forward* through the tag space, skipping any tag in the exclude mask. Give it an offset of 1 and it hands you the next usable tag going up. There's no way to ask it for the previous one. So a `SUBG` that passes `uimm4` straight through increments the tag exactly like `ADDG` does. The address goes down, the tag goes up, and `ADDG #1` followed by `SUBG #1` never gets you back where you started.

The fix is to walk the *two's complement* of the offset instead, which lands on the right tag whether or not any tags are excluded:

```cpp
    // chooseNonExcludedTag() only ever advances through the non-excluded
    // sequence, so SUBG has to walk (16 - offset) rather than the offset.
    const uint8_t subOffset = (uint8_t)((16 - (uimm4 & 0xF)) & 0xF);
    uint8_t rtag = IsMTEEnabled(tc, el)
        ? chooseNonExcludedTag(tc, startTag, subOffset, gcr_el1.exclude)
        : 0;
    *result = addressWithAllocationTag(
        operand - (uimm6 << LOG2_TAG_GRANULE), rtag);
```
{: file='src/arch/arm/mte_helpers.cc'}

`GMI` is so simple it can live as an inline in `mte_helpers.hh`: extract the tag from bits [59:56] of the tagged pointer, set that bit in the mask:

```cpp
inline uint64_t opGMI(uint64_t mask, uint64_t taggedAddr)
{
    uint8_t tag = allocationTagFromAddress(taggedAddr);
    return mask | (1ULL << tag);
}
```
{: file='src/arch/arm/mte_helpers.hh'}

## ISA DSL Definitions

With the C++ helpers done, the ISA DSL definitions follow the same factory pattern from [Part 1](/dev_log/2026/06/29/devlog-gem5-advertising-a-cpu-feature-part2.html#sctlr_el1-the-mapsto-trap). `ADDG` and `SUBG` use the new `DataX1Reg2ImmOp` template; `GMI` reuses `DataX2RegOp`:

```python
addgCode = mteEnabledCode() + """
    uint64_t res;
    fault = opADDG(xc->tcBase(), Op164, imm1, imm2, &res);
    XDest = res;
    """
iop = ArmInstObjParams("addg", "Addg", "DataX1Reg2ImmOp", addgCode, [])
header_output  += DataX1Reg2ImmDeclare.subst(iop)
decoder_output += DataX1Reg2ImmConstructor.subst(iop)
exec_output    += BasicExecute.subst(iop)

# SUBG mirrors ADDG
subgCode = mteEnabledCode() + """
    uint64_t res;
    fault = opSUBG(xc->tcBase(), Op164, imm1, imm2, &res);
    XDest = res;
    """
iop = ArmInstObjParams("subg", "Subg", "DataX1Reg2ImmOp", subgCode, [])
# ... same pattern

# GMI uses two register operands (address, mask)
gmiCode = mteEnabledCode() + """
    XDest = opGMI(Op264, Op164);
    """
iop = ArmInstObjParams("gmi", "Gmi", "DataX2RegOp", gmiCode, [])
# ... same pattern
```
{: file='src/arch/arm/isa/insts/mte.isa'}

One thing to pay attention to: for `GMI`, the operand order in the DSL is `opGMI(Op264, Op164)`: `Op264` is the existing mask (second source) and `Op164` is the tagged address (first source). The ARM encoding has `Xn` as the address and `Xm` as the mask, and the DSL wires them as `Op1` -> `Xn`, `Op2` -> `Xm`, so the call order has to match.

## Testing

Here’s a sample gem5 SE test for `ADDG`, `SUBG`, and `GMI`:

```c
/* ADDG: add 16-byte offset, keep tag from source */
uint64_t base = 0x0700000000454200ULL;  // tag=7, addr=0x454200
uint64_t r = addg(base, 16, 0);
// expected: tag=7, addr=0x454210

/* SUBG: subtract 32-byte offset */
r = subg(base, 32, 0);
// expected: tag=7, addr=0x4541e0

/* GMI: build a bitmask from two tagged pointers */
uint64_t addr_tag3 = 0x0300000000001000ULL;
uint64_t addr_tag7 = 0x0700000000002000ULL;
uint64_t mask = 0;
mask = gmi(addr_tag3, mask);  // sets bit 3
mask = gmi(addr_tag7, mask);  // sets bit 7
// expected: mask == (1<<3)|(1<<7) == 0x88
```

Running in SE mode:

```
ADDG/SUBG/GMI instruction tests in gem5 SE mode
 Test: addg xd, xn, #16, #0  (offset +16 bytes, same tag)
   input  = 0x0700000000454200  (tag=0x7, addr=0x454200)
   result = 0x0700000000454210  (tag=0x7, addr=0x454210)
   addr offset: expected 0x454210  got 0x454210  OK
 Test: addg xd, xn, #0, #1  (no offset, advance tag by 1 slot)
   result = 0x0800000000454200  (tag=0x8)
   addr unchanged: OK
 Test: subg xd, xn, #32, #0  (offset -32 bytes, same tag)
   input  = 0x0700000000454200  (tag=0x7, addr=0x454200)
   result = 0x07000000004541e0  (tag=0x7, addr=0x4541e0)
   addr offset: expected 0x4541e0  got 0x4541e0  OK
 Test: gmi — build a mask from two tagged addresses
   after gmi(tag=3): mask=0x0000000000000008  bit3=1  OK
   after gmi(tag=7): mask=0x0000000000000088  bit7=1  OK
   mask has exactly bits 3 and 7 set: OK
ADDG/SUBG/GMI tests done!
```

Everything checks out. Address arithmetic is correct for both `ADDG` and `SUBG`, tag selection advances properly when `uimm4 > 0`, and `GMI` accumulates bits exactly as expected.

I hope this two-part series has given you a better understanding of gem5’s ISA description system and helped you feel more comfortable applying the same principles and workflow when extending an ISA for your own needs.

Again, if you're interested in exploring the full implementation of Arm MTE in gem5, feel free to check out my repo, [gem5-mte](https://github.com/Archimantix/gem5-mte).
