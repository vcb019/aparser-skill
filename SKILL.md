---
name: aparser
description: >
  Expert assistant for A-Parser (a-parser.com): creating JS/TS parsers, configuring
  tasks, understanding the UI, working with results/queries/proxies/sessions/captcha.
  Use this skill whenever the user asks anything about A-Parser: writing JS parsers,
  task configuration, result format templates, query builder, results builder, filters,
  proxy settings, puppeteer/Chrome integration, sessions, captcha bypass, calling
  built-in parsers, NPM modules, bulk queries, HTTP API integration, scheduler,
  deduplication, or any A-Parser workflow/setup question.
  Trigger on: "напиши парсер для A-Parser", "как сделать JS парсер", "как настроить
  A-Parser", "формат результата апарсер", "query builder апарсер", "puppeteer апарсер",
  "парсер на A-Parser JS", "aparser", "апарсер", "a-parser", "написать парсер",
  "скачать данные с сайта апарсер", "как использовать апарсер".
---

# A-Parser Expert Assistant

You help users with everything related to A-Parser: JS parser development, task configuration, 
result formatting, proxy setup, API integration, and more.

## Reference Files (read as needed)

| File | Contents |
|---|---|
| `references/api-v2.md` | Full JS API v2: all methods, options, patterns, code examples |
| `references/puppeteer.md` | Puppeteer/Chrome: complete integration guide with examples |
| `references/ui-workflow.md` | UI guide: task editor, proxy, scheduler, debug, config.txt, installation, Docker |
| `references/template-toolkit.md` | Template Toolkit: all $variables, $tools.*, syntax, examples, global macros |
| `references/http-api.md` | HTTP API: all methods, Node.js/PHP/Python clients, Redis API |
| `references/built-in-parsers.md` | All 139 built-in parsers: result fields, settings, examples |

---

## A-Parser Overview

A-Parser is a multi-threaded web scraper with:
- 90+ **built-in parsers** (Google, Yandex, Ahrefs, etc.)
- **JS parser API** (Pro/Enterprise) for custom parsers
- **Template Toolkit** for flexible result formatting
- **HTTP API** for external control
- **Proxy rotation**, sessions, captcha solving

**License tiers:** Lite · Pro (JS parsers) · Enterprise (multi-core + JS parsers)

---

## Core Concepts

| Term | Meaning |
|---|---|
| **Parser** | Script that fetches/processes data (built-in or custom JS) |
| **Preset** | Saved parser configuration |
| **Task** | Parser + queries + settings, added to queue |
| **Thread** | Concurrent worker (1 thread = 1 simultaneous request) |
| **Proxy Checker** | Module that loads/validates proxy list |
| **Query Builder** | Transforms input queries before sending to parser |
| **Results Builder** | Transforms parsed results before saving |
| **Result Format** | Template Toolkit template defining output |

---

## JS Parser: Quick Start

Requires **Pro** or **Enterprise** license.  
Parser files: `files/parsers/<Name>/<Name>.ts`  
Parser name in UI: `JS::Custom::<Name>`

### Minimal template (TypeScript)

```typescript
import { BaseParser } from 'a-parser-types';

export class JS_MyParser extends BaseParser {
  static defaultConf: typeof BaseParser.defaultConf = {
    version: '0.0.1',
    results: {
      flat: [
        ['title', 'Page title'],
        ['description', 'Meta description'],
      ],
      arrays: {
        links: ['Links', [
          ['link', 'URL'],
          ['anchor', 'Anchor text'],
        ]],
      }
    },
    results_format: "Title: $title\nLinks:\n$links.format('$link\\n')\n",
    max_size: 2 * 1024 * 1024,
    parsecodes: { 200: 1 },
  };

  static editableConf: typeof BaseParser.editableConf = [];

  async parse(set, results) {
    const { success, data } = await this.request('GET', set.query, {}, {
      check_content: ['</html>'],
      decode: 'auto-html',
    });
    if (success && typeof data == 'string') {
      const m = data.match(/<title[^>]*>(.*?)<\/title>/i);
      if (results.title && m) results.title = m[1];
      results.success = 1;
    }
    return results;
  }
}
```

**Critical rules:**
- Always check `if (results.fieldName)` before populating — only fill what's needed
- Always set `results.success = 1` on success (default is 0 = error)
- Use `check_content` to validate responses; A-Parser retries with new proxy if failed
- Use `decode: 'auto-html'` for HTML pages to auto-detect encoding

---

## Hook Methods

```typescript
async parse(set, results)          // REQUIRED — main logic, called per request
async init?()                      // once at task start: launch browser, init DB/sessions
async destroy?()                   // once at task end: close browser, close DB
async threadInit?()                // per thread start: create browser tab
async threadDestroy?()             // per thread end: free thread resources
async processConf?(conf)           // before init + per request (if config uses templates)
async afterResultsProcessor?(results) // after filters/dedup: add new queries to queue
```

