---
layout: archives
icon: fas fa-archive
order: 4
---
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>About Me</title>
    <style>
        * {
            margin: 0;
            padding: 0;
            box-sizing: border-box;
        }

        body {
            font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
            line-height: 1.6;
            color: #333;
            background: linear-gradient(135deg, #547159 0%, #3a4e3f 100%);
            min-height: 100vh;
            padding: 20px;
        }

        .container {
            max-width: 1200px;
            margin: 0 auto;
            background: rgba(255, 255, 255, 0.95);
            border-radius: 20px;
            padding: 40px;
            box-shadow: 0 20px 40px rgba(0, 0, 0, 0.1);
            backdrop-filter: blur(10px);
        }

        .profile-section {
            display: flex;
            align-items: stretch;
            gap: 40px;
            margin-bottom: 40px;
            flex-wrap: wrap;
        }

        .profile-picture {
            width: 250px;
            height: 250px;
            border-radius: 50%;
            object-fit: cover;
            border: 5px solid #fff;
            box-shadow: 0 10px 30px rgba(0, 0, 0, 0.2);
            flex-shrink: 0;
            background: linear-gradient(45deg, #547159, #3a4e3f);
            display: flex;
            align-items: center;
            justify-content: center;
            font-size: 64px;
            color: white;
            font-weight: bold;
        }

        .profile-info {
            flex: 1;
            min-width: 300px;
            display: flex;
            flex-direction: column;
            justify-content: center;
        }

        .name {
            font-size: 2.8em;
            font-weight: 700;
            color: #2c3e50;
            margin-bottom: 10px;
            background: linear-gradient(45deg, #547159, #3a4e3f);
            -webkit-background-clip: text;
            -webkit-text-fill-color: transparent;
            background-clip: text;
        }

        .position {
            font-size: 1.4em;
            color: #7f8c8d;
            margin-bottom: 25px;
            font-weight: 300;
        }

        .links {
            display: flex;
            gap: 20px;
            margin-bottom: 0;
            flex-wrap: wrap;
        }

        .link {
            width: 50px;
            height: 50px;
            background: linear-gradient(45deg, #547159, #3a4e3f);
            color: white;
            text-decoration: none;
            border-radius: 50%;
            display: flex;
            align-items: center;
            justify-content: center;
            font-size: 20px;
            transition: all 0.3s ease;
            box-shadow: 0 4px 15px rgba(84, 113, 89, 0.3);
            position: relative;
        }

        .link:hover {
            transform: translateY(-3px);
            box-shadow: 0 6px 20px rgba(84, 113, 89, 0.4);
        }

        .link::after {
            content: attr(data-tooltip);
            position: absolute;
            bottom: -35px;
            left: 50%;
            transform: translateX(-50%);
            background: rgba(0, 0, 0, 0.8);
            color: white;
            padding: 5px 10px;
            border-radius: 4px;
            font-size: 12px;
            white-space: nowrap;
            opacity: 0;
            visibility: hidden;
            transition: all 0.3s ease;
        }

        .link:hover::after {
            opacity: 1;
            visibility: visible;
        }

        .about-text {
            background: rgba(255, 255, 255, 0.7);
            padding: 30px;
            border-radius: 15px;
            font-size: 1.1em;
            line-height: 1.8;
            border-left: 4px solid #547159;
        }

        .skills {
            margin-top: 40px;
            background: rgba(255, 255, 255, 0.1);
            padding: 30px;
            border-radius: 20px;
            backdrop-filter: blur(10px);
        }

        .skills h3 {
            color: #2c3e50;
            margin-bottom: 25px;
            font-size: 1.6em;
            text-align: center;
            position: relative;
        }

        .skills h3::after {
            content: '';
            position: absolute;
            bottom: -10px;
            left: 50%;
            transform: translateX(-50%);
            width: 80px;
            height: 3px;
            background: linear-gradient(45deg, #547159, #3a4e3f);
            border-radius: 2px;
        }

        .skill-categories {
            display: grid;
            grid-template-columns: repeat(auto-fit, minmax(250px, 1fr));
            gap: 25px;
            margin-top: 30px;
        }

        .skill-category {
            background: rgba(255, 255, 255, 0.9);
            padding: 20px;
            border-radius: 15px;
            box-shadow: 0 5px 15px rgba(0, 0, 0, 0.1);
            transition: transform 0.3s ease;
        }

        .skill-category:hover {
            transform: translateY(-5px);
        }

        .skill-category h4 {
            color: #547159;
            margin-bottom: 15px;
            font-size: 1.2em;
            display: flex;
            align-items: center;
            gap: 10px;
        }

        .skill-category-icon {
            width: 24px;
            height: 24px;
            background: linear-gradient(45deg, #547159, #3a4e3f);
            border-radius: 50%;
            display: flex;
            align-items: center;
            justify-content: center;
            color: white;
            font-size: 12px;
            font-weight: bold;
        }

        .skill-bars {
            display: flex;
            flex-direction: column;
            gap: 12px;
        }

        .skill-bar {
            display: flex;
            justify-content: space-between;
            align-items: center;
            margin-bottom: 8px;
        }

        .skill-name {
            font-weight: 500;
            color: #2c3e50;
            font-size: 0.9em;
        }

        .skill-level {
            display: flex;
            gap: 3px;
        }

        .skill-dot {
            width: 8px;
            height: 8px;
            border-radius: 50%;
            background: #e0e0e0;
            transition: all 0.3s ease;
        }

        .skill-dot.filled {
            background: linear-gradient(45deg, #547159, #3a4e3f);
            transform: scale(1.1);
        }

        .skill-tag {
            background: rgba(84, 113, 89, 0.1);
            color: #547159;
            padding: 8px 15px;
            border-radius: 20px;
            font-size: 0.9em;
            font-weight: 500;
            border: 1px solid rgba(84, 113, 89, 0.3);
        }

        @media (max-width: 768px) {
            .profile-section {
                flex-direction: column;
                text-align: center;
            }
            
            .container {
                padding: 20px;
            }
            
            .name {
                font-size: 2em;
            }
            
            .links {
                justify-content: center;
            }
        }
    </style>
</head>
<body>
    <div class="container">
        <div class="profile-section">
            <div class="profile-picture">
                <!-- Replace with your actual image -->
                <!-- <img src="your-profile-picture.jpg" alt="Profile Picture" class="profile-picture"> -->
                JD
            </div>
            
            <div class="profile-info">
                <h1 class="name">John Doe</h1>
                <p class="position">Full Stack Developer & UI/UX Designer</p>
                
                <div class="links">
                    <a href="https://github.com/johndoe" class="link" data-tooltip="GitHub">
                        <svg width="20" height="20" viewBox="0 0 24 24" fill="currentColor">
                            <path d="M12 0C5.37 0 0 5.37 0 12c0 5.31 3.435 9.795 8.205 11.385.6.105.825-.255.825-.57 0-.285-.015-1.23-.015-2.235-3.015.555-3.795-.735-4.035-1.41-.135-.345-.72-1.41-1.23-1.695-.42-.225-1.02-.78-.015-.795.945-.015 1.62.87 1.845 1.23 1.08 1.815 2.805 1.305 3.495.99.105-.78.42-1.305.765-1.605-2.67-.3-5.46-1.335-5.46-5.925 0-1.305.465-2.385 1.23-3.225-.12-.3-.54-1.53.12-3.18 0 0 1.005-.315 3.3 1.23.96-.27 1.98-.405 3-.405s2.04.135 3 .405c2.295-1.56 3.3-1.23 3.3-1.23.66 1.65.24 2.88.12 3.18.765.84 1.23 1.905 1.23 3.225 0 4.605-2.805 5.625-5.475 5.925.435.375.81 1.095.81 2.22 0 1.605-.015 2.895-.015 3.3 0 .315.225.69.825.57A12.02 12.02 0 0024 12c0-6.63-5.37-12-12-12z"/>
                        </svg>
                    </a>
                    <a href="https://linkedin.com/in/johndoe" class="link" data-tooltip="LinkedIn">
                        <svg width="20" height="20" viewBox="0 0 24 24" fill="currentColor">
                            <path d="M20.447 20.452h-3.554v-5.569c0-1.328-.027-3.037-1.852-3.037-1.853 0-2.136 1.445-2.136 2.939v5.667H9.351V9h3.414v1.561h.046c.477-.9 1.637-1.85 3.37-1.85 3.601 0 4.267 2.37 4.267 5.455v6.286zM5.337 7.433c-1.144 0-2.063-.926-2.063-2.065 0-1.138.92-2.063 2.063-2.063 1.14 0 2.064.925 2.064 2.063 0 1.139-.925 2.065-2.064 2.065zm1.782 13.019H3.555V9h3.564v11.452zM22.225 0H1.771C.792 0 0 .774 0 1.729v20.542C0 23.227.792 24 1.771 24h20.451C23.2 24 24 23.227 24 22.271V1.729C24 .774 23.2 0 22.222 0h.003z"/>
                        </svg>
                    </a>
                    <a href="https://scholar.google.com/citations?user=johndoe" class="link" data-tooltip="Google Scholar">
                        <svg width="20" height="20" viewBox="0 0 24 24" fill="currentColor">
                            <path d="M5.242 13.769L0 9.5 12 0l12 9.5-5.242 4.269C17.548 11.249 14.978 9.5 12 9.5c-2.977 0-5.548 1.748-6.758 4.269zM12 10a7 7 0 1 0 0 14 7 7 0 0 0 0-14z"/>
                        </svg>
                    </a>
                    <a href="mailto:john@example.com" class="link" data-tooltip="Email">
                        <svg width="20" height="20" viewBox="0 0 24 24" fill="currentColor">
                            <path d="M20 4H4c-1.1 0-1.99.9-1.99 2L2 18c0 1.1.9 2 2 2h16c1.1 0 2-.9 2-2V6c0-1.1-.9-2-2-2zm0 4l-8 5-8-5V6l8 5 8-5v2z"/>
                        </svg>
                    </a>
                    <a href="/cv.pdf" class="link" data-tooltip="Download CV">
                        <svg width="20" height="20" viewBox="0 0 24 24" fill="currentColor">
                            <path d="M14,2H6A2,2 0 0,0 4,4V20A2,2 0 0,0 6,22H18A2,2 0 0,0 20,20V8L14,2M18,20H6V4H13V9H18V20Z"/>
                        </svg>
                    </a>
                </div>
            </div>
        </div>
        
        <div class="about-text">
            <p>Hello! I'm a passionate full-stack developer with over 5 years of experience creating beautiful, functional web applications. I love turning complex problems into simple, elegant solutions that users enjoy interacting with.</p>
            
            <p>When I'm not coding, you can find me exploring new technologies, contributing to open-source projects, or enjoying a good cup of coffee while reading about the latest trends in web development. I believe in writing clean, maintainable code and creating user experiences that make a difference.</p>
            
            <p>I'm always open to new opportunities and interesting projects. Feel free to reach out if you'd like to collaborate or just chat about technology!</p>
        </div>
        
        <div class="skills">
            <h3>Skills & Technologies</h3>
            <div class="skill-tags">
                <span class="skill-tag">JavaScript</span>
                <span class="skill-tag">React</span>
                <span class="skill-tag">Node.js</span>
                <span class="skill-tag">Python</span>
                <span class="skill-tag">HTML/CSS</span>
                <span class="skill-tag">MongoDB</span>
                <span class="skill-tag">Git</span>
                <span class="skill-tag">Figma</span>
            </div>
        </div>
    </div>
</body>
</html>