# PYNK — Developer Handoff

Take the pink one — an animated 3D supplement store built around one pink capsule that transforms across the entire page.

- **Live site:** https://rosicjacob-sys.github.io/pynk-handoff/
- **This guide:** https://rosicjacob-sys.github.io/pynk-handoff/dev/
- **Source + build zip:** https://rosicjacob-sys.github.io/pynk-handoff/dl/pynk.zip

> These links are hosted and always available — they do not depend on anyone's
> machine being on. The zip is rebuilt with the site on every publish, so
> always download it fresh rather than keeping a local copy.

---

## What this is

A production-built **React 18 + Vite + TypeScript** single-page landing page using
GSAP + ScrollTrigger, three.js / react-three-fiber (WebGL) and Lenis smooth scroll,
with a working demo cart and a **swappable checkout seam**.

> ⚠️ **Read first:** this is a React SPA, **not** a Shopify Liquid theme. You cannot drop
> the folder into a theme as-is. The three real integration paths are below.

**Product:** PYNK Daily — a daily wellness capsule (dietary supplement). Variants: 1 bottle / 3-pack / subscribe & save.

---

## What's in the zip

```
pinkpage2/
  src/            all source (components, three/ WebGL, styles, data)
  public/         self-hosted fonts + any generated imagery
  dist/           PRODUCTION BUILD — preview or deploy immediately
  index.html
  package.json + package-lock.json
  scripts/serve-dist.cjs    zero-dependency static preview server
  HANDOFF.md / HANDOFF.pdf  this guide
```

Not included (regenerate locally): `node_modules/`, `.git/`.

## Run it locally

```bash
cd pinkpage2
npm install          # Node 18+

npm run dev          # hot-reloading dev server
# —or— preview the exact production build (no install needed):
node scripts/serve-dist.cjs
```

---

## Getting this onto Shopify — three real paths

### Path A — Reference + reimplement in the theme ★ recommended for production
Treat this page as the pixel-and-motion spec and rebuild the sections as **Liquid
sections/blocks**. Reuse the same libraries (GSAP, three.js, Lenis) as theme assets and
wire commerce to Shopify's native **AJAX Cart** (`/cart/add.js`, `/cart.js`). You keep
theme-native cart, SEO and Theme-Editor control.

- `src/styles/global.css` — design tokens + layout. **Copy the `:root` variables first.**
- `src/components/*.tsx` — section structure and copy.
- `src/three/*` — the WebGL; framework-light and portable almost verbatim.

### Path B — Embed the built SPA ★ fastest
Host `dist/` (theme assets or external CDN) and load it in a full-width custom section or
a dedicated template. Point `checkout()` at a Shopify **cart permalink**
(`/cart/{variantId}:{qty}`) or the Storefront API. Trade-off: heavier bundle, the SPA cart
is separate from the theme cart, less Theme-Editor control.

### Path C — Headless (Hydrogen / Storefront API)
If the store is headless, these React components port **directly**.
`checkout()` → Storefront API `cartCreate` / `cart.checkoutUrl`.

---

## The four Shopify seams (already isolated)

| Seam | File | What to do |
|---|---|---|
| **Checkout** | `src/lib/checkout.ts` | `setCheckoutProvider(fn)` — implement ONE adapter. Nothing else changes. |
| **Cart state** | `src/store/cart.ts` | Zustand + localStorage demo cart. Mirror to Shopify AJAX Cart, or replace calls with `/cart/add.js`. |
| **Products / prices / copy** | `src/data/product.ts` | Every variant, price and string in ONE file. Map each `variantId` to a real Shopify variant ID. |
| **Analytics** | `src/lib/analytics.ts` | `view_item`, `add_to_cart`, `begin_checkout` already fire to `window.dataLayer`. Point at GA4 / Shopify analytics. |

Example checkout adapter (Path B — cart permalink):

```ts
import { setCheckoutProvider } from './lib/checkout'

setCheckoutProvider(async (items) => {
  const q = items.map(i => `${SHOPIFY_VARIANT_ID[i.variantId]}:${i.qty}`).join(',')
  window.location.href = `https://YOUR-STORE.myshopify.com/cart/${q}`
  return { ok: true }
})
```

---

## This theme specifically

**Palette** — `--porcelain #FDF6FA` canvas · `--pink-royal #FF2E88` accent · `--pink-deep #C4126B` · `--plum-ink #1A0A14` for the rare dark sections.

**Type** — Clash Display (display) + Satoshi (body) — self-hosted variable woff2 in `public/fonts/`.

**Sections, in order**

1. Loader
2. Hero
3. Trust marquee
4. The Science (dark, pinned)
5. How it works
6. Benefits grid
7. Proof / reviews
8. Buy block
9. FAQ + compliance
10. Footer
11. Sticky buy bar
12. Cart drawer

**Key animation work to preserve**

- One persistent 3D capsule on a single fixed WebGL canvas, choreographed across every section
- Signature beat: the capsule splits open and dissolves into GPU-instanced granules, which reassemble into labeled ingredient nodes
- Masked character reveals on headlines (split-type, 0.032 stagger, power4.out)
- Magnetic CTAs, custom cursor, fly-to-cart arc on add

---

## What's hardened — please keep

- **`prefers-reduced-motion`** fully disables the spectacle; the page stays calm and buyable.
- **WebGL never blanks** — capability probe + error boundary → static fallback. No GPU still gives a complete page.
- **~3.5 s reveal failsafes** — anything that animates in force-reveals; content cannot get stuck invisible.
- **One WebGL context**, DPR-clamped `[1,2]`, paused offscreen, adaptive quality under load.
- **Cart drawer is a real modal** — focus trap, background `inert`, `Esc` to close.
- **No layout shift** — canvas and images reserve their space.

## Stack

React 18 · Vite (rolldown) · TypeScript · three ~0.169 · @react-three/fiber 8 · drei 9 ·
gsap 3 (free plugins only — no Club plugins) · lenis 1 · split-type · zustand.
Evergreen browsers; WebGL2 preferred, WebGL1 / no-GL gracefully degraded.
Fonts are self-hosted woff2 (no runtime Google Fonts request).

---

## Compliance — read before launch

Dietary supplement. Structure/function claims only ("supports calm focus", "supports clean energy") with the standard FDA disclaimer, a Supplement Facts panel, suggested use and a consult-your-physician line. Do not substitute disease/treatment claims.

---

*The four files in the seams table are the entire commerce surface. Start there.*
