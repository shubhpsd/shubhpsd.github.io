---
title: "My Portfolio Website: Hugo, Gruvbox"
date: 2025-01-28
draft: false
description:
  "Building a modern portfolio with Hugo and Gruvbox theme - because apparently
  I enjoy making things harder than they need to be"
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

You know that moment when you decide to build "just a simple portfolio website"
and somehow end up with a fully-featured static site generator setup, custom CSS
architecture, and automated deployment pipelines, with markdown blogging? Yeah,
that's this project.

What started as "I need a website to showcase my projects" turned into a deep
dive into Hugo templating, performance optimization, and way too many hours
tweaking CSS variables. But hey, at least I learned a lot and now have a pretty
neat portfolio that loads faster than most JavaScript frameworks can initialize.

## Why Not Just Use a Dynamic Template?

I wanted to understand every piece of my website. Plus, most portfolio templates
felt either too generic or over-engineered and flashy with features I'd never
use. I wanted something that was:

- **Fast to write content** - because nobody has time for to keep typing, so
  markdown was the way
- **Distinctive** - Gruvbox theme because it's easier to see and more retro
- **Maintainable** - JSON Resume integration so I never have to manually update
  my CV again
- **Mine** - custom everything, down to the build process

## The Hugo Journey

### Why Hugo Over Everything Else

After wrestling with React, WordPress, and even considering hand-coding HTML
(dark times), I landed on Hugo because:

**It's stupid fast**: Build times measured in milliseconds, not minutes. Coming
from JavaScript land, this felt like cheating.

**Go templates**: Once you get past the initial "what the hell is this syntax"
phase, they're actually pretty powerful.

**No runtime**: Static files mean no server crashes at 3 AM nor lag on weaker
browsers or extreme load times.

```toml
# config/_default/hugo.toml
# The magic numbers that make it all work
baseURL = "https://shubhamprasad.me"
languageCode = "en-us"
title = "Shubham Prasad"
theme = "gruvbox"
```

### Custom Domain Setup

Since the site is now hosted at `shubhamprasad.me`, I set up a proper domain
infrastructure:

- **Domain**: Namecheap (better pricing than most)
- **DNS**: Cloudflare (for subdomain management and my homelab projects)
- **Hosting**: Still GitHub Pages, just with custom domain configured
- **SSL**: Cloudflare handles certificates and CDN caching

```toml
# config/_default/hugo.toml
baseURL = "https://shubhamprasad.me"
languageCode = "en-us"
title = "Shubham Prasad"
theme = "gruvbox"

[params]
  themeColor = "aqua"  # Because blue is boring
  author = "Shubham Prasad"
  subtitle = "Finance and Machine Learning Enthusiast"
```

### Wrestling with Gruvbox Theme

