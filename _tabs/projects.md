---
# the default layout is 'page'
icon: fas fa-laptop-code
order: 2
---

<style>
.projects-portfolio {
  font-family: 'Source Sans Pro', 'Microsoft Yahei', sans-serif;
  line-height: 1.6;
  color: var(--text-color);
}

.projects-portfolio > p.intro {
  margin-bottom: 2.5rem;
  font-size: 1.1rem;
  color: var(--text-muted-color);
}

.project-item {
  display: flex;
  align-items: flex-start;
  gap: 1.75rem;
  padding: 1.75rem 0;
  border-bottom: 1px solid var(--main-border-color);
}

.project-item:first-of-type {
  border-top: 1px solid var(--main-border-color);
}

.project-logo {
  flex-shrink: 0;
  width: 96px;
  height: 96px;
  display: flex;
  align-items: center;
  justify-content: center;
}

.project-logo img {
  max-width: 100%;
  max-height: 100%;
  object-fit: contain;
}

.project-logo-placeholder {
  width: 96px;
  height: 96px;
  border-radius: 12px;
  background: linear-gradient(135deg, #547159, #3a4e3f);
  color: #fff;
  font-size: 1.75rem;
  font-weight: 700;
  display: flex;
  align-items: center;
  justify-content: center;
}

.project-body {
  flex: 1;
  min-width: 0;
}

.project-body h3 {
  margin: 0 0 0.6rem;
  font-size: 1.35rem;
  color: var(--heading-color);
}

.project-body p {
  margin: 0 0 0.9rem;
  color: var(--text-color);
}

.project-link {
  display: inline-flex;
  align-items: center;
  gap: 0.4rem;
  color: #547159;
  font-weight: 600;
  text-decoration: none;
}

.project-link:hover {
  text-decoration: underline;
}

html[data-mode='dark'] .project-link,
html:not([data-mode='light']) .project-link {
  color: #8fbf9a;
}

@media (max-width: 576px) {
  .project-item {
    flex-direction: column;
    align-items: center;
    text-align: center;
  }

  .project-link {
    justify-content: center;
  }
}
</style>

<div class="projects-portfolio">
  <p class="intro">
    A selection of projects I am working on across computer architecture,
    systems security, and architectural simulation.
  </p>

  <article class="project-item">
    <div class="project-logo">
      <img
        src="/assets/img/figures/arm-mte-logo.png"
        alt="ARM Memory Tagging Extension logo"
      />
    </div>
    <div class="project-body">
      <h3>ARM Memory Tagging Extension in gem5</h3>
      <p>
        This project develops a functional model of the Arm Memory Tagging Extension (MTE) in the gem5 architectural simulator, enabling research on hardware-supported memory safety in both syscall-emulation and full-system configurations. The implementation is validated against a dedicated regression suite as well as Linux full-system boots in which tagged userspace workloads execute correctly under the kernel’s native fault path. 
      </p>
      <a
        class="project-link"
        href="https://github.com/archimantix/gem5-mte"
        target="_blank"
        rel="noopener noreferrer"
      >
        <i class="fab fa-github"></i>
        View on GitHub
      </a>
    </div>
  </article>
</div>
