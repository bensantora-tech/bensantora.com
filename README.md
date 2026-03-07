# bensantora.com — Site Guide

## Preview and Build

```bash
# Live preview with hot-reload at localhost:1313
hugo server

# Build the site into public/ for deployment
hugo
```

---

## Adding a New Post

Create a new `.md` file in `content/posts/`:

```
content/posts/your-post-slug.md
```

The filename becomes the URL: `/posts/your-post-slug/`

**Frontmatter template:**

```markdown
---
title: "Your Post Title"
date: 2026-03-07
description: "One or two sentences shown on the homepage card and in search results."
tags: ["linux", "go", "bare-metal"]
---

Your content starts here.
```

- `title` — displayed as the H1 on the post page and as the card link on the homepage
- `date` — controls sort order (newest first); use `YYYY-MM-DD`
- `description` — shown as the card summary on the homepage and `/posts/` list; also used for OpenGraph and the Article schema. Keep it to 1–2 sentences.
- `tags` — any number of lowercase tags; each gets a page at `/tags/<tag>/`

**The homepage and `/posts/` list update automatically** — no editing of `layouts/index.html` required.

### Headings in posts

Use `##` (H2) and `###` (H3) for section headings. These feed the right-sidebar Table of Contents automatically on wide screens. Don't use `#` (H1) inside your post — the template renders the title as H1.

### Existing tags in use

`linux` `llm` `bare-metal` `cpu` `ffmpeg` `video`

---

## Adding to Notes

Notes are short-form entries — TIL items, one-liners, config details that don't need a full post.

Create a file in `content/notes/`:

```
content/notes/your-note-slug.md
```

**Frontmatter template:**

```markdown
---
title: "AVX-512 detection one-liner"
date: 2026-03-07
description: "Optional summary line."
tags: ["linux", "cpu"]
---

`lscpu | grep -E --color=always "avx(2|512)"`

Look for `avx512_vnni` to confirm AI-optimized instruction support.
```

The note appears automatically in the `/notes/` section list.

---

## Adding to Tools

Tools are individual entries for Go or Bash utilities you've built.

Create a file in `content/tools/`:

```
content/tools/your-tool-slug.md
```

**Frontmatter template:**

```markdown
---
title: "hwprobe"
date: 2026-03-07
description: "A Go binary that audits CPU flags, RAM, and storage in one pass. No dependencies."
tags: ["go", "linux", "hardware"]
---

## What It Does

...

## Install

```bash
wget https://...
chmod +x hwprobe
./hwprobe
```
```

---

## Adding to Hardware

Hardware entries document machines — current rigs, restored hardware, purpose-built setups.

Create a file in `content/hardware/`:

```
content/hardware/machine-slug.md
```

**Frontmatter template:**

```markdown
---
title: "ThinkPad X220 — Daily Driver"
date: 2026-03-07
description: "Intel Core i7-2640M, 16GB RAM. Running CrunchBang++ with a custom Openbox config."
tags: ["thinkpad", "debian", "hardware"]
---

## Specs

...

## What Was Done

...
```

---

## Adding to Links

Links are individual reference entries — documentation, tools, projects you actually use.

Create a file in `content/links/`:

```
content/links/link-slug.md
```

**Frontmatter template:**

```markdown
---
title: "llama.cpp — ggml-org/llama.cpp"
date: 2026-03-07
description: "The source. C++ LLM inference engine, actively maintained."
---

Notes on why this is useful, what to look at, etc.
```

---

## Section Index Pages

Each section has a `_index.md` that controls the section's title, description, and intro text shown at the top of the list:

| File | URL |
|------|-----|
| `content/tools/_index.md` | `/tools/` |
| `content/notes/_index.md` | `/notes/` |
| `content/hardware/_index.md` | `/hardware/` |
| `content/links/_index.md` | `/links/` |

Edit these to update the intro text for each section. The list of entries below the intro is generated automatically.

---

## File Structure

```
hugo.toml                        # Site config: base URL, title, taxonomy, TOC settings

content/
  posts/                         # Long-form articles
    llama-cpp-bare-metal-linux.md
    ffmpeg-mandelbrot.md
  notes/                         # Short-form TIL entries
    _index.md                    # Section intro text
  tools/                         # Go/Bash utility entries
    _index.md
  hardware/                      # Machine documentation
    _index.md
  links/                         # Reference links
    _index.md

layouts/
  _default/
    baseof.html                  # Master template — all pages extend this
    list.html                    # Generic section list (notes, tools, hardware, links)
    single.html                  # Generic single page
  partials/
    head.html                    # All CSS + meta/OG tags (edit here for style changes)
    left-nav.html                # Left sidebar navigation
    footer.html                  # Site footer
  index.html                     # Homepage template
  posts/
    single.html                  # Post template (TOC, back-link, tags, schema)
    list.html                    # /posts/ list page

static/
  images/                        # Static images (reference as /images/filename.png)

public/                          # Built output — generated by `hugo`, do not edit directly
```

---

## Style Changes

All CSS lives in one place: `layouts/partials/head.html`. Font sizes, colors, column widths, spacing — edit there. Run `hugo server` to preview changes live.

The CSS variables at the top of the stylesheet control the color palette:

```css
:root {
    --green:   #98c9a3;   /* headings, links, accents */
    --text:    #e0e0e0;   /* body text */
    --bg:      #0d0d0d;   /* page background */
    --card-bg: #1e1e1e;   /* card/code block background */
    --border:  #2a2a2a;   /* borders and dividers */
    --muted:   #666;      /* metadata, footer */
    --dim:     #888;      /* sidebar links, secondary text */
}
```

---

## Tags

Tags are defined in post frontmatter (`tags: ["linux", "go"]`) and generate pages automatically at `/tags/<tag>/`. No configuration needed — just use them in frontmatter and Hugo handles the rest.

To see all tag pages, visit `/tags/` on the live site.