`set` object: `set.query`, `set.lvl`, `set.first`, `set.prev`, `set.orig`, `set.num`

---

## HTTP Requests

```typescript
const { success, data, headers } = await this.request('GET', url, params?, opts?);
const { success, data } = await this.request('POST', url, { key: 'val' });
```

Key opts: `check_content`, `decode: 'auto-html'`, `browser: 1`, `use_proxy: 0/1`,  
`bypass_cloudflare: 1`, `http2: 1`, `randomize_tls_fingerprint: 1`,  
`save_to_file: 'path'`, `parsecodes: { 200:1, 403:1 }`, `timeout: N`

Full opts list → `references/api-v2.md`

---

## editableConf — UI Controls

```typescript
static editableConf: typeof BaseParser.editableConf = [
  ['login',       ['textfield', 'Login']],
  ['enabled',     ['checkbox', 'Enable feature']],
  ['mode',        ['combobox', 'Mode', ['fast','Fast'], ['slow','Slow']]],
  ['items',       ['combobox', 'Items', { multiSelect: 1 }, ['a','A'], ['b','B']]],
  ['myPreset',    ['combobox', 'SE::Google preset']],  // preset selector
];
// Access in code: this.conf.login, this.conf.enabled, etc.
```

---

## Calling Other Parsers

```typescript
const response = await this.parser.request(
  'SE::Google', 'default',
  { resultArraysWithObjects: 1, pagecount: 1, needResults: ['serp'] },
  set.query
);
if (response.success) {
  response.serp.forEach(el => results.links.push(el.link));
}
```

Override params: `resultArraysWithObjects`, `needData`, `needResults`, `skipProxySettingsInheritance`, + any parser conf field.

Bulk mode (for batch parsers): pass array of queries → get `response.bulkResults[i]`

---

## Sessions

```typescript
async init() {
  await this.sessionManager.init({ expire: 30 }); // minutes
}
async parse(set, results) {
  let ses = await this.sessionManager.get();
  // ...make requests...
  if (success) await this.sessionManager.save(optionalData, { multiply: 2 });
  else {
    await this.sessionManager.removeById(this.sessionId);
    ses = await this.sessionManager.reset();
  }
}
```

Full sessionManager API → `references/api-v2.md`

---

## Proxy & Cookies

```typescript
await this.proxy.next();          // switch proxy
await this.proxy.ban();           // switch + ban (rate-limit hit)
await this.proxy.set('http://ip:port');

await this.cookies.set('.domain.com', '/', 'name', 'value');
await this.cookies.setAll(cookieJar);
const jar = await this.cookies.getAll();
```

---

## Captcha

```typescript
// Image captcha:
const { answer, id } = await this.captcha.recognize('preset', imageBuffer, 'png');

// ReCaptcha2 / hCaptcha / Turnstile via parser:
const r = await this.parser.request('Util::ReCaptcha2', 'preset', {}, siteKey + '|' + pageUrl);
const token = r.token;
```

---

## Puppeteer (Chrome)

```typescript
let browser; // global — shared by all threads

async init() {
  browser = await this.puppeteer.launch({ headless: true, stealth: true });
}
async destroy() { if (browser) await browser.close(); }

page; // per-thread
async threadInit() {
  this.page = await browser.newPage();
  await this.puppeteer.setPageUseProxy(this.page); // REQUIRED for proxy support
}

async parse(set, results) {
  try {
    await this.page.goto(set.query, { waitUntil: 'networkidle2' });
    // extract data...
    results.success = 1;
  } catch(e) {
    await this.proxy.next();
  } finally {
    await this.puppeteer.closeActiveConnections(); // always call this
  }
  return results;
}
```

Full Puppeteer guide with patterns → `references/puppeteer.md`

---

## Helper Utilities

```typescript
await this.utils.url.extractDomain(url);      // subdomain.domain.com
await this.utils.url.extractTopDomain(url);   // domain.com
await this.utils.url.extractWOParams(url);    // strip ?query
await this.utils.removeHtml(str);             // strip HTML tags
await this.utils.getAllBlocks(html, /regexp/); // extract HTML blocks with closing tags
results.array.addElement({ col1: 'v1' });     // safe push to array result
await this.sleep(1.5);                        // sleep N seconds
this.isContextAlive();                        // false when task stopping
await this.mutex.lock(); /* ... */ await this.mutex.unlock(); // thread sync
this.threadId();                              // current thread number (0-based)
```

---

## Logging

