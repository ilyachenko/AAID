# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Architecture

This is a Jekyll-based GitHub Pages site for "Agentic AI Development" documentation. The site serves as a collection of scaffolding guides for full-stack development patterns.

**Key Structure:**
- `_config.yml`: Jekyll configuration with collections enabled
- `_scaffolding/`: Collection of setup guides (converted to pages via collections)
- `_best-practices/`: Collection of best practices articles
- `_layouts/default.html`: Main page layout
- `_includes/header.html`: Site header with navigation
- `assets/css/style.scss`: Custom styles extending minima theme
- `index.md`: Homepage with links to both collections

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

The site uses Jekyll collections for content:

**Scaffolding Guides Collection (`_scaffolding/`):**
- Each `.md` file becomes a page with URL pattern: `/scaffolding/{filename}/`
- Setup guides for specific technology stacks

**Best Practices Collection (`_best-practices/`):**
- Each `.md` file becomes a page with URL pattern: `/best-practices/{filename}/`
- General development practices and patterns

All guides use the default layout and require front matter with `title` and `layout: default`.

## Available Content

**Best Practices:**
1. **CLAUDE.md Maintenance & Documentation Guide** (`_best-practices/claude-md-maintenance.md`)
   - Best practices for maintaining CLAUDE.md and related documentation
   - Team-wide consistency and long-term clarity guidelines

**Scaffolding Guides:**
1. **Express + React + TypeScript** (`_scaffolding/pnpm-express-react.md`)
   - Full pnpm workspace setup with Express backend, React frontend, shared types
   - Key pattern: Build shared types first, then server, then client

2. **NestJS + React** (`_scaffolding/pnpm-nest-react.md`)
   - NestJS backend with Swagger docs, React frontend, shared TypeScript types
   - Includes CORS, environment configuration, API documentation

All setup guides follow the same workspace pattern: `server/`, `client/`, `shared/` packages.

## Theme Configuration

Uses minima theme with:
- Auto skin (respects system dark/light preference)
- Custom header with theme toggle functionality
- Footer hidden (`show_excerpts: false`)
- Social links disabled
- Do not forget update CLAUDE.md if needed.