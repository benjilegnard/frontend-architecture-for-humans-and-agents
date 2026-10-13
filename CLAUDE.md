# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this repo is

A reveal.js slide deck for a 45-minute French-language conference talk: "Renforçons les règles d'architecture frontend, pour les humains comme pour les agents" (subtitle: "Votre architecture n'existe que si elle est exécutable"). `README.md` holds the abstract, the detailed section-by-section plan, open decisions, and sources — treat it as the spec for slide content. Slide content and speaker notes are written in French.

## Commands

Package manager is pnpm (version pinned via `packageManager` / `devEngines`).

- `pnpm dev` — Vite dev server with live reload
- `pnpm build` — type check (`tsc -b`) then production build into `dist/`; type errors fail the build
- `pnpm preview` — serve the built `dist/`
- `pnpm exec tsc` — type check only

There are no tests or linters.

## Checking slides visually

With `pnpm dev` running, the deck is at `http://localhost:5173`. After editing slides, check the rendering in the browser (Chrome DevTools MCP when available): screenshot the changed slides and check the console for errors.

- URLs are 1-based: `/#/<section>/<vertical>`, e.g. `/#/2/3`
- PageDown / PageUp walk every slide in order; arrow keys move between sections (Right/Left) and within one (Down/Up)

## Architecture

- **Slide content lives in `public/frontend-architecture-for-humans-and-agents.md`**, loaded at runtime by the reveal.js Markdown plugin via `data-markdown` in `index.html`. It sits in `public/` so it's served as-is (not bundled).
- **Slide separators** (configured in `index.html`):
  - A line containing only `---` (`^---$`) → new horizontal slide (a big section)
  - Blank lines (`^\n\n`) → new vertical slide within a section. A single blank line inside a slide is safe, but two blank lines in a row will split it, so be careful when editing whitespace.
  - `Notes:` at the start of a line → speaker notes for the rest of the slide
- **Deck structure:** the first section is the talk itself, following the README plan. Separate "Exemples mermaid" and "Exemples de code" sections after it hold reference diagrams and snippets (based on the README sources).
- `src/main.ts` initializes Reveal with Markdown, Mermaid (`@benjilegnard/reveal.js-mermaid-plugin`, the author's own plugin), Notes, and Highlight plugins. Plugin order matters: Mermaid after Markdown, before Highlight.
- `src/style.scss` builds a custom reveal.js theme by injecting Catppuccin Mocha colors into reveal's theme template (`node_modules/reveal.js/css/theme/template/*`).
- **Mermaid styling** is global: theme variables in `src/main.ts` (mapped from Catppuccin Mocha), CSS overrides in `src/style.scss` for what the variables can't reach, and a shared SVG gradient in `index.html`. Fix diagram styling there, not per diagram.
- **Code highlights:** by default a code block shows all the code, with no line metadata. To walk through parts of it, the reveal.js line metadata must start with `[|` so the first step shows all the code before the highlighted steps, e.g. ```` ```ts [|6-8|11] ````.
- **Writing Mermaid slides:** use ```` ```mermaid ```` blocks, never with blank lines inside. Prefer top-down layouts and short labels so diagrams stay readable at slide size.
- `vite.config.ts` contains a small custom plugin that forces a full page reload when any `.md` file changes (Vite HMR doesn't pick up the runtime-fetched markdown otherwise).

## Deployment

`.github/workflows/publish.yml` builds on every push to `main` and deploys `dist/` to GitHub Pages with `--base=/frontend-architecture-for-humans-and-agents/`. Asset paths must work under that base path.
