---
name: obsidian-plugin-dev-skill
description: Build, modify, debug, and validate Obsidian community plugins. Use when working on Obsidian plugin development tasks involving TypeScript plugin APIs, manifest/build structure, sample-plugin scaffolds, Svelte or scoped-CSS plugin UI, Dataview/Datacore/Tasks/Templater integrations, Bases-related behavior, test-vault workflows, plugin reloads, Obsidian CLI diagnostics, DOM/screenshot checks, or syncing a built plugin into a vault for validation.
---

# Obsidian Plugin Dev Skill

Use this skill for Obsidian plugin development work, not for writing ordinary notes. Start with the development loop below, then load only the reference file needed for the current step.

## Development Workflow

1. **Classify the plugin shape.** Identify whether the work is command-only, settings-heavy, editor extension, markdown postprocessor, view/sidebar, Bases integration, Dataview/Datacore/Tasks/Templater integration, or complex interactive UI.
2. **Choose the smallest scaffold.** Default to the official sample plugin. Use Svelte/scoped CSS only for substantial UI. Use Tailwind/UnoCSS templates only when the project already wants utility CSS.
3. **Verify APIs before coding.** Check upstream type definitions or local reference submodules before naming Obsidian APIs, plugin APIs, fields, filters, lifecycle methods, or DOM classes.
4. **Implement in small loops.** Keep plugin state, settings, commands, views, and CSS explicit. Prefer Obsidian CSS variables and scoped selectors. Avoid speculative abstractions.
5. **Build and sync.** Build the plugin, then sync the output files into a test vault at `.obsidian/plugins/<plugin-id>/`.
6. **Validate in Obsidian.** Open or focus the GUI outside sandbox when needed, reload/enable the plugin, inspect errors and console logs, and verify UI or vault state through CLI/DOM/screenshot checks when available.
7. **Keep test data contained.** Generate notes only under a dedicated test folder in the test vault, and clean up only files created for the current test.

## Routing

| Need | Read |
|---|---|
| End-to-end implementation loop, scope decisions, build/sync cadence | [`references/development-workflow.md`](references/development-workflow.md) |
| Obsidian API, Tasks, Bases, Dataview, Datacore, or Templater API source-of-truth links | [`references/api-sources.md`](references/api-sources.md) |
| Plugin scaffold, manifest/build files, Svelte UI templates, CSS conventions | [`references/plugin-structure.md`](references/plugin-structure.md) |
| Test vault files, Obsidian CLI, GUI versus terminal control, sandbox limits | [`references/testing.md`](references/testing.md) |
| Adding a new GitHub API/template source to this skill | [`references/adding-sources.md`](references/adding-sources.md) |

## Core Rules

- Treat upstream docs and local submodules as source of truth; do not rely on memory for API names.
- Keep changes surgical and tied to the requested plugin behavior.
- Keep the `description` field in `SKILL.md` under 1024 characters; Codex rejects longer skill descriptions.
- Prefer reproducible validation: build output, vault sync path, reload command, error output, and UI evidence.
- Do not run GUI launch commands from sandbox. Ask for approval only when external GUI, network, or OS-level access is required.
