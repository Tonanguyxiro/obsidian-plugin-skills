# Plugin Structure

Use this reference when scaffolding a plugin, choosing a UI stack, or reviewing build and release files.

## Default Scaffold

Default to the official sample plugin for ordinary plugins:

- Upstream: `https://github.com/obsidianmd/obsidian-sample-plugin`
- Best for: commands, settings, editor actions, markdown postprocessors, simple views, lifecycle examples.
- Expected deployable files: `manifest.json`, `main.js`, optional `styles.css`.

Typical source files:

| File | Purpose |
|---|---|
| `manifest.json` | Plugin id, name, version, minimum app version, description |
| `package.json` | scripts and dependency versions |
| `main.ts` or `src/main.ts` | plugin class, lifecycle, commands, views, settings |
| `styles.css` | plugin styles shipped to the vault |
| `versions.json` | compatibility map for releases when used |
| `esbuild.config.mjs` or equivalent | TypeScript bundling into `main.js` |

Keep the sample-plugin shape unless the requested UI or build needs something else.

## UI Framework Choice

| Need | Recommended choice |
|---|---|
| Small settings tab or simple modal | Plain Obsidian UI helpers and scoped CSS |
| Custom view with moderate interaction | Plain DOM or a tiny local component pattern |
| Substantial stateful UI | Svelte with scoped CSS and Obsidian CSS variables |
| Existing utility-CSS project or explicit Tailwind request | Svelte plus Tailwind/UnoCSS template |

Useful Svelte sources:

- `https://github.com/alexlafroscia/obsidian-svelte-ui` for Svelte 5 components/utilities intended for Obsidian plugin UI.
- `https://github.com/emilio-toledo/obsidian-svelte-plugin` for a Svelte/TailwindCSS/UnoCSS plugin template.

Do not introduce Svelte, Tailwind, or UnoCSS into an existing plain plugin unless the UI complexity justifies the extra build surface or the user asks for it.

## CSS Rules

- Prefer Obsidian CSS variables such as `--background-primary`, `--text-normal`, `--interactive-accent`, and `--border-color`.
- Scope selectors under a plugin-owned class or view root.
- Avoid global resets and broad selectors such as `button`, `input`, `.modal`, or `.workspace-leaf` unless intentionally extending Obsidian UI.
- Keep CSS in `styles.css` or framework-scoped component styles according to the existing build.

## Manifest and Plugin ID

- Use the `manifest.json` `id` as the test-vault folder name.
- Keep `id` stable once users may have installed the plugin.
- Ensure `version` and `minAppVersion` are intentional before release work.
- For development-only changes, do not churn manifest metadata unless required.

## Build and Release Boundaries

For local validation, build only what is needed to produce `main.js` and optional `styles.css`. Release packaging, community-plugin submission metadata, changelogs, and version bumps are separate tasks unless the user asks for them.
