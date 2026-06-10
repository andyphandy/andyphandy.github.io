# Handoff — andyphandy.github.io

Single-page portfolio. Static HTML + CSS + ~120 lines of vanilla JS. No build
step, no framework, no package manager.

---

## Quick start

```bash
python3 -m http.server 8000
# → http://localhost:8000
```

Edit `index.html` or `styles.css`, refresh the browser. That's it.

---

## File layout

| File | Purpose |
|---|---|
| `index.html` | All markup + inline JS (scrollspy, reveal observer, cursor glow, mobile nav). |
| `styles.css` | All styles. Tokens at the top, components below, animations near the bottom. |
| `assets/` | Résumé PDF, favicons, and a pile of legacy images mostly unused. See "Cleanup" below. |
| `README.md` | Points to the live URL. Currently still references the old UW CSE host — worth updating. |
| `.gitlab-ci.yml` | Orphan from the old UW CSE GitLab deploy. Safe to delete now that the site lives on GitHub Pages. |
| `jquery.js` | Orphan — jQuery is no longer used. Safe to delete. |

---

## Design tokens

All in `:root` at the top of `styles.css`. Change once, applies everywhere.

```css
--bg / --surface / --surface-2 / --border / --border-strong   /* neutral palette */
--text / --text-muted / --text-dim                            /* typographic hierarchy */
--accent / --accent-hover / --accent-dim                      /* violet #8b5cf6 family */
--on-accent                                                   /* text color on accent fills */
--radius / --radius-sm                                        /* corner radii */
--max-width                                                   /* 1080px container */
--nav-h                                                       /* 64px nav height */
--font-sans                                                   /* Inter */
--font-display                                                /* Instrument Serif */
```

**Gotcha:** the radial gradients in `.hero::before` and `.project-media-empty`
hardcode the accent's RGB triplet (`139, 92, 246`) for alpha control. If you
swap the accent, search the file for `rgba(139, 92, 246` and update those too.

---

## Page structure

```
<header.site-nav>  fixed; mobile dropdown; scrollspy indicator inside
<main>
  <section.hero #top>     name + lede + CTAs
  <section.about #about>  bio + skill chips
  <section.work #work>    alternating project rows
  <section.contact #contact>  email + GitHub + LinkedIn
<footer.site-footer>
```

Section order matters for the nav scrollspy; if you reorder sections, also
reorder the `.nav-links` `<li>` items so the indicator reads left-to-right.

---

## Animation system

Two mechanisms, both reduced-motion aware.

**1. Hero stagger on load** — pure CSS keyframes on `.hero .eyebrow`,
`.hero h1` (per-word via `.hero-word-inner`), `.hero .lede`, `.hero .hero-cta`.
Delays are set via `animation-delay` in `styles.css`.

**2. Scroll reveals via IntersectionObserver** — JS adds `.in-view` to any
`[data-reveal]` element when it intersects the viewport. CSS transitions do
the rest.

Two reveal variants:

- **Default** (`data-reveal` alone): opacity 0→1, translateY 24px→0. Soft.
- **Wipe** (`data-reveal class="reveal-wipe"`): clip-path polygon left→right.
  Sharper, more "architectural." Used on the section `<h2>` headings.

Per-element delays can be tuned via inline style: `style="--reveal-delay: 200ms;"`.

Other moving parts:

- Cursor-tracked accent glow in the hero (`--mx`, `--my` CSS vars, rAF-throttled).
- Decorative grid behind the hero drifts on a 50s loop.
- Eyebrows have a small horizontal dash drawn via `::before` (`scaleX 0→1`).
- Nav scrollspy: an absolutely-positioned 2px bar slides between nav links as
  you scroll past each section's top (threshold = 35vh).
- Animated underlines on nav links and project links (hover-only).

All cursor/scroll effects skip on `(pointer: coarse)` and
`prefers-reduced-motion: reduce`.

---

## Common tasks

### Add a new project

In `index.html`, inside `<div class="project-list">`, copy one of the existing
`<article class="project-row" data-reveal>` blocks and fill in:

```html
<article class="project-row" data-reveal>
  <div class="project-media">
    <!-- Replace this empty placeholder with a real screenshot -->
    <img src="assets/your-screenshot.jpg" alt="..." loading="lazy">
  </div>
  <div class="project-content">
    <span class="project-number">Project 04</span>
    <h3 class="project-title">Project name</h3>
    <p class="project-desc">One or two sentences.</p>
    <ul class="chips">
      <li>Tech</li>
    </ul>
    <div class="project-links">
      <a href="..." target="_blank" rel="noopener noreferrer">Live ↗</a>
      <a href="..." target="_blank" rel="noopener noreferrer">Code ↗</a>
    </div>
  </div>
</article>
```

Rows automatically alternate left/right via `:nth-child(even) .project-media { order: 2 }`.

### Change the hero text

The `<h1>` is inside `<section class="hero">`. Each word is wrapped for the
per-word reveal:

```html
<span class="hero-word"><span class="hero-word-inner">Word</span></span>
```

Add/remove words by adding/removing those nested spans. If the word count
changes, update the `:nth-child(N) .hero-word-inner { animation-delay: ... }`
rules in `styles.css` so each word gets its own delay.

### Swap the accent color

1. Update `--accent`, `--accent-hover`, `--accent-dim`, `--on-accent` in `:root`.
2. Find all `rgba(139, 92, 246, ...)` instances (currently in `.hero::before`
   and `.project-media-empty`) and update the RGB to match the new accent.
3. Confirm `--on-accent` still passes AA contrast on the button background.

### Update placeholders

Search the HTML for `[Add:` — anything still bracketed is yours to fill in.
At handoff time the remaining ones are the three project rows (name,
description, tech chips, links each).

---

## Deployment

GitHub Pages serves from `main`. Push → live in ~30s. Configure custom domain
in repo Settings → Pages if/when you want one.

---

## Browser support

Modern evergreen (Chrome, Safari, Firefox, Edge). IntersectionObserver has a
fallback path that flips every reveal to its final state if the API is
missing. `clip-path: polygon()` is universally supported. Touch and
reduced-motion users get a calmer, simpler page.

---

## Cleanup worth doing

Low-priority but everything below is unreferenced and safe to delete when
you're confident nothing's pinned to it:

- `jquery.js` (no longer included from the HTML).
- `.gitlab-ci.yml` (old UW CSE deploy config).
- Most of `assets/`: `cherrytrees.jpg`, `rainier.jpg`, `me.png`, `udub.png`,
  `website.png`, `maze.gif`, `tetris.gif`, `spotify_website.JPG`, and all the
  social-icon PNGs (`facebook.png`, `twitter.png`, `instagram.png`,
  `gitlab.png`, `gmail.png`, `linkedin.png`, `github.png`, `star.png`).
  Kept for now in case you want to revert anything.

Worth keeping in `assets/`: `Andy_Phan_Resume.pdf`, the favicons
(`favicon-*.png`, `apple-touch-icon.png`), and any project screenshot you
later wire into a project row.
