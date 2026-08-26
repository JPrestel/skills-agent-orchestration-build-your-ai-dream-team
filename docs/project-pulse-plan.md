# Project Pulse Dashboard — Implementation Plan

## 1. Summary

Project Pulse is a small static dashboard (no build step, no server framework) that helps
contributors see, at a glance: which projects are active, who owns each project, current
status, recent activity, priority/risk, and a short summary — presented as a polished set
of project cards with status badges and clear spacing.

The deliverable is four files:
- `app/index.html` — semantic HTML structure + rendering logic (vanilla JS, no build tools)
- `app/styles.css` — visual design: cards, badges, spacing, responsive layout, accessibility
- `app/project-data.json` — static data source with a top-level `projects` array
- `.vscode/launch.json` — a "Run Project Pulse Dashboard" launch configuration that serves
  `app/` and opens `index.html` directly (not a directory listing)

The repo currently has an **empty `app/` directory** and no `.vscode/launch.json` — both are
net-new. `.vscode/tasks.json` already exists (used only for the Copilot CLI exercise terminal
on `folderOpen`) and has no port/server conflicts with the new launch config, so it can be
left untouched.

Two agents own the four files with **no overlapping file scope**:
- **Coder** owns `app/index.html`, `app/project-data.json`, `.vscode/launch.json` (structure,
  data, JS wiring, run/launch config).
- **Designer** owns `app/styles.css` (visual design, CSS hooks, accessibility, responsive
  layout), and provides markup/class-hook guidance that Coder incorporates into
  `app/index.html`.

Because HTML and CSS are different files but tightly coupled (class names/hooks must match),
the plan sequences a short "structure-first" step before parallel styling/data work, then a
final integration/validation pass.

## 2. Ordered implementation steps

1. **Define data shape and CSS hook contract (planning-level, already specified by brief)**
   - Confirm `project-data.json` schema: top-level `projects` array; each item has `name`,
     `owner`, `status`, `recentActivity`, `priority`. Optionally allow a short `summary`
     field to satisfy "short contributor-friendly summary" from the brief (see Open
     Questions — recommend adding this field explicitly since the brief calls for a
     "short contributor-friendly summary" but the required-fields list in the task does not
     include it).
   - Confirm deterministic CSS hooks: `.dashboard` (page/container), `.project-card` (each
     project), plus card sub-elements such as `.project-card__header`, `.project-card__name`,
     `.project-card__owner`, `.status-badge` (with modifier classes per status, e.g.
     `.status-badge--active`, `.status-badge--blocked`), `.priority` / `.priority--high`,
     `.project-card__activity`, `.project-card__summary`.
   - Output: a short written contract (can live as comments at top of `index.html` and
     `styles.css`, or in this plan) so Coder and Designer build against the same names.

2. **Coder: scaffold `app/index.html` structure + author `app/project-data.json`**
   - Build minimal semantic HTML: `<main class="dashboard">`, a heading, and a container
     where project cards render, using the agreed CSS hooks as empty/placeholder classes.
   - Decide and implement the data-loading approach (see Open Questions): either (a) fetch
     `project-data.json` via `fetch()` in an inline `<script>`/`app/app.js` and render cards
     client-side, or (b) statically hard-code sample cards in HTML mirroring the JSON (not
     recommended — breaks single-source-of-truth and the brief's data-file intent).
     **Recommended:** fetch-and-render JS approach so `project-data.json` is the real source
     of truth.
   - Author `app/project-data.json` with realistic sample projects (4–6 entries) covering a
     range of statuses/priorities to give Designer real content to style against.
   - This step must land the HTML skeleton and class-hook names **before** Designer begins
     hook-specific CSS, so hooks are not renamed mid-stream.

3. **Designer: build `app/styles.css` against the agreed hooks**
   - Style `.dashboard` layout (page padding, max-width, responsive grid/flex for cards).
   - Style `.project-card` (rounded corners, shadow, spacing/padding, hover state).
   - Style `.status-badge` variants with color + text (not color alone) for accessibility.
   - Style priority treatment (`.priority--high/medium/low`) with icon/text distinction, not
     color-only.
   - Ensure readable typography, spacing rhythm, and responsive breakpoints (mobile stacking
     of cards).
   - Verify contrast ratios (WCAG AA) for badge/text/background combinations.

4. **Coder: create `.vscode/launch.json`**
   - Add a "Run Project Pulse Dashboard" configuration that serves the `app/` directory as
     the working directory/root and opens `index.html` (not a folder listing).
   - Use a strict, comment-free JSON file (VS Code launch.json normally allows comments, but
     keep strict JSON per Coder agent rules for determinism/tooling compatibility).
   - Set `cwd` to `${workspaceFolder}/app`.
   - Choose a concrete mechanism (see Open Questions — depends on what's available in this
     environment/extensions):
     - If the "Live Preview" / "Live Server" style launch is available via a debug adapter,
       configure it to serve `${workspaceFolder}/app` and open `/index.html`.
     - If using the built-in `chrome`/`msedge` debug type with a `file://` URL, point
       directly at `${workspaceFolder}/app/index.html` (simplest, no server dependency,
       avoids any directory-listing risk entirely).
     - If a lightweight static-server task is required (e.g. `npx serve`, `python -m
       http.server`), pair a `.vscode/tasks.json` prelaunch task with the launch config,
       and set the launch `url`/`webRoot` to open `index.html` explicitly rather than `/`.
   - Do not modify the existing task in `.vscode/tasks.json`; add only what's needed (a new
     task, if the chosen approach requires one) without touching the existing entry.
   - Verify the config name is exactly `Run Project Pulse Dashboard` to match the brief.

