---
name: publish-blog-post
description: Publish Hugo blog posts to ceeyang.com via GitHub Pages. Use when writing, previewing, committing, or publishing articles on ceeyang.github.io, or when the user says 发布文章、上线、push 博客、ceeyang.com 看不到新文章.
---

# Publish Blog Post (ceeyang.github.io)

Hugo static site → push `main` → GitHub Actions → `gh-pages` → https://ceeyang.com

## Critical rule

**Commit alone does NOT publish.** The site updates only after:

```
git push origin main  →  GitHub Actions "GitHub Pages" workflow  →  ceeyang.com
```

If the user says the article is missing on ceeyang.com, **check push status first**:

```bash
git status          # "ahead of origin/main" = NOT published
git log origin/main..HEAD --oneline
```

## Workflow

### 1. Write the post

```bash
hugo new posts/my-slug.md
```

Front matter (see `archetypes/default.md`):

```yaml
---
title: "..."
date: 2026-06-29
tags: ["..."]
keywords: []
description: "..."
cover:
  image: "images/posts/<slug>/cover.png"   # SEO/OG only
  alt: "..."
  hiddenInList: true                       # do NOT show cover in list cards
  hiddenInSingle: true                     # use {{< figure >}} in body instead
draft: true                                # preview only until publish
---
```

**Images:** put under `static/images/posts/<slug>/`. Reference in body with PaperMod figure shortcode:

```markdown
{{< figure src="/images/posts/<slug>/diagram.svg" alt="..." caption="..." >}}
```

Do **not** rely on `cover.image` for in-article display — it also renders on the home/posts list.

### 2. Local preview

```bash
hugo serve
# development config (config/development/hugo.toml) keeps links on localhost:1313
```

- Preview URL: `http://localhost:1313/posts/<filename-without-md>/`
- Use `draft: true` + `hugo serve` (includes drafts) OR `draft: false` for publish-ready preview
- **Never** open `https://ceeyang.com/...` to preview unpublished posts — production 404 until pushed

Verify before publish:

```bash
hugo list all | grep <slug>
curl -s -o /dev/null -w "%{http_code}" http://localhost:1313/posts/<slug>/
```

### 3. Publish-ready checklist

- [ ] `draft: false` in front matter
- [ ] Images exist under `static/images/posts/`
- [ ] SVG files validate: `xmllint --noout static/images/posts/**/*.svg`
- [ ] `hugo --minify` succeeds locally
- [ ] Post appears at localhost URL

### 4. Commit

Only commit when user asks. Stage post + static assets + related config:

```bash
git add content/posts/... static/images/posts/...
git commit -m "$(cat <<'EOF'
Add <title> blog post.

Brief why (one sentence).
EOF
)"
```

Do **not** commit `.cursor/` unless user requests; `graphify-out/` stays gitignored.

### 5. Push (required for live site)

```bash
git push origin main
```

Monitor deploy:

```bash
gh run list --repo ceeyang/ceeyang.github.io --workflow "GitHub Pages" --limit 3
gh run watch --repo ceeyang/ceeyang.github.io   # optional, wait for latest run
```

Workflow file: `.github/workflows/gh-pages.yml` — triggers on **push to `main` only**.

### 6. Verify production

Wait ~1–2 min after green CI, then:

```bash
curl -s -o /dev/null -w "%{http_code}" https://ceeyang.com/posts/<slug>/
```

Expect `200`. If `404`:

| Symptom | Fix |
|---------|-----|
| Local ahead of origin | `git push origin main` |
| CI failed | `gh run view <id> --log-failed` |
| CI never ran | Confirm push landed on `main`, not another branch |
| 200 on localhost, 404 on prod | Cache/CDN — hard refresh or wait |

## Site conventions

| Item | Value |
|------|-------|
| Repo | `ceeyang/ceeyang.github.io` |
| Source branch | `main` |
| Deploy branch | `gh-pages` (auto, do not push manually) |
| Domain | https://ceeyang.com |
| Hugo | Extended 0.160.1 |
| Theme | PaperMod (submodule) |
| Post path | `content/posts/YYYY-MM-DD-slug.md` → `/posts/YYYY-MM-DD-slug/` |

## Common mistakes

1. **Committed but not pushed** — most common; user sees 404 on ceeyang.com
2. **`draft: true` on CI** — `hugo --minify` in Actions excludes drafts
3. **Broken SVG** — truncated XML → broken image icon in browser
4. **Using production URL during local preview** — `baseURL` in root `hugo.toml` is `https://ceeyang.com/`; use `hugo serve` (development env) not raw `hugo` without env
5. **README says `master`** — workflow uses **`main`**

## One-liner publish (after user approval)

```bash
git push origin main && gh run list --repo ceeyang/ceeyang.github.io --workflow "GitHub Pages" --limit 1
```

Tell the user the live URL: `https://ceeyang.com/posts/<slug>/`
