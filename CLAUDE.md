# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a Hugo static site blog (mgorbatyuk.dev) using the PaperMod theme. The `public/` directory is a git submodule pointing to the GitHub Pages deployment repository (maximgorbatyuk.github.io).

## Commands

- **Local development server:** `hugo server -D` (includes drafts)
- **Build site:** `hugo`
- **Deploy to production:** `./deploy.sh` or `./deploy.sh "commit message"` - builds and pushes to GitHub Pages
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

Standalone pages live directly in `content/` (about.md, archive.md, search.md, speaker.md).

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
---
```

### Theme Customization

- Theme: PaperMod (in `themes/PaperMod/`, managed as submodule)
- Custom layouts override theme in `layouts/` (currently only `_default/sitemap.xml`)
- Custom CSS can be added in `assets/`
- Static files (images, PDFs) go in `static/`

### Configuration

Main config in `config.yml`:
- Profile mode enabled on homepage
- Search enabled via Fuse.js
- Google Analytics configured
- Social links: Instagram, Telegram, GitHub
