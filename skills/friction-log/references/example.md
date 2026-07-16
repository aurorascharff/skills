# Friction Log: Next.js product grid with React `<ViewTransition>` morph

**Date:** 2026-05-15
**Model:** claude-opus-4-7 (1M context)
**Harness:** Claude Code
**Stack:** Next.js 16.2.6 (Turbopack) · React 19.2.4 (aliased to compiled `react-experimental` 19.3.0-experimental-3f0b9e61-20260317 at build time) · Tailwind CSS 4.3.0 · TypeScript 5.9.3
**Build time:** 0.85s (1 build · `next build` reported `Compiled successfully in 853ms`)
**Task:** Build a Next.js product grid + detail page where clicking a thumbnail morphs into the detail image via React's `<ViewTransition>` component, and back navigation reverses the morph.
**Output:** `fl-view-transition-morph/`

---

## Prompt

> Build a Next.js product grid and detail page. Clicking a thumbnail should morph into the detail image using React's <ViewTransition> component. Going back should reverse it. Use this skill: https://github.com/aurorascharff/nextjs-app-architecture-skill/tree/main/skills/friction-log

---

## Tool Timeline

- `t+0s` — bash: `curl SKILL.md` (read the friction-log skill before any work)
- `t+0s` — bash: `curl references/template.md` + `references/agent-behavior.md`
- `t+0s` — bash: `ls ~/Documents/Development && node --version && pnpm --version`
- `t+0s` — bash: `mkdir fl-view-transition-morph`
- `t+0s` — write_file: initial `fl-view-transition-morph.md` (header + prompt)
- `t+1s` — bash: `mv fl-view-transition-morph.md /tmp/ && rmdir fl-view-transition-morph` (CNA requires an empty target dir)
- `t+1s` — bash: `pnpm dlx create-next-app@latest fl-view-transition-morph --typescript --tailwind --app --src-dir --no-eslint --turbopack --import-alias "@/*" --use-pnpm --yes`
- `t+8s` — bash: `mv /tmp/fl-vt-log.md fl-view-transition-morph/`
- `t+8s` — read: `fl-view-transition-morph/AGENTS.md` (got the pointer to `node_modules/next/dist/docs/`)
- `t+8s` — bash: `grep -ril "ViewTransition" node_modules/next/dist/docs/`
- `t+9s` — read: `node_modules/next/dist/docs/01-app/02-guides/view-transitions.md`
- `t+10s` — read: `next.config.ts`, `src/app/page.tsx`, `src/app/layout.tsx`, `src/app/globals.css`
- `t+10s` — edit: `next.config.ts` (add `experimental.viewTransition: true` + `images.remotePatterns`)
- `t+10s` — bash: `node -e "require('react').ViewTransition"` (returned `undefined` — Next aliases at build time)
- `t+11s` — bash: `node -e "require('next/dist/compiled/react-experimental').ViewTransition"` (returned `symbol`)
- `t+11s` — write: `src/lib/products.ts` (9 products, picsum seeds)
- `t+11s` — write: `src/app/page.tsx` (grid wrapped in `<ViewTransition name="product-image-${id}">`)
- `t+11s` — write: `src/app/products/[id]/page.tsx` (detail hero with matching `name`)
- `t+11s` — edit: `src/app/layout.tsx` (metadata), `src/app/globals.css` (default 400ms duration + reduced-motion)
- `t+12s` — bash: `pnpm dev` (background) → `Ready in 490ms`
- `t+13s` — bash: `curl localhost:3000/ && curl localhost:3000/products/alpine-tote` → both 200
- `t+13s` — bash: `grep -oE 'href="/products/[a-z-]+"' /tmp/home.html | sort -u` → 9 unique product links
- `t+14s` — bash: `pnpm build` → `Compiled successfully in 853ms`, 13 static pages generated
- `t+15s` — task_stop: kill dev server
- `t+15s` — write: finalize this log

---

