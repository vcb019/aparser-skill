# A-Parser JS API v2 — Full Reference

## this.request() — all options

```typescript
const { success, data, headers } = await this.request(method, url, queryParams, opts);
```

**Return value:**
- `success` — `1` if request succeeded, `0` otherwise
- `data` — response body as `string` (or `Buffer` if `data_as_buffer: 1`)
- `headers` — response headers object, includes `headers.Status` (HTTP code)

### opts reference

| Option | Default | Description |
|---|---|---|
| `check_content` | — | Array of conditions for successful response. If check fails → retry with new proxy. Conditions: string (substring match), RegExp, or `(data, hdr) => bool`. Wrap in inner array for negation: `[/pattern/]` = must NOT match |
| `decode` | — | `'auto-html'` = auto-detect encoding → UTF-8 (recommended). `'utf8'` = force UTF-8. Or any encoding name |
| `headers` | — | Object with request headers (lowercase keys). E.g. `{ 'user-agent': '...', cookie: '...' }` |
| `headers_order` | — | Array of header names to reorder |
| `browser` | `0` | `1` = emulate browser headers automatically |
| `use_proxy` | (from conf) | Override proxy usage for this request: `1` = on, `0` = off |
| `recurse` | `7` | Max redirect follow count. `0` = disable redirects |
| `redirect_filter` | — | `(hdr) => 1\|0` — filter function for redirect following |
| `follow_common_redirects` | `0` | Follow standard redirects (http→https, www→non-www) regardless of `recurse` |
| `follow_meta_refresh` | `0` | Follow `<meta http-equiv="refresh">` redirects |
| `onlyheaders` | `0` | `1` = fetch headers only, skip body |
| `proxyretries` | (from conf) | Override retry count for this request |
| `parsecodes` | (from conf) | HTTP codes to treat as success: `{ 200: 1, 403: 1 }`. Use `{ '*': 1 }` for all |
| `timeout` | (from conf) | Override timeout in seconds |
| `do_gzip` | `1` | `0` = disable gzip/deflate/br compression |
| `max_size` | (from conf) | Max response size in bytes |
| `cookie_jar` | — | Provide cookies as jar object |
| `attempt` | — | Set current attempt number; disables built-in retry handler |
| `save_to_file` | — | Path to save response directly to disk (skips data/check_content) |
| `data_as_buffer` | `0` | `1` = return `data` as `Buffer` instead of `string` |
| `body` | — | POST body as string or Buffer |
| `bypass_cloudflare` | `0` | `1` = bypass CF JS challenge via Chrome |
| `http2` | `0` | `1` = use HTTP/2 |
| `randomize_tls_fingerprint` | `0` | `1` = randomize TLS fingerprint to avoid bans |
| `tlsOpts` | — | Object with TLS connection settings |
| `noextraquery` | `0` | `1` = don't append Extra query string to URL |

### check_content examples

```typescript
// Positive conditions (all must match):
check_content: [
  '</html>',                              // must contain this string
  /<\/body>/,                             // must match this regex
  (data, hdr) => hdr.Status == 200,      // custom function must return true
]

// Negation (wrap in array):
check_content: [
  'expected text',       // must be present
  [/captcha/i],          // must NOT match (negated)
]
```

### POST examples

```typescript
// Form data (auto-converted to application/x-www-form-urlencoded):
const { success, data } = await this.request('POST', url, { key: 'value', id: 123 });

// JSON body:
const { success, data } = await this.request('POST', url, {}, {
  headers: { 'content-type': 'application/json' },
  body: JSON.stringify({ key: 'value' }),
});

// File upload with form-data:
const FormData = require('form-data');
const form = new FormData();
form.append('file', fs.readFileSync('path/to/file'), 'filename.ext');
const { success, data } = await this.request('POST', url, {}, {
  headers: form.getHeaders(),
  body: form.getBuffer(),
});
```

---

## defaultConf — all fields

```typescript
static defaultConf: typeof BaseParser.defaultConf = {
  version: '0.0.1',           // Major.Minor.Revision (Revision auto-increments on save)
  results: {
    flat: [
      ['fieldName', 'Label'], // scalar result field
    ],
    arrays: {
      arrayName: ['Array Label', [
        ['col1', 'Col1 Label'],
        ['col2', 'Col2 Label'],
      ]],
    }
  },
  results_format: "$fieldName\n$arrayName.format('$col1: $col2\\n')\n",
  // Optional settings (all have defaults):
  timeout: 60,                // seconds
  useproxy: 1,                // 0 or 1
  max_size: 1048576,          // bytes
  proxyretries: 10,
  requestdelay: 0,            // seconds, or '10,30' for random range
  proxybannedcleanup: 600,    // seconds
  pagecount: 1,
  parsecodes: { 200: 1 },
  queryformat: '$query',
  bulkQueries: 10,            // enable bulk mode: N queries per iteration
  // Custom fields (editable via editableConf):
  myCustomField: 'default value',
};
```

