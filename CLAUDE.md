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

## Checking slides visually

With `pnpm dev` running, the deck is at `http://localhost:5173`. When the Chrome DevTools MCP is available, use it to check rendering (especially Mermaid diagrams) after editing slides:

- URLs use 1-based hashes: `/#/<section>/<vertical>`, e.g. `/#/2/3` (`hashOneBasedIndex: true`)
- Navigate with PageDown / PageUp (walks every slide in order), arrow keys (Right/Left between sections, Down/Up within one), or Home for the first slide
- Take a screenshot per slide and check the console for errors

## Architecture

- **Slide content lives in `public/frontend-architecture-for-humans-and-agents.md`**, loaded at runtime by the reveal.js Markdown plugin via `data-markdown` in `index.html`. It sits in `public/` so it's served as-is (not bundled).
- **Slide separators** (configured in `index.html`):
  - A line containing only `---` (`^---$`) → new horizontal slide (a big section)
  - Blank lines (`^\n\n`) → new vertical slide within a section. A single blank line inside a slide is safe, but two blank lines in a row will split it, so be careful when editing whitespace.
  - `Notes:` at the start of a line → speaker notes for the rest of the slide
- **Deck structure:** the first horizontal section is the talk itself (title slide, then content following the README plan). After a `---`, a separate "Exemples mermaid" section holds reference diagrams (feature spaghetti, Sheriff depRules between types, features via `index.ts`, agent stop-hook loop). Keep examples there unless asked to move them into the talk.
- `src/main.ts` initializes Reveal with Markdown, Mermaid (`@benjilegnard/reveal.js-mermaid-plugin`, the author's own plugin), Notes, and Highlight plugins. Plugin order matters: Mermaid must come after Markdown and before Highlight. Mermaid theme variables are mapped from the Catppuccin Mocha palette (including `edgeLabelBackground`, otherwise edge labels get black boxes). Mermaid 12 defaults to the "neo" look, which adds grey drop shadows: `dropShadow: "none"` in `themeVariables` removes them on flowcharts, and a CSS rule in `src/style.scss` removes the SVG `filter` attribute that sequence diagrams hard-code on actors.
- **Writing Mermaid slides:** use ```` ```mermaid ```` fenced blocks in the markdown. Never put blank lines inside a diagram (they split the slide). Prefer `flowchart TD`: `LR` flowcharts render wide and tiny in the slide viewport. Keep node and participant labels short, since long sequence-diagram participant names overflow their boxes. A diagram that fails to render stays as source with a `mermaid-error` class and logs to the console.
- `src/style.scss` builds a custom reveal.js theme by injecting Catppuccin Mocha colors into reveal's theme template (`node_modules/reveal.js/css/theme/template/*`).
- `vite.config.ts` contains a small custom plugin that forces a full page reload when any `.md` file changes (Vite HMR doesn't pick up the runtime-fetched markdown otherwise).

## Deployment

`.github/workflows/publish.yml` builds on every push to `main` and deploys `dist/` to GitHub Pages with `--base=/frontend-architecture-for-humans-and-agents/`. Asset paths must work under that base path.
