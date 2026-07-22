# Adding a new plugin to this skill

Goal: add coverage for another Obsidian plugin without reading its entire source. The expensive
mistake is reading the whole plugin repo. Don't — go straight to the few files that define the
user-facing API, verify names against them, and write an example-first doc.

## 1. Add the upstream source as a submodule under `references/`

```
git submodule add <repo-url> references/<plugin-name>
```

This is the source of truth for verifying API names, fields, and behavior. Treat it as read-only.

## 2. Read only the API-relevant files (skip the rest)

Most of a plugin repo (build config, tests, settings UI, styles, i18n) is irrelevant to authoring
the skill. In priority order, read:

1. **The upstream `docs/` directory if it exists** — already written for users; it is the cheapest
   and highest-signal source. Read this first.
2. **The file defining the public API object** — the global the user scripts against
   (`dv`, `dc`, `tp`, the Tasks API).
3. **The functions / filters module** — the enumerated built-in functions or query filters.
4. **The data model** — the fields available on a page/task/file.

Skip: `main.ts`/plugin bootstrap, settings, UI components, tests, build tooling.

### Where the API lives in the plugins already covered

Use these as a template for what to look for in a new plugin (paths are inside each
`references/<plugin>/`):

| Plugin | API object | Functions / filters | Data model | Docs |
|---|---|---|---|---|
| Dataview | `src/api/plugin-api.ts` (`dv`), `src/api/data-array.ts` | `src/query/parse.ts`, `src/expression/parse.ts` | implicit page/task fields | `docs/docs/queries/`, `docs/docs/api/` |
| Datacore | `src/api/api.ts` (`dc`), `src/api/local-api.tsx` | `src/expression/functions.ts`, `src/expression/parser.ts` | `src/index/datacore.ts` | `docs/docs/expressions/`, `docs/docs/data/` |
| Tasks | `src/Api/TasksApiV1.ts` | `src/Query/Filter/`, `src/Query/FilterParser.ts` | `src/Task/Task.ts` | `docs/Queries/`, `docs/Reference/` |
| Templater | `src/core/functions/FunctionsGenerator.ts` (`tp`) | `src/core/functions/internal_functions/` | — | `docs/src/syntax.md`, `docs/src/internal-functions/` |

General rule: **`docs/` first → public-API file → functions module.** That's usually 3-5 files, not
the whole repo.

## 3. Write `plugins/<name>/SKILL.md`

Match the existing style (see [`plugins/dataview/SKILL.md`](plugins/dataview/SKILL.md)):

- Start with `# <Plugin>` and a one-line pointer: `> Upstream source: ../../../references/<name>/`.
- **No YAML frontmatter** — activation triggers belong in the router, not the per-plugin file.
- Lead with tables and short runnable snippets. Example-first.
- End with a **"common mistakes to avoid"** section — the non-obvious API gotchas you found while
  verifying against source. This is the highest-value part.
- Verify every API name, field, and function against the submodule before writing it down.

## 4. Register it in the router

Add a row to the routing table in [`SKILL.md`](SKILL.md) and extend the `description` frontmatter
with the new plugin's trigger keywords. Add a disambiguation note if it overlaps an existing plugin.

## 5. (Optional) Add evals

If you want coverage checks, add `plugins/<name>/evals.json` with prompt → expected-shape pairs,
following [`plugins/dataview/evals.json`](plugins/dataview/evals.json).
