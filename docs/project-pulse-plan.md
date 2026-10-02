# Project Pulse — Implementation Plan

## 1. Overview

**Project Pulse** is a lightweight, static project-status dashboard served from the `app/` directory. It renders a sample set of projects (name, owner, status, progress, last update, etc.) from a local JSON file into a clean, scannable visual dashboard.

**User goal:** A project stakeholder opens the dashboard locally in VS Code and immediately sees the health of every tracked project — status at a glance (on-track / at-risk / blocked), progress bars, and owners — without needing a backend, build step, or external service.

**Scope guardrails:**
- Static only: HTML + CSS + a small amount of vanilla JS (fetch + render).
- No frameworks, no bundlers, no package.json required.
- Must be runnable and debuggable from VS Code using `.vscode/launch.json`.
- Data lives in `app/project-data.json` and is loaded at runtime via `fetch`.

---

## 2. File Assignments

| File | Owner | Purpose | Required Contents |
|---|---|---|---|
| `app/index.html` | **Coder** (structure & wiring) — **Designer** contributes semantic layout guidance | Dashboard markup and entry point | Semantic HTML5 skeleton, a dashboard header with title + summary counters (total / on-track / at-risk / blocked), a container element (e.g. `<ul id="project-list">`) that JS populates, a `<template id="project-card">` for a single project card, link to `styles.css`, inline or linked `<script>` that fetches `project-data.json` and renders cards. Must include ARIA labels and a `<noscript>` fallback message. |
| `app/styles.css` | **Designer** (sole owner) | Visual styling, layout, design system | CSS custom properties for color tokens (status colors, surface, text, accent), typography scale, spacing scale, responsive grid/flex layout for the card list, project card component styles, progress-bar component, status pill/badge component, focus-visible styles, prefers-reduced-motion handling, and a high-contrast palette. |
| `app/project-data.json` | **Coder** (sole owner) | Sample project status data | A JSON array of 6–10 project objects with a stable schema. Required fields: `id`, `name`, `owner`, `status` (one of `on-track`, `at-risk`, `blocked`, `complete`), `progress` (0–100), `lastUpdated` (ISO 8601), `description`, optional `tags`. Must be valid JSON. |
| `.vscode/launch.json` | **Coder** (sole owner) | VS Code run/debug configuration | A `launch.json` (version `0.2.0`) with a configuration that opens `app/index.html` in a browser. Recommended: Chrome/Edge launch against `http://localhost:8080` with a `preLaunchTask` that runs `python3 -m http.server 8080` from `app/` (because `fetch` of a local JSON file is blocked under `file://`). Must coexist with the existing `.vscode/tasks.json`. |

Note: `.vscode/tasks.json` already exists. The Coder must add `launch.json` next to it, not replace the folder. If a `preLaunchTask` is used, add the corresponding task to `tasks.json` (shared-file edit — see §6).

---

## 3. Designer Responsibilities

The Designer produces the visual direction and the implemented stylesheet.

Deliverables:
1. Visual direction brief (short, as a comment at top of `styles.css`):
   - Mood: calm, professional, scannable.
   - Primary surfaces + accent color.
   - Status color mapping: `on-track` → green, `at-risk` → amber, `blocked` → red, `complete` → neutral/blue.
2. Design tokens in `styles.css` as `:root` CSS custom properties:
   - Color tokens (surface, surface-elevated, text-primary, text-muted, border, accent, status-*).
   - Spacing scale (e.g. `--space-1` … `--space-6`).
   - Typographic scale (system font stack; sizes for h1/h2/body/meta).
   - Border radius + shadow tokens.
3. Layout:
   - Responsive grid of project cards (1 column mobile, 2–3 columns tablet/desktop).
   - Prominent header with summary counts.
4. Component styling:
   - Project card (title, owner, description, meta row).
   - Status pill/badge (uses status color tokens).
   - Progress bar (accessible — text label present, not color-only).
5. Accessibility:
   - WCAG AA contrast for all status colors on their backgrounds.
   - Visible `:focus-visible` outlines.
   - Status pills include text, not color alone.
   - Respect `prefers-reduced-motion`.
   - Minimum tap target 44×44 CSS px for interactive elements.
6. Structural guidance to Coder for `index.html`:
   - Required class names / data attributes the stylesheet expects.
   - Required DOM structure of the card template (short snippet in the brief).

Files owned: `app/styles.css`.
Files contributed to: `app/index.html` (class-name contract and card template structure only — does not edit the file).

---

## 4. Coder Responsibilities

The Coder produces the structure, data, data-wiring, and local run/debug config.

Deliverables:
1. `app/project-data.json`:
   - Define and freeze the schema (see §2).
   - Populate with 6–10 realistic sample projects covering all four status values.
2. `app/index.html`:
   - Semantic HTML implementing the Designer's class-name contract.
   - A `<template id="project-card">` matching the Designer's card structure.
   - A vanilla-JS block (`<script defer>` or `type="module"`) that:
     - fetches `./project-data.json`,
     - computes summary counts by status and updates header counters,
     - clones the template per project and injects name, owner, description, status pill, progress bar (width + `aria-valuenow`), and formatted `lastUpdated`,
     - handles fetch/parse errors by rendering a visible error message.
   - `<noscript>` fallback explaining JS is required.
