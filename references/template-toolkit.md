# Template Toolkit Reference for A-Parser

Template Toolkit is used in: Result Format, Query Format, Query Builder, Results Builder, Filters, Parser Settings fields.

## Syntax

```
[% variable %]                    — output variable
[% IF cond %]...[% ELSE %]...[% END %]
[% FOREACH item IN array %]...[% END %]
[% WHILE cond %]...[% END %]
[% SET var = value %]
```

Outside `[% %]` tags, `$variable` works for simple substitution.  
Use `\n` for literal newlines in format strings.

---

## Built-in Variables

### Query variables (always available)
```
$query          — formatted query
$query.orig     — original query (before Query Format transformation)
$query.num      — sequence number
$query.lvl      — nesting level (0 = from input)
$query.first    — first query in chain
$query.prev     — parent level query
```

### Result variables (depend on parser)
```
$fieldName           — flat result field (e.g., $title, $totalcount)
$arrayName.N.col     — N-th array element, column col (0-indexed)
$p1.fieldName        — field from parser 1 (multi-parser tasks)
$p2.arrayName.0.link — first link from parser 2
$info.success        — parsing success (1/0)
$info.retries        — number of retries used
$data                — raw page data (HTML)
$pages.N.data        — page N's raw data
```

### Global variables
```
$datefile       — current datetime: 2024-01-15_14-30-00 (for filenames)
```

---

## Array Methods

```
# Iterate with template:
$serp.format('$link\n')
$serp.format('$link\t$anchor\t$snippet\n')

# Count elements:
[% serp.size %]

# Access by index:
$serp.0.link        — first element's link
$serp.last.link     — last element's link

# Slice:
[% FOREACH i IN serp.slice(0,9) %][% i.link %][% END %]

# JSON serialize:
[% serp.json %]

# Format without template (shows all fields):
$serp.format()
```

---

## Filters (pipes)

```
[% value | upper %]              — UPPERCASE
[% value | lower %]              — lowercase
[% value | html %]               — HTML escape: &amp; &lt; etc.
[% value | html_entity %]        — HTML entities decode
[% value | uri %]                — URL encode
[% value | trim %]               — strip whitespace
[% value | replace('old','new') %]  — replace text
[% value | remove('\s+') %]      — remove pattern
[% value | truncate(100) %]      — truncate to N chars
[% value | length %]             — string length
```

---

## $tools.* Methods

### Query management
```
[% tools.query.add(url) %]                   — add URL to queue
[% tools.query.add(url, maxLevel) %]         — add with max level limit
[% tools.query.add({query=>url, lvl=>1}) %]  — add at specific level
[% tools.query.addAll(serp, 'link', 2) %]    — add all links from array
```

### JSON
```
[% tools.parseJSON(data) %]      — parse JSON string into variables
[% tools.error %]                — error message if parseJSON failed
```

### CSV output
```
[% tools.CSVline(query, p1.title, p1.totalcount) %]
```
Auto-quotes fields with commas, adds newline.

### SQLite database
```
[% tools.sqlite.run("CREATE TABLE IF NOT EXISTS t (url TEXT, title TEXT)") %]
[% tools.sqlite.run("INSERT INTO t VALUES (?, ?)", query, p1.title) %]
[% rows = tools.sqlite.all("SELECT * FROM t WHERE url = ?", query) %]
[% row = tools.sqlite.get("SELECT title FROM t WHERE url = ?", query) %]
```

### User-Agent
```
[% tools.ua.random() %]          — random UA string
[% tools.ua.list() %]            — array of all UAs
```

### JavaScript functions
```
[% tools.js.myFunction(arg1, arg2) %]    — call custom function from tools.js
```
Custom functions defined in Tools → JavaScript Editor.

### Base64
```
[% tools.base64.encode(value) %]
[% tools.base64.decode(value) %]
```

### Data references
```
[% tools.data.GoogleDomains %]   — array of Google country domains
[% tools.data.CountryCodes %]    — ISO country codes
[% tools.data.GoogleLangs %]     — Google language codes
```

### Memory (cross-request storage)
```
[% tools.memory.set('key', value) %]
[% val = tools.memory.get('key') %]
[% tools.memory.delete('key') %]
```

### Task info
```
[% tools.aparser.version() %]    — A-Parser version
[% tools.task.id %]              — current task ID
[% tools.task.threadsCount %]    — number of threads
[% tools.task.stop('reason') %]  — stop task with message
```

### Static template flag
```
[% isStaticTemplate() %]         — marks template as static (evaluates once at task start)
```
Use for stable filenames when using API file links.

