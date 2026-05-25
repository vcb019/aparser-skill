# A-Parser UI & Workflow Reference

## Interface Overview

A-Parser has three main areas: left navigation menu, bottom status bar, right workspace.

### Navigation tabs
| Tab | Purpose |
|---|---|
| **Home & News** | Updates, recipes, video tutorials, forum messages |
| **Quick Task** | Run simple parsing without full editor |
| **Task Editor** | Full configuration: parsers, queries, filters, format, presets |
| **Tasks Queue** | Active and completed tasks with stats |
| **Scheduler** | Schedule recurring tasks |
| **Logs Viewer** | Per-thread logging for debugging |
| **Proxy Checker** | Manage and validate proxy lists |
| **Tools** | Template testing, JS editor, regex builder, updates |
| **Settings** | All configuration: general, threads, parsers, proxy, advanced |
| **Parser Test** | Debug individual parsers |

### Status bar
Shows: A-Parser status, active/total tasks, live/total proxies, engaged threads, update notifications.

---

## Creating a Task (Task Editor)

### Step by step
1. **Select parser** — from dropdown (e.g., SE::Google, HTML::LinkExtractor, JS::Custom::MyParser)
2. **Enter queries** — one per line in textarea, or import from file (unlimited)
3. **Configure parser settings** — parser-specific fields appear
4. **Set result format** — Template Toolkit template defining output
5. **Set result file** — filename (supports variables like `$datefile`, `$query`)
6. **Enable "Do Log"** — recommended for debugging
7. **Click "Add task"** — task appears in Tasks Queue

### Multiple parsers in one task
Click "Add parser" to add more parsers. Each gets a number.  
Reference results: `$p1.fieldName`, `$p2.fieldName`, `$p1.serp.format(...)`

**Benefits:** Increases speed by spreading proxy usage across services, reduces bans.

**Example use cases:**
- Check multiple SEO metrics (Ahrefs + Moz + Majestic) in one pass
- Collect keywords from multiple sources simultaneously
- Combine SERP results + domain metrics

---

## Query Format & Variables

### Query format field (transforms the query before sending)
```
$query         — query as formatted
$query.orig    — original query before any formatting
$query.num     — query sequence number (1-based)
$query.lvl     — nesting level (0 = from input file)
$query.first   — first query in the chain (for multi-level)
$query.prev    — previous level's query
```

### Query macros (substitutions in query list)
```
{num:1:100}         — numbers 1 to 100
{num:1:100:2}       — numbers 1, 3, 5... (step 2)
{num:100:1}         — reverse: 100, 99, 98...
{az:a:z}            — letters a through z
{each:cat,dog,bird} — iterate over comma list
{subs:filename}     — load from queries/subs/filename.txt
```
Macros multiply: `site.com/{num:1:10}` generates 10 queries.

---

## Result Format (Template Toolkit)

### Basic output
```
$query                          — current query
$query.orig                     — original query
$fieldName                      — any flat result field
$arrayName.N.colName            — N-th array element, specific column
$arrayName.format('$col1\t$col2\n')  — iterate array with template
```

### Array formatting examples
```
# All SERP links, one per line:
$serp.format('$link\n')

# Links with anchors tab-separated:
$serp.format('$link\t$anchor\n')

# All H2 headers:
$h2.format('$header\n')

# Custom separator:
$ns.format('$ns ')
```

### Template Toolkit syntax
```
[% IF condition %]...[% ELSE %]...[% END %]
[% FOREACH item IN array %][% item.field %][% END %]
[% query | upper %]            — uppercase
[% value | html %]             — HTML escape
[% value | replace('a','b') %] — replace
[% value | remove('\s+') %]    — remove pattern
```

### File naming with variables
```
results/$query.txt             — one file per query
$datefile.txt                  — timestamp: 2024-01-15_14-30-00
reports/$queriesfile/$query.txt — folder per queries file + per query
```

