---
title: "Building a Modern Portfolio Website with Hugo and GitHub Pages"
date: 2025-05-22
draft: false
description:
  "Creating a clean, professional portfolio website using Hugo static site
  generator, custom CSS, and automated deployment"
tags:
  [
    "hugo",
    "web-development",
    "github-pages",
    "portfolio",
    "static-site",
    "css",
    "automation",
  ]
---

Every developer needs a professional online presence. This website represents my
journey in creating a clean, fast, and maintainable portfolio using modern web
development practices and static site generation.

## Design Philosophy {#design-philosophy}

When designing this portfolio, I followed several key principles:

**Simplicity First**: Clean, minimalist design focusing on content over flashy
effects **Performance**: Fast loading times with optimized assets and static
generation **Accessibility**: Proper semantic HTML and responsive design
**Professionalism**: Removed unnecessary emojis and distractions for a
business-ready appearance **Blog-Centric**: Emphasis on sharing knowledge and
project insights

## Technology Stack

### Hugo Static Site Generator

**Why Hugo?**

- **Speed**: Builds sites in milliseconds, not seconds
- **Flexibility**: Powerful templating and content management
- **SEO-friendly**: Clean URLs and meta tag support
- **GitHub Pages integration**: Seamless deployment workflow

### PaperMod Theme

Started with PaperMod theme for its clean aesthetic and extensive customization
options:

```toml
[params]
  env = "production"
  title = "Shubham Prasad"
  description = "Finance and Machine Learning Enthusiast"
  keywords = ["finance", "machine learning", "portfolio"]
  author = "Shubham Prasad"

  defaultTheme = "auto"
  disableThemeToggle = false

  ShowReadingTime = true
  ShowShareButtons = true
  ShowPostNavLinks = true
  ShowBreadCrumbs = true
  ShowCodeCopyButtons = true
  ShowWordCount = true
  ShowRssButtonInSectionTermList = true
```

## Site Architecture

### Content Structure

```
content/
├── _index.md           # Homepage with blog focus
├── about/
│   └── _index.md       # Personal introduction and featured projects
├── cv/
│   └── _index.md       # Professional resume/CV
└── blog/
    ├── _index.md       # Blog listing page
    ├── yoga-pose-detector.md
    ├── sign-language-detection.md
    ├── home-server-infrastructure.md
    └── portfolio-website.md
```

### Navigation Design

Implemented a clean three-section navigation inspired by modern developer
portfolios:

```toml
[[menu.main]]
name = "About"
url = "/about/"
weight = 10

[[menu.main]]
name = "CV"
url = "/cv/"
weight = 20

[[menu.main]]
name = "Blog"
url = "/blog/"
weight = 30
```

## Custom Styling

### Professional CSS Enhancements

Created `/static/css/custom.css` for site-specific improvements:

```css
/* Professional typography */
.post-content h1,
.post-content h2,
.post-content h3 {
  color: var(--primary);
  margin-top: 2rem;
  margin-bottom: 1rem;
}

/* Blog post styling */
.blog-post-meta {
  display: flex;
  align-items: center;
  gap: 1rem;
  margin-bottom: 1.5rem;
  padding-bottom: 1rem;
  border-bottom: 1px solid var(--border);
}

/* Code syntax highlighting */
.chroma .hl {
  background-color: var(--code-bg);
}

pre code {
  background: transparent;
}

/* Responsive design improvements */
@media (max-width: 768px) {
  .cv-section {
    padding: 1rem 0;
  }

  .cv-item {
    flex-direction: column;
    align-items: flex-start;
  }
}
```

### CV/Resume Styling

Implemented professional resume formatting with custom CSS classes:

```css
.cv-section {
  margin-bottom: 2.5rem;
  padding: 1.5rem 0;
  border-bottom: 1px solid var(--border);
}

.cv-item {
  display: flex;
  justify-content: space-between;
  align-items: flex-start;
  margin-bottom: 1.5rem;
}

.cv-title-date {
  display: flex;
  justify-content: space-between;
  align-items: center;
  width: 100%;
  margin-bottom: 0.5rem;
}

.cv-company-location {
  display: flex;
  justify-content: space-between;
  align-items: center;
  font-style: italic;
  color: var(--secondary);
  margin-bottom: 1rem;
}
```

## Content Strategy

### Homepage Design

Transformed the traditional portfolio approach into a blog-centric homepage:

```markdown
# Hello, I'm Shubham 👋

Welcome to my corner of the internet where I share insights about **finance**,
**machine learning**, and **technology**.

Currently pursuing Post Graduate Diploma in Finance and Machine Learning at
Indian Statistical Institute.

## Latest Posts

- [Building a Real-time Yoga Pose Detector with PyTorch and MediaPipe](yoga-pose-detector/)
- [Building a Sign Language Detection System with Computer Vision](sign-language-detection/)
- [Building a Self-Hosted Home Server Infrastructure](home-server-infrastructure/)

## What I Do

I'm passionate about applying AI solutions to real-world problems and exploring
the intersection of finance and technology.
```

### Blog Integration

Implemented proper blog functionality with:

- **Tag system**: Categorize posts by technology and topic
- **Reading time**: Estimated reading duration for each post
- **Code highlighting**: Syntax highlighting for technical content
- **Social sharing**: Built-in sharing buttons
- **SEO optimization**: Meta descriptions and structured data

## Development Workflow

### Local Development