5. **Integration pass (Coder, then Designer spot-check)**
   - Coder confirms `index.html`'s class names exactly match what `styles.css` targets (no
     drift from step 2 naming).
   - Coder confirms `project-data.json` parses and every field referenced by the render
     script exists.
   - Coder launches via the new launch config and confirms `index.html` renders (not a
     directory listing), cards populate from JSON, and no console errors appear.
   - Designer does a visual pass against the brief: cards visible, badges legible, spacing
     readable, responsive at narrow width.

6. **Final validation (see Section 7)** — run through all validation checks below before
   reporting completion.

## 3. File assignments

| File | Owner | Responsibilities |
|---|---|---|
| `app/index.html` | **Coder** | Semantic HTML structure; container `<main class="dashboard">`; card-rendering logic (inline `<script>` or `app/app.js` if a separate JS file is preferred — confirm with Orchestrator before adding a new file not in the brief's file list); fetch/parse `app/project-data.json`; apply the agreed CSS hook class names to rendered markup; basic empty-state and malformed-data handling in JS. |
| `app/project-data.json` | **Coder** | Author the `projects` array and all sample project entries; ensure valid JSON; ensure every required field (`name`, `owner`, `status`, `recentActivity`, `priority`) is present per project; decide on and document any optional fields (e.g. `summary`) in coordination with the plan. |
| `app/styles.css` | **Designer** | All visual design: layout of `.dashboard`, card design `.project-card`, status badge styling `.status-badge` (+ status modifiers), priority/risk visual treatment, typography, spacing scale, responsive breakpoints, hover/focus states, accessibility (contrast, focus rings, not-color-alone indicators). Designer may also propose/confirm exact class-hook names used by Coder in `index.html`, but does not edit `index.html` directly — hands the hook contract to Coder. |
| `.vscode/launch.json` | **Coder** | Create the new file; define the `Run Project Pulse Dashboard` configuration; set working directory to `app/`; ensure it opens `index.html` and not a directory listing; keep strict, comment-free JSON; avoid touching unrelated existing VS Code config (`.vscode/tasks.json`) unless a prelaunch task is strictly required, in which case add a new task without altering the existing "Open Copilot CLI exercise terminal" task. |

Note: Designer does not touch `app/index.html`, `app/project-data.json`, or
`.vscode/launch.json`. Coder does not touch `app/styles.css`. This keeps the two agents'
file scopes fully non-overlapping.

## 4. Dependencies between steps

- **HTML class-hook names must be agreed/fixed before Designer finalizes CSS** — if Coder
  renames a hook after Designer has written CSS against it, styles will silently stop
  applying. Mitigate by fixing the hook contract in step 1 and having Coder scaffold the
  container/card markup (even with placeholder/sample content) before Designer starts.
- **`project-data.json` field shape must be fixed before the render script in `index.html`
  is finalized** — the JS that maps JSON fields to DOM/class hooks depends on exact key
  names (`name`, `owner`, `status`, `recentActivity`, `priority`, optional `summary`).
- **The launch config depends on `app/index.html` existing** at the expected path
  (`app/index.html`) — build/launch config after or alongside HTML scaffolding, but validate
  only after `index.html` exists and renders something.
- **Final integration/validation depends on all four files being present**: launching before
  `styles.css` exists will still "work" (unstyled page), but the visual/brief validation
  requires Designer's CSS to be in place.
- `.vscode/tasks.json` is a pre-existing, unrelated file — no dependency, but Coder must read
  it first (already done in this research) to avoid task-label collisions if a prelaunch
  task is added.

## 5. Parallel work decisions

**Can run in parallel:**
- Once the HTML skeleton + CSS hook contract exist (end of step 2's markup scaffolding),
  **Designer writing `app/styles.css`** and **Coder writing `.vscode/launch.json`** can
  proceed in parallel — these touch entirely different files with no data dependency between
  them.
- Authoring `app/project-data.json` (Coder) can happen in parallel with Designer's CSS work,
  since CSS styles class hooks/structure, not the literal data values (Designer only needs
  representative sample content, which Coder can provide early/in parallel).

**Must run sequentially:**
- Step 1 (hook/schema agreement) → Step 2 (HTML scaffold with those hooks) must precede
  Designer starting CSS in earnest, to avoid rework from renamed classes.
- Step 2 (HTML skeleton with render script) should substantially precede or be developed
  alongside Step "author JSON data," since Coder is doing both and the render script's field
  usage and the JSON's field names must match — but since both are Coder-owned, this is an
  internal sequencing detail for Coder, not a cross-agent handoff.
- Final integration/validation (step 5–6) must happen **after** all of: HTML, CSS, JSON, and
  launch.json are complete — this is a hard gate, not parallelizable.

**Practical two-phase split for the Orchestrator:**
- Phase A (sequential, Coder only): scaffold `index.html` structure + class hooks, author
  `project-data.json`. Produces the hook contract Designer needs.
- Phase B (parallel): Designer builds `app/styles.css`; Coder builds `.vscode/launch.json`.
- Phase C (sequential, gated on A+B): integration pass — launch, visually inspect, validate
  against brief and JSON schema, fix any hook mismatches.

## 6. Edge cases to handle

- **Empty `projects` array** — render script must show a clear "no projects" empty state
  in the `.dashboard` container instead of a blank page or JS error.
- **Missing/undefined field on a project** (e.g. no `recentActivity`) — render script should
  substitute a fallback ("No recent activity") rather than printing `undefined` or throwing.
- **Unexpected `status` or `priority` values** not covered by CSS modifier classes (e.g. a
  typo like `"Actve"`) — CSS should have a default/fallback badge style so unknown values
  don't render unstyled/invisible; consider normalizing casing in the render script.
- **Long text overflow** — long `name`, `owner`, or `recentActivity`/summary strings must wrap
  or truncate gracefully in the card (CSS `overflow-wrap`/`text-overflow`) rather than
  breaking the card layout.
- **Many projects (10+)** — grid/flex layout in `.dashboard` must remain responsive and not
  overflow horizontally; consider CSS grid with `auto-fill`/`minmax`.
- **Malformed JSON** (trailing comma, syntax error) — `fetch()`/`JSON.parse` failure should be
  caught and surfaced as a visible on-page error state, not a silent blank page.
- **`project-data.json` fetched via `file://` protocol** — some browsers block `fetch()` on
  `file://` URLs due to CORS; this directly affects the launch.json choice (see Open
  Questions) — if using a `chrome`/`file://` launch approach, `fetch()` of a local JSON file
  may fail silently. This must be resolved before finalizing the launch mechanism.
- **Launch config path resolution** — `cwd`/`webRoot`/`url` must resolve to `app/index.html`
  specifically; verify no trailing-slash-only URL that could resolve to a directory listing
  if any server-based approach is used.
- **Accessibility**: status/priority must not be conveyed by color alone (add text/icon);
  ensure sufficient color contrast for badges; ensure focus states are visible if any
  interactive elements exist.
- **Data/HTML drift**: if Coder adds an optional `summary` field to JSON but the brief's
  literal required-field list doesn't mention it, ensure the render script tolerates its
  absence (optional field, not required) so brief-compliant minimal data still works.

## 7. Validation expectations

- **JSON validity**: `app/project-data.json` parses with `JSON.parse` (or `python -m
  json.tool` / `jq` locally) with no errors; top-level key is `projects` and is an array;
  every entry has non-empty `name`, `owner`, `status`, `recentActivity`, `priority`.
- **Launch behavior**: Launching "Run Project Pulse Dashboard" from VS Code opens the actual
  Project Pulse UI (rendered cards from `index.html`), not a bare directory/file listing and
  not a 404. Confirm the browser/tab URL or webview resolves to `index.html` specifically.
- **Rendering correctness**: Number of rendered `.project-card` elements equals number of
  entries in `project-data.json`'s `projects` array; each card shows name, owner, status
  badge, recent activity, and priority indicator.
- **Hook contract compliance**: `.dashboard` and `.project-card` classes are present in the
  rendered DOM and are the elements actually styled by `styles.css` (spot check via browser
  dev tools that the CSS rules are applied, not overridden/unused).
- **Accessibility spot checks**: run an automated check (e.g. axe/browser accessibility
  panel) for contrast and landmark/heading structure; confirm status/priority are
  distinguishable in a color-blind simulation or grayscale screenshot.
- **Responsive check**: resize/narrow the viewport (or use device toolbar) and confirm cards
  reflow to a single column without horizontal scroll or overlap.
- **Empty/edge-case data check**: temporarily test with an empty `projects` array and with a
  project missing a non-required/optional field to confirm graceful fallback (then restore
  real sample data).
- **No unrelated regressions**: confirm `.vscode/tasks.json`'s existing "Open Copilot CLI
  exercise terminal" task still runs on folder open and was not altered.
- **Scope check**: confirm Designer only modified `app/styles.css` (and did not edit
  `index.html`/`project-data.json`/`launch.json`), and Coder only modified `index.html`,
  `project-data.json`, and `.vscode/launch.json` (and did not edit `styles.css`).

## 8. Open questions

1. **Data-loading mechanism**: Should `index.html` use `fetch()` to load
   `project-data.json` at runtime (recommended, keeps JSON as single source of truth), or
   should Coder inline the data into a `<script>` variable to sidestep `file://` CORS issues
   with `fetch`? This choice affects both the render script and the launch.json mechanism
   (a `file://` launch is simplest but may block `fetch()` of local JSON in some browsers;
   a served-via-`http://` launch avoids that but adds a server dependency). Recommend the
   Orchestrator have Coder confirm which VS Code extensions/tools are available in this
   environment (e.g., Live Preview, Live Server, `python3`, `npx serve`) before finalizing
   `.vscode/launch.json`.
2. **Separate JS file vs. inline script**: The brief lists exactly three `app/` files
   (`index.html`, `styles.css`, `project-data.json`). If render logic needs to be
   non-trivial, should it live inline in `index.html` (keeps to the brief's exact file list)
   or should Coder request Orchestrator approval to add `app/app.js`? Recommend inline
   `<script>` in `index.html` unless the Orchestrator explicitly approves an additional file.
3. **Optional `summary` field**: The brief's prose asks for "a short contributor-friendly
   summary" per project, but the explicit required-field list (`name`, `owner`, `status`,
   `recentActivity`, `priority`) omits it. Recommend adding an optional `summary` field to
   the JSON schema and rendering it if present, so both the prose intent and the literal
   field list are satisfied without breaking minimal/required-field compliance — needs
   Orchestrator/Mona sign-off.
4. **Launch config type**: Is there a preferred/pre-installed VS Code extension in this
   Codespace for serving static sites (e.g., Live Preview/Live Server), or should the launch
   config rely only on built-in browser debug types (`chrome`/`msedge`) pointed at a `file://`
   path? This determines the exact shape of `.vscode/launch.json` and whether a companion
   task in `.vscode/tasks.json` is needed.
5. **Status/priority vocabulary**: The brief doesn't define a fixed enum for `status` (e.g.
   Active/At Risk/Blocked/Complete) or `priority` (e.g. High/Medium/Low). Designer's CSS
   modifier classes and Coder's sample data need an agreed, finite vocabulary to keep badge
   styling deterministic — recommend Orchestrator confirm or delegate this small decision to
   Coder+Designer jointly during Phase A/B handoff.