### Output formats
- **Text** (default): `$query\t$title\n`
- **CSV**: use `[% tools.CSVline(query, p1.field1, p2.field2) %]`
- **JSON**: `[% data.json %]` or `[% tools.parseJSON(data) %]`
- **Binary/files**: use `save_to_file` in JS parser

### Prepend/Append text
Add headers (CSV) or wrappers (XML/HTML) in "More Options" → Prepend/Append.

---

## Query Builder

Transforms input queries **before** sending to parser.

### Available operations
| Operation | Description | Example |
|---|---|---|
| Split | Divide by regex or delimiter | `site.com;keyword` → `site.com` |
| Replace | Substitution (text or regex) | Add `!` before each word |
| Extract domain | Get domain from URL | `https://site.com/page` → `site.com` |
| Extract top domain | Root domain only | `sub.domain.co.uk` → `domain.co.uk` |
| Add "!" to each word | For Yandex WordStat | `buy laptop` → `!buy !laptop` |
| Uppercase / Lowercase | Case conversion | — |

**$proxy variable:** Create proxy from query — format `http://ip:port`, bypasses standard proxy checker.

---

## Results Builder

Transforms parsed results **after parsing, before saving**.

### Available operations
| Operation | Description |
|---|---|
| Split | Divide field by regex/delimiter |
| Replace | Text or regex substitution |
| Extract domain / top domain | From URL fields |
| Uppercase / Lowercase | Case conversion |
| Remove HTML tags | `<b>text</b>` → `text` |
| HTML entities decode | `&copy;` → `©` |
| XPath extraction | `//*[@id="rso"]//a/@href` |

**Pipeline:** parsing → Results Builder → Result Format → save to disk

---

## Results Filters

Keep only results matching conditions. Multiple filters = logical AND.

### Filter types
- String: equals / not equals
- Contains / not contains (substring)
- Matches / not matches (regex)
- Numeric: greater than / less than / equals

### Filter behavior
- **Array result filtered**: only matching elements remain in array
- **Flat result filtered**: entire query is skipped/excluded

### Examples
```
# Yandex TIC > 10:
$yandex_tic > 10

# URL contains "blog":
$link contains blog

# Exclude pages with captcha:
$data not matches /captcha/i

# Dynamic: query must appear in snippet:
$snippet contains [% query %]
```

---

## Results Deduplication

Remove duplicate results from output.

### Dedup types
| Type | Treats as same |
|---|---|
| String | Exact line match |
| Domain | www.site.com = www.site.com (but ≠ sub.site.com) |
| Top-Level Domain | sub.site.com = other.site.com |
| Second-Level Domain | www.site.com = sub.site.com |
| Path | https://site.com/path/ = https://site.com/path/?param=1 |
| Without Parameters | Ignores URL query string |

### Scope
- **Within task**: clears on restart
- **Cross-task**: create named dedup database → reuse across tasks, only new items pass

### Query deduplication
Send only unique queries to parser — prevents reprocessing duplicates.

---

## Request Processing Order

1. Read queries from file/text (level 0)
2. Apply Query Builder transformations
3. Apply query macros/substitutions (still level 0)
4. Send to parser threads
5. Apply Results Builder transformations
6. Apply Filters
7. Apply Deduplication
8. Format via Result Format template
9. Save to result file
10. **afterResultsProcessor**: add new queries to queue (level +1)
11. Process new queries (repeat from step 2)

---

## Task Settings — Common Parameters

| Setting | Default | Description |
|---|---|---|
| Threads | 1 | Concurrent requests |
| Use proxy | On | Enable proxy rotation |
| Proxy checker | — | Which proxy source to use |
| Request retries | 10 | Attempts before skipping |
| Request timeout | 60s | Max wait per request |
| Request delay | 0 | Delay between requests ("10,30" = random) |
| Proxy ban time | 600s | How long to ban blocked proxy |
| Parse codes | {200:1} | HTTP codes = success |
| Max file size | 1MB | Max response body size |
| Page count | 1 | Pages per query |
| Do Log | Off | Enable task log (enable for debugging!) |