Found
[Michael Schnerring's Hugo Gruvbox theme](https://github.com/schnerring/hugo-theme-gruvbox)
and thought "this looks cool, should be easy to customize".

**Why This Theme Rocks:**

The [Gruvbox theme](https://hugo-theme-gruvbox.schnerring.net/) comes with
everything:

- **Prism code highlighting** with dark mode switching
- **FlexSearch** for fast full-text search
- **JSON Resume support** (game-changer for CV pages)
- **Image optimization** with WebP and lazy loading
- **Responsive design** that actually works on mobile
- **Markdown blogging** - write posts, push to GitHub, done

**My Customizations:**

- Critical CSS splitting for performance
- Projects carousel (had to show off somehow)
- Enhanced JSON Resume integration
- Custom build pipeline with PostCSS

The retro aesthetic matched my ricing setup perfectly, and the blogging workflow
is stupidly simple: write markdown, push to GitHub, site rebuilds automatically.

The theme's retro aesthetic perfectly matched my terminal setup, and the high
contrast made everything readable. Plus, it has both light and dark modes that
actually don't hurt your eyes at 2 AM.

## The CSS Architecture

### Critical vs Non-Critical CSS

This is where things got interesting (read: unnecessarily complex). I
implemented a system where critical CSS gets inlined in the `<head>` for
above-the-fold content, while non-critical CSS loads asynchronously.

```text
assets/css/
├── critical/           # The important stuff
│   ├── 00-vendor.css  # Reset the browser's bad decisions
│   ├── 15-colors.css  # Gruvbox color variables
│   ├── 20-base.css    # Typography that doesn't suck
│   ├── 25-layout.css  # Make things not look broken
│   └── 30-header.css  # Navigation magic
└── non-critical/      # The nice-to-have stuff
    ├── 15-footer.css
    └── 20-pagination.css
```

**Did I need this level of optimization for a personal portfolio?** Absolutely
not.  
**Did I do it anyway?** Obviously.  
**Was it worth the 6-hour CSS debugging session?** The 75/100 Lighthouse
performance score says "mostly yes" (those project images are heavy!).

### Typography That Actually Works

Spent way too much time choosing fonts. Ended up with:

```css
:root {
  --font-monospace: "Fira Code", Monaco, monospace;
  --font-sans-serif: Verdana, Helvetica, sans-serif;
  --font-serif: "Roboto Slab", Georgia, serif;
}
```

Fira Code for code blocks (because ligatures are life), Roboto Slab for headings
(because it's readable), and Verdana for body text (because it works on
everything).

## JSON Resume: Easy edits

### The Problem with Traditional CVs

You know what sucks? Having your resume in a Word doc that you have to manually
update every time you change jobs, learn a new skill, or remember that project
you forgot to mention.

### Enter JSON Resume

Found the JSON Resume standard and thought "this is it, this is the future." One
JSON file to rule them all:

```json
{
  "$schema": "https://raw.githubusercontent.com/jsonresume/resume-schema/v1.0.0/schema.json",
  "basics": {
    "name": "Shubham Prasad",
    "label": "Finance and Tech Enthusiast",
    "email": "shubhampsd@tuta.io",
    "summary": "Currently pursuing Post Graduate Diploma in Statistical Methods and Analytics..."
  },
  "work": [
    {
      "name": "Previous Company",
      "position": "Data Analyst",
      "startDate": "2023-01-01",
      "summary": "Did data things, made charts"
    }
  ],
  "skills": [
    {
      "name": "Python",
      "level": "Advanced",
      "keywords": ["pandas", "scikit-learn", "matplotlib"]
    }
  ]
}
```

Then created Hugo shortcodes to render different sections:

```markdown
## Experience

{{</* json-resume "work" */>}}

## Education

{{</* json-resume "education" */>}}

## Skills

{{</* json-resume "skills" */>}}
```

**Result**: Update one JSON file, and it automatically updates everywhere. My CV
page and any future integrations all stay in sync. It's beautiful.

## Build Process: Making Simple Things Complex

### PostCSS Pipeline

Because apparently using regular CSS is for peasants, the theme had a PostCSS
pipeline:

```js
// postcss.config.js
module.exports = {
  plugins: [
    require("postcss-import"), // Import other CSS files
    require("postcss-nesting"), // Nest CSS like Sass
    require("postcss-preset-env"), // Use future CSS today
    require("cssnano")({
      // Make CSS tiny
      preset: "default",
    }),
    require("@fullhuman/postcss-purgecss")({
      // Remove unused CSS
      content: ["./layouts/**/*.html", "./hugo_stats.json"],
    }),
  ],
};
```

**Translation**: Take perfectly good CSS, run it through 6 different processors,
and spit out optimized CSS that's smaller than the original and renders 10x
faster. Modern web development in a nutshell.

### Automated Deployment

Set up GitHub Actions to automatically deploy when I push to main:

```yaml
# .github/workflows/hugo.yml
name: Deploy Hugo site to Pages

on:
  push:
    branches: ["main"]

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - name: Install Hugo CLI
        run: |
          wget -O ${{ runner.temp }}/hugo.deb https://github.com/gohugoio/hugo/releases/download/v${HUGO_VERSION}/hugo_extended_${HUGO_VERSION}_linux-amd64.deb
          && sudo dpkg -i ${{ runner.temp }}/hugo.deb

      - name: Build with Hugo
        run: hugo --minify --baseURL "${{ steps.pages.outputs.base_url }}/"
```

Now I can write a blog post, push to GitHub, and have it live in under 2
minutes. Future me will thank present me for this automation.

## Performance: Because Speed Matters

### Lighthouse Scores That Don't Suck

Current performance metrics from PageSpeed Insights:

- **Performance**: 75/100 (room for improvement, but solid)
- **Accessibility**: 96/100 (almost perfect)
- **Best Practices**: 96/100 (excellent)
- **SEO**: 100/100 (Google loves me)

**Performance opportunities identified**:

- Image delivery optimization (potential 9.5MB savings)
- Better cache lifetimes (potential 6.7MB savings)
- Reduce network payload size (currently 9.9MB)

Not bad for a portfolio site packed with project images!

### Image Optimization

Hugo's built-in image processing is criminally underrated:

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

Automatically converts images to WebP for modern browsers, with fallbacks for
older ones. Plus lazy loading because we're not savages.

## Things I Learned (The Hard Way)

### 1. Hugo's Learning Curve is Real

Go templates look like someone took HTML and put it in a blender with
programming syntax. Things like `{{ range .Site.RegularPages }}` and
`{{ with .Params.author }}` make sense... eventually.

**Pro tip**: The Hugo documentation is actually good and easy to understand. For
other things LLMs are your friend.

### 2. CSS Optimization Can Be Addictive

Started with "let me just inline some critical CSS" and ended up with a full
build pipeline that optimizes, minifies, and purges unused styles.

**Lesson**: Sometimes good enough is actually good enough. But sometimes you
need that extra 0.1 second page load improvement.

### 3. JSON Resume is Brilliant

Having your professional information in a standardized format opens up so many
possibilities:

- Auto-generate different CV formats
- Integrate with job application systems
- Keep everything in version control
- Never manually copy-paste experience descriptions again

### 4. Static Sites Are Underrated

After years of seeing database crashes, lag spikes, bad load times, serving
static files feels like cheating. It's fast, secure, and basically
indestructible.

## What I'd Do Differently

**More content, less optimization**: Spent too much time tweaking performance
and not enough time writing. The content is what people actually care about.

**Better image workflow**: Need to set up automated image optimization and
proper alt-text workflows.

## Future Plans

**Domain Infrastructure**: The Cloudflare setup isn't just for this site - I'm
planning to use subdomains for various homelab projects and self-hosted
services. Expect a detailed blog post about my server setup, Docker containers,
and whatever other self-hosting shenanigans I get into.

**Comments**: Utterances for blog discussions  
**Analytics**: Something privacy-friendly to see if anyone actually reads this

## The Real Results

**Professional presence**: A website that actually represents my skills and
interests  
**Learning platform**: Documented every project so I remember what I built  
**Performance obsession**: Sub-second load times because life's too short for
slow websites  
**Future-proof**: Easy to maintain and extend as my career evolves

Would I recommend this approach? If you enjoy learning web technologies and want
complete control over your online presence, absolutely. If you just want a quick
portfolio, maybe stick with a template.

But where's the fun in that?

---

**Built with**: Hugo, Gruvbox Theme, GitHub Pages and too much caffeine

**Live site**: [shubhamprasad.me](https://shubhamprasad.me)  
**Source code**: [GitHub](https://github.com/shubhpsd/shubhpsd.github.io)