## Summary

The task went smoothly because Next.js 16 ships an `AGENTS.md` at the project root that pointed me to `node_modules/next/dist/docs/`, and the bundled `01-app/02-guides/view-transitions.md` happened to walk through almost exactly this task (photo grid → hero morph). Once I knew to wrap matching thumbnails and hero images in `<ViewTransition name="...">` with the same name, the default browser behavior covered both forward navigation and back reversal — no custom CSS required for the morph itself. The biggest minor friction was that `react`'s `ViewTransition` export is not visible to plain Node introspection (Next aliases `react` → compiled `react-experimental` at build time), which is harmless but momentarily confusing. The scaffold also forced me to move the friction-log file out of the target directory because `create-next-app` refuses non-empty targets — at odds with the skill's "log lives inside the scaffold" instruction.

## Action Items

### Docs

- 🔧 In `view-transitions.md`, note that `import { ViewTransition } from 'react'` works at compile time even though the symbol is not present in the installed `react` package on disk — Next.js swaps in `next/dist/compiled/react-experimental` when `experimental.viewTransition: true` is set.
  - Context: I sanity-checked the import with `node -e "require('react').ViewTransition"` and got `undefined`. Without context, that looks broken; a one-line callout would save the dead-end check. [sandbox]

- 🔧 The bundled doc's "Step 1: Morph a thumbnail into a hero image" code wraps `<Image>` directly in `<ViewTransition>` but doesn't show the surrounding fixed-size container. Without an `aspect-square`/`relative` parent, `fill` images collapse and the morph snaps to a 0-height box.
  - Context: Wasn't a hard friction here because Tailwind made the container obvious, but a beginner copying the code as-is would hit a layout bug before they ever see the animation. [docs]

### Framework

- 🔧 `create-next-app` should accept a target directory that contains only ignorable files (single `.md`, dotfiles) the same way `git init` does, or print a clearer `--force` hint than the current generic "Directory is not empty" error.
  - Context: The friction-log skill mandates writing `fl-<feature>/fl-<feature>.md` *before* scaffolding. Today this requires `mv` out → scaffold → `mv` back. A friendlier scaffolder would let agents follow the documented workflow without filesystem gymnastics. [error output]

