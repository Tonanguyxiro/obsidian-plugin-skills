---
name: obsidian-plugins
description: |
  Use for any task involving the query/scripting/templating languages of major Obsidian plugins:
  Dataview, Datacore, Tasks, and Templater. This is a router skill — it points to a per-plugin
  reference file; read only the file for the plugin in question.

  Trigger on any of:
  - "dataview", DQL, dataviewjs, the `dv` API, inline fields, "table of my books", "list pages with tag X"
  - "datacore", datacorejs, datacorejsx, the `dc` API, React-based vault views
  - "obsidian tasks", Tasks query blocks, filter/sort/group by, task.due / task.status, recurrence, custom statuses
  - "templater", `tp.` functions (tp.date, tp.file, tp.frontmatter, tp.web, tp.system), `<% %>` template syntax, daily-note templates
  - "list callouts", colored/highlighted list items, marker characters like `- & text` / `- ! text`, callouts inside lists or checkboxes
  - "bases", `.base` files, Bases views/filters/properties, the timeline / Gantt view for Bases (`type: timeline`, startDate/endDate)
  - more generally: querying, filtering, listing, or displaying data from an Obsidian vault, or generating/templating note content.
---

# Obsidian Plugins Skill

A single skill covering the scripting surface of four widely-used Obsidian plugins. Each plugin's
full reference lives in its own file under [`plugins/`](plugins/) — **read only the file matching
the user's request** to keep context small.

## Routing

| If the request involves… | Read |
|---|---|
| Dataview — DQL queries, DataviewJS, the `dv` API, inline fields | [`plugins/dataview.md`](plugins/dataview.md) |
| Datacore — `dc` API, datacorejs/datacorejsx, React views | [`plugins/datacore.md`](plugins/datacore.md) |
| Tasks — query blocks, filters, task properties, custom statuses, Tasks API | [`plugins/tasks.md`](plugins/tasks.md) |
| Templater — `tp.` functions, `<% %>` syntax, note templates | [`plugins/templater.md`](plugins/templater.md) |
| List Callouts — colored list items via marker chars (`- & text`) | [`plugins/list-callouts.md`](plugins/list-callouts.md) |
| Bases — `.base` files, views/filters/properties, the timeline/Gantt view | [`plugins/bases.md`](plugins/bases.md) |

Disambiguation: Dataview and Datacore are both query engines and share vocabulary ("query my
vault", "table", "list"). Pick Datacore only when the user names Datacore / `dc` / datacorejsx or
asks for React-based views; otherwise default to Dataview. Templater is the only one for generating
or templating note content (`tp.`), and Tasks is the only one for the `- [ ]` task query language.

## Per-plugin extras

- Each `plugins/<name>.md` begins with a pointer to its upstream source under
  [`../references/`](../references/) — read that source to verify any API name before documenting it.
- Some plugins ship eval pairs alongside their doc: `plugins/dataview.evals.json`,
  `plugins/datacore.evals.json` (prompt → expected query/template shape).
- Templater has deeper per-topic docs under [`plugins/templater/`](plugins/templater/).

## Adding a new plugin

See [`ADDING-A-PLUGIN.md`](ADDING-A-PLUGIN.md) for a token-efficient workflow: which upstream files
to read (and which to skip), how to write the per-plugin doc, and how to register it here.
