# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a Hugo static site blog (mgorbatyuk.dev) using the PaperMod theme. The blog is primarily in Russian with some English translations. The `public/` directory is a git submodule pointing to the GitHub Pages deployment repository (maximgorbatyuk.github.io).

**Requires:** Hugo v0.158.0+ extended (`brew install hugo`)

## Blogpost rules

- Articles are placed in folder `/content/blog/`
- Images for blogposts should be placed to folder `/static/images/blog/<category_name>/<YYYY-MM-DD>/`

## Docs

- Text style is described in `editorial-instructions.md`
- File `prompts.md` contains useful prompts to review and correct articles, and generate images

## Commands

- **Clone with submodules:** `git clone --recurse-submodules` (or run `git submodule update --init` after clone)
- **Local development server:** `hugo server -D` (includes drafts)
- **Build site:** `hugo`
- **Deploy to production:** `./deploy.sh` or `./deploy.sh "commit message"` — builds and pushes `public/` submodule to GitHub Pages. **This updates the live site.**
- **Create new post:** `hugo new blog/<category>/<date>-<slug>.md`

## Architecture

### Content Structure

Blog posts are organized in `content/blog/` by category:
- `development/` - Technical programming posts
- `management/` - Team leadership and management
- `employment/` - Job interviews and career advice
- `opinion/` - Personal opinions and reviews
- `different/` - Miscellaneous topics
- `videos/` - Video content references

Standalone pages live directly in `content/` (about.md, archive.md, ev-charging-tracker.md, photography.md, privacy.md, projects.md, search.md, speaker.md).

**Bilingual posts:** English translations use the `-en` suffix (e.g., `2026-02-19-how-to-build-pet-projects.md` → `2026-02-20-how-to-build-pet-projects-en.md`).

### Post Front Matter

Posts use YAML front matter with these fields:
```yaml
---
layout: post
title: "Post Title"
category: development
tags: [tag1, tag2]
date: "YYYY-MM-DD"
description: "Short description for SEO"
cover:
  image: "/images/blog/<category>/<date>/cover.png"  # optional
---
```

### Theme Customization

- Theme: PaperMod (in `themes/PaperMod/`, managed as submodule)
- Custom layouts override theme in `layouts/` (`_default/sitemap.xml`, `partials/extend_head.html`, `partials/footer.html`, `partials/header.html`)
- Custom CSS goes in `assets/css/extended/`
- Static files (images, PDFs) go in `static/`

### Configuration

Main config in `config.yml`:
- Profile mode enabled on homepage
- Search enabled via Fuse.js
- Google Analytics configured
- Social links: Instagram, Telegram, GitHub
