# API Sources

Use this reference when a task requires Obsidian API names, third-party plugin APIs, query fields, template functions, or integration behavior. Verify against source before writing code.

## Source Map

| API area | Source of truth | Use for |
|---|---|---|
| Obsidian core API | `https://github.com/obsidianmd/obsidian-api` and project `node_modules/obsidian/obsidian.d.ts` | `Plugin`, `App`, `TFile`, `Vault`, `Workspace`, `ItemView`, editor APIs, events, settings |
| Official sample behavior | `https://github.com/obsidianmd/obsidian-sample-plugin` | Expected plugin lifecycle, manifest fields, build output shape |
| Tasks API | Local `references/obsidian-tasks/` if present, otherwise upstream Tasks repo/docs | Tasks query blocks, task model fields, filters, sorting, grouping, custom statuses |
| Bases API/behavior | Current Obsidian docs/types and any local Bases-related reference such as `references/timeline-for-bases/` | `.base` files, properties, views, timeline/Gantt behavior |
| Dataview API | Local `references/obsidian-dataview/` if present | `dv`, DataviewJS, DQL, page/task metadata, DataArray |
| Datacore API | Local `references/obsidian-datacore/` if present | `dc`, datacorejs/datacorejsx, React-based views, Datacore data model |
| Templater API | Local `references/obsidian-templater/` if present | `tp.*`, `<% %>` commands, user functions, internal modules |

## Verification Order

1. Read official docs or the upstream `docs/` folder first.
2. Inspect type definitions or exported API files next.
3. Inspect parser/filter/model files only when field names or query semantics are unclear.
4. Use README examples as examples, not as proof of full API shape.
5. Avoid issue comments, old blog posts, or generated summaries for exact names unless confirmed by source.

## Local Reference Patterns

When this repo has a local upstream reference, prefer targeted searches over opening whole repos:

```bash
rg -n "class .*Plugin|interface .*Api|export .*API|addCommand|registerMarkdownPostProcessor|PluginSettingTab" references
rg -n "DataArray|Dataview|Datacore|TasksApi|Templater|tp\\.|dc\\.|dv\\." references
```

Useful existing paths in this repository:

- `references/obsidian-dataview/docs/docs/api/`
- `references/obsidian-dataview/src/api/`
- `references/obsidian-datacore/docs/docs/`
- `references/obsidian-datacore/src/api/`
- `references/obsidian-tasks/src/Api/`
- `references/obsidian-tasks/src/Query/`
- `references/obsidian-templater/docs/src/`
- `references/obsidian-templater/src/core/functions/`

## API Safety Rules

- Do not invent `app.plugins.plugins[...]` access patterns unless the target plugin documents a public API or the user accepts private API risk.
- Treat Bases APIs as drift-prone; verify against the user's installed Obsidian version or current docs before naming exact types.
- For Dataview versus Datacore, choose Datacore only when the user names Datacore, `dc`, `datacorejs`, `datacorejsx`, or React views. Otherwise default to Dataview for vault queries.
- For Templater, distinguish generated template code from plugin development code; `tp.*` runs in template context, not in an arbitrary plugin method.
