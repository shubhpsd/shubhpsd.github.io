---
title: "Building a Modern Portfolio Website with Hugo and Gruvbox Theme"
date: 2025-01-28
draft: false
description:
  "Creating a professional portfolio website using Hugo static site generator,
  Gruvbox theme, JSON Resume integration, and optimized CSS architecture"
tags:
  [
    "hugo",
    "web-development",
    "github-pages",
    "portfolio",
    "gruvbox",
    "json-resume",
    "css-optimization",
    "static-site",
  ]
toc: true
---

Building a professional developer portfolio requires balancing aesthetics,
performance, and maintainability. This website represents my approach to
creating a modern, fast, and content-focused portfolio using Hugo's static site
generation power combined with the distinctive Gruvbox theme.

## Design Philosophy {#design-philosophy}

The foundation of this portfolio rests on several key principles:

**Retro-Modern Aesthetic**: Utilizing the Gruvbox color scheme for a
distinctive, high-contrast design that's both nostalgic and professional

**Performance-First**: Critical CSS inlining and optimized asset delivery for
sub-second load times

**Content-Centric**: Clean typography and layout that prioritizes readability
and knowledge sharing

**Structured Data**: JSON Resume integration for maintainable and standardized
professional information

**Developer Experience**: Modern tooling with PostCSS, automated deployment, and
comprehensive linting

## Technology Stack

### Hugo Static Site Generator

**Why Hugo?**

- **Speed**: Lightning-fast builds measured in milliseconds
- **Flexibility**: Powerful templating system with Go templates
- **Ecosystem**: Rich theme and module ecosystem
- **SEO-Friendly**: Built-in support for sitemaps, robots.txt, and structured
  data

### Gruvbox Theme

Chose the Gruvbox theme for its distinctive retro aesthetic and technical
excellence:

```toml
[params]
  # Aqua color scheme for a professional yet distinctive look
  themeColor = "aqua"
  author = "Shubham Prasad"
  subtitle = "Finance and Machine Learning Enthusiast"
  description = "Portfolio showcasing finance, machine learning, and technology projects"

  [params.logo]
    text = "shubhpsd.github.io"
    url = "/"
```

**Theme Features:**

- High-contrast color palettes optimized for accessibility
- Built-in dark/light mode support with system preference detection
- Prism.js integration for syntax highlighting
- Responsive design with mobile-first approach

## Site Architecture

### Content Structure

```
content/
├── _index.md           # Homepage with personal introduction
├── about.md            # Detailed bio and featured projects
├── cv.md              # JSON Resume integration
└── blog/
    ├── _index.md       # Blog listing
    ├── yoga-pose-detector/
    ├── githubfetch/
    ├── movie-recommendation-system/
    └── portfolio-website.md
```

### Navigation Design

Implemented a clean three-section navigation focusing on core content areas:

```toml
[menu]
  [[menu.main]]
    identifier = "blog"
    name = "Blog"
    url = "/blog"
    weight = 10
  [[menu.main]]
    identifier = "cv"
    name = "CV"
    url = "/cv"
    weight = 20
  [[menu.main]]
    identifier = "about"
    name = "About"
    url = "/about"
    weight = 30
```

## CSS Architecture & Performance

### Critical/Non-Critical CSS Split

Implemented an advanced CSS architecture separating critical and non-critical
styles:

```
assets/css/
├── critical/           # Above-the-fold styles
│   ├── 00-vendor.css  # Normalize.css and reset
│   ├── 15-colors.css  # Gruvbox color variables
│   ├── 20-base.css    # Typography and base styles
│   ├── 25-layout.css  # Grid and layout
│   └── 30-header.css  # Navigation and header
└── non-critical/      # Below-the-fold styles
    ├── 15-footer.css  # Footer styles
    └── 20-pagination.css # Pagination components
```

### Typography System

Custom typography stack combining modern web fonts:

```css
:root {
  --font-monospace: "Fira Code", "Lucida Console", Monaco, monospace;
  --font-sans-serif: Verdana, Helvetica, sans-serif;
  --font-serif: "Roboto Slab", Georgia, serif;
}

html {
  font-family: var(--font-serif);
  font-size: 1rem;
  scroll-behavior: smooth;
}
```

### Gruvbox Color Variables

The theme's distinctive color palette provides excellent contrast and
readability:

```css
:root {
  --bg: #fbf1c7; /* Light background */
  --fg: #3c3836; /* Dark foreground */
  --bg1: #ebdbb2; /* Subtle background */
  --fg0: #282828; /* Strong foreground */
  --aqua1: #689d6a; /* Primary accent */
  --yellow1: #d79921; /* Warning/highlight */
}
```

## JSON Resume Integration

### Structured Professional Data

Implemented JSON Resume standard for maintainable CV content:

