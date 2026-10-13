# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this repo is

A reveal.js slide deck for a 45-minute French-language conference talk: "Renforçons les règles d'architecture frontend, pour les humains comme pour les agents" (subtitle: "Votre architecture n'existe que si elle est exécutable") (enforcing front-end architecture rules with Sheriff, ts-arch, ESLint, and agent feedback loops). `README.md` holds the abstract, the detailed section-by-section plan with timings, open decisions, and sources — treat it as the spec for slide content. Slide content and speaker notes are written in French.

## Commands

Package manager is pnpm (version pinned via `packageManager` / `devEngines`).

- `pnpm dev` — Vite dev server with live reload
- `pnpm build` — type check (`tsc -b`) then production build into `dist/`; type errors fail the build
- `pnpm preview` — serve the built `dist/`
- `pnpm exec tsc` — type check only

There are no tests or linters.

## Architecture

- **Slide content lives in `public/frontend-architecture-for-humans-and-agents.md`**, loaded at runtime by the reveal.js Markdown plugin via `data-markdown` in `index.html`. It sits in `public/` so it's served as-is (not bundled).
- **Slide separators** (configured in `index.html`):
  - A line containing only `---` (`^---$`) → new horizontal slide (a big section)
  - Blank lines (`^\n\n`) → new vertical slide within a section. A single blank line inside a slide is safe, but two blank lines in a row will split it, so be careful when editing whitespace.
  - `Notes:` at the start of a line → speaker notes for the rest of the slide
- `src/main.ts` initializes Reveal with Markdown, Mermaid (`@benjilegnard/reveal.js-mermaid-plugin`, the author's own plugin), Notes, and Highlight plugins. Mermaid theme variables are mapped from the Catppuccin Mocha palette.
- `src/style.scss` builds a custom reveal.js theme by injecting Catppuccin Mocha colors into reveal's theme template (`node_modules/reveal.js/css/theme/template/*`).
- `vite.config.ts` contains a small custom plugin that forces a full page reload when any `.md` file changes (Vite HMR doesn't pick up the runtime-fetched markdown otherwise).

## Deployment

`.github/workflows/publish.yml` builds on every push to `main` and deploys `dist/` to GitHub Pages with `--base=/frontend-architecture-for-humans-and-agents/`. Asset paths must work under that base path.
