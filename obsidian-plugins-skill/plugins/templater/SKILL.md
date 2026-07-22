# Templater Skill

> Upstream source: [`references/obsidian-templater/`](../../../references/obsidian-templater/). Deep-dive docs live alongside this file. Activation triggers live in [`../../SKILL.md`](../../SKILL.md).

[Templater](https://github.com/SilentVoid13/Templater) is an Obsidian plugin that lets you insert **variables** and **function results** into notes, and execute JavaScript to manipulate them.

## Core Syntax

### Command Tags

All Templater commands use `<%` as the opening tag and `%>` as the closing tag. There are two command types:

**Interpolation** — `<% expression %>`
Evaluates the expression and **outputs the result** into the note.

```javascript
<% tp.date.now() %>                        // outputs: 2026-03-24
<% tp.file.title %>                        // outputs: current file's title
```

**Execution** — `<%* expression %>`
Executes JavaScript for **side effects only** (no output by default).

```javascript
<%* tp.file.rename(tp.date.now("YYYY-MM-DD")) %>  // renames the file
```

### Function Hierarchy

All Templater functions live under the `tp` object. You call them with dot notation and parentheses:

```
tp.<module>.<function>()
```

Modules: `tp.date`, `tp.file`, `tp.frontmatter`, `tp.web`, `tp.system`, `tp.config`, `tp.app`, `tp.obsidian`, `tp.hooks`, `tp.user`

### Argument Types

When calling functions, pass values only — no names, no types:

| Type | How to pass |
|------|-------------|
| `string` | `"value"` or `'value'` |
| `number` | `15`, `-5` |
| `boolean` | `true` or `false` (lowercase) |

**Wrong**: `tp.date.now(format: string = "YYYY-MM-DD")`
**Right**: `tp.date.now("YYYY-MM-DD")`

### Optional Arguments

Arguments marked with `?` in docs are optional. Arguments with `= <default>` have a default value.

```
tp.date.now(format?: string = "YYYY-MM-DD", offset?: number|string, ...)
```

Valid calls:
```javascript
<% tp.date.now() %>                              // uses all defaults
<% tp.date.now("YYYY-MM-DD", 7) %>               // format + offset
<% tp.date.now("dddd, MMMM Do YYYY", 0, tp.file.title, "YYYY-MM-DD") %>
```

## Internal Modules Quick Reference

| Module | Prefix | What it does |
|--------|--------|-------------|
| Date | `tp.date` | Current date/time, formatting, offsets |
| File | `tp.file` | File name, title, creation/mod date, renaming |
| Frontmatter | `tp.frontmatter` | Read/parse YAML frontmatter |
| Web | `tp.web` | Fetch quotes, random links, web content |
| System | `tp.system` | User prompts, input dialogs, clipboard |
| Config | `tp.config` | Templater plugin configuration |
| App | `tp.app` | Obsidian app instance, vault access |
| Obsidian | `tp.obsidian` | Obsidian app functions (like `tp.obsidian.inputPrompt()`) |
| Hooks | `tp.hooks` | Templater lifecycle hooks |
| User | `tp.user` | User-defined script/command functions |

## Writing Templates

### Pattern: Daily Note with Navigation

```javascript
# <% tp.file.title %>

<< [[<% tp.date.now("YYYY-MM-DD", -1) %>]] | [[<% tp.date.now("YYYY-MM-DD", 1) %>]] >>

**Today:** <% tp.date.now("YYYY-MM-DD") %> (<% tp.date.now("dddd") %>)

## Gratitude
1.
2.
3.

## Goals
- [ ]
- [ ]

## Daily Quote
<% tp.web.daily_quote() %>
```

**Two ways to do date arithmetic with `tp.date.now`:**
```javascript
// Option A: offset parameter (simplest for yesterday/tomorrow)
<% tp.date.now("YYYY-MM-DD", -1) %>   // yesterday
<% tp.date.now("YYYY-MM-DD", 1) %>    // tomorrow

// Option B: format string + offset (gives you the day name too)
<% tp.date.now("dddd", -1) %>         // "Monday" (or whatever yesterday was)
```

### Pattern: File Rename on Creation

Use the execution command to rename a file during creation:

```javascript
<%* await tp.file.rename(tp.date.now("YYYY-MM-DD") + " - " + tp.file.title) %>
```

### Pattern: Conditional with JS Execution

```javascript
<%*
const title = tp.file.title;
if (title.includes("Meeting")) {
  return "📅 Meeting Notes";
}
return "Note";
%>
```

### Pattern: Reading Frontmatter

```javascript
<% tp.frontmatter.tags %>           // outputs tags from YAML frontmatter
<% tp.frontmatter.author %>          // outputs author field
<% tp.frontmatter.summary || "No summary provided." %>  // with fallback
```

## Debugging Templates

### Common Errors

**Error: "Cannot read property X of undefined"**
→ The object or property doesn't exist in the current context. For example, `tp.frontmatter.tags` fails if the file has no frontmatter.

**Error: "tp.date.now is not a function"**
→ Likely a typo. The function name is exactly `tp.date.now` (with dot notation). Check for missing parentheses.

**Error: Template outputs nothing**
→ You likely used `<%*` (execution) instead of `<%` (interpolation). `<%*` does not output by default — use `<%` to output values.

**Error: "Invalid argument" or wrong date format**
→ Check argument order and types. `tp.date.now()` expects format as first arg as a string. If passing a reference date, you also need its format.

**Error: Function not found (tp.user.X)**
→ User functions (`tp.user.X`) only work if defined in Templater settings and only on desktop (not mobile).

**Error: Template looks broken or partially literal**
→ Common cause: a missing `%>` closing tag. If a line like `<% tp.date.now("YYYY-MM-DD")` is missing its `%>`, everything after it will be treated as literal text. Check every opening `<%` has a matching `%>`.

### Debugging Approach

1. **Count your tags** — every `<%` needs a `%>`. If tags are imbalanced, find the missing closer.
2. **Check for literal text in output** — if template code appears literally in the output, the missing `%>` is somewhere before it.
3. **Verify module names** — `tp.date`, not `tp.Date` (case-sensitive)
4. **Check argument types** — strings in quotes, numbers without
5. **Test incrementally** — start with `<% tp.date.now() %>` alone, then add arguments one at a time

## Detailed Reference

For full documentation on syntax and commands, see the bundled references:

- [`syntax.md`](syntax.md) — full command and function syntax guide
- [`commands/overview.md`](commands/overview.md) — command types (`<%` vs `<%*`) and utilities
- [`commands/execution-command.md`](commands/execution-command.md) — JS execution in templates
- [`commands/whitespace-control.md`](commands/whitespace-control.md) — controlling whitespace
- [`commands/dynamic-command.md`](commands/dynamic-command.md) — dynamic command syntax
- [`user-functions/overview.md`](user-functions/overview.md) — user-defined functions (tp.user)
- [`user-functions/script-user-functions.md`](user-functions/script-user-functions.md) — script-based user functions
- [`user-functions/system-user-functions.md`](user-functions/system-user-functions.md) — system command user functions