---

## sessionManager — full API

```typescript
// In init():
await this.sessionManager.init({
  name: 'JS::MyParser',          // optional: override parser name for session storage
  waitForSession: false,          // if true: .get()/.reset() wait for session to appear
  domain: '.site.com',           // optional: scope sessions to domain
  sessionsKey: 'custom_key',     // optional: manual storage key
  expire: 60,                    // optional: session TTL in minutes
});

// In parse():
let sessionData = await this.sessionManager.get({
  waitTimeout: 10,               // wait up to N minutes for session
  tag: 'my-tag',                 // get session by tag
});

// Reset (on failure):
sessionData = await this.sessionManager.reset({ waitTimeout: 5, tag: 'my-tag' });

// Save (on success):
await this.sessionManager.save(
  { any: 'data' },               // arbitrary data stored in session
  { multiply: 3, tag: 'my-tag' } // multiply: create N copies of this session
);

// Utilities:
let count = await this.sessionManager.count();
let removed = await this.sessionManager.removeById(this.sessionId);
```

**Complete session pattern:**
```typescript
async init() {
  await this.sessionManager.init({ expire: 30 });
}

async parse(set, results) {
  let ses = await this.sessionManager.get();
  
  for (let attempt = 1; attempt <= this.conf.proxyretries; attempt++) {
    const { success, data } = await this.request('GET', set.query, {}, { attempt });
    if (success) {
      // process data
      results.success = 1;
      await this.sessionManager.save(null, { multiply: 2 });
      break;
    } else if (attempt < this.conf.proxyretries) {
      await this.sessionManager.removeById(this.sessionId);
      ses = await this.sessionManager.reset();
    }
  }
  return results;
}
```

---

## Proxy API

```typescript
await this.proxy.next();                            // switch proxy, old one discarded
await this.proxy.ban();                             // switch + ban (for rate-limiting)
const proxyStr = await this.proxy.get();            // get current proxy string
await this.proxy.set('http://user:pass@host:port'); // set specific proxy
await this.proxy.set('http://host:port', true);     // set, don't change between retries
```

---

## Cookies API

```typescript
// Get all cookies as jar object:
const jar = await this.cookies.getAll();

// Set from jar:
await this.cookies.setAll({
  "version": 1,
  ".domain.com": {
    "/": {
      "cookieName": { "value": "cookieValue" }
    }
  }
});

// Set single cookie:
// Use '.domain.com' (with dot) for subdomain scope
// Use 'domain.com' (no dot) for host-only cookie
await this.cookies.set('.domain.com', '/', 'name', 'value');
```

---

## Captcha API

```typescript
// Image captcha (AntiGate):
const { answer, id, error } = await this.captcha.recognize(
  'AntiGate_preset_name',  // preset name configured in A-Parser
  imageBuffer,             // Buffer with image data
  'png'                    // 'jpeg', 'gif', or 'png'
);

// Load captcha from URL:
const { answer, id } = await this.captcha.recognizeFromUrl(
  'AntiGate_preset_name',
  'https://site.com/captcha.jpg'
);

// Report bad solve:
await this.captcha.reportBad('AntiGate_preset_name', id);

// ReCaptcha2 via built-in parser:
const r = await this.parser.request('Util::ReCaptcha2', 'default', {},
  siteKey + '|' + pageUrl
);
const token = r.token; // use as g-recaptcha-response

// hCaptcha:
const r = await this.parser.request('Util::hCaptcha', 'default', {},
  siteKey + '|' + pageUrl
);

// Turnstile (Cloudflare):
const r = await this.parser.request('Util::Turnstile', 'default', {},
  siteKey + '|' + pageUrl
);
```

---

## Calling other parsers — full API

```typescript
const response = await this.parser.request(
  'SE::Google',        // 'Namespace::Parser::SubParser'
  'default',           // preset name
  {
    // overrideParams:
    resultArraysWithObjects: 1,    // arrays as [{col1, col2}, ...] not flat
    needData: 1,                   // include data/pages[] in response
    needResults: ['serp'],         // only return these result fields
    skipProxySettingsInheritance: 0, // 1 = don't inherit proxy settings
    pagecount: 2,                  // any parser conf field can be overridden
  },
  set.query             // or array for bulk: ['query1', 'query2', ...]
);

// Response structure:
response.success      // 1 or 0
response.serp         // array field (if resultArraysWithObjects: 1 → array of objects)
response.totalcount   // flat field
response.bulkResults  // for bulk queries: array of per-query results
```

