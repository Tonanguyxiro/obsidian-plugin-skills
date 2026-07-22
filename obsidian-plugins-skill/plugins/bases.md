# Obsidian Bases

> Upstream source for the timeline view: [`references/timeline-for-bases/`](../../references/timeline-for-bases/). Activation triggers live in [`../SKILL.md`](../SKILL.md).

**Bases** is Obsidian's built-in database plugin. A `.base` file defines one or more **views** over a
set of notes selected by **filters**, showing their frontmatter **properties** as columns. Bases
ships table and card views; third-party plugins register additional view types.

A `.base` file is YAML:

```yaml
filters:
  and:
    - file.hasTag("project")
    - "!status.isEmpty()"
views:
  - type: table
    name: All projects
    order:
      - file.name
      - status
      - due
    sort:
      - property: due
        direction: ASC
```

- `filters` — which notes are in the base (`and` / `or` / `not` groups of expressions). Expressions
  use note properties (`status`, `due`), `file.*` helpers (`file.folder`, `file.hasTag(...)`), and
  methods like `.isEmpty()`.
- `views` — one or more views. Each has a `type`, a `name`, optional per-view `filters`, `sort`, and
  `order` (the visible properties, in column order). The first ordered property is treated as the
  primary column.
- Properties are referenced bare in expressions (`status`) but as `note.<prop>` in some view config
  keys (see the timeline view below).

## Timeline view (Timeline for Bases plugin)

A community plugin (install via BRAT — not in the core plugin list) that registers a **Gantt-style
timeline** view type, `type: timeline`. Point a base at notes that have start/end date frontmatter
and they render as bars on a horizontal timeline by day, week, month, quarter, or year.

### Minimal base with a timeline view

```yaml
filters:
  and:
    - "!start.isEmpty()"
views:
  - type: timeline
    name: Project timeline
    startDate: note.start      # frontmatter prop for the bar's start
    endDate: note.end          # frontmatter prop for the bar's end
    order:
      - file.name              # first ordered prop = primary frozen column
      - status
    sort:
      - property: start
        direction: ASC
```

A note only needs date frontmatter to appear:

```yaml
---
start: 2026-07-01
end: 2026-07-10
status: open
---
```

A note with only `start` (no `end`) renders as a single-day **point** marker.

### Timeline view config keys

Set in the Bases **Configure View** panel (standard Bases keys):

| Key | Meaning |
|-----|---------|
| `startDate` | frontmatter property for the bar start — written as `note.<prop>` |
| `endDate` | frontmatter property for the bar end — written as `note.<prop>` |
| `order` | visible properties; first becomes the primary frozen column, rest become sticky property columns |
| `sort` / `filters` | native Bases sorting and filtering; reflected by the timeline |

Set via the timeline's own gear/config panel (persisted into the view's YAML by the plugin):

| Key | Type | Meaning |
|-----|------|---------|
| `timeScale` | `day`\|`week`\|`month`\|`quarter`\|`year` | time scale (default `week`) |
| `zoom` | number | horizontal zoom factor |
| `colorBy` | `note.<prop>` | property driving bar fill color |
| `colorMap` | string | `Value=color;...` mapping (e.g. `High=#b02a37;Low=var(--color-green)`) |
| `borderBy` | `note.<prop>` | property driving the bar border color |
| `borderColorMap` | string | `Value=color;...` mapping for borders |
| `borderWidth` | number | border thickness in px |

Full example (the plugin's "Create Sample Base" output):

```yaml
views:
  - type: timeline
    name: Family Vacation Planning
    filters:
      and:
        - file.folder == "Timeline Sample"
    sort:
      - property: start
        direction: ASC
    startDate: note.start
    endDate: note.end
    order: [file.name, priority, assigned, status]
    colorBy: note.priority
    colorMap: 'High=#b02a37;Medium=#c27c0e;Low=var(--color-green)'
    borderBy: note.assigned
    borderColorMap: 'Patrick=var(--color-blue);Maya=var(--color-pink)'
    borderWidth: 2
    timeScale: day
    zoom: 1
```

Other capabilities (no YAML needed): drag a bar to move dates / drag edges to resize (writes back to
frontmatter), draw on an empty row to create dates, Shift+click multi-select, collapsible groups with
drag-between-groups, inline property-cell editing, undo/redo (50 steps), "Color by" palette, export PNG.

## Common mistakes to avoid

- ❌ Bare property name in timeline date config — use `startDate: note.start`, **not** `startDate: start`.
  (Filter expressions, by contrast, use the bare name: `"!start.isEmpty()"`.)
- ❌ Expecting bars without a date — a note needs at least a `start` value to appear; `start`-only
  renders as a single-day point, not an error.
- ❌ Treating `timeline` as a core Bases view — it requires the Timeline for Bases plugin (installed
  via BRAT); core Bases only ships table/card views.
- ❌ Wrong `colorMap` syntax — it's a single string of `Value=color` pairs joined by `;`, not YAML
  list/map nesting.
