# Provenir Redesign — Dev Handoff (Noir Couture)

**For:** Claude Code / implementing developer
**From:** Design build, June 10, 2026
**Live preview:** https://provenir-redesign.vercel.app
**Repo:** https://github.com/alnany/provenir-redesign
**Production target:** https://www.provenir.design/

---

## 1. What this is

An approved, fully coded visual redesign of provenir.design covering three pages. The static HTML in this repo is the **design source of truth** — typography, spacing, color, motion, and content treatment are all final and user-approved. Your job is to port this design into the production codebase, not to redesign it.

| Page | File | Production route |
|------|------|------------------|
| Landing | `index.html` | `/` |
| World Wall (feed) | `world-wall.html` | `/world-wall` |
| Product detail | `product.html` | `/post/:id` |

Direction history (for context): the approved language is **"Noir Couture"** — a dark luxury fashion-house aesthetic (Celine / Saint Laurent energy) chosen explicitly to avoid the overused "warm cream + light serif" AI-default look. Do not drift back toward cream/paper backgrounds or generic SaaS styling.

## 2. Hard constraints (non-negotiable)

1. **English only. Zero CJK characters anywhere in rendered output.** Audience is US/AU designers and architects. Chinese product titles/tags/descriptions are translated; manufacturer names are romanized. Add a CI check if possible (the design build greps output for `[\u4e00-\u9fff]`).
2. **All original site content preserved** — every post, stat, showroom card, mechanism step, footer link. Nothing dropped for aesthetics.
3. **Respect `prefers-reduced-motion`** — all animation (entrance, slideshow, reveals) must no-op for users who opt out. Already implemented in `app.js` / CSS; keep the behavior.
4. **Photography carries the luxury.** Images are shown slightly dimmed (`brightness(.86) saturate(.92)`) and lift to full on hover. Don't "fix" this — it's intentional lookbook treatment.

## 3. Design system

### Color tokens

```css
--noir:        #0E0D0B;                  /* page ground */
--noir-2:      #161311;                  /* raised band / image placeholder */
--card:        #141210;                  /* card surfaces (login gate etc.) */
--paper:       #ECE7DE;                  /* primary text, warm off-white */
--mut:         rgba(236,231,222,.62);    /* secondary text */
--mut-soft:    rgba(236,231,222,.52);    /* tertiary / labels */
--line:        rgba(236,231,222,.16);    /* hairlines */
--line-strong: rgba(236,231,222,.34);    /* emphasized rules / borders */
--champagne:   #C2AD7E;                  /* THE ONLY ACCENT */
--ease:        cubic-bezier(.16,1,.3,1);
```

**Champagne accent discipline:** used ONLY for verified badges, the active tab underline, and the step tags. Never for buttons, links, or large surfaces. Everything else is monochrome.

### Typography

Google Fonts (single link, already in each page `<head>`):
`Cormorant Garamond` ital,wght 0,300/0,400/0,500/1,300/1,400 + `Jost` 300/400/500

- **Display serif — Cormorant Garamond.** All headlines, card titles, stat numerals, step names. Weight 300–400, italics used for emphasis words in headlines.
- **Body / UI — Jost.** Weight 300 body, 400 for UI labels. All labels/buttons/nav are **uppercase with wide tracking** (`.22em–.5em letter-spacing`). No mono font anywhere.
- **Wordmark:** "PROVENIR" in Jost 300, uppercase, `letter-spacing: .52em` (compensate trailing space with negative right margin). This is the brand mark treatment — keep it exact.

### Layout & components

- Max width 1280px, 48px gutters (22px mobile).
- **No border-radius anywhere** except circular avatars. Sharp rectangles are part of the language.
- Buttons: tracked-caps 11px, `17px 36px` padding; solid paper-on-noir for primary, 1px hairline outline for secondary.
- Cards have no boxes/shadows — they're defined by the image, a serif title, and hairline-separated meta rows.
- World Wall images: **4:5 portrait crop**, `object-fit: cover`. Showroom cards: 4/4.8. Product gallery main: 4/3.4.
- Section heads: tiny tracked kicker + large serif title, bottom hairline.

All of this is in `src/style.css` (~250 lines, heavily sectioned with comments). Port the tokens into your styling system but treat the CSS file as the spec.

## 4. Motion spec (GSAP)

