# Obsidian Datacore Scripting

> Upstream source: [`references/obsidian-datacore/`](../../../references/obsidian-datacore/). Activation triggers live in [`../../SKILL.md`](../../SKILL.md).

Datacore is a reactive data engine for Obsidian.md that indexes vault metadata (pages, sections, blocks, tasks, files) and provides live-updating views via React-based codeblocks.

Scripts are written in `datacorejsx` (or `datacorejs`/ts/tsx) codeblocks. The `dc` global provides the API.

## View Template

Every Datacore view follows this pattern:

```jsx
return function View() {
    // 1. Fetch data with a query
    const data = dc.useQuery("@page and #myTag");

    // 2. Optionally process with DataArray operations or dc.useMemo
    const processed = dc.useMemo(() => {
        return data.filter(item => item.value("rating") > 5);
    }, [data]);

    // 3. Return a dc component — ALWAYS enable paging for tables and lists
    return <dc.Table rows={processed} columns={COLUMNS} paging={25} />;
}
```

> **IMPORTANT**: Always enable `paging` on `dc.Table` and `dc.List` components — it dramatically improves performance and is required by the Datacore rendering contract. Use `paging={true}` for the default size or `paging={number}` for a specific page size.

## Codeblock Format

Use a fenced codeblock with the appropriate language:

~~~markdown
```datacorejsx
// your script here
```
~~~

Available languages: `datacorejs`, `datacorejsx`, `datacorets`, `datacoretsx`.

---

## 1. Queries

Queries use the Datacore query language to filter vault objects. They return arrays of matching metadata objects.

### Query Types

| Query | Meaning |
| ----- | ------- |
| `@page` | All markdown pages |
| `@file` | All files (including non-markdown) |
| `@section` | All markdown sections |
| `@block` | All markdown blocks |
| `@block-list` | All blocks containing lists |
| `@codeblock` | All code blocks |
| `@datablock` | Codeblocks annotated with `yaml:data` |
| `@list-item` | All list items |
| `@task` | All task items (`- [ ]`) |

### Filters

Combine queries with:

- **`#tag`** — Filter by tag: `#game`, `#project/status`
- **`and`** — Both conditions: `@page and #book`
- **`or`** — Either condition: `@block or @section`
- **`!`** — Negation: `!#draft`
- **`path("folder")`** — Objects in a folder
- **`exists(field)`** — Objects with a field defined (use this in queries, NOT in JS filter)
- **`connected([[Page]])`** — Objects linking to or from a page
- **`linkedto([[Page]])`** — Objects linking TO a page
- **`linkedfrom([[Page]])`** — Objects linking FROM a page
- **`parentof(query)`** — Objects that are parents of matching children
- **`childof(query)`** — Objects that are children of matching parents

### Expressions

Use expressions for comparisons and text matching:

```jsx
// Field comparisons (case-insensitive field names)
rating >= 9
$name != "Daily"
genre.contains("Sci-Fi")

// Combine with query operators
@page and #book and rating >= 7

// Date arithmetic
$ctime > date(now) - dur(30d)
```

**Expression literals**: `1`, `true`/`false`, `"text"`, `date(2024-01-15)`, `dur(7d)`, `[[Link]]`, `[1, 2, 3]`, `{ a: 1 }`

**Operators**: `+`, `-`, `*`, `/`, `%`, `>`, `<`, `=`, `!=`, `<=`, `>=`

---

## 2. Data Fetching

### dc.useQuery(query) — Reactive Query

```jsx
const pages = dc.useQuery("@page and #book");
```

- Runs the query and returns a reactive array of results
- Updates automatically when the index changes
- Use inside the `View()` function (React hook)
- For one-time queries (no reactivity), use `dc.query()` instead

### dc.query(query) — One-time Query

```jsx
const pages = dc.query("@page");
```

- Same as `useQuery` but runs once (no reactivity)
- Use for non-reactive contexts or manual subscriptions

> **Note**: There is also `dc.useFullQuery()` which returns an object with `{ results, duration }`. Prefer `dc.useQuery()` for reactive views — only use `dc.useFullQuery()` if you specifically need the timing information from the returned object.

### dc.useCurrentFile() — Current Page Metadata

```jsx
const current = dc.useCurrentFile();
return <p>On: {current.$path}</p>;
```

### dc.useFile(path) — Specific File Metadata

```jsx
const file = dc.useFile("Notes/Project.md");
```

---

## 3. DataArray Operations

DataArray wraps query results with chainable operations. It uses **swizzling** — indexing with a field name maps all elements to that field.

```jsx
const books = dc.useQuery("#book and @page");

// Swizzling: extract field values
books.$name    // => array of all book names
books.$ctime   // => array of all created times

// Chainable methods (all return new DataArrays):
.where(predicate)      // Filter — for simple predicates; for exists() checks, use the query instead
.filter(predicate)     // Alias for .where()
.map(func)             // Transform
.flatMap(func)         // Transform and flatten
.mutate(func)          // Mutate in place
.limit(n)              // First n elements
.slice(start?, end?)   // Range
.concat(iterable)      // Combine
.sort(key, direction?) // Sort (direction: "asc" | "desc")
.groupBy(key)          // Group into { key, rows } objects — SEE SECTION 7 FOR DETAILS
.distinct(key?)        // Unique values
.every(predicate)       // All match?
.some(predicate)       // Any matches?
.none(predicate)       // None match?
.first() / .last()     // First/last element
.expand(key)           // Recursively expand a tree key
.forEach(fn)           // Side effects
.array()               // Convert to plain array
.length                // Number of elements
```