```bash
# Clone repository
git clone https://github.com/shubhpsd/shubhpsd.github.io.git
cd shubhpsd.github.io

# Install Hugo (macOS)
brew install hugo

# Start development server
hugo server -D

# Build for production
hugo --minify
```

### GitHub Actions Deployment

Automated deployment pipeline using GitHub Actions:

```yaml
name: Deploy Hugo site to Pages

on:
  push:
    branches: ["main"]
  workflow_dispatch:

permissions:
  contents: read
  pages: write
  id-token: write

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout
        uses: actions/checkout@v4
        with:
          submodules: recursive

      - name: Setup Hugo
        uses: peaceiris/actions-hugo@v2
        with:
          hugo-version: "latest"
          extended: true

      - name: Build
        run: hugo --minify

      - name: Upload artifact
        uses: actions/upload-pages-artifact@v2
        with:
          path: ./public

  deploy:
    environment:
      name: github-pages
      url: ${{ steps.deployment.outputs.page_url }}
    runs-on: ubuntu-latest
    needs: build
    steps:
      - name: Deploy to GitHub Pages
        id: deployment
        uses: actions/deploy-pages@v2
```

## Performance Optimizations

### Static Generation Benefits

- **Sub-second load times**: Pre-generated HTML serves instantly
- **CDN optimization**: GitHub Pages global distribution
- **Minimal JavaScript**: Enhanced performance on all devices
- **SEO advantages**: Search engines can easily crawl static content

### Image Optimization

```yaml
# Hugo image processing
{{ $image := .Page.Resources.GetMatch "featured-image.jpg" }}
{{ $resized := $image.Resize "800x q80" }}
<img src="{{ $resized.RelPermalink }}" alt="Featured image" loading="lazy">
```

### Lighthouse Scores

- **Performance**: 95/100
- **Accessibility**: 100/100
- **Best Practices**: 92/100
- **SEO**: 100/100

## Content Management

### Markdown Workflow

All content written in Markdown with frontmatter for metadata:

```yaml
---
title: "Building a Modern Portfolio Website"
date: 2025-05-22
draft: false
description: "Creating a professional portfolio with Hugo"
tags: ["hugo", "web-development", "portfolio"]
---
```

### Asset Organization

```
static/
├── css/
│   └── custom.css
├── images/
│   ├── avatar.jpg
│   └── projects/
└── docs/
    └── resume.pdf
```

## SEO and Analytics

### Meta Tags Implementation

```html
<meta name="description" content="{{ .Description }}">
<meta name="keywords" content="{{ delimit .Keywords ", " }}">
<meta property="og:title" content="{{ .Title }}">
<meta property="og:description" content="{{ .Description }}">
<meta property="og:type" content="website">
```

### Structured Data

```json
{
  "@context": "https://schema.org",
  "@type": "Person",
  "name": "Shubham Prasad",
  "jobTitle": "Finance and ML Student",
  "url": "https://shubhpsd.github.io",
  "sameAs": ["https://linkedin.com/in/shubhpsd", "https://github.com/shubhpsd"]
}
```

## Challenges and Solutions

### 1. Theme Customization

**Challenge**: PaperMod theme needed extensive customization **Solution**:
Override theme templates and add custom CSS

### 2. Blog Integration

**Challenge**: Converting from project-focused to blog-centric design
**Solution**: Restructured navigation and created comprehensive blog posts

### 3. Professional Appearance

**Challenge**: Removing casual elements while maintaining personality
**Solution**: Selective emoji use and professional color scheme

### 4. Mobile Responsiveness

**Challenge**: Ensuring perfect mobile experience **Solution**: Custom CSS media
queries and testing across devices

## Maintenance and Updates

### Content Updates

- **Regular blogging**: Share new projects and insights
- **CV updates**: Keep professional information current
- **Project additions**: Document new work and learnings

### Technical Maintenance

```bash
# Update Hugo version
brew upgrade hugo

# Update theme
git submodule update --remote --merge

# Deploy updates
git add .
git commit -m "Update content"
git push origin main
```

## Results and Impact

**Professional Presence**: Clean, modern website representing technical
capabilities **Knowledge Sharing**: Platform for documenting projects and
insights **Career Benefits**: Professional URL for resumes and applications
**Learning Platform**: Hands-on experience with modern web technologies

## Technical Skills Developed

**Static Site Generation**: Hugo templating and build processes **CSS/HTML**:
Custom styling and responsive design **Git Workflow**: Version control and
automated deployment **Performance Optimization**: Web vitals and loading speed
**SEO**: Search engine optimization and structured data

## Future Enhancements

**Planned Features**:

- **Comments system**: Utterances or Disqus integration
- **Newsletter**: RSS feed and email subscriptions
- **Project galleries**: Enhanced project showcases
- **Search functionality**: Site-wide content search

**Technical Improvements**:

- **Progressive Web App**: Offline capabilities
- **Advanced analytics**: Detailed visitor insights
- **Performance monitoring**: Core Web Vitals tracking
- **Content management**: Headless CMS integration

This portfolio represents more than just a website—it's a living document of my
professional journey and a platform for sharing knowledge with the developer
community.

---

**Technologies Used**: Hugo, GitHub Pages, GitHub Actions, HTML/CSS, Markdown,
Git

**Live Site**: [shubhpsd.github.io](https://shubhpsd.github.io) **Source Code**:
[GitHub Repository](https://github.com/shubhpsd/shubhpsd.github.io)
