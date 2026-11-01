# Linear

A Linear panel on the right of the Herdr window, beside Files and Changes. It lists
the team's issues by status and project with sub-issues folded under their parent,
opens an issue's details (description, cycle, project, parent, sub-issues), changes
status, priority, assignee and the agent delegated to it, and starts an agent on an
issue in a new tab with a prompt the person writes. It reads and writes Linear
through the person's own `linear` CLI login; no API key is stored in Herdr.

## Installing

Copying `mods/linear/` is not enough: `mod.json` is `"place": "none"`, so the panel
shows only once a layout puts it beside Files and Changes. Shipped layouts and mods
are read-only, so after copying the folder:

1. `herdr-app do layout.clone '{"name":"main","as":"my-main"}'` (switches to it) and
   `herdr-app do mod.clone '{"name":"tabs","as":"my-tabs"}'`. If the person already
   has their own layout or tab bar clone, edit those instead.
2. In the layout, add `{ "id": "linear", "type": "block", "content": { "mod": "linear" } }`
   as a child of the `panel-body` stack, and point the `tabs` block at `my-tabs`.
3. In `my-tabs/main.js`, append `"linear"` to `enabled` (the panels from
   `shared("workspace.panels")`), and add
   `.child(panelButton("linear", "Linear", "mods/linear/linear.svg"))` after the
   Changes button.
4. `herdr-app check` both mods, then click Linear in the tab bar: the welcome checks the
   `linear` CLI (`brew install schpet/tap/linear`), the login and write access.

## What you see

One block, `mods/linear`, filling the right panel (368px wide in `main`), drawn as a
card: `background` fill, 1px `border`, radius 10. Every color is a `theme()` token
except Linear's own state colors, which come from the API.

- **Header** (42px, bottom border): the page title, then icon buttons on the right.
  On the list: "Linear" and the shown issue count, then ? (welcome), ⚙ (settings) and
  ⟳ (refresh; a spinner while loading). On an issue: ‹ back, the issue ID, then ▶
  (green `ansi2`), copy and ↗ (open in Linear).
- **Filter bar** (116px): a search box (30px, `raised` fill, clear × when not empty);
  a full-width segmented switch Anyone / Me / Unassigned; two dropdown buttons side by
  side, Status (the selected statuses' icons) and Project ("All projects", one name,
  or "N projects").
- **List**: status sections in workflow order (started, unstarted, backlog,
  completed, canceled). Each is a 30px band (`raised`, radius 7) with a fold chevron,
  a status icon in the state's color, the name and a count; under it, project
  headings (11px, `dim`, with fold chevron and count), then 32px issue rows: fold
  chevron for parents, priority glyph, ID (one fixed column width), status icon,
  title, "in ENG-123" when a filter hides the parent, a sub-issue count (done/total),
  then agent (rounded square) and assignee (circle) avatars. Sub-issues nest 20px in.
- **Issue details**: "Sub-issue of" link to the parent, the title (16px semibold),
  property rows (Status, Priority, Assignee with "with <agent>", Cycle with dates and
  a Current tag, Project with milestone), the description as markdown, then
  sub-issues: "N of M done", one tag per status with its count, and the rows.
- **Settings**: one card per play action (`raised` fill, radius 10): ▶, the editable
  name, the agent button (icon, name, ▾) and ×; a framed prompt box (`background`
  fill, `input` border, `accent` while focused) where variables are accent chips;
  insert buttons for `$issueId $title $url $branch $description`; a "Sends, for
  ENG-123" preview. Then Add action, and a Welcome row with Show again.
- **Welcome** (first run): a title, five feature lines, and three setup steps (Install
  the Linear CLI, Log in to Linear, Allow editing) each marked ✓, a spinner, or a
  numbered circle with the fix.

Status icons: backlog a dashed ring, todo a ring, started a ring with a half fill,
done a filled disc with a check, canceled a filled disc with an ×. Priority: urgent an
orange (`blocked`) square with !, high/medium/low three bars with 3/2/1 lit in
`foreground`, none three `dim` dots.

## How it behaves

- Agent statuses don't apply; the panel is about Linear issues.
- The list refetches every 60s while the panel shows, on ⟳, and on every filter
  change (a newer fetch wins over one still running). It loads up to 250 issues and
  says so when there are more.
- Status, Project and the segmented switch filter in Linear's query; search filters
  on screen by ID and title and opens folded sections and parents so matches show.
- Tapping a status band or project heading folds it; parents' chevrons fold their
  sub-issues. Folds persist.
