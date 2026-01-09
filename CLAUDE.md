# Eliasz Sawicki's Blog - Project Context

## Overview
Jekyll-based GitHub Pages blog hosted at eliaszsawicki.com. Minimalist theme based on Indigo template. No JavaScript dependencies - purely static HTML/CSS.

## Directory Structure
```
├── _config.yml          # Main Jekyll config
├── _config-dev.yml      # Dev overrides (localhost:4000)
├── _layouts/            # HTML layout templates
├── _includes/           # Reusable components
├── _posts/              # Blog posts (Markdown)
├── _sass/               # SCSS stylesheets
├── assets/images/       # Post images and assets
├── blog/                # Blog listing page
├── Gemfile              # Ruby dependencies
└── CNAME                # Custom domain config
```

## Layouts (_layouts/)
- **default.html** - Base HTML structure, analytics, icon sprites
- **page.html** - Standard pages with header/footer
- **post.html** - Blog posts with tags, date, reading time, related posts, Disqus
- **compress.html** - HTML minification utility

## Creating a Blog Post
1. Create file in `_posts/` named `YYYY-MM-DD-title.md`
2. Add front matter:
```yaml
---
layout: post
title: "Post Title"
category: blog
tags: [tag1, tag2]
header: /assets/images/post-image.jpg  # optional
excerpt: "Brief description"           # optional
---
```
3. Write content in Markdown below front matter

## Key Includes (_includes/)
- `header.html` - Profile image, name, bio, social links
- `nav.html` - Navigation (Home, Blog, Speaking, About)
- `blog-post.html` - Post list item template
- `read-time.html` - Reading time calculation (words / 180)
- `related.html` - Related posts by tag similarity
- `disqus.html` - Comments section
- `social-links.html` - Social media icons

## Styling (_sass/)
**Variables** (_sass/base/variables.sass):
- Colors: alpha (gray), beta (dark), delta (indigo accent)
- Breakpoints: 400px (mobile), 780px, 1050px

**Organization**:
- `base/` - Reset, typography, code syntax
- `components/` - Header, nav, footer, blog items, etc.
- `pages/` - Page-specific styles

## Configuration Highlights (_config.yml)
- **Analytics**: Google Analytics (UA-72230383-1)
- **Comments**: Disqus (shortname: "Eluss")
- **Plugins**: jekyll-seo-tag, jekyll-gist, jekyll-feed, jemoji
- **Permalink**: `/:title/` (clean URLs)

## Main Pages
- `index.html` - Homepage
- `blog/index.html` - Blog listing (filters category: blog)
- `tags.html` - Posts grouped by tags
- `speaking.md` - Talks and presentations
- `about.md` - Author bio
- `projects.html` - Projects (posts with projects: true)

## Development Commands
```bash
# Install dependencies
bundle install

# Local development
bundle exec jekyll serve --config _config.yml,_config-dev.yml

# Run tests
bundle exec rake test
```

## Post Features
- **Tags**: Clickable, filter on tags.html page
- **Reading time**: Auto-calculated
- **Related posts**: Based on shared tags (up to 5)
- **Comments**: Disqus for blog/project categories
- **Hidden posts**: Set `hidden: true` in front matter

## Assets
- Post images go in `assets/images/`
- Profile image: `assets/images/EliaszSawicki.png`
- Favicons in root directory (multiple sizes)
- SVG icons embedded via `_includes/icons.html`