---

## Template Examples

### TSV (tab-separated values)
```
$query\t$p1.title\t$p1.totalcount\n
```

### CSV with header
Prepend: `Query,Title,Count\n`  
Format: `[% tools.CSVline(query, p1.title, p1.totalcount) %]`

### Multi-field array output
```
$serp.format('$link\t$anchor\t$snippet\n')
```

### Conditional output
```
[% IF p1.totalcount > 1000 %]$query — popular: $p1.totalcount[% ELSE %]$query — low: $p1.totalcount[% END %]
```

### Combine queries from SERP
```
[% tools.query.addAll(p1.serp, 'link', 1) %]
```
This feeds all SERP links as new level-1 queries.

### JSON output of full result
```
[% data.json %]
```

### Write to SQLite and output nothing
```
[% tools.sqlite.run("INSERT INTO results (query, title) VALUES (?, ?)", query, p1.title) %][% SET skip = 1 %]
```

### Date-stamped result file
```
result_$datefile.csv
```

### Folder structure by query
```
results/$queriesfile/$query.txt
```

---

## Regex in Templates

```
# Use query in regex pattern:
[% FOREACH item IN p1.serp %]
  [% IF item.link.match(query) %]$item.link[% END %]
[% END %]
```

---

## Plugins

| Plugin | Usage |
|---|---|
| Date | `[% USE date %][% date.format(date.now, '%Y-%m-%d') %]` |
| Math | `[% USE Math %][% Math.round(1.5) %]` |
| String | `[% USE String %][% s = String.new(value) %]` |
| HTML | `[% USE HTML %][% HTML.escape('<tag>') %]` |
| URL | `[% USE URL %][% URL.new(href).scheme %]` |

---

## Custom Functions (tools.js)

Define reusable functions in Tools → JavaScript Editor:

```javascript
// Example: strip domain from URL
exports.getDomain = function(url) {
  return url.replace(/^https?:\/\/(www\.)?/, '').split('/')[0];
};
```

Use in templates:
```
[% tools.js.getDomain(p1.serp.0.link) %]
```

---

## Advanced Examples

### Output all array fields (for debugging)
Call `.format()` without arguments to dump all field names and values:
```
$serp.format()
```
Output shows: `amp: 0`, `anchor: ...`, `link: ...`, `snippet: ...` for each element.

### Iterate key-value pairs of each array element
```
[% FOREACH el = serp -%]
[% FOREACH el -%]
$key: $value
[% END -%]
[% END %]
```

### Output with position numbers
```
[% i = 1 -%]
[% FOREACH el IN serp -%]
$query: [% i %] - $el.link
[% i = i + 1 -%]
[% END %]
```

### Only first N elements of array
```
[% FOREACH el IN serp.slice(0,4) %]$el.link
[% END %]
```

### Only even-indexed elements
```
[% i = 0; FOREACH el IN serp; i = i + 1; NEXT IF i % 2 != 0 -%]
[% i %] - $el.link
[% END %]
```

### Conditional based on result count
```
[% IF p1.totalcount > 1000000 %]<b>$query: $p1.totalcount</b>[% ELSE %]$query: $p1.totalcount[% END %]
```

### Source query alongside each result
```
$serp.format('$query: $link\n')
```

### All SERP links on separate lines (SE::Google)
```
$serp.format('$link\n')
```

### Snippets with HTML stripped
Use Results Builder with "Remove HTML" transform on the `snippet` field, then:
```
$serp.format('$link\t$snippet\n')
```

### WordStat CSV with date
```
"$query",$p1.wordstat_count,$p1.update_date\n
```

### Domain expiry and NS servers (Net::Whois)
```
$query: $p1.expire_date [% FOREACH ns IN p1.ns %]$ns.server [% END %]\n
```

### JSON dump of full result
```
$results.json
```

---

## Global Macros

Define global variables and macros in Settings → Additional Settings.

The default macro defines `$datefile`:
```
[%- USE date; SET datefile = date -%]
```

**Custom global variable example** (for shared cookies across parsers):
```
[%- SET instagram_cookie = 'sessionid=abc123; csrftoken=xyz' -%]
```
Then use `$instagram_cookie` in any parser's settings fields.

**Syntax note:** Use `-%]` closing tag to suppress the trailing newline, otherwise an extra blank line appears at the start of every result.

---

## Template Testing Tool

Go to **Tools → Template Testing** to test result format templates interactively without running a full task. Enter your template and sample data, see the rendered output immediately.