### Settings Override
Add per-task overrides without changing preset:
- Click "Add override" in Task Editor
- Override any parser conf field (timeout, useproxy, pagecount, etc.)

### Additional Options (More Options)
| Option | Description |
|---|---|
| Keep Unique Data | Save dedup data for future tasks |
| Prepend/Append | Header/footer for result file |
| Log Limit | Auto-delete old logs if > N entries |
| Task Priority | Higher priority = gets threads first |
| Run Next Task | Auto-launch another task on completion |
| Callback URL | POST notification on task completion |
| Override tools.js | Custom tools.js for this preset |
| Delete Task on Completion | Auto-remove from queue |
| Stop Task on Error | Halt on first failed request |

---

## Multi-Level Parsing (Parse All Results / Parse to Level)

Use to crawl multi-page sites or follow links:

- **Parse all results**: parse each result as a new query (follow links)
- **Parse to level N**: limit depth of following (0 = original only, 1 = follow once, etc.)
- New queries created have `lvl` = parent `lvl + 1`
- Use `$query.first` to access original query at any depth
- Use `$query.prev` to access parent query

---

## Proxy Configuration

### Proxy formats
```
ip:port
ip:port:login:password
login:password@ip:port
http://ip:port
http://login:password@ip:port
socks://ip:port
socks5://ip:port
socks4://ip:port
```

### Proxy sources (files in `files/proxy/<checker_name>/`)
- `proxy.txt` — direct proxy list
- `sites.txt` — URLs to load proxies from (one per line)
- `alive.txt` — auto-saved working proxies (every 5 sec)
- `regex.txt` — regex patterns for parsing proxy from external URLs

### Proxy Checker settings
| Setting | Default | Description |
|---|---|---|
| Loading threads | 5 | Concurrent check threads |
| Check interval | 30s | Re-check interval for alive proxies |
| Timeout | 5s | Check request timeout |
| Auth type | IP-based | IP access / shared credentials / per-proxy |

### A-Parser own proxies
- **Proxy Unlimited**: `http://work.a-poster.info/prx/perm_socks.txt` (fixed IP) or `rand_socks.txt` (random IP)
- **Proxy Premium**: SOCKS5 with login/password from Member Area

---

## Captcha Configuration

A-Parser uses external captcha-solving services.

### Supported services
RuCaptcha, Anti-Captcha, 2captcha, CapMonster, CapSolver, XEvil, and others.

### Setup
1. Configure captcha parser preset (e.g., Util::AntiGate) with API key
2. In target parser settings → select captcha preset
3. Save as new preset

### Built-in captcha parsers
| Parser | Type |
|---|---|
| Util::AntiGate | Image captchas |
| Util::ReCaptcha2 | Google ReCaptcha v2 |
| Util::ReCaptcha3 | Google ReCaptcha v3 |
| Util::hCaptcha | hCaptcha |
| Util::Turnstile | Cloudflare Turnstile |

---

## Scheduler

Schedule recurring tasks (e.g., daily SERP checks, monitoring).

### Configuration
- Start date and time
- Number of repetitions (unlimited or N times)
- Repeat frequency (every N hours/days)
- **Unique**: prevent duplicate runs if previous task still running

Saved schedules appear in dropdown for quick reuse.

---

## Debugging

### Methods
1. **Do Log**: enable in task settings → view in Logs Viewer during/after task
2. **Task Tester**: test full preset (up to 5 threads, 10 requests) with detailed thread logs
3. **Test Parsing**: test individual parser without constructor/multi-level
4. **Debug mode**: shows full request/response details, page source, regex builder link