- Hovering a row shows a copy button in place of the avatars. While a dropdown is
  open, rows don't hover and taps stop at the dropdown.
- Tapping an issue, a sub-issue or a parent pushes its details; ‹ pops.
- Status, Priority and Assignee open an inline picker. Assignee lists People
  (assignee) and Agents (Linear's delegate); both can be set, as in Linear. A change
  saves through `issueUpdate`, then the details and list refresh.
- ▶ with one action runs it; with several it opens a menu of actions with their
  agents. Running opens `tab.new { name: <ID>, argv: agent.run, task: <prompt> }`
  and toasts.
- The agent button opens this mod as a popup (props `{ pickAgent }`) listing the
  installed agents (icon, name, command); ↑/↓/Enter/Esc work.
- Insert buttons put a variable chip at the cursor; a variable typed by hand becomes a
  chip when the box loses focus.
- If the CLI's key can only read, editing is locked and the details show "Editing is
  off" with Create key ↗, Log in and Check again.
- The welcome shows once; its checks re-run every 5s while a step is unfinished, so a
  login in the split pane turns ✓ by itself.

## Built from

- Files: `mods/linear/main.js`, `mod.json` (`{ "place": "none", "title": "Linear" }`)
  and its SVGs (status icons, play, copy, help, refresh, sub-issues, the Linear
  glyph). Icons for chevrons, search, x, check, plus, settings and the spinner come
  from the shipped `mods/workspace-chrome`.
- Data: `exec("linear", ["api", "--no-input", "--variables-json", …, <GraphQL>])` for
  every read and the `issueUpdate` mutation. On failure `linear api` prints the
  error body to stderr. `linear --version` for the setup check.
- Write check: `issueUpdate` on the all-zero UUID. Linear checks the key's scope
  before looking the issue up, so "Invalid scope" means read-only and "Entity not
  found" means it may write; nothing changes either way.
- herdr: `exec`, `run` (`tab.new`, `terminal.open`), `theme`, `layout`, `notify`,
  `shared`/`setShared` (`linear.prefs`: welcomed, filter, collapsed, actions),
  `knownAgents`, `props`, `openOverlay`, `closeOverlay`. gpui-base: `TextView`
  (description), `InputState`/`Input` (search, action names),
  `TextareaState`/`Textarea` with `.token()` (prompts).
- Placement: a third child of `main`'s `panel-body` stack,
  `{ "id": "linear", "type": "block", "content": { "mod": "linear" } }`, and a tab-bar
  button that runs `layout.select { stack: "panel-body", child: "linear" }` plus
  `layout.show { id: "panel" }`. Both need clones: `layout.clone` main and
  `mod.clone` tabs, then point the layout's tabs block at the clone and add
  `"linear"` to its enabled panels and a `panelButton("linear", "Linear",
  "mods/linear/linear.svg")`.
- Programs: the `linear` CLI (`brew install schpet/tap/linear`), `open`, `pbcopy`.

## Watch out

- Text selection in `TextView` paints the theme's `selection` color on top of the
  text; with an opaque selection the text vanishes. A layout `colors.selection` with
  alpha (e.g. `#3944548c`) fixes it.
- An element with `overflow_y_scrollbar()` loses absolute placement: put the scroll
  inside the positioned box.
- GPUI gives a click and hover to everything under the pointer; mods can't occlude.
  Stop propagation on dropdowns and their backdrop, and skip hover styles behind them.
- `localStorage` is per mod copy and starts over when the layout rebuilds; keep
  preferences in `shared`.
- `linear auth login` stores a new key beside the old one under the organization's
  slug without making it default; the Log in button runs `linear auth default <org>`
  after it.
- Text states (`InputState`, `TextareaState`) are created only in `init` or event
  handlers. `TextareaState.replace` inserts at the cursor.
- macOS only as written (`open`, `pbcopy`, the Homebrew tap).

## Done when

- `herdr-app check mods/linear` passes.
- The Linear button opens the panel; the list shows status sections with project
  headings, and ENG-style parents fold their sub-issues.
- Status and Project dropdowns open under their buttons and change the list; tapping
  a dropdown row doesn't open the issue beneath.
- Tapping an issue shows its properties, description and sub-issues; a sub-issue
  shows "Sub-issue of" its parent.
- With a write key, changing priority saves and the list updates; with a read-only
  key the details say "Editing is off".
- In settings, inserting `$issueId` puts a chip at the cursor and the preview fills
  it in; ▶ on an issue opens a tab running the chosen agent with that prompt.
