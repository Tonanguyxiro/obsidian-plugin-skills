# Obsidian Tasks Plugin API

> Upstream source: [`references/obsidian-tasks/`](../../references/obsidian-tasks/). Activation triggers live in [`../SKILL.md`](../SKILL.md).

This skill provides knowledge about the Obsidian Tasks plugin (https://github.com/obsidian-tasks-group/obsidian-tasks) - its task data model, query language, scripting capabilities, and plugin API. All information is embedded directly in this file.

## Task Data Model

### Task Fields

Each task has these properties:

**Status fields:**
- `task.isDone` - boolean
- `task.status.name` - e.g. 'Todo', 'In Progress'
- `task.status.type` - 'TODO', 'DONE', 'IN_PROGRESS', 'ON_HOLD', 'CANCELLED', 'NON_TASK'
- `task.status.symbol` - the checkbox character (e.g. ' ', '/')
- `task.status.nextSymbol` - what the checkbox becomes when toggled (e.g. 'x')

**Dates (each is a TasksDate object):**
- `task.created` - creation date
- `task.start` - start date
- `task.scheduled` - scheduled date
- `task.due` - due date
- `task.done` - done date
- `task.cancelled` - cancelled date
- `task.happens` - earliest of due/scheduled/start

**Priority:**
- `task.priorityNumber`: 0 (Highest) to 5 (Lowest), 3 = Normal
- `task.priorityName`: 'Highest', 'High', 'Medium', 'Normal', 'Low', 'Lowest'

**Dependencies (Tasks 6.1.0+):**
- `task.id` - unique string identifier
- `task.dependsOn` - array of task IDs this task depends on
- `task.isBlocked(query.allTasks)` - boolean
- `task.isBlocking(query.allTasks)` - boolean

**Other properties:**
- `task.description` - task text (includes tags)
- `task.descriptionWithoutTags` - text with tags stripped
- `task.tags` - array of tags, e.g. ['#todo', '#health']
- `task.isRecurring` - boolean
- `task.recurrenceRule` - standardized recurrence text or ''
- `task.onCompletion` - 'delete', 'keep', or ''
- `task.urgency` - numeric urgency score
- `task.originalMarkdown` - full task line as written
- `task.lineNumber` - line number in file (0-indexed)

**File properties:**
- `task.file.path` - full path including filename
- `task.file.pathWithoutExtension`
- `task.file.root` - root folder
- `task.file.folder` - parent folder (trailing slash)
- `task.file.filename` - filename with extension
- `task.file.filenameWithoutExtension`
- `task.hasHeading` - boolean
- `task.heading` - heading text or null

**Obsidian Properties (Tasks 7.7.0+):**
- `task.file.hasProperty('name')` - boolean
- `task.file.property('name')` - returns the value or null

**Links (Tasks 7.21.0+):**
- `task.outlinks` - links in task description
- `task.file.outlinks` - all links in the file

### TasksDate Formatting

TasksDate objects (for task.due, task.start, etc.) expose these methods:

| Method | Returns | Example |
|--------|---------|---------|
| `task.due.moment` | Moment.js object or null | moment('2023-07-04') |
| `task.due.formatAsDate()` | '2023-07-04' or '' | |
| `task.due.formatAsDate('no date')` | '2023-07-04' or 'no date' | |
| `task.due.format('dddd')` | 'Tuesday' (locale-aware) | |
| `task.due.format('dddd', 'no date')` | day name or fallback | |
| `task.due.toISOString()` | '2023-07-04T00:00:00.000Z' | |
| `task.due.toISOString(true)` | with local timezone offset | |
| `task.due.category.name` | 'Overdue', 'Today', 'Future', 'Undated' | |
| `task.due.fromNow.name` | 'in 22 days', '8 days ago' | |

Note: All stored dates have no time - they represent midnight at the start of the day in local time. Use moment.js for date arithmetic.

## Tasks Query Syntax

Tasks queries go inside a tasks block in Obsidian:

~~~markdown
```tasks
query instructions
```
~~~

### Filters

| Filter | Description |
|--------|-------------|
| `done` / `not done` | Task completion status |
| `status.type is IN_PROGRESS` | Match by status type |
| `status.name (includes) "Review"` | Match by status name |
| `is recurring` / `is not recurring` | Recurrence filter |
| `priority is (above, below, not)? (lowest, low, none, medium, high, highest)` | Priority filter |
| `due (on, before, after, in, ...)` | Date filters |
| `has due date` / `no due date` | Date existence check |
| `path (includes) "Projects/"` | File path filter |
| `folder (includes) "Work/"` | Folder filter |
| `filename (includes) "tasks"` | Filename filter |
| `heading (regex matches) /Meeting/` | Heading filter |
| `description (includes) "review"` | Description filter |
| `has tags` / `tag (includes) #work` | Tag filters |
| `is blocked` / `is not blocked` | Dependency filters |
| `has id` / `no id` | ID filters |
| `filter by function <expression>` | Custom JavaScript filter |

### Date Expressions

Date filters support many forms:

```
due on 2024-01-15
due before 2024-01-15
due after 2024-01-15
due in 5 days
due today
due tomorrow
due Saturday
due next week
due (last, this, next) (week, month, quarter, year)
due in YYYY-Www  (ISO week)
```

### Sorting

```
sort by due
sort by status
sort by priority
sort by created
sort by urgency
sort by path
sort by folder
sort by filename
sort by happens
sort by (function) <expression>
```

### Grouping

```
group by due
group by status
group by priority
group by folder
group by filename
group by (function) <expression>
```

### Combining Filters

```
(done) AND (priority is high)
(done) OR (priority is highest)
NOT (path includes "Archive")
(due today) AND ((priority is high) OR (priority is highest))
```

### Layout Options

- `hide due date` / `hide done date` / `hide start date` / `hide scheduled date`
- `hide tags` / `hide priority` / `hide recurrence rule`
- `hide edit button` / `hide postpone button`
- `short mode` / `full mode`
- `limit to 10 tasks`

## Custom Expressions (JavaScript)

Use `filter by function`, `sort by function`, or `group by function` with JavaScript expressions:

~~~javascript
// Simple boolean expression
filter by function task.description.length > 100

// Date filtering
filter by function task.due.format('dddd') === 'Tuesday'

// Using moment.js for date math
filter by function task.due.moment?.isSameOrBefore(moment(), 'day') || false

// Priority check
filter by function task.priorityNumber < 2

// Folder matching
filter by function task.file.folder.includes("Work/Projects/")

// Custom grouping
group by function task.due.category.name
~~~

Expressions support:
- Variables: `const x = task.due.moment;`
- `if/else` blocks
- `return` statements (if you include `return`, you must write it explicitly)
- `||` (OR with fallback): `task.due.format('dddd') || 'no date'`
- moment.js for date operations

## The Tasks Plugin API

For integration from other plugins (available at app.plugins.plugins['obsidian-tasks-plugin'].apiV1):

### Methods

**`createTaskLineModal(): Promise<string>`** (Tasks 2.0.0+)
- Opens the Tasks Create/Edit UI and returns the markdown string for the created task
- Returns empty string if cancelled

**`editTaskLineModal(taskLine: string): Promise<string>`** (Tasks 7.21.0+)
- Opens the Tasks Create/Edit UI pre-filled with the provided task
- Returns empty string if cancelled or if task was deleted via onCompletion: delete
- Use this to pre-set fields: pass a task line like '- [ ] Buy groceries [calendar] tomorrow'

**`executeToggleTaskDoneCommand(line: string, path: string): string`** (Tasks 7.2.0+)
- Toggles a task line and returns the updated markdown string
- Handles recurrence - returns two lines if a recurring task was completed

### Auto-Suggest Integration (Tasks 7.2.0+)

Plugins extending Obsidian's markdown editor can implement:
```typescript
showTasksPluginAutoSuggest(
  cursor: EditorPosition,
  editor: Editor,
  lineHasGlobalFilter: boolean
): boolean | undefined
```
Return `true` to show, `false` to hide, `undefined` for default behavior.

## Task Line Format

Tasks recognize this format (all parts optional except checkbox):

```
- [ ] Task description 🔼 🏁 delete 🔁 every day ➕ 2023-07-01 🛫 2023-07-02 ⏳ 2023-07-03 📅 2023-07-04 ❌ 2023-07-06 ✅ 2023-07-05 🆔 abc123 ⛔ 123456,abc123
```

Key markers (in task line text):
- 🔼 / 🔽 - priority (High/Low)
- 🔁 - recurrence rule
- ➕ - created date
- 🛫 - start date
- ⏳ - scheduled date
- 📅 - due date
- ❌ - cancelled date
- ✅ - done date
- 🏁 - onCompletion action
- 🆔 - task ID
- ⛔ - depends on

## Date Abbreviations in the Create/Edit Modal

| Abbreviation | Expands to |
|-------------|------------|
| `td` | `today` |
| `tm` | `tomorrow` |
| `yd` | `yesterday` |
| `tw` | `this week` |
| `nw` | `next week` |
| `weekend` / `we` | `sat` |

## Priority Symbols

| Symbol | Priority | priorityNumber |
|--------|----------|----------------|
| 🔺 | Highest | 0 |
| 🔼 | High | 1 |
| (none) | Normal | 3 |
| 🔽 | Low | 4 |
| ⬇ | Lowest | 5 |

## Recurrence Patterns

Standard forms:
- `every day`
- `every week`
- `every month`
- `every year`
- `every 3 days`
- `every 2 weeks on Thursday`
- `every 3 months`
- `every 3 months on the 15th`

With date tracking:
- `every day when done` - next occurrence's date = done date
- `every day` (default) - next occurrence's date = previous occurrence's date

## Status Types

Core status types: `TODO`, `DONE`, `IN_PROGRESS`, `ON_HOLD`, `CANCELLED`, `NON_TASK`

Default statuses:
- `[ ]` - Todo (symbol: ' ', next: 'x')
- `/` - In Progress (symbol: '/', next: 'x')
- `x` - Done (symbol: 'x', next: '')
- `-` - Cancelled (symbol: '-', next: '')

## Quick Reference: Relative Date Filtering

The `due this week` filter uses ISO 8601 weeks (Monday through Sunday). It was changed in Tasks 2.0.0 from a single date (Sunday before) to a full date range (Monday-Sunday of the current week). Supported relative date keywords: last, this, next for week, month, quarter, year.