GSAP core is loaded from CDN; logic is in `src/app.js` (small, readable — port as-is or adapt).

1. **Hero entrance (landing):** eyebrow → headline → side paragraph → stat rail fade-up; `y:28, autoAlpha:0, duration:1.2, ease:power3.out, stagger:0.12, delay:0.2`.
2. **Hero slideshow (landing):** 4 slides. Hold **6.5s**, crossfade **1.6s** (`power2.inOut`), each incoming slide gets a Ken Burns drift (`scale 1.07 → 1` over ~9s, linear). z-index swap pattern in `app.js` prevents flash-through.
3. **Scroll reveals:** `.reveal` elements fade-up via IntersectionObserver (threshold .12), one-shot. CSS-only fallback: without the `js` class on `<html>`, everything is visible (no-JS safe).
4. **Image hover:** `scale 1.04` over 1.2s + brightness lift, on the `--ease` curve.

All gated behind `prefers-reduced-motion`.

## 5. Page-by-page inventory

### Landing (`index.html`)
- Nav (absolute, on-photo) · hero slideshow (4 images) with headline + side paragraph + 2 CTAs + vertical stat ticker (9 cities / 1,260 factories / 42 countries)
- Tracked-caps "vertical network" strip
- Stats band (3 columns, hairline-topped, serif numerals)
- Showrooms: 3 cards (Ceramics·Jingdezhen VERIFIED · Lighting·Zhongshan IN PROD · Textiles·Suzhou OPEN) with MOQ / lead time / sample data
- Mechanism: raised `--noir-2` band, 3 steps (Présenter / Connecter / Accompagner) with champagne tags
- CTA band · footer

### World Wall (`world-wall.html`)
- Centered editorial header (serif, italic emphasis) + sub
- Tools row: Latest/Popular tabs (client-side sort via `data-pop`) + search input (client-side text filter)
- 20 posts: 4:5 image, serif title, avatar + romanized maker name + MANUFACTURER tag
- **Production note:** sort/search are demo-grade client-side implementations — replace with real API-driven sort/pagination/search.

### Product (`product.html`)
- Breadcrumb · 5-image gallery (arrows, thumbnails, `1 / 5` counter, ArrowLeft/ArrowRight keyboard nav)
- Maker block (avatar, name, champagne VERIFIED badge), date, serif H1, translated description, tracked-caps tags, view/comment stats, 2 CTAs
- Comments section with login gate
- **Production note:** gallery is static; wire to real post data. Keep the keyboard nav.

## 6. Assets

- `assets/hero-sofa.jpg`, `assets/hero-pendant.jpg`, `assets/hero-velvet.jpg` — **AI-generated** hero slideshow images (slides 2–4). Slide 1 is a real product photo from `public.provenir.design` CDN. Replace generated images with real campaign photography when available; until then they're approved for use.
- All product/showroom imagery hotlinks the existing `public.provenir.design` CDN with `x-oss-process` resize params — keep using the CDN in production.
- `mocks/` directory — exploration mocks from the direction-picking phase. **Ignore; do not ship.**

## 7. Build pipeline (this repo only)

The deployed pages are assembled from `src/` by `scripts/build.py` (templates + inlined CSS/JS + image refs) into flat HTML. This pipeline is for design iteration only — in production, integrate the markup/styles/JS into your real stack however fits. The flat HTML files at repo root are the rendered reference.

## 8. Open items / judgment calls to confirm before launch

1. **Romanized manufacturer names are designer transliterations** (e.g. 摩登翡丽 → "Modeng Feili"). Confirm each maker's preferred English name.
2. **Translated product titles/descriptions** are working translations — fine for design, but worth an editorial pass.
3. Generated hero images (above) — swap for real photography when available.
4. World Wall sort/search and product gallery need real data wiring (noted per page above).
5. Login/Sign up/CTA links currently point at `https://www.provenir.design/` — wire to real auth routes.

## 9. QA bar that was met (keep it green)

- Zero JS console errors on all three pages
- Zero CJK characters in rendered output (automated grep)
- Functional: all nav links, CTAs, wall→product navigation, tab sorting, search filter, gallery arrows/thumbs/keyboard, back links
- Slideshow cycles correctly (no double-visible or blank frames)
- Desktop 1440 + mobile 390 verified; reduced-motion verified
- Contrast: secondary text ≥ `--mut-soft` (.52 alpha) — don't dim text below this