### In JS parsers
```typescript
this.logger.put("value: " + someVar);        // → task log (UI)
console.log("debug info");                    // → aparser.log file
this.console.log("thread:", this.threadId()); // thread-aware log
if (this.doLog) this.logger.put(expensive()); // optimize: check before heavy computation
```

---

## JS Parser Integration

### File location
```
files/parsers/
└── MyParser/
    ├── MyParser.ts    ← main file (TypeScript, preferred)
    ├── MyParser.js    ← compiled JS (auto-generated on save)
    └── icon.png       ← optional icon (32x32 px)
```

### Parser name in UI
`JS::Custom::MyParser` — appears in parser dropdown under JS::Custom:: namespace.

### Enable remote JS Editor access
1. Set password: Settings → General Settings
2. Add to `config/config.txt`: `allow_javascript_editor: 1`
3. Restart A-Parser

### Version format
`"Major.Minor.Revision"` — Revision auto-increments on each save.  
Wrap in double quotes `"0.1.0"` to prevent auto-increment.

---

## config/config.txt — Hidden Settings

Create file manually, requires restart.

| Parameter | Default | Description |
|---|---|---|
| `bind` | 0.0.0.0:9091 | IP:port for web interface |
| `outgoing_ip` | 0.0.0.0 | Outbound connection IP |
| `dns` | Google/CF | Custom DNS servers |
| `dns_retries` | 2 | DNS retry attempts |
| `dns_timeout` | 5 | DNS timeout (seconds) |
| `proxies_reuse` | 0 | Reuse same proxy on retry |
| `https` | 1 | HTTPS support |
| `save_interval` | 10 | Progress save frequency (seconds) |
| `allow_javascript_editor` | 0 | Enable JS parser editor |
| `allow_outside_files` | 0 | Allow external file access |
| `allow_dangerous_node_modules` | 0 | Unrestricted Node.js modules |
| `proxies_dns_local` | 0 | DNS resolution method |

---

## Command Line Parameters

```bash
aparser.exe -resetpassword          # Reset web interface password
aparser.exe -stoptasks              # Start with all tasks paused
aparser.exe -foreground             # Run with console output (no fork)
aparser.exe -morelogs               # Verbose logging
aparser.exe -asynchttpx-disable-cert-check  # Disable TLS validation
aparser.exe -nofork                 # Disable multicore result processing
```

Linux: `./aparser -stoptasks`

---

## Tools

| Tool | Location | Purpose |
|---|---|---|
| **Template Testing** | Tools tab | Test result format templates without parsing |
| **JavaScript Editor** | Tools tab | Edit tools.js custom functions |
| **Regex Constructor** | Tools tab | Build regex patterns visually |
| **A-Parser Update** | Tools tab | Update to latest version |
| **Service** | Tools tab | Restart/shutdown, translation editor |
| **Parser Test** | Tools tab | Debug individual parser with test query |

---

## Common Workflows

### Collect SERP + extract metadata
1. Task with SE::Google → get URLs
2. Task with HTML::LinkExtractor or custom JS parser → parse each URL
   Or: use "Parse all results" + multi-parser in one task

### Bulk domain analysis
1. Queries: list of URLs
2. Query Builder: Extract top domain
3. Parsers: Rank::Ahrefs + Rank::Moz + Net::Whois (multiple parsers)
4. Result format: `$query.orig\t$p1.domain_rating\t$p2.pa\n`

### Monitor positions (recurring)
1. Scheduler → weekly/daily
2. Parser: SE::Google::Position
3. Deduplicate: keep new changes only

### Screenshot bulk sites
Puppeteer JS parser → see `references/puppeteer.md`

### Login + scrape authenticated pages
JS parser with sessionManager → see `references/api-v2.md`

---

## Special Parser Options

### Parse All Results
Available for: SE::Google, SE::Yandex, SE::Bing, SE::Yahoo

Bypasses the 1000-result limit per query:
- A-Parser checks total result count for the query
- Automatically adds sub-queries to collect all results
- Enable in parser settings: **Parse all results**

