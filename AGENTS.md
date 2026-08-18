# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Info

- **Domain**: duzeradev.com.br
- **Repository**: https://github.com/eduardonk9999/blogDuzera.git

## Commands

```bash
npm run dev      # Start dev server at localhost:4321
npm run build    # Build production site to ./dist/
npm run preview  # Preview production build locally
```

## Architecture

This is a blog built with Astro 7, featuring a Linux terminal visual theme.

### Content System

Posts are Markdown files in `src/content/posts/` using Astro Content Collections. The schema is defined in `src/content.config.ts`:

```yaml
---
title: "Post title"
date: 2024-01-15
description: "Short description"
tags: ["tag1", "tag2"]  # optional
draft: false            # optional, hides from listing
---
```

### Routing

- `/` - Home with post listing (`src/pages/index.astro`)
- `/about` - About page (`src/pages/about.astro`)
- `/posts/[slug]` - Dynamic post pages (`src/pages/posts/[slug].astro`)

### Layouts

- `Layout.astro` - Base layout with Header, Footer, and global styles
- `PostLayout.astro` - Post-specific layout with terminal window wrapper and animations

### Theme

Terminal aesthetic using CSS variables in `src/styles/global.css`:
- Colors: `--accent-green`, `--accent-cyan`, `--accent-yellow`, `--accent-magenta`, `--accent-red`
- Font: JetBrains Mono (Google Fonts)
- Animations: `fadeInLine`, `typewriter`, `blink`, `slideIn`

### Key Components

- `TerminalWindow.astro` - Visual wrapper resembling a terminal window with title bar buttons
- `PostCard.astro` - Post listing item with staggered animation
- `Header.astro` - Navigation with terminal prompt style (`edu@blog:~$`)

## Development Notes

- Always test mobile responsiveness when making UI changes
