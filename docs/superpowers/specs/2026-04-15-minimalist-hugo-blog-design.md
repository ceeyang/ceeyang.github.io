# Minimalist Hugo Blog Design Spec

## 1. Project Overview
- **Goal**: Rebuild the existing Jekyll blog into a modern, minimalist Hugo-based site.
- **Target Platform**: GitHub Pages (via GitHub Actions).
- **Core Aesthetic**: Minimalist, elegant, high whitespace ("breathable"), text-focused.
- **Base Theme**: [Hugo-PaperMod](https://github.com/adityatelange/hugo-PaperMod) with custom styling.

## 2. Information Architecture
- **Home**: Welcome text + chronological list of article cards (text-only).
- **Tags**: Taxonomy page for categorizing content.
- **Search**: Fast, local full-text search using Fuse.js.
- **About**: Personal profile page (migrated from existing Jekyll about.html).

## 3. Visual Design (The "Elegant" Look)
- **Card Style**: Text-only cards with generous padding (32px+).
- **Shadows**: Extremely subtle soft shadows (`rgba(0,0,0,0.02)`) instead of borders.
- **Colors**:
  - Light: `#F9F9FB` background, `#1A1A1A` text.
  - Dark: `#121212` background, `#E0E0E0` text.
- **Typography**: System native sans-serif stack; 1.75 line height for Chinese text.

## 4. Technical Strategy
- **Framework**: Hugo Extended (for SCSS support).
- **Content Migration**:
  - Move `.md` files from `_posts/` to `content/posts/`.
  - Convert Jekyll front-matter (YAML) to Hugo-compatible format.
  - Replace Liquid tags with Hugo shortcodes.
- **Deployment**: Configure GitHub Actions to build and deploy from the `main` branch.

## 5. Success Criteria
- Site loads under 500ms.
- 100/100 Lighthouse score for Performance and SEO.
- Fully responsive and accessible (AA standard).
