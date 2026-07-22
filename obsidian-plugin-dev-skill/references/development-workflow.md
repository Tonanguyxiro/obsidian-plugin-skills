# Development Workflow

Use this reference when starting or modifying an Obsidian plugin. The goal is a tight loop from user intent to verified behavior in a test vault.

## 1. Clarify the Plugin Shape

Classify the requested change before editing:

| Shape | Common implementation surface |
|---|---|
| Command-only | `addCommand`, callbacks, active file/editor checks |
| Settings-heavy | settings interface, defaults, load/save, settings tab |
| Editor extension | CodeMirror extensions, editor commands, selections |
| Markdown postprocessor | rendered markdown hooks, DOM cleanup on unload |
| View/sidebar | custom `ItemView`, workspace leaves, icons |
| Status/ribbon/tooling | ribbon icons, status bar items, commands |
| Bases integration | `.base` files, properties, views, data derived from vault metadata |
| Dataview/Datacore/Tasks/Templater integration | plugin APIs or generated note/query/template content |
| Complex UI | Svelte component tree, scoped CSS, Obsidian CSS variables |

If more than one shape applies, implement the smallest vertical slice first.

## 2. Inspect the Existing Project

Read the local project before choosing an approach:

```bash
rg --files -g 'manifest.json' -g 'package.json' -g 'main.ts' -g 'src/**' -g 'styles.css'
rg -n "addCommand|PluginSettingTab|ItemView|MarkdownPostProcessor|registerMarkdownPostProcessor|CodeMirror|Svelte|Dataview|Datacore|Templater|Tasks" .
```

Prefer the repo's existing build system, folder layout, and UI framework. If the project is absent or empty, choose the simplest scaffold from `plugin-structure.md`.

## 3. Verify APIs Before Coding

Before documenting or using an API name:

- Check `obsidianmd/obsidian-api` or installed `obsidian` types for core Obsidian APIs.
- Check the target plugin's docs/source for third-party APIs such as Dataview, Datacore, Tasks, or Templater.
- Prefer type definitions and exported API files over examples copied from issues or blog posts.
- If an API is not verified, state the uncertainty and inspect source before implementation.

## 4. Implement in Small Loops

Use a narrow implementation loop:

1. Add or adjust one behavior.
2. Build.
3. Sync the built files into a test vault plugin folder.
4. Reload the plugin.
5. Inspect errors and verify visible behavior.
6. Repeat.

Keep settings, state, commands, views, and cleanup explicit. Register disposables with Obsidian lifecycle helpers when possible. Do not add framework layers, global CSS, or broad refactors unless the requested behavior needs them.

## 5. Build and Sync

For a normal Obsidian plugin, the deployable test-vault folder usually needs:

- `manifest.json`
- `main.js`
- `styles.css` if the plugin has styles

Sync into:

```text
<test-vault>/.obsidian/plugins/<plugin-id>/
```

Use the plugin id from `manifest.json`. Keep the test vault separate from the user's real vault unless the user explicitly names a delivery vault.

## 6. Validate and Iterate

Minimum validation for meaningful plugin changes:

- Build succeeds.
- Test-vault plugin folder contains fresh output.
- Plugin reloads/enables in Obsidian.
- `obsidian dev:errors` and console checks show no new relevant errors.
- UI changes are verified with DOM/screenshot checks when applicable.
- Generated test notes stay under a dedicated test folder.

If sandbox restrictions prevent GUI or CLI validation, document the exact manual command sequence and what result to expect.
