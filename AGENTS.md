# QRSmith

A single-purpose, standalone tool: turn any URL or text into a styled QR
code with an embedded logo or initials. 100% client-side. No signup.

This project was split out of the multi-tool "anyconvert" site into its
own standalone domain/deploy. **It is deliberately single-purpose.** Do
not add other tools, a multi-tool nav, or cross-links to sibling tools
(anyconvert, pixelforge, reelshift, stillmotion, scriptflow, etc). The
SEO strategy for this entire site family is one keyword-matched domain
per tool — "qrsmith" should read and rank as the QR code generator, not
as a page inside a bigger utility dump.

## Stack

- Astro 7, static output (`output: "static"`, the default) — zero
  server runtime, deploys as pure static files.
- Tailwind v4 via `@tailwindcss/vite` (NOT the old `@astrojs/tailwind`
  integration — Tailwind v4 is Vite-plugin-based).
- `qr-code-styling` for QR generation/rendering/download (PNG + SVG),
  all client-side in a page `<script>` block (no server function).
- `@astrojs/sitemap` for `sitemap-index.xml` generation at build time.
- No React/Vue/other UI framework — plain Astro components + vanilla
  TypeScript in `<script>` tags.

## Design tokens (do not deviate — shared family look across all split-out tools)

- Background: `#FAF6EC` (warm cream)
- Card/surface: `#FFFFFF`, border `#E8DFC8` (warm, not gray)
- Heading text: `#2B2013` (warm espresso brown, not pure black)
- Body text: `#5C4F3D` (warm brown-gray)
- Accent / CTA / links / active state: `#C9982E` (warm gold), hover
  `#B8860B` (darker gold)
- Headings: **Fraunces** (Google Font, soft-serif/display), weight
  600–700, slightly negative letter-spacing
- Body / labels / buttons / UI: **Inter** (Google Font)
- Both fonts loaded via `<link>` in `Layout.astro` head with
  `display=swap`
- Rounded-xl corners, soft shadows only (`shadow-sm`), generous
  whitespace, no gradients, no dark mode toggle (cream-only is the
  point — do not add a `prefers-color-scheme` dark variant)
- The whole point is "small, confident, premium product" — not a
  generic free-tool dump. Resist adding ad-dump styling, banner clutter,
  or "50 other tools" cross-promo blocks.

## Pages

- `/` — the QR generator tool itself (`src/pages/index.astro`)
- `/about` — what the tool is and why it's single-purpose
- `/privacy` — no data collection (everything is client-side), hosting
  disclosure, ad-network disclosure
- `/faq` — common questions (logo scannability, formats, expiry, etc)
- `/404` — custom not-found page

These four content pages (about/privacy/faq/404) exist specifically
because they're a baseline AdSense eligibility requirement across this
whole site family — do not remove them when trimming the site down.

## Security

Any user-controlled text that is ever rendered via `innerHTML` MUST go
through the `escapeHtml()` helper defined in `index.astro` first. This
is a standing requirement across the whole tool family, even though the
current implementation mostly uses `textContent` and canvas APIs (which
are inherently safe) rather than `innerHTML`.

## Dev / build

```bash
npm install
npm run dev      # http://localhost:4350
npm run build    # static output to dist/
npm run preview  # serve the built dist/ locally
```

Dev server is pinned to port **4350** (many sibling tools in this
split-out family occupy 4331-4342; pick a free port in the 4340–4360
range if this one ever collides — check with `lsof -i :<port>` first).

### iCloud sync hygiene

`node_modules` is a **symlink** to `node_modules.nosync` — the real
directory. This keeps iCloud Drive from trying to sync hundreds of
thousands of tiny package files (which chokes sync and corrupts state
across the two Macs). Never delete the symlink and re-materialize
`node_modules` as a real directory; if you need to reinstall from
scratch:

```bash
rm -rf node_modules node_modules.nosync
npm install
mv node_modules node_modules.nosync
ln -s node_modules.nosync node_modules
```

`tsconfig.json`'s `exclude` array includes **both** `"node_modules"`
and `"node_modules.nosync"`. This is required — `astro check` will
otherwise crash trying to type-check into the real
`node_modules.nosync` directory through the symlink. Do not remove
either entry.

`dist/` and `.astro/` are also build-local and untracked/regenerable.

## Deploy

Static site on Netlify.

```bash
source ~/.claude/credentials/netlify.env
export NETLIFY_AUTH_TOKEN="$NETLIFY_API_KEY"
npm run build
netlify deploy --prod --dir=dist
```

Site is linked via `.netlify/state.json` (site name/ID: `qrsmith`, or
whatever suffix Netlify assigned if the exact name was taken — check
`.netlify/state.json` for the actual site ID and the deploy output for
the live URL).

## CLAUDE.md

`CLAUDE.md` in this directory is a symlink to this file (`AGENTS.md`).
Edit `AGENTS.md`; do not create a separate `CLAUDE.md`.
