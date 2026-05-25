# A-Parser Skill for Claude Code

A Claude Code skill that turns Claude into an expert for [A-Parser](https://a-parser.com) — a professional multi-threaded web scraper.

## Installation

```bash
npx skills add https://github.com/vcb019/aparser-skill
```

## What the skill covers

- **JS/TS parser development** — writes complete, ready-to-run TypeScript parsers using the v2 API
- **Full JS API v2** — HTTP requests, sessions, proxy management, cookies, mutex, bulk queries, calling built-in parsers
- **Puppeteer/Chrome** — multi-threaded browser automation, per-tab proxy, screenshots, request interception
- **Task configuration** — Task Editor, Query Builder, Results Builder, filters, deduplication, multi-level parsing
- **Template Toolkit** — all `$variables`, `$tools.*` methods, array formatting, SQLite, CSV, JSON output
- **HTTP API** — all API methods with Node.js examples for external integration
- **150+ built-in parsers** — complete result fields and settings for every parser
- **Proxy & captcha** — proxy checkers, ReCaptcha2/hCaptcha/Turnstile integration
- **Scheduler, debugger, config** — all A-Parser settings and tools

## Test Prompts

### 1. Basic JS parser
```
Write a TypeScript parser for A-Parser that takes a URL and returns:
page title, meta description, and all links (href + anchor text). Use API v2.
```

### 2. Parser with login and sessions
```
Create an A-Parser parser that logs into a site via POST form
(login and password configurable via the UI), saves the session using sessionManager,
and for each input URL returns the H1 heading.
```

### 3. Puppeteer screenshot parser
```
Write an A-Parser TypeScript parser that takes screenshots of websites using Puppeteer/Chrome.
Support proxies, multi-threading (one tab per thread), viewport size configurable via the UI.
```

### 4. Calling a built-in parser from JS
```
How do I call SE::Google from an A-Parser JS parser and get the list of SERP links?
Show a complete code example.
```

### 5. Task configuration
```
How do I configure an A-Parser task to take a list of URLs from a file,
run HTML::LinkExtractor on each URL, keep only links containing "product",
and output as TSV: source URL + found link?
```

### 6. Template Toolkit result format
```
How do I write a result format in A-Parser to output CSV with a header row,
where column 1 is the query, column 2 is Google result count, column 3 is the first SERP link?
```

### 7. HTTP API integration
```
How do I use the A-Parser HTTP API to add a task with queries from text
and track its completion? Show a Node.js example.
```

### 8. CloudFlare bypass
```
How do I write an A-Parser JS parser that bypasses CloudFlare protection?
```

### 9. Bulk query mode
```
Write an A-Parser parser that uses bulk query mode to check Yandex Direct keyword frequency
for 10 keywords at once and returns monthly search volume for each.
```

### 10. Multi-parser domain analysis
```
How do I create an A-Parser task that checks Ahrefs DR, Moz DA, and WHOIS expiry
simultaneously for a list of domains?
```

## Skill Structure

```
aparser-skill/
├── SKILL.md                        — main skill file (overview + JS API + built-in parsers list)
├── README.md                       — this file
├── evals/evals.json                — test cases
└── references/
    ├── api-v2.md                   — complete JS API v2 reference
    ├── puppeteer.md                — Puppeteer/Chrome integration guide
    ├── ui-workflow.md              — UI: tasks, filters, proxy, config, scheduler
    ├── template-toolkit.md         — Template Toolkit: $tools.*, syntax, examples
    ├── http-api.md                 — HTTP API: all methods, Node.js examples
    └── built-in-parsers.md         — all 150+ parsers with result fields and settings
```

## License

MIT