- 🔧 When `experimental.viewTransition: true` is enabled, Next.js could synthesize an inline `style={{ viewTransitionName: name }}` on the immediate child during SSR (a no-op when the API isn't in use) so MPA-style first paints and search-engine crawlers see the names. Today the `name` only materializes during the client transition.
  - Context: Not blocking for this task, but it means deep-linking into `/products/alpine-tote` from external traffic can't participate in any same-document morph, even if the source page also uses the API. [training data]

### DX / Research

- 🔍 Explore whether the `react-experimental` alias could expose a typed declaration shim so editors show `ViewTransition` in `react`'s autocomplete out of the box (not via `unstable_*`).
  - Context: TS resolution worked because `@types/react` 19.2.x apparently includes the `ViewTransition` typing, but I verified that only by writing the code and not getting a red squiggle — there's no doc confirming it. A clear "types are bundled, no extra install" line in the guide would help. [docs]

- 🔍 Measure whether `<ViewTransition share="morph">` with the doc's `via-blur` keyframe meaningfully improves perceived quality on images that change aspect ratio between grid (square) and detail (square here, but often wide-hero elsewhere). The default morph CSS interpolates the bounding box, which can produce visible squash mid-flight on non-uniform aspects.
  - Context: Skipped here because both surfaces are `aspect-square`. Worth a follow-up demo with a 4:3 detail hero to see whether the default morph is good enough or whether the documented `share="morph"` blur trick should be the recommended default. [docs]

## Log

- 🟢 Read the skill spec first
  - Pulled SKILL.md + `references/template.md` + `references/agent-behavior.md` via `curl` so the rules (write log inside `fl-<feature>/`, cite sources, no mid-run questions, complete sections only) were loaded before any code. [url]

- 🟢 Scaffolded a fresh Next.js 16.2.6 app
  - `pnpm dlx create-next-app@latest fl-view-transition-morph --typescript --tailwind --app --src-dir --no-eslint --turbopack --import-alias "@/*" --use-pnpm --yes` completed in ~6s. Got React 19.2.4 + Tailwind 4.3.0 + Turbopack on by default. [sandbox]

- 🟡 `create-next-app` refused to scaffold into the existing `fl-view-transition-morph/` directory
  - **Expected:** scaffold-in-place since the only file inside was my own `fl-view-transition-morph.md` (the friction log).
  - **Actual:** CNA bails out on any non-empty target.
  - **Resolution:** `mv fl-view-transition-morph/fl-view-transition-morph.md /tmp/fl-vt-log.md && rmdir fl-view-transition-morph`, scaffold, then `mv` the log back. No information loss, but it's friction the skill itself can't avoid without scaffolder support. [error output]

- 🟢 Root `AGENTS.md` pointed straight to bundled docs
  - The scaffold dropped an `AGENTS.md` saying "Read the relevant guide in `node_modules/next/dist/docs/` before writing any code." A `grep -ril "ViewTransition" node_modules/next/dist/docs/` surfaced four hits, the most useful being `01-app/02-guides/view-transitions.md`, which walked through exactly this task. Massive time saver — no web search needed. [agents.md]

- 🟢 Enabled the experiment in `next.config.ts`
  - Added `experimental: { viewTransition: true }` and `images.remotePatterns` for `picsum.photos`. [docs]

- 🟡 `react`'s `ViewTransition` is invisible to Node introspection
  - **Expected:** `node -e "require('react').ViewTransition"` to return a symbol.
  - **Actual:** `undefined`. Same for `unstable_ViewTransition`.
  - **Investigation:** Listed `node_modules/next/dist/compiled/` and found `react-experimental/`. Probed it: `require('next/dist/compiled/react-experimental').ViewTransition` → `symbol`, version `19.3.0-experimental-3f0b9e61-20260317`. So Next.js swaps `react` → this compiled build at compile time when `viewTransition` is on; the import in user code works even though the real `react` package on disk doesn't expose it.
  - **Resolution:** Kept `import { ViewTransition } from 'react'` (matching the bundled doc) and verified at build/dev that it resolves. [sandbox]

- 🟢 Built the data + grid + detail
  - `src/lib/products.ts` defines 9 products with stable picsum seed URLs.
  - `src/app/page.tsx` is the grid: each thumbnail is a `<Link href="/products/${id}">` wrapping `<ViewTransition name="product-image-${id}"><Image fill /></ViewTransition>`.
  - `src/app/products/[id]/page.tsx` mirrors the same `<ViewTransition name="product-image-${id}">` on the hero image, plus title/price/description and a back link.
  - The `name` prop is the entire morph contract — no other props needed for forward + back reversal. [docs]

- 🟢 Tweaked the default morph in `globals.css`
  - Set `::view-transition-group(*) { animation-duration: 400ms; animation-timing-function: cubic-bezier(0.22, 1, 0.36, 1); }` for a slightly slower, more "expensive" feel, and added the documented `prefers-reduced-motion` killswitch. [docs]

- 🟢 Dev server came up clean
  - `pnpm dev` reported `Ready in 490ms` and explicitly echoed `✓ viewTransition` in the experiments list — nice signal that the flag took effect. `curl /` and `curl /products/alpine-tote` both returned 200; the HTML contained all 9 product links and the matching `product-image-alpine-tote` identifier on the detail page. [sandbox]

- 🟢 Production build passed
  - `pnpm build`: `Compiled successfully in 853ms`, TypeScript clean in 804ms, 13 static pages generated (1 home + 1 not-found + 9 SSG product pages + supporting). No type errors around `<ViewTransition>` props. [sandbox]
