# Obsidian Dataview Scripting

> Upstream source: [`references/obsidian-dataview/`](../../../references/obsidian-dataview/). Activation triggers live in [`../../SKILL.md`](../../SKILL.md).

Dataview treats your Obsidian vault as a queryable database. It indexes metadata from markdown files and lets you query it with a SQL-like language or JavaScript API.

## Query Types

The first and only mandatory part of a DQL query. Four types:

| Type | Output | Example |
|------|--------|---------|
| `LIST` | Bullet list of pages | `LIST FROM #projects` |
| `TABLE` | Tabular view with columns | `TABLE name, status FROM #tasks` |
| `TASK` | Interactive task list | `TASK WHERE !completed` |
| `CALENDAR` | Calendar view with dots | `CALENDAR file.ctime` |

```dataview
TABLE author, rating, year
FROM #books
WHERE rating >= 4
SORT rating DESC
```

## Data Commands

Commands that refine your query (order matters, except FROM):

- `FROM` — source (tag, folder, link): `FROM #projects`, `FROM "Tasks"`, `FROM [[Page]]`
- `WHERE` — filter by condition: `WHERE status = "open" AND due < date(today)`
- `SORT` — sort field: `SORT file.ctime DESC`
- `GROUP BY` — group results: `GROUP BY folder`
- `LIMIT` — limit count: `LIMIT 10`
- `FLATTEN` — expand array fields: `FLATTEN tags AS tag`

## Sources

Sources define which pages to query:

```
FROM #tag              -- pages with tag
FROM "folder"          -- pages in folder (use quotes for folders)
FROM [[Page]]          -- pages linking to Page
FROM !#tag             -- pages WITHOUT tag
FROM #a OR #b          -- union
FROM #a AND "folder"   -- intersection
FROM outgoing([[Page]]) -- pages this page links to
```

## Implicit Fields (Available on Every Page)

