# SynapseHealthTech — Developer Handover
**Version:** 1.0.0  
**Prepared:** May 2025  
**Status:** Ready for production deployment

---

## Quick Start

```bash
# No build step. No package manager. No dependencies to install.
# Simply serve the root folder as static HTML.

# Option A — Local preview with Python (built-in)
python3 -m http.server 8080
# → open http://localhost:8080

# Option B — Local preview with Node (if installed)
npx serve .
# → open http://localhost:3000

# Option C — Deploy to any static host (see §6 Deployment)
```

---

## 1. Project Overview

Pure static website. **Zero JavaScript frameworks. Zero CSS frameworks. Zero build tools.**

- 7 HTML pages, all self-contained
- 1 shared CSS file (`assets/synapse.css`) — for reference/extension
- 1 logo PNG (`assets/Logo_Hi_Res.png`)
- All fonts loaded from Google Fonts CDN
- All JavaScript is inline `<script>` at end of each `<body>`

---

## 2. File Structure

```
synapse-website/
│
├── index.html            ← Landing page (hero, manifesto, services, products, industries, careers teaser)
├── services.html         ← Professional Services (4 full sections + process + CTA)
├── products.html         ← Platform Suite (NeuraScan, CareFlow, PredictRx, HealthOS)
├── industries.html       ← Industries (6 sectors, each with use cases and product links)
├── case-studies.html     ← Client Impact (4 studies, filterable by product/industry)
├── careers.html          ← Careers (About Us, values, 24 open roles, benefits)
├── contact.html          ← Contact (split layout: offices + contact form)
│
└── assets/
    ├── Logo_Hi_Res.png   ← Brand logo (PNG with black background — see §4.1)
    ├── synapse.css       ← Shared design system stylesheet (reference + extend)
    └── style-guide.html  ← Living design system documentation (open in browser)
```

---

## 3. Design System

Full visual documentation is in **`assets/style-guide.html`** — open it in a browser.

### 3.1 Colour Tokens

All colours are CSS custom properties on `:root`. **Never use raw hex in new code.**

| Token | Value | Role |
|-------|-------|------|
| `--bg` | `#0F0E0B` | Page background |
| `--bg2` | `#161410` | Alternate section background |
| `--bg3` | `#1D1A14` | Tertiary / quote sections |
| `--faint` | `#252219` | Card / inset block backgrounds |
| `--fg` | `#EFEBE2` | Primary text |
| `--sub` | `#706960` | Secondary / muted text |
| `--a` | `#BEFC42` | **Acid Lime — primary accent** |
| `--a-dim` | `rgba(190,252,66,0.10)` | Faint accent fill |
| `--ln` | `rgba(239,235,226,0.07)` | Default rule / border |
| `--lnh` | `rgba(239,235,226,0.14)` | Hover rule / border |

### 3.2 Typography

| Role | Font | Size | Weight |
|------|------|------|--------|
| Hero display | DM Serif Display | clamp(3rem, 7vw, 7rem) | 400 |
| Section H2 | DM Serif Display | clamp(2rem, 3.2vw, 2.8rem) | 400 |
| Pull quote | DM Serif Display · italic | clamp(1.35rem, 2.1vw, 1.85rem) | 400 |
| Body copy | DM Sans | 0.9–0.95rem | 300 |
| Section labels | Chivo Mono | 0.67rem | 300 |
| Nav / UI strings | Chivo Mono | 0.66rem | 300 |
| Captions / micro | Chivo Mono | 0.62–0.65rem | 300 |

### 3.3 Google Fonts Link (required in every page `<head>`)

```html
<link rel="preconnect" href="https://fonts.googleapis.com">
<link href="https://fonts.googleapis.com/css2?family=DM+Serif+Display:ital@0;1&family=Chivo+Mono:wght@300;400&family=DM+Sans:opsz,wght@9..40,300;9..40,400;9..40,500&display=swap" rel="stylesheet">
```

### 3.4 Layout Rules