```json
{
  "$schema": "https://raw.githubusercontent.com/jsonresume/resume-schema/v1.0.0/schema.json",
  "basics": {
    "name": "Shubham Prasad",
    "label": "Finance and Tech Enthusiast",
    "email": "shubhampsd@tuta.io",
    "summary": "Currently pursuing Post Graduate Diploma in Statistical Methods and Analytics...",
    "location": {
      "city": "Delhi",
      "country": "India"
    }
  },
  "work": [...],
  "education": [...],
  "skills": [...],
  "projects": [...]
}
```

### Shortcode Integration

Created reusable shortcodes for dynamic content rendering:

```html
<!-- layouts/shortcodes/json-resume.html -->
{{ $section := .Get 0 }} {{ $lang := .Page.Language.Lang }} {{ $data := index
.Site.Data.json_resume $lang }} {{ partial (printf "json-resume/%s.html"
$section) $data }}
```

**Usage in Markdown:**

```markdown
## Experience

{{< json-resume "work" >}}

## Education

{{< json-resume "education" >}}
```

## Build Process & Tooling

### PostCSS Pipeline

Modern CSS processing with performance optimizations:

```js
// postcss.config.js
module.exports = {
  plugins: [
    require("postcss-import"),
    require("postcss-custom-media"),
    require("postcss-nesting"),
    require("postcss-preset-env"),
    require("cssnano")({
      preset: "default",
    }),
    require("@fullhuman/postcss-purgecss")({
      content: ["./layouts/**/*.html", "./hugo_stats.json"],
      defaultExtractor: (content) => content.match(/[\w-/:]+(?<!:)/g) || [],
    }),
  ],
};
```

### Dependencies & Asset Management

```json
{
  "dependencies": {
    "@tabler/icons": "^3.31.0",
    "flexsearch": "0.7.43",
    "normalize.css": "^8.0.1",
    "prism-themes": "^1.9.0",
    "prismjs": "^1.29.0",
    "simple-icons": "^13.7.0",
    "typeface-fira-code": "^1.1.13",
    "typeface-roboto-slab": "^1.1.13"
  }
}
```

### Linting & Code Quality

Comprehensive linting setup for maintainable code:

```json
{
  "scripts": {
    "lint": "npm run lint:css && npm run lint:js && npm run lint:md",
    "lint:css": "stylelint --fix **/*.css",
    "lint:js": "eslint --fix .",
    "lint:md": "markdownlint --fix **/*.md"
  }
}
```

## Deployment Pipeline

### GitHub Actions Workflow

Automated deployment with optimized build process:

```yaml
name: Deploy Hugo site to Pages

on:
  push:
    branches: ["main"]
  workflow_dispatch:

jobs:
  build:
    runs-on: ubuntu-latest
    env:
      HUGO_VERSION: 0.147.8
    steps:
      - name: Install Hugo CLI
        run: |
          wget -O ${{ runner.temp }}/hugo.deb https://github.com/gohugoio/hugo/releases/download/v${HUGO_VERSION}/hugo_extended_${HUGO_VERSION}_linux-amd64.deb \
          && sudo dpkg -i ${{ runner.temp }}/hugo.deb

      - name: Install Dart Sass
        run: sudo snap install dart-sass

      - name: Checkout
        uses: actions/checkout@v4
        with:
          submodules: recursive

      - name: Install Node.js dependencies
        run: "[[ -f package-lock.json ]] && npm ci || true"

      - name: Build with Hugo
        env:
          HUGO_ENVIRONMENT: production
        run: hugo --minify --baseURL "${{ steps.pages.outputs.base_url }}/"
```

### Build Optimizations

- **Hugo Extended**: WebP image processing and advanced features
- **Dart Sass**: Latest Sass compilation
- **Asset Minification**: Automatic CSS/JS/HTML minification
- **PurgeCSS**: Remove unused CSS for optimal bundle size

## Performance Optimizations

### Critical CSS Inlining

The theme automatically inlines critical CSS for above-the-fold content:

```html
<!-- layouts/partials/head/stylesheets.html -->
{{ if .Site.IsServer }}
<!-- Development: separate files for debugging -->
<link rel="stylesheet" href="/css/critical.css" />
{{ else }}
<!-- Production: inline critical CSS -->
<style>
  {{ readFile "public/css/critical.css" | safeCSS }}
</style>
{{ end }}
```

### Image Processing

Hugo's built-in image processing for responsive images:

```html
{{ $image := .Page.Resources.GetMatch "featured-image.*" }} {{ if $image }} {{
$webp := $image.Resize "800x webp q80" }} {{ $fallback := $image.Resize "800x
q80" }}
<picture>
  <source srcset="{{ $webp.RelPermalink }}" type="image/webp" />
  <img src="{{ $fallback.RelPermalink }}" alt="{{ .Title }}" loading="lazy" />
</picture>
{{ end }}
```

### Lighthouse Performance Scores

Current performance metrics:

- **Performance**: 98/100
- **Accessibility**: 100/100
- **Best Practices**: 95/100
- **SEO**: 100/100

## Content Strategy

### Blog-First Approach

Transformed from a traditional portfolio to a knowledge-sharing platform:

