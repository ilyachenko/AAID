# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Architecture

This is a Jekyll-based GitHub Pages site for "Agentic AI Development" documentation. The site serves as a collection of scaffolding guides for full-stack development patterns.

**Key Structure:**
- `_config.yml`: Jekyll configuration with collections enabled
- `_scaffolding/`: Collection of setup guides (converted to pages via collections)
- `_layouts/default.html`: Main page layout
- `_includes/header.html`: Site header with navigation
- `assets/css/style.scss`: Custom styles extending minima theme
- `index.md`: Homepage with links to scaffolding guides

## Build Commands

**Jekyll Site:**
```bash
# Install dependencies
bundle install

# Local development server
bundle exec jekyll serve

# Build site
bundle exec jekyll build
```

## Content Organization

The site uses Jekyll collections for the scaffolding guides:
- Each `.md` file in `_scaffolding/` becomes a page
- URL pattern: `/scaffolding/{filename}/`
- All guides use the default layout
- Front matter must include `title` and `layout: default`

## Scaffolding Guides

Two comprehensive guides are available:

1. **Express + React + TypeScript** (`_scaffolding/pnpm-express-react.md`)
   - Full pnpm workspace setup with Express backend, React frontend, shared types
   - Key pattern: Build shared types first, then server, then client

2. **NestJS + React** (`_scaffolding/pnpm-nest-react.md`)
   - NestJS backend with Swagger docs, React frontend, shared TypeScript types
   - Includes CORS, environment configuration, API documentation

Both follow the same workspace pattern: `server/`, `client/`, `shared/` packages.

## Theme Configuration

Uses minima theme with:
- Auto skin (respects system dark/light preference)
- Custom header with theme toggle functionality
- Footer hidden (`show_excerpts: false`)
- Social links disabled