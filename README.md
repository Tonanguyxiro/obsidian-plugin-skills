# obsidian-plugin-skills

A single Claude skill covering the query, scripting, and templating languages of widely-used
Obsidian plugins: **Dataview**, **Datacore**, **Tasks**, and **Templater**.

## Layout

```text
obsidian-plugins-skill/
  SKILL.md              # router: activation triggers + a table mapping each plugin to its file
  ADDING-A-PLUGIN.md    # token-efficient guide for adding a new plugin from a GitHub URL
  plugins/
    <plugin>.md         # per-plugin reference (dataview, datacore, tasks, templater)
    <plugin>.evals.json # optional prompt/expected-output eval pairs
    templater/          # templater's deeper per-topic docs
references/             # upstream plugin sources as git submodules (read-only reference)
```

The skill is a **router**: only `SKILL.md` carries activation triggers, and each plugin's full
reference loads on demand from `plugins/<plugin>.md` — so only the relevant plugin enters context.

## Setup

The upstream plugin sources are git submodules. After cloning:

```sh
git submodule update --init
```

## Adding a plugin

Given a plugin's GitHub URL, follow
[ADDING-A-PLUGIN.md](obsidian-plugins-skill/ADDING-A-PLUGIN.md): add it as a submodule under
`references/`, read only the API-relevant upstream files, write `plugins/<name>.md`, and register
it in the router.