3. `.vscode/launch.json`:
   - Launch configuration that opens the dashboard for debugging.
   - Recommended: Chrome/Edge launch against `http://localhost:8080` with a `preLaunchTask` that starts `python3 -m http.server 8080` from `app/`.
   - If adding a `preLaunchTask`, add the corresponding task to the existing `.vscode/tasks.json` (background task, problem matcher that resolves on "Serving HTTP").

Files owned: `app/index.html`, `app/project-data.json`, `.vscode/launch.json`.
Files contributed to: `.vscode/tasks.json` (append a server task if needed — shared edit, coordinate).

---

## 5. Dependencies

Explicit ordering (A → B means A must finish before B can finalize):

1. Data schema (Coder, `project-data.json`) → `index.html` render script. The JS renderer needs finalized field names.
2. Designer's class-name / DOM contract → Coder's `index.html` final markup.
3. Designer's status token names (`--status-on-track`, etc.) → Coder's status pill class strategy in JS (renderer sets `class="pill pill--{status}"` which must match CSS selectors).
4. Final `app/` file paths → `.vscode/launch.json`. The launch URL and server `cwd` depend on where `index.html` lives.
5. `tasks.json` server task (if added) → `launch.json` `preLaunchTask` reference. The task name must exist before the launch config can reference it.

---

## 6. Parallel Work Decisions

Can run in parallel (no file-scope overlap, no data dependency yet):
- Designer drafts visual direction brief + design tokens in `styles.css`.
- Coder scaffolds `app/project-data.json` (schema + sample rows).
- Coder drafts `.vscode/launch.json` against the known target path `app/index.html`.

Must run sequentially (file overlap or contract dependency):
- Final `app/index.html` markup — blocked on both the Designer's class-name contract and the Coder's finalized JSON schema. Coder is the single owner of the edit; integrates the Designer's documented contract.
- `.vscode/tasks.json` edit (if a server task is required) — shared with the pre-existing file; Coder owns the edit and must preserve existing tasks.
- Component styling in `styles.css` that targets specific class names can be written in parallel against the agreed contract, but is only *verified* once `index.html` lands.

Recommended execution order:
1. Parallel kickoff: Designer → tokens + direction brief + DOM contract; Coder → `project-data.json` + draft `launch.json`.
2. Sync point: publish Designer DOM contract + Coder JSON schema.
3. Coder finalizes `index.html` and JS renderer against both contracts.
4. Designer finalizes component CSS against actual `index.html`.
5. Coder verifies `launch.json` + `tasks.json` end-to-end.

---

## 7. Edge Cases to Handle

- `fetch` under `file://` — Chrome/Edge block it. Launch config must use a local HTTP server, or the renderer must fall back gracefully with a clear error.
- Empty / malformed `project-data.json` — renderer shows a visible error state, not a blank page.
- Progress values out of range (negative or >100) — clamp to 0–100 before setting bar width and `aria-valuenow`.
- Missing optional fields (`tags`, `description`) — render without breaking layout.
- Long project names / owner strings — CSS must handle wrap/ellipsis.
- Zero projects — empty-state message.
- Status value not in the known enum — render a neutral "unknown" pill, do not crash.
- Date formatting — use `Intl.DateTimeFormat` for `lastUpdated`; handle invalid dates.
- Color-only status signaling — status pills must always include the text label.
- Reduced motion — disable progress-bar transitions when `prefers-reduced-motion: reduce`.
- Port already in use for the preLaunch server — document fallback port.

---

## 8. Validation Expectations

Acceptance checks before considering the dashboard "done":

1. Launch works: Pressing F5 in VS Code with the Project Pulse config opens the dashboard in a browser (via the configured server) with no manual steps.
2. Data renders: All projects from `app/project-data.json` appear as cards. Editing the JSON and reloading reflects the change (confirms live data wiring).
3. Summary counters match: Header counts equal the actual counts per status in the JSON.
4. Visual check vs. Designer direction: Colors, spacing, typography, and card layout match the Designer's brief / tokens. Status pills use the correct color per status.
5. Accessibility pass (basic):
   - Keyboard: tab order is logical, focus is visible on all interactive elements.
   - Screen reader: progress bars expose `role="progressbar"` with `aria-valuenow`, `aria-valuemin`, `aria-valuemax`.
   - Contrast: status pill text passes WCAG AA against pill background.
   - Status is not conveyed by color alone.
6. No console errors / no 404s on load (check DevTools Console + Network).
7. Graceful failure: Temporarily rename `project-data.json`; the page shows a visible error, not a blank screen.
8. Responsive: Resize from ~360px up to ~1440px — layout reflows without horizontal scroll or clipped content.

---

## 9. Open Questions

1. Server strategy for `launch.json`: Prefer `python3 -m http.server` (zero install, available in the devcontainer) or the "Live Server" VS Code extension? Recommendation: `python3 -m http.server` via `preLaunchTask`.
2. Browser target: Chrome or Edge for the debug config? Codespaces typically uses the Simple Browser or forwards ports — confirm whether this plan targets local VS Code or Codespaces.
3. Dark mode: In-scope for v1, or ship light-only with tokens prepared for a future dark theme?
4. Data refresh: Manual "Reload" button or page refresh sufficient for v1? (Assumed: page refresh only.)
5. Sorting / filtering: Out of scope for v1 unless specified. Confirm.
6. Date i18n: Fixed `en-US` or browser locale? (Assumed: browser locale via `Intl.DateTimeFormat`.)