```markdown
---
title: Hello, I'm Shubham 👋
---

This is the place where I share my thoughts and notes about things that I'm
excited about and working on and hope to connect with people having a similar
mindset. I'd love to hear from you!

Check out my latest blog posts below.
```

### Technical Writing

Each blog post includes:

- **Comprehensive project documentation** with architecture details
- **Code examples** with syntax highlighting
- **Performance metrics** and optimization techniques
- **Lessons learned** and technical challenges

## Advanced Features

### Search Functionality

Integrated FlexSearch for fast client-side search:

```html
<!-- layouts/home.searchindex.json -->
{{- $index := slice -}} {{- range .Site.RegularPages -}} {{- $index = $index |
append (dict "id" .Permalink "title" .Title "content" (.Plain | truncate 200)
"tags" .Params.tags ) -}} {{- end -}} {{- $index | jsonify -}}
```

### Social Sharing

Built-in social sharing with configurable platforms:

```toml
[[params.socialShare]]
  iconSuite = "simple-icon"
  iconName = "linkedin"
  formatString = "https://www.linkedin.com/sharing/share-offsite/?url={url}"
```

### Code Highlighting

Advanced Prism.js configuration with multiple plugins:

```toml
[params.prism]
  languages = [
    "markup", "css", "javascript", "python",
    "bash", "diff", "toml", "json"
  ]
  plugins = [
    "normalize-whitespace",
    "toolbar",
    "copy-to-clipboard",
    "line-numbers",
    "command-line"
  ]
```

## Development Workflow

### Local Development

```bash
# Clone repository
git clone https://github.com/shubhpsd/shubhpsd.github.io.git
cd shubhpsd.github.io

# Install dependencies
npm install

# Start development server
hugo server -D --disableFastRender

# Build for production
npm run lint && hugo --minify
```

### Content Creation Workflow

1. **Create content**: Write Markdown files with frontmatter
2. **Update JSON Resume**: Modify structured data for CV sections
3. **Lint and format**: Run automated code quality checks
4. **Preview locally**: Test with development server
5. **Deploy**: Push to main branch for automatic deployment

## Maintenance & Updates

### Dependency Management

Using Renovate for automated dependency updates:

```json
{
  "extends": ["config:base"],
  "packageRules": [
    {
      "matchDepTypes": ["devDependencies"],
      "automerge": true
    }
  ]
}
```

### Content Updates

Regular maintenance tasks:

- **JSON Resume updates**: Keep professional information current
- **Blog content**: Document new projects and learnings
- **Performance monitoring**: Track Core Web Vitals
- **Security updates**: Automated dependency updates

## Results & Impact

**Professional Presence**: Clean, distinctive website showcasing technical
capabilities

**Knowledge Platform**: Centralized location for documenting projects and
insights

**Performance Excellence**: Sub-second load times with 98/100 Lighthouse score

**Developer Experience**: Modern tooling and automated workflows

**SEO Optimization**: Perfect SEO score with structured data

## Technical Skills Developed

**Hugo Mastery**: Advanced templating, shortcodes, and build optimization

**CSS Architecture**: Critical CSS patterns and performance optimization

**JSON Schema**: Structured data implementation with JSON Resume

**Build Automation**: GitHub Actions, PostCSS, and asset optimization

**Performance Optimization**: Web Vitals, image processing, and bundle
optimization

## Challenges & Solutions

### 1. CSS Performance Optimization

**Challenge**: Balancing design richness with loading performance **Solution**:
Critical/non-critical CSS splitting with automated inlining

### 2. JSON Resume Integration

**Challenge**: Making structured resume data editable and maintainable
**Solution**: Hugo shortcodes with JSON schema validation

### 3. Build Process Complexity

**Challenge**: Managing multiple build tools and dependencies **Solution**:
Comprehensive npm scripts and GitHub Actions automation

### 4. Theme Customization

**Challenge**: Extending Gruvbox theme without breaking updates **Solution**:
Strategic CSS overrides and custom layout partials

## Future Enhancements

**Planned Features:**

- **Comments System**: Utterances integration for blog discussions
- **Newsletter**: RSS feed improvements and email subscriptions
- **Project Galleries**: Enhanced visual project showcases
- **Analytics Dashboard**: Custom visitor insights and metrics

**Technical Improvements:**

- **Progressive Web App**: Service worker for offline capabilities
- **Advanced Search**: Full-text search with filtering
- **Performance Monitoring**: Real User Monitoring (RUM)
- **Content Management**: Headless CMS integration for non-technical updates

This portfolio represents a modern approach to developer websites, combining
aesthetic appeal with technical excellence and optimal performance.

---

**Technologies Used**: Hugo, Gruvbox Theme, GitHub Pages, GitHub Actions,
PostCSS, JSON Resume, Prism.js, FlexSearch

**Live Site**: [shubhpsd.github.io](https://shubhpsd.github.io)  
**Source Code**:
[GitHub Repository](https://github.com/shubhpsd/shubhpsd.github.io)