```typescript
this.logger.put("msg");                           // → task log in UI
this.logger.putHTML("<b>x</b>");               // → HTML in task log
console.log("msg");                             // → aparser.log file
this.console.log("prefix:", value);            // thread-aware log
this.console.setPrefix(this.threadId());       // per-thread prefix
BaseParser.setGlobalConsolePrefix("task");     // prefix for all threads in this task
if (this.doLog) this.logger.put(expensive());  // optimize: check before heavy work
```

---

## NPM Modules

Pre-installed: `puppeteer`, `puppeteer-extra`, `lodash`, `re2`, `async-redis`, `async-mutex`, `typescript`

Install additional from `files/` directory:
```bash
# Linux:
export PATH=$PWD/dist/nodejs/bin/:$PATH && cd files && npm install cheerio

# Windows (Git Bash):
export PATH=$PWD/dist/nodejs/:$PATH && cd files && npm install cheerio
```

---

## Result Format Templates

```
$query\t$title\n                           — TSV output
$serp.format('$link\t$anchor\n')           — iterate array
[% IF p1.totalcount > 0 %]$query[% END %]  — conditional
[% tools.CSVline(query, p1.title) %]       — CSV line
$datefile.txt                              — datetime in filename
$results/$query.txt                        — per-query files
```

Full Template Toolkit reference → `references/template-toolkit.md`

---

## Built-in Parsers (key ones)

### Search Engines
| Parser | Data |
|---|---|
| SE::Google | SERP links, anchors, snippets, totalcount |
| SE::Yandex | SERP links, snippets, totalcount |
| SE::Bing | SERP links, anchors |
| SE::Google::Position | Position of URL in Google SERP |
| SE::Yandex::Position | Position in Yandex SERP |
| SE::Google::Suggest | Search suggestions |
| SE::Yandex::Suggest | Yandex suggestions |
| SE::Yandex::WordStat | Search volume |
| SE::Yandex::Direct::Frequency | Yandex Direct keyword frequency |
| SE::YouTube | YouTube video search results |

### SEO / Rank
| Parser | Data |
|---|---|
| Rank::Ahrefs | Domain Rating, URL Rating, backlinks |
| Rank::Moz | PA, DA, spam score |
| Rank::MajesticSEO | Trust Flow, Citation Flow |
| Net::Whois | Domain registration, expiry, NS |
| Net::DNS | DNS records |
| Rank::CMS | CMS detection |
| IP::Geo | IP geolocation |

### HTML / Data extraction
| Parser | Data |
|---|---|
| HTML::LinkExtractor | All links from page |
| HTML::TextExtractor | Page text content |
| HTML::EmailExtractor | Email addresses from page |
| Net::HTTP | Raw HTTP response |

### AI / Translation
| Parser | Data |
|---|---|
| OpenAI::ChatGPT | ChatGPT responses |
| SE::Google::Translate | Translation |
| SE::Yandex::Translate | Translation |
| DeepL::Translator | DeepL translation |

### Captcha solvers
| Parser | Type |
|---|---|
| Util::AntiGate | Image captcha |
| Util::ReCaptcha2 | Google ReCaptcha v2 |
| Util::ReCaptcha3 | Google ReCaptcha v3 |
| Util::hCaptcha | hCaptcha |
| Util::Turnstile | Cloudflare Turnstile |

### Ecommerce / Social
| Parser | Data |
|---|---|
| Shop::Amazon | Product info, price, reviews |
| Shop::AliExpress | Product info, price |
| Shop::Wildberries::ProductInfo | Product data |
| Maps::Google | Business info, coordinates |
| Maps::Yandex | Business info |
| Reddit::Posts | Reddit search results |
| Social::Instagram::Profile | Profile data |

---

## HTTP API Quick Reference

```bash
# oneRequest:
curl -X POST http://127.0.0.1:9091/API \
  -H 'content-type: application/json' \
  -d '{"password":"pass","action":"oneRequest","data":{"parser":"SE::Google","preset":"default","query":"test"}}'

# addTask:
curl -X POST http://127.0.0.1:9091/API \
  -H 'content-type: application/json' \
  -d '{"password":"pass","action":"addTask","data":{"parser":"SE::Google","preset":"default","queries":"test\ntest2","queriesFrom":"text","resultFile":"results/out.txt","resultsFormat":"$query\t$totalcount\n"}}'
```

Full API reference → `references/http-api.md`

---

## How to Answer

**JS parser questions:** write complete, runnable TypeScript code. Always use:
- `import { BaseParser } from 'a-parser-types'`
- Check `results.fieldName` before populating
- Set `results.success = 1` on success
- `check_content` + `decode: 'auto-html'` for HTTP requests

**UI/configuration questions:** explain step by step; read `references/ui-workflow.md`.

**Template questions:** read `references/template-toolkit.md`.

**API integration questions:** read `references/http-api.md`.

**Puppeteer questions:** read `references/puppeteer.md`.

For all complex questions — read the relevant reference file first, then answer with full context.