- Max content width: **1280px**, centered with `margin: 0 auto`
- Horizontal padding: **5% viewport width** on all sections
- Section vertical padding: **7rem top/bottom**
- Inter-section grid gap: **4–6rem**

---

## 4. Technical Notes

### 4.1 Logo Rendering

The logo PNG has a black background. It is rendered on dark pages using:

```css
.logo img {
  mix-blend-mode: screen;
}
```

This blends the black background away, leaving only the coloured wordmark visible. **Do not pre-process or crop the PNG** — the blend mode handles it. If the site ever introduces a light-coloured page or section, use `mix-blend-mode: multiply` instead, or provide a separate transparent-background logo.

### 4.2 Custom Cursor

Every page has a lime dot cursor (`#dot`) that tracks `mousemove` and enlarges on interactive elements via `body.lh`. It uses `mix-blend-mode: exclusion` to invert colours beneath it.

```html
<!-- Required in every page <body> before closing tag -->
<div id="dot"></div>
```

```js
// Required cursor JS — already in every page
const dot = document.getElementById('dot');
document.addEventListener('mousemove', e => {
  dot.style.left = e.clientX + 'px';
  dot.style.top  = e.clientY + 'px';
});
document.querySelectorAll('a, .sr, .pr, .cert, button, input, select, textarea')
  .forEach(el => {
    el.addEventListener('mouseenter', () => document.body.classList.add('lh'));
    el.addEventListener('mouseleave', () => document.body.classList.remove('lh'));
  });
```

> **Mobile note:** The cursor is hidden on mobile naturally (no mouse) — `cursor: none` on `body` has no visible effect on touch devices.

### 4.3 Fit-Text (Hero Headline)

The hero headline fills the full viewport width using a binary-search algorithm that runs on `document.fonts.ready` (so fonts are loaded before sizing) and again on `window.resize`:

```js
function ft() {
  const c = document.getElementById('fittext');
  if (!c) return;
  const W = c.clientWidth;
  document.querySelectorAll('.hl').forEach(el => {
    let lo = 8, hi = 500;
    el.style.fontSize = hi + 'px';
    while (hi - lo > 1) {
      const m = (lo + hi) >> 1;
      el.style.fontSize = m + 'px';
      el.scrollWidth <= W ? (lo = m) : (hi = m);
    }
    el.style.fontSize = lo + 'px';
  });
}
document.fonts.ready.then(() => { ft(); requestAnimationFrame(ft); });
window.addEventListener('resize', () => { clearTimeout(rfTimer); rfTimer = setTimeout(ft, 80); });
```

### 4.4 Scroll Reveal Animations

Elements with class `.fade` are observed by `IntersectionObserver`. When they enter the viewport (threshold 0.1), class `.on` is added:

```js
const io = new IntersectionObserver(entries =>
  entries.forEach(e => {
    if (e.isIntersecting) { e.target.classList.add('on'); io.unobserve(e.target); }
  }), { threshold: 0.1 });
document.querySelectorAll('.fade').forEach(el => io.observe(el));
```

Stagger delays: add `.d1` (0.1s), `.d2` (0.2s), `.d3` (0.3s), `.d4` (0.4s) alongside `.fade`.

### 4.5 Accordions

Two accordion types: **services** (`.sr` rows) and **products** (`.pr` rows). Both use `max-height` transitions to 0/open value. JS simply toggles `.open` class:

```js
// Services
document.querySelectorAll('.sr').forEach(r => {
  r.querySelector('.st').addEventListener('click', () => {
    const o = r.classList.contains('open');
    document.querySelectorAll('.sr').forEach(x => x.classList.remove('open'));
    if (!o) r.classList.add('open');
  });
});
```

### 4.6 Nav Dimming

```js
const nav = document.getElementById('nav');
window.addEventListener('scroll', () =>
  nav.classList.toggle('dim', scrollY > 80), { passive: true });
```

When `body.dim`, nav gets `background: rgba(15,14,11,0.88)` + `backdrop-filter: blur(16px)`.

### 4.7 Active Nav State

Each page's nav link for the current page has class `act`. Update manually per page:

```html
<!-- e.g. on products.html -->
<a href="products.html" class="act">Products</a>
```