| Field | Type | Description |
|-------|------|-------------|
| `file.name` | string | File name without extension |
| `file.link` | link | Link to the file |
| `file.path` | string | Full path |
| `file.folder` | string | Folder path |
| `file.ctime` | date | Creation time |
| `file.mtime` | date | Modification time |
| `file.tags` | array | All tags in file |
| `file.etags` | array | Explicit tags (#tag) |
| `file.inlinks` | array | Links TO this file |
| `file.outlinks` | array | Links FROM this file |
| `file.tasks` | array | All tasks in file |
| `file.lists` | array | All list items in file |
| `file.day` | date | Date from filename (2024-01-15) |

## Task Implicit Fields

| Field | Type | Description |
|-------|------|-------------|
| `task.completed` | boolean | Whether the task checkbox is checked |
| `task.text` | string | The task text (everything after `- [ ]`) |
| `task.fullyCompleted` | boolean | True if task and all subtasks are checked |
| `task.link` | link | Link to the file containing the task |
| `task.due` | date | Due date if set on the task |
| `task.created` | date | When the task was created |
| `task.start` | date | Start date if set |
| `task.done` | date | Completion date (set when task is checked via Dataview) |
| `task.subtasks` | array | Array of subtask objects |

**Note:** There is NO `task.completion` field. Use `task.done` for completion date.

## Metadata Annotation

### Frontmatter (top of file)
```yaml
---
author: Jane Austen
genre: fiction
year: 1813
rating: 5
---
```

### Inline Fields (anywhere in file)
```markdown
Author:: Jane Austen
Genre:: fiction
Rating:: 5
```

Query with: `author`, `genre`, `rating`

## Key Functions

### Filtering
```js
where(x => x.rating >= 4)           // filter
contains(field, "value")             // string/list contains
startswith(field, "prefix")          // starts with
endswith(field, "suffix")            // ends with
regextest(pattern, string)           // regex match
typeof(field) = "number"             // type check
```

### Array/Object
```js
map(array, x => x.field)            // transform
filter(array, x => condition)        // filter
flatten(array)                       // flatten nested arrays
unique(array)                        // deduplicate
length(array)                        // count
join(array, ", ")                    // join to string
nonnull(array)                       // remove nulls
```

### Numeric
```js
sum(array), average(array), min(a,b,...), max(a,b,...)
round(number, 2), floor(number), ceil(number)
```

### String
```js
lower(string), upper(string)
replace(string, "old", "new")
split(string, delimiter)
substring(string, start, end)
contains("hello", "lo")              // case-sensitive contains
icontains("Hello", "lo")             // case-insensitive
```

### Date/Time
```js
date("2024-01-15")                   // parse date (DQL: date(), DataviewJS: dv.date())
dateformat(date, "yyyy-MM-dd")       // format
striptime(date)                      // DQL only: remove time component (NOT in DataviewJS)
dur("3 days")                        // duration
```

### Utility
```js
default(field, "N/A")               // null fallback
choice(condition, then, else)       // if-then-else
link("path", "display")             // create link
typeof(value)                        // "number", "string", "date", etc.
```

## DataviewJS API

For complex queries, use `dataviewjs` code blocks:

~~~dataviewjs
const pages = dv.pages("#books").where(p => p.rating >= 4);
dv.table(
  ["Book", "Author", "Rating"],
  pages.sort(p => p.rating, 'desc')
       .map(p => [p.file.link, p.author, p.rating])
);
~~~

### DataArray Methods

`dv.pages()` returns a DataArray with chainable methods:

```js
dv.pages("#projects")
  .where(p => !p.completed)           // filter
  .sort(p => p.due)                   // sort
  .limit(10)                          // limit
  .groupBy(p => p.status)            // group (NOT dv.groupBy())
  .map(g => [g.key, g.rows.length])   // transform
```

### Grouping in DataviewJS

Use `.groupBy()` on a DataArray — **NOT `dv.groupBy()`** (that doesn't exist):

```js
// Correct:
const grouped = dv.pages("#books").groupBy(p => p.genre);

// For each group:
for (const group of grouped) {
    dv.header(3, group.key);           // group key (e.g., genre name)
    dv.paragraph(`Count: ${group.rows.length}`);
    dv.list(group.rows.file.link);     // use dv.list() for links
}
```

### Task Rendering

**Prefer `dv.taskList()` for task lists** — it renders interactive checkboxes:

```js
// Correct - use dv.taskList():
dv.taskList(dv.pages("#projects").file.tasks.where(t => !t.completed));

// Avoid manually building task strings with dv.list():
// BAD:  dv.list(group.rows.map(t => "- [ ] " + t.text))
// GOOD: dv.taskList(group.rows)
```

### Date Handling in DataviewJS

Dates in DataviewJS are Luxon `DateTime` objects. Use Luxon methods directly:

```js
const today = dv.date("today");           // Luxon DateTime
const startOfWeek = today.minus({ days: today.weekday });  // Sunday
const endOfWeek = startOfWeek.plus({ days: 7 });

// Compare dates directly (Luxon supports <, >, >=, <=):
.where(t => t.done >= startOfWeek && t.done < endOfWeek)
```

**Common mistakes to avoid:**
- ❌ `const { striptime } = this` — `this` doesn't have striptime in DataviewJS
- ❌ `task.completion` — field doesn't exist; use `task.done`
- ❌ `dv.list(group.rows.map(t => "- [ ] " + t.text))` — use `dv.taskList(group.rows)`

### Common Patterns

**Uncompleted tasks from a tag:**
```dataview
TASK
FROM #projects
WHERE !completed
```

**Table with calculations:**
```dataview
TABLE
  file.ctime AS "Created",
  date(today) - file.ctime AS "Age (days)",
  rating * 2 AS "Doubled Rating"
FROM #books
```

**Grouped and sorted list:**
```dataview
LIST rows.file.link
FROM #reading
GROUP BY genre
SORT key
```

**Inline expression:**
```markdown
Total books: `= length(#books)`
This book was published: `= date(published).year`
```

**DataviewJS task list:**
```dataviewjs
dv.taskList(
  dv.pages("#projects")
    .file.tasks
    .where(t => !t.completed && t.due)
    .sort(t => t.due)
);
```

**DataviewJS: Completed tasks this week, grouped by file:**
```dataviewjs
const today = dv.date("today");
const startOfWeek = today.minus({ days: today.weekday });
const endOfWeek = startOfWeek.plus({ days: 7 });

const completedThisWeek = dv.pages("")
  .flatMap(p => p.file.tasks)
  .where(t => t.completed && t.done)
  .where(t => t.done >= startOfWeek && t.done < endOfWeek);

const grouped = completedThisWeek.groupBy(t => t.link);

for (const group of grouped) {
    dv.header(3, group.key.display);
    dv.taskList(group.rows);
}
```

## DataviewJS `dv.io` (Async)

```js
// Load CSV
const data = await dv.io.csv("data.csv");

// Load file contents
const content = await dv.io.load("notes/summary.md");

// Normalize paths
const path = dv.io.normalize("MyFile", "Folder/File.md");
```

## Error Handling in DataviewJS

```js
try {
  const result = await dv.query("LIST FROM #nonexistent");
  if (result.successful) {
    dv.list(result.value.values);
  } else {
    dv.paragraph("No results: " + result.error);
  }
} catch(e) {
  dv.paragraph("Error: " + e.message);
}
```

## Writing Queries - Quick Tips

1. **Start simple** — begin with just `LIST` or `TABLE`, then add filters
2. **Use quotes for folders** — `FROM "My Folder"` not `FROM My Folder`
3. **Dates need `date()`** — `WHERE due < date(today)` not `WHERE due < today`
4. **Tags use #** — `FROM #projects` not `FROM projects` (but field values don't)
5. **Case matters** — `contains` is case-sensitive, `icontains` isn't
6. **Use `default()` for nulls** — `default(field, "N/A")`
7. **Array fields need FLATTEN** — to iterate: `FLATTEN tags AS tag`
8. **Debug with inline** — test expressions: `= typeof(field)` or `= field`
9. **Task completion date** — use `task.done`, NOT `task.completion`
10. **In DataviewJS, use `.groupBy()`** on DataArrays, not `dv.groupBy()`
11. **Render task lists with `dv.taskList()`**, not `dv.list()` with manual checkbox strings
