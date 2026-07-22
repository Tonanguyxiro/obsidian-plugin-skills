# Adding API or Template Sources

Use this reference when extending the skill with a new Obsidian API, plugin API, template, or UI library.

## Classification

Classify the new source before adding it:

| Category | Examples | Where to document |
|---|---|---|
| Core Obsidian API | `obsidianmd/obsidian-api`, official developer docs | `api-sources.md` |
| Plugin API | Dataview, Datacore, Tasks, Templater, another plugin with a public API | `api-sources.md` |
| Scaffold/template | sample plugin, Svelte template, build template | `plugin-structure.md` |
| UI/component library | Svelte components, CSS utilities, Obsidian-themed UI helpers | `plugin-structure.md` |
| Test/debug tooling | Obsidian CLI, vault sync tools, screenshot/DOM helpers | `testing.md` |

## Add a GitHub Source

1. Record the repository URL and why it is useful.
2. Identify the source-of-truth files to inspect:
   - `README.md` for overview,
   - `docs/` for user-facing behavior,
   - `src/**/api*`, exported types, or generated `.d.ts` for API names,
   - build/template files only when documenting scaffold structure.
3. Add a short entry to the right reference file.
4. Add routing text in `SKILL.md` only if the new source creates a new top-level need.
5. Avoid vendoring large docs into the skill. Link upstream and describe what to inspect.

## Add a Local Reference

If repeated work needs offline verification, add the upstream repo as a local reference or submodule under the parent repository's `references/` directory, following the local repo convention. Treat local references as read-only source material.

Suggested naming:

```text
references/<upstream-or-plugin-name>/
```

Then add a targeted search hint to `api-sources.md` or `plugin-structure.md`.

## Quality Bar

- Include exact upstream links.
- State when to use the source and when not to.
- Keep additions short and example-driven.
- If changing `SKILL.md` triggers, keep the frontmatter `description` under 1024 characters.
- Verify API names from source before documenting them as facts.
- Do not add broad README-style docs or duplicated upstream prose.