---

## 5. Contact Form

The contact form on `contact.html` is **HTML-only** — no backend is wired. To make it functional, choose one of:

| Option | Service | Effort |
|--------|---------|--------|
| **Recommended** | [Formspree](https://formspree.io) | Add `action="https://formspree.io/f/{id}"` and `method="POST"` to `<form>` |
| Self-hosted | PHP mailer / Node Express | Replace `<button>` click with `fetch()` to your API endpoint |
| CRM integration | HubSpot / Salesforce Web-to-Lead | Replace form fields with vendor embed code |
| Low-code | Netlify Forms | Add `netlify` attribute to `<form>` tag if hosting on Netlify |

### Current form field names (for backend mapping):

| Field | HTML name attribute to add | Type |
|-------|---------------------------|------|
| First Name | `name="first_name"` | text |
| Last Name | `name="last_name"` | text |
| Work Email | `name="email"` | email |
| Organization | `name="organization"` | text |
| Role / Title | `name="role"` | text |
| Area of Interest | `name="interest"` | select |
| Message | `name="message"` | textarea |

---

## 6. Deployment

### 6.1 Static Hosting (Recommended)

The site is a folder of static files — deploy to any static host with zero configuration:

| Platform | Command / Method | Free tier |
|----------|-----------------|-----------|
| **Netlify** | Drag-and-drop the folder at netlify.com/drop | Yes |
| **Vercel** | `npx vercel` in the project root | Yes |
| **GitHub Pages** | Push to repo → Settings → Pages → root | Yes |
| **AWS S3 + CloudFront** | Upload files, enable static website hosting | Usage-based |
| **Azure Static Web Apps** | Connect GitHub repo or upload zip | Free tier |
| **Cloudflare Pages** | Connect repo or drag-and-drop | Yes |

### 6.2 Netlify (Recommended for quickest deploy)

```bash
# 1. Install Netlify CLI (one-time)
npm install -g netlify-cli

# 2. Deploy
cd synapse-website
netlify deploy --dir . --prod
```

Or drag the `synapse-website/` folder to [netlify.com/drop](https://netlify.com/drop) — no account needed for a preview URL.

### 6.3 Custom Domain Setup

After deploying to your chosen host:

1. Add a CNAME record pointing `www.synapsehealthtech.ai` → your host's domain
2. Add an A record for the apex domain (`synapsehealthtech.ai`) if required
3. Enable HTTPS (all platforms listed above offer free Let's Encrypt certificates)

### 6.4 Performance Checklist Before Go-Live

- [ ] Compress `Logo_Hi_Res.png` with [squoosh.app](https://squoosh.app) (target < 100KB)
- [ ] Add `<meta name="description">` to each page for SEO
- [ ] Add `<link rel="canonical">` to each page
- [ ] Add `robots.txt` in root (allow all initially)
- [ ] Add `sitemap.xml` in root
- [ ] Test all inter-page links (especially `href="assets/Logo_Hi_Res.png"` path)
- [ ] Wire contact form backend (see §5)
- [ ] Validate Google Fonts load correctly behind corporate firewalls (self-host if needed)
- [ ] Test on Safari (check `backdrop-filter` and `mix-blend-mode` support)
- [ ] Test cursor behaviour on mobile (should be invisible — no issues expected)
- [ ] Add Google Analytics or Plausible snippet if needed

---

## 7. Self-Hosting Fonts (for air-gapped / intranet deployments)

If Google Fonts CDN is not accessible:

1. Download fonts from [google-webfonts-helper.herokuapp.com](https://google-webfonts-helper.herokuapp.com)
2. Select: DM Serif Display (400, 400i), Chivo Mono (300, 400), DM Sans (300, 400, 500)
3. Place `.woff2` files in `assets/fonts/`
4. Replace the Google Fonts `<link>` in each HTML file with:

```css
@font-face {
  font-family: 'DM Serif Display';
  src: url('assets/fonts/dm-serif-display-v15-latin-regular.woff2') format('woff2');
  font-weight: 400;
  font-style: normal;
  font-display: swap;
}
@font-face {
  font-family: 'DM Serif Display';
  src: url('assets/fonts/dm-serif-display-v15-latin-italic.woff2') format('woff2');
  font-weight: 400;
  font-style: italic;
  font-display: swap;
}
/* Repeat for Chivo Mono 300, 400 and DM Sans 300, 400, 500 */
```

---

## 8. Adding New Pages

To add a new page, use this shell:

```html
<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>PAGE TITLE — SynapseHealthTech</title>
<link rel="preconnect" href="https://fonts.googleapis.com">
<link href="https://fonts.googleapis.com/css2?family=DM+Serif+Display:ital@0;1&family=Chivo+Mono:wght@300;400&family=DM+Sans:opsz,wght@9..40,300;9..40,400;9..40,500&display=swap" rel="stylesheet">
<link rel="stylesheet" href="assets/synapse.css">
<style>
  /* Page-specific styles here */
</style>
</head>
<body>
<div id="dot"></div>

<nav id="nav">
  <a href="index.html" class="logo"><img src="assets/Logo_Hi_Res.png" alt="SynapseHealthTech"></a>
  <ul class="nl">
    <li><a href="services.html">Services</a></li>
    <li><a href="products.html">Products</a></li>
    <li><a href="industries.html">Industries</a></li>
    <li><a href="case-studies.html">Case Studies</a></li>
    <li><a href="careers.html">Careers</a></li>
    <li><a href="contact.html">Contact</a></li>
  </ul>
</nav>

<!-- PAGE CONTENT HERE -->

<footer>
  <a href="index.html" class="logo"><img src="assets/Logo_Hi_Res.png" alt="SynapseHealthTech"></a>
  <ul class="fl">
    <li><a href="services.html">Services</a></li>
    <li><a href="products.html">Products</a></li>
    <li><a href="careers.html">Careers</a></li>
    <li><a href="contact.html">Contact</a></li>
    <li><a href="#">Privacy</a></li>
  </ul>
  <span class="fc">© 2025 SynapseHealthTech · Abu Dhabi</span>
</footer>

<script>
const dot = document.getElementById('dot');
document.addEventListener('mousemove', e => {
  dot.style.left = e.clientX + 'px';
  dot.style.top  = e.clientY + 'px';
});
document.querySelectorAll('a, button, input, select, textarea').forEach(el => {
  el.addEventListener('mouseenter', () => document.body.classList.add('lh'));
  el.addEventListener('mouseleave', () => document.body.classList.remove('lh'));
});
const nav = document.getElementById('nav');
window.addEventListener('scroll', () => nav.classList.toggle('dim', scrollY > 80), { passive: true });
const io = new IntersectionObserver(entries =>
  entries.forEach(e => {
    if (e.isIntersecting) { e.target.classList.add('on'); io.unobserve(e.target); }
  }), { threshold: 0.1 });
document.querySelectorAll('.fade').forEach(el => io.observe(el));
</script>
</body>
</html>
```

> **Note:** Current HTML pages have styles inline (in `<style>` tags) rather than linked to `synapse.css`. This is intentional for portability — each file works standalone. `synapse.css` serves as the canonical reference for shared patterns. When the developer refactors to a linked stylesheet, use `synapse.css` as the base.

---

## 9. Key Contacts & Handover Checklist

| Item | Status |
|------|--------|
| All 7 HTML pages | ✅ Complete |
| Logo asset (PNG) | ✅ Provided in assets/ |
| Shared CSS (synapse.css) | ✅ Provided in assets/ |
| Visual design system (style-guide.html) | ✅ Provided in assets/ |
| Contact form backend | ⚠️ Needs wiring (see §5) |
| SEO meta tags | ⚠️ Add before go-live |
| Sitemap / robots.txt | ⚠️ Generate before go-live |
| Analytics | ⚠️ Add snippet if required |
| Custom domain DNS | ⚠️ Configure at host |
| Logo PNG optimization | ⚠️ Compress before launch |

---

*SynapseHealthTech Design & Development Handover · May 2025*