### Parse to Level
Available for: SE::Google::Suggest, SE::Yandex::WordStat, HTML::LinkExtractor

- **Suggest parsers:** Re-feeds parsed keywords as new queries, collecting nested suggestions down to N levels
- **HTML::LinkExtractor:** Recursively follows internal links, building a full site map

**Important:** Always enable unique queries deduplication when using this option, or the parser will loop.

### Parse Related to Level
Available for: SE::Google, SE::Bing, SE::Yahoo

Recursively collects related keywords at each SERP level.

---

## Installation

### Windows
1. Download archive from account → A-Parser → Downloads
2. Extract to any folder
3. Run `aparser.exe` (first start takes 30s–2min)
4. Open `http://127.0.0.1:9091/` — default password is empty, click Login

**Troubleshooting Windows:**
- If A-Parser won't start, disable Windows Search indexing service (can lock files)
- Incompatible programs: Norton, Emsisoft Anti-Malware, Guard Mail.ru, HTTPDebugger
- Update errors: delete or rename the `dist` folder if Windows blocks it

### Linux
```bash
# Download using one-time link from your account:
wget https://a-parser.com/members/onetime/LINK/aparser-linux-x64.tar.gz
tar zxf aparser-linux-x64.tar.gz && rm -f aparser-linux-x64.tar.gz
cd aparser/ && chmod +x aparser && ./aparser
```
Open `http://SERVER_IP:9091/` — default password is empty.

**Increase thread limits (recommended):**
```bash
echo 'root soft nofile 10240' >> /etc/security/limits.conf
echo 'root hard nofile 10240' >> /etc/security/limits.conf
sysctl -w net.ipv4.netfilter.ip_conntrack_max=262144
```
Re-login to SSH after applying, then restart A-Parser.

### macOS
macOS requires Docker. See Docker section below.

### Docker
```bash
# Download and run:
curl -O https://a-parser.com/members/onetime/LINK/aparser-linux-x64.tar.gz
tar zxf aparser-linux-x64.tar.gz && rm -f aparser-linux-x64.tar.gz
docker run --rm --name aparser -v $(pwd)/aparser:/app -p 9091:9091 -t aparser/runtime ./aparser -foreground
```

**docker-compose:**
```yaml
version: '3'
services:
  a-parser:
    image: aparser/runtime:latest
    command: ./aparser
    restart: always
    volumes: [./aparser:/app]
    ports: ["9091:9091"]
    ulimits:
      nofile: { soft: 10240, hard: 10240 }
```
```bash
docker compose up -d
```

**Update via Docker:**
```bash
docker stop aparser
docker run --rm --name aparser -v $(pwd)/aparser:/app -p 9091:9091 -t aparser/runtime ./aparser -foreground -doupdate
```

**Note for macOS (Apple Silicon):** Enable Rosetta emulation in Docker Desktop settings.

### Multiple instances on one machine
Each instance needs a unique port. In `config/config.txt`:
```
bind: 0.0.0.0:9092
```

### File structure
```
aparser/
├── config/         — configuration (backup before updates)
│   ├── config.db   — main settings and presets
│   ├── config.txt  — additional config (manual edits)
│   ├── queue.db    — task queue
│   └── scheduler.db
├── dist/           — A-Parser runtime + Node.js
├── files/
│   ├── parsers/    — JS parser source files
│   └── proxy/      — proxy checker settings
├── logs/           — task execution logs
├── queries/        — query files
├── results/        — result files
├── tmp/
├── aparser.exe     — (or ./aparser on Linux)
└── aparser.log     — main diagnostic log
```

### Password reset
```bash
aparser.exe -resetpassword   # Windows
./aparser -resetpassword      # Linux
```

### Updating
**Via UI:** Tools → Update A-Parser → select files → click Update.

**Manual Linux:**
```bash
killall aparser
# Download new binary from account
chmod +x aparser && ./aparser
```