> **Performance tip**: Use `exists(field)` in the query itself rather than `.filter(p => p.value("field") != null)` in JavaScript. Query-side filtering runs in the Datastore index and is much faster.

**Example chaining:**
```jsx
const topBooks = dc.useArray(books, array =>
    array.filter(book => book.value("rating") >= 7)
         .sort(book => book.value("rating"), "desc")
         .limit(10)
);
```

---

## 4. Table Views

```jsx
const COLUMNS = [
    { id: "link", value: row => row.$link },
    { id: "Rating", value: row => row.value("rating") },
    { id: "Genre", value: row => row.value("genre") },
];

return function View() {
    const books = dc.useQuery("#book and @page");
    return <dc.Table rows={books} columns={COLUMNS} paging={25} />;
}
```

> **Always set `paging`**: `paging={25}` or `paging={true}` for the default size.

### Column Options

```jsx
{
    id: "Genre",                      // Required: unique column ID
    value: row => row.value("genre"), // Required: extract cell value
    render: (value, row) => <b>{value}</b>, // Optional: custom JSX renderer
    title: "Genre Column",            // Optional: column header override
    width: "200px" | "50%" | "minimum" | "maximum", // Optional: column width
}
```

### Table Options

```jsx
<dc.Table
    rows={data}
    columns={COLUMNS}
    paging={25}           // ALWAYS enable paging — this is required
    scrollOnPaging={true}   // scroll to top on page change
    groupings={(key) => dc.fileLink(key)} // group header renderer
/>
```

---

## 5. List Views

```jsx
return function View() {
    const items = dc.useQuery("#note and @page");
    return <dc.List rows={items} renderer={item => item.$link} paging={true} />;
}
```

> **Always set `paging`**: `paging={true}` or `paging={number}`.

### List Types

```jsx
<dc.List type="unordered" rows={items} renderer={...} paging={true} />  // bullets (default)
<dc.List type="ordered" rows={items} renderer={...} paging={true} />    // numbered
<dc.List type="block" rows={items} renderer={dc.embed} paging={true} /> // no formatting
```

### List Options

```jsx
<dc.List
    rows={data}
    renderer={item => item.$link}
    paging={25}
    groupings={(key) => dc.fileLink(key)}
    maxChildDepth={3}
    childSource={"children"}
/>
```

---

## 6. Fields API

All indexable objects (pages, sections, blocks, tasks) support field access:

```jsx
page.value("fieldName")      // Get typed value (case-insensitive)
page.field("fieldName")      // Get Field object { key, value, raw }
page.fields()                // All fields as Field[]
```

Intrinsic fields (prefixed with `$`):
```jsx
page.$path     // Full path
page.$name     // Page name
page.$link     // Link object
page.$ctime    // Creation time
page.$mtime    // Modification time
page.$tags     // Array of tags
page.$links    // Array of links
page.$sections // Array of sections
page.$frontmatter  // Frontmatter map
```

**Special field access for names with spaces:**
```jsx
page.value("last reviewed")        // JavaScript
$row["last reviewed"] >= date(now)  // In expressions
```

---

## 7. Grouping with dc.useArray + groupBy

**This is the correct pattern for grouping** — use `dc.useArray` with `groupBy`, not `dc.useMemo` with manual object construction:

```jsx
const tasks = dc.useQuery("@task and $completed = false");

// CORRECT: Use dc.useArray with groupBy
const grouped = dc.useArray(tasks, array =>
    array.groupBy(task => task.$path)  // group by parent page path
);

// Render with dc.List — groupings prop renders group headers automatically
return <dc.List
    rows={grouped}
    renderer={item => item.$link}
    groupings={(key) => dc.fileLink(key)}
    paging={true}
/>;
```

**For task hierarchies (grouped by parent page with subtasks):**

```jsx
const tasks = dc.useQuery("@task and $completed = false");

const grouped = dc.useArray(tasks, array =>
    array.groupBy(task => task.$path)
);

return <dc.List
    rows={grouped}
    renderer={item => item.$link}
    groupings={(key) => dc.fileLink(key)}
    type="unordered"
    maxChildDepth={3}
    childSource={item => item.rows || []}
    paging={true}
/>;
```

> **Common mistake**: Do NOT use `dc.useMemo` with manual object grouping like `const byParent = {}; for (const t of tasks) { byParent[parent].push(...) }`. Use `dc.useArray` with `groupBy` — it returns a DataArray of `{ key, rows }` objects that `dc.List` understands natively.

---

## 8. Hierarchies (Children)

Lists support recursive child rendering via `$children`:

```jsx
const DATA = [
    { title: "Parent", children: [{ title: "Child1" }, { title: "Child2" }] }
];

return <dc.List rows={DATA} renderer={item => item.title} childSource={"children"} paging={true} />;
```

Custom child source:
```jsx
<dc.List
    rows={data}
    renderer={item => item.$link}
    childSource={"$children"}  // or "children" or item => item.subItems
    maxChildDepth={3}
    paging={true}
/>
```

---

## 9. Embeds and Links

### dc.embed — Render File Embeds

```jsx
<dc.List rows={blocks} renderer={dc.embed} type="block" paging={true} />
```

### dc.fileLink — Create Link HTML

```jsx
value: row => dc.fileLink(row.$path)
// or in JSX:
<span>{dc.fileLink(page.$path)}</span>
```

---

## 10. Code Sharing with dc.require

Define reusable code in a codeblock, then import it elsewhere.

**In `scripts/helpers.md`:**
~~~markdown
# ListItem

```jsx
function ListItem({ text }) {
    return <li>Item: {text}</li>;
}
return { ListItem };
```
~~~

**Import and use:**
```jsx
const { ListItem } = await dc.require(dc.headerLink("scripts/helpers.md", "ListItem"));

return function View() {
    return <ListItem text="Hello!" />;
}
```

---

## 11. Memoization

Use `dc.useMemo` for expensive computations that shouldn't rerun on every render:

```jsx
const ratingBuckets = dc.useMemo(() => {
    const buckets = {};
    for (const game of games) {
        const rating = (game.value("rating") || 0) + "";
        buckets[rating] = (buckets[rating] || 0) + 1;
    }
    return buckets;
}, [games]);
```

> Do NOT use `dc.useMemo` for grouping — use `dc.useArray` with `groupBy` instead.

---

## 12. Direct Array Creation

Create `DataArray` from plain arrays:

```jsx
const nums = dc.array([1, 2, 3, 4, 5]);
const doubled = nums.map(x => x * 2).limit(2).array();  // [2, 4]
```

---

## Quick Reference: Common Patterns

**Table of books with rating (ALWAYS enable paging):**
```jsx
const COLUMNS = [
    { id: "title", value: r => r.$link },
    { id: "rating", value: r => r.value("rating") },
    { id: "genre", value: r => r.value("genre") },
];

return function View() {
    const books = dc.useQuery("#book and @page");
    return <dc.Table rows={books} columns={COLUMNS} paging={25} />;
}
```

**Grouped list by tag (use dc.useArray + groupBy):**
```jsx
return function View() {
    const pages = dc.useQuery("@page");
    const grouped = dc.useArray(pages, a => a.groupBy(p => p.$tags[0] || "Untagged"));
    return <dc.List rows={grouped} renderer={p => p.$link} groupings={(k) => dc.fileLink(k)} paging={true} />;
}
```

**Tasks from a section (use exists() in query, not JS filter):**
```jsx
return function View() {
    const tasks = dc.useQuery("@task and $completed = false and childof(@section and $name = 'Daily')");
    return <dc.List rows={tasks} renderer={t => t.$link} paging={true} />;
}
```

**Pages with a specific field (use exists() in query):**
```jsx
return function View() {
    const projects = dc.useQuery("#project and @page and exists(status)");
    return <dc.Table rows={projects} columns={COLUMNS} paging={25} />;
}
```

**Pages linking to current:**
```jsx
return function View() {
    const current = dc.useCurrentFile();
    const linked = dc.useQuery(`linkedto(${current.$path}) and @page`);
    return <dc.List rows={linked} renderer={p => p.$link} paging={true} />;
}
```

**Embed list of blocks:**
```jsx
return function View() {
    const notes = dc.useQuery("#important and @block");
    return <dc.List rows={notes} renderer={dc.embed} type="block" paging={true} />;
}
```

**Table with formatted dates and timing info:**
```jsx
const COLUMNS = [
    { id: "name", value: r => r.$link },
    { id: "created", value: r => r.$ctime.toLocaleDateString("en-US", { year: "numeric", month: "short", day: "numeric" }) },
];

return function View() {
    const start = Date.now();
    const data = dc.useQuery("@page");
    return (
        <div>
            <p style={{color:"#666"}}>Found {data.length} pages</p>
            <dc.Table rows={data} columns={COLUMNS} paging={25} />
        </div>
    );
}
```

---

## Key Conventions

- **Always return `function View()`** from codeblocks — this is the React component
- **`dc` global** is always available in codeblocks
- **Field names are case-insensitive**: `rating` and `Rating` both work
- **Intrinsic fields start with `$`**: `$path`, `$name`, `$tags`, `$links`
- **DataArrays are immutable**: all operations return new DataArrays
- **Swizzling**: `dataArray.fieldName` extracts that field from all elements
- **ALWAYS enable `paging`** — set `paging={true}` or `paging={25}` on `dc.Table` and `dc.List`
- **Use `exists()` in queries** for filtering by field presence — do NOT use `.filter(p => p.value("x") != null)` in JavaScript
- **Use `dc.useArray` + `groupBy`** for grouping — NOT `dc.useMemo` with manual object construction
- **JSX uses Preact** (React-compatible, aliased as `react` in the build)
