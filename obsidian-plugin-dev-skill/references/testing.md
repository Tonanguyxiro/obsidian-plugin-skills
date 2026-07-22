# Testing and Validation

Use this reference when validating a plugin in an Obsidian test vault or deciding what can run from terminal versus GUI.

## Test Vault Convention

Use a dedicated test vault whenever possible. Put generated test notes under a clearly named folder, for example:

```text
_codex-plugin-tests/<plugin-id>/<scenario-name>/
```

Keep test files small and scenario-specific. Do not create notes in the vault root. Do not clean unrelated files.

Install or sync the built plugin to:

```text
<test-vault>/.obsidian/plugins/<plugin-id>/
```

The plugin folder should contain `manifest.json`, `main.js`, and `styles.css` if styles exist.

## Terminal Control Versus GUI

Obsidian plugin validation has two different surfaces:

| Surface | What it means | Sandbox stance |
|---|---|---|
| GUI app | The Electron Obsidian application with a visible window and loaded vault | Do not launch from sandbox without approval; user may need to open/focus it manually |
| Terminal CLI | `obsidian ...` commands that control or inspect a running Obsidian instance | Use when available, but requires Obsidian to already be running |
| TUI | A terminal UI program | Obsidian itself is not a TUI app; if the user says TUI, clarify whether they mean terminal CLI control |

Cross-platform invariant: first get a vault open in the GUI, then use terminal commands to reload, inspect, and verify. OS-specific launch commands differ; the validation loop does not.

## Obsidian CLI

Official docs: `https://help.obsidian.md/cli`

Run `obsidian help` for the installed command set. Common plugin-development commands include:

```bash
obsidian plugin:reload id=<plugin-id>
obsidian dev:errors
obsidian dev:console level=error
obsidian eval code="app.vault.getFiles().length"
obsidian dev:dom selector=".workspace-leaf" text
obsidian dev:screenshot path=/tmp/obsidian-plugin-check.png
```

Target a specific vault when supported:

```bash
obsidian vault="<vault-name>" plugin:reload id=<plugin-id>
```

If `obsidian` is not found, ask the user to install or expose the CLI. Do not assume a platform-specific binary name.

## Validation Loop

1. Build the plugin.
2. Sync output files to the test vault plugin folder.
3. Ensure the vault is open in the Obsidian GUI.
4. Reload the plugin with CLI or manually from settings.
5. Check `dev:errors` and error-level console logs.
6. Verify behavior with the smallest reliable evidence:
   - command appears and runs,
   - test note is created or updated,
   - DOM selector exists,
   - screenshot shows expected UI,
   - `eval` confirms app state.
7. Record blocked steps if GUI/CLI access is unavailable.

## Sandbox and Approval Rules

- Building and copying inside the workspace or test vault may be normal filesystem work.
- Opening GUI apps, using OS launchers, or controlling external windows generally requires approval.
- Network installs may require approval if dependencies are missing.
- If validation cannot run, provide exact manual commands and expected outputs instead of claiming success.
