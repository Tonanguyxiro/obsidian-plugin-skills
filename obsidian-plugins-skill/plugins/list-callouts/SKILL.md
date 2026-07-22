# Obsidian List Callouts

> Upstream source: [`references/obsidian-list-callouts/`](../../../references/obsidian-list-callouts/). Activation triggers live in [`../../SKILL.md`](../../SKILL.md).

List Callouts turns individual **list items** into colored callouts. Type a configured marker
character immediately after the list bullet (and a trailing space), and the item is styled with that
marker's color and optional icon. Unlike Obsidian's built-in `> [!note]` block callouts, these live
inside list items and are per-line.

## Syntax

A marker only triggers when it is the **first thing after the list marker**, followed by a space:

```markdown
- & This item is highlighted yellow
- ? A question / uncertain item (orange)
* ! Important — works with -, *, or + bullets
1. @ Ordered lists work too
- [ ] & Works inside Tasks-plugin / checkbox items as well
```

Rules (from the matching regex in `src/main.ts`):

- Allowed list markers: `-`, `*`, `+`, or an ordered `1.` / `1)`.
- An optional checkbox `[ ]` may sit between the bullet and the marker (`- [ ] & text`).
- The marker char must be followed by a single space, then the content.
- Nested list items are matched independently — indent and add a marker to style a sub-item.

## Default markers

| Char | Color (RGB) | Typical use |
|------|-------------|-------------|
| `&`  | `255, 214, 0`  | yellow — highlight |
| `?`  | `255, 145, 0`  | orange — question |
| `!`  | `255, 23, 68`  | red — important / warning |
| `~`  | `124, 77, 255` | purple |
| `@`  | `0, 184, 212`  | cyan |
| `$`  | `0, 200, 83`   | green |
| `%`  | `158, 158, 158`| grey — muted |

These are configurable in the plugin's settings tab: edit a marker's character, set an **icon**
(any Obsidian icon, shown in place of the character), change colors for custom callouts, or add new
custom markers. Colors are stored as `"r, g, b"` strings.

## Where it renders

- Works in both **Reading mode** (markdown post-processor) and **Live Preview** (CodeMirror 6
  editor extension), so callouts look the same while editing and reading.
- Recognized list items get the CSS class `lc-list-callout`, a `data-callout="<char>"` attribute,
  and a `--lc-callout-color` CSS variable — useful if you want to target them from a CSS snippet.
- Further visual tuning (padding, etc.) is exposed through the **Style Settings** plugin.

## Common mistakes to avoid

- ❌ Marker not at the start of the item — `- text & more` does **not** trigger; the marker must
  immediately follow the bullet (`- & text`).
- ❌ Missing the trailing space — `- &text` does not match; it must be `- & text`.
- ❌ Using `+`, `*`, `-`, `>`, or `#` as a **custom** callout character — the plugin warns these can
  disrupt reading mode (they collide with markdown list/quote/heading syntax). Prefer punctuation
  like `&`, `?`, `!`, `@`, `$`, `%`, `~`.
- ❌ Expecting block-level callouts — this styles single list items, not multi-line blocks. For
  multi-line admonitions use Obsidian's native `> [!note]` callouts instead.