**Parser name format:** `Category::Name` or `Category::Name::SubName`
Examples: `SE::Google`, `SE::Yandex`, `HTML::LinkExtractor`, `Util::AntiGate`,
`Util::ReCaptcha2`, `JS::Custom::MyParser`

---

## Helper utilities — full list

```typescript
// Results helpers:
await this.utils.updateResultsData(results, htmlData); // fill $data/$pages.$i.data
results.myArray.addElement({ col1: 'v1', col2: 'v2' }); // type-safe array push

// URL utilities:
await this.utils.urlFromHTML(url, baseUrl);            // decode HTML entities, resolve relative
await this.utils.url.extractDomain(url, removeWww?);  // 'sub.domain.com' → 'sub.domain.com'
await this.utils.url.extractTopDomain(url);            // 'sub.domain.com' → 'domain.com'
await this.utils.url.extractTopDomainByZone(url);      // works with all regional TLDs
await this.utils.url.extractWOParams(url);             // strip ?query=string
await this.utils.url.extractMaxPath(url);              // extract URL from string

// String utilities:
await this.utils.removeHtml(str);       // strip HTML tags
await this.utils.removeNoDigit(str);    // keep only digits
await this.utils.removeComma(str);      // remove . , \r \n

// Block extraction (for parsing HTML without regex):
const blocks = await this.utils.getAllBlocks(html, /<div[^>]*class="item"/);
const blocks = await this.utils.getAllBlocksByAttr(html, 'div', 'class', /item/);
// Returns array of full HTML blocks with matched opening tags and their closing tags

// Thread sync:
await this.mutex.lock();
// critical section
await this.mutex.unlock();

// Control flow:
await this.sleep(2.5);           // sleep 2.5 seconds
this.isContextAlive();           // false when task is stopping
this.threadId();                 // current thread number (0-based)

// Template Toolkit in JS:
let tmpl = await tools.createTemplate("Hello [% name %]!");
if (typeof tmpl == 'function') tmpl = await tmpl({ name: 'World' });
this.logger.put(tmpl); // "Hello World!"

// Global tools object (same as $tools.* in Template Toolkit):
await tools.ua.random();         // random User-Agent string
// tools.query is NOT available here — use this.query instead
```

---

## Bulk queries pattern

```typescript
static defaultConf = {
  // ...
  bulkQueries: 10,  // take 10 queries at once
};

async parse(set, results) {
  // set.bulkQueries = array of {query, lvl, num, first, prev, orig, queryUid}
  const queries = set.bulkQueries.map(el => el.query);

  const { success, bulkResults } = await this.parser.request(
    'SE::Yandex::Direct::Frequency', 'default', {}, queries
  );

  if (success) {
    for (let i = 0; i < set.bulkQueries.length; i++) {
      results.bulkResults[i].views = bulkResults[i].views;
      results.bulkResults[i].success = bulkResults[i].success;
    }
  }
  return results;
}
```

---

## Common parser patterns

### Pagination (crawling multiple pages)
```typescript
async parse(set, results) {
  for (let page = 1; page <= this.conf.pagecount; page++) {
    const { success, data } = await this.request('GET', set.query + '?page=' + page);
    if (!success) break;
    // extract and push to results.items
    if (!hasNextPage(data)) break;
    if (page < this.conf.pagecount) await this.sleep(1);
  }
  results.success = 1;
  return results;
}
```

### Adding discovered URLs to queue
```typescript
async afterResultsProcessor(results) {
  for (const link of results.links) {
    await this.query.add({ query: link, lvl: 1 });
  }
}
```

### File download to disk
```typescript
const filename = 'files/downloads/' + Date.now() + '.pdf';
await this.request('GET', fileUrl, {}, { save_to_file: filename });
results.file = filename;
results.success = 1;
```

### JSON API
```typescript
const { success, data } = await this.request('GET', apiUrl, { param: 'value' }, {
  headers: { accept: 'application/json' },
  parsecodes: { 200: 1 },
});
if (success) {
  const json = JSON.parse(data);
  results.title = json.title;
  results.success = 1;
}
```

### Login + session
```typescript
async login() {
  const { success, data } = await this.request('POST', 'https://site.com/login', {
    username: this.conf.login,
    password: this.conf.password,
  });
  return success && data.includes('logout');
}

async parse(set, results) {
  let ses = await this.sessionManager.get();
  if (!ses) {
    const ok = await this.login();
    if (!ok) return results;
  }
  // make authenticated request
  const { success, data } = await this.request('GET', set.query);
  // ...
  if (success) await this.sessionManager.save();
  else await this.sessionManager.reset();
  return results;
}
```
