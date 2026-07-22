# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this repository is

A single authored **Claude skill** (`obsidian-plugins-skill`) covering the query/scripting/templating
languages of widely-used Obsidian plugins: Dataview, Datacore, Tasks, and Templater. There is no
application to build, lint, or test — the deliverables are Markdown files that teach an agent how to
write queries/templates for each plugin. The upstream plugin sources are bundled as git submodules
under `references/` so skill content can be authored and verified against the real implementation.

## Repository layout

```text
obsidian-plugins-skill/
  SKILL.md              # router: frontmatter triggers + a table mapping each plugin to its file
  ADDING-A-PLUGIN.md    # token-efficient guide for adding a new plugin
  plugins/
    <plugin>.md         # per-plugin reference (dataview, datacore, tasks, templater)
    <plugin>.evals.json # optional eval pairs (dataview, datacore have them)
    templater/          # templater's deeper per-topic docs (syntax, commands/, user-functions/)
references/             # upstream plugin sources as git submodules (read-only reference)
  obsidian-dataview/  obsidian-tasks/  obsidian-templater/  obsidian-datacore/
```

Submodule URLs/paths are in [.gitmodules](.gitmodules).

## How the skill is structured

It is a **router**: [SKILL.md](obsidian-plugins-skill/SKILL.md) holds the only YAML frontmatter
(`name` + a `description` merging every plugin's trigger keywords) and a routing table. The heavy
content lives in `plugins/<plugin>.md`, loaded on demand — so only the relevant plugin's reference
enters context. Per-plugin files have **no frontmatter**; they open with a `# <Plugin>` heading and
a pointer to their upstream source under `references/`.

The per-plugin body style is dense and example-first: lead with tables and short runnable snippets,
and end with a "common mistakes to avoid" section calling out API confusions (e.g. `task.done` not
`task.completion`, `.groupBy()` not `dv.groupBy()`). See
[plugins/dataview.md](obsidian-plugins-skill/plugins/dataview.md).

## Working with submodules

The submodules under `references/` are the upstream plugin repos and the source of truth when
authoring skill content. After cloning, initialize them with `git submodule update --init`. Treat
them as read-only — skill changes go in `obsidian-plugins-skill/` only.

## When authoring or editing the skill

- Verify every API name, field, and function against the submodule source before documenting it —
  the "common mistakes" sections exist because these plugins have non-obvious gotchas.
- Keep the example-first, table-heavy style consistent across the per-plugin files.
- When adding a new plugin, follow
  [ADDING-A-PLUGIN.md](obsidian-plugins-skill/ADDING-A-PLUGIN.md): add the source as a submodule
  under `references/`, read only the API-relevant upstream files, write `plugins/<name>.md`, and
  register it in the router table + `description`.
- Eval files (`plugins/<name>.evals.json`) hold prompt/expected-output pairs; add or update them
  when adding a capability so coverage stays checkable.
