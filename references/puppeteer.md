# A-Parser Puppeteer Integration

A-Parser integrates Chrome/Chromium via the `puppeteer` library, with built-in proxy support per tab and multi-threaded tab management.

## Key differences from plain puppeteer

- Use `this.puppeteer.launch()` instead of `puppeteer.launch()` — integrates with A-Parser proxy system
- Must call `await this.puppeteer.setPageUseProxy(page)` after creating each page — enables per-tab proxy
- Must call `await this.puppeteer.closeActiveConnections()` after each request — Chrome keeps connections alive; this controls resource usage
- Use browser as a **global variable** (shared across threads), pages as **per-thread** (`this.page`)
- Recommended: 1-2 tabs per CPU core

## Resource limits

The browser is resource-heavy. Keep thread count low (1-2 per CPU core). Unlike HTTP threads, each browser thread = one Chrome tab.

## Complete working example (screenshot maker)

```typescript
import { BaseParser, PuppeteerTypes } from 'a-parser-types';
const jimp = require('jimp');

let browser: PuppeteerTypes.Browser;

class JS_ScreenshotMaker extends BaseParser {
  static defaultConf: typeof BaseParser.defaultConf = {
    version: '1.0.0',
    results: {
      flat: [
        ['screenshot', 'PNG screenshot'],
      ]
    },
    results_format: '$screenshot',
    load_timeout: 30,
    width: 1024,
    height: 768,
    headless: 1,
    log_screenshots: 0,
  };

  static editableConf: typeof BaseParser.editableConf = [
    ['headless', ['checkbox', 'Chrome Headless']],
    ['width', ['textfield', 'Viewport Width']],
    ['height', ['textfield', 'Viewport Height']],
    ['resize_width', ['textfield', 'Resize to Width (optional)']],
    ['log_screenshots', ['checkbox', 'Log Screenshots']],
  ];

  async init() {
    // Launch browser once — shared by all threads
    browser = await this.puppeteer.launch({
      headless: !!this.conf.headless,
      stealth: true,           // mask as real Chrome
      logConnections: false,
      defaultViewport: {
        width: parseInt(this.conf.width),
        height: parseInt(this.conf.height),
      },
    });
  }

  async destroy() {
    if (browser) await browser.close();
  }

  // Each thread gets its own page
  page: PuppeteerTypes.Page;

  async threadInit() {
    this.page = await browser.newPage();
    await this.page.setCacheEnabled(true);
    await this.page.setDefaultNavigationTimeout(this.conf.timeout * 1000);
    // Critical: bind this page to A-Parser proxy for this thread
    await this.puppeteer.setPageUseProxy(this.page);
    this.logger.put(`Thread #${this.threadId()} initialized`);
  }

  async threadDestroy() {
    if (this.page) await this.page.close();
  }

  async parse(set, results) {
    for (let attempt = 1; attempt <= this.conf.proxyretries; attempt++) {
      try {
        this.logger.put(`Attempt #${attempt}: ${set.query}`);
        
        await this.page.goto(set.query, { waitUntil: 'networkidle2' });
        
        // Hide scrollbar for cleaner screenshot
        await this.page.evaluate(() => {
          (document.querySelector('html') as HTMLElement).style.overflow = 'hidden';
        });

        let screenshot: Buffer = await this.page.screenshot({ fullPage: false });

        if (this.conf.resize_width) {
          const image = await jimp.read(screenshot);
          image.resize(parseInt(this.conf.resize_width), jimp.AUTO);
          screenshot = await image.getBufferAsync('image/png');
        }

        results.screenshot = screenshot;
        
        if (this.conf.log_screenshots) {
          this.logger.putHTML(
            `<img src='data:image/png;base64,${screenshot.toString('base64')}'>`
          );
        }

        this.logger.put(`OK, size: ${Math.round(screenshot.length / 1024)}KB`);
        results.success = 1;
        await this.puppeteer.closeActiveConnections();
        break;

      } catch (error) {
        this.logger.put(`Error: ${error}`);
        await this.puppeteer.closeActiveConnections();
        await this.proxy.next(); // switch proxy for next attempt
      }
    }
    return results;
  }
}
```

## puppeteer.launch() options

```typescript
browser = await this.puppeteer.launch({
  headless: true,          // headless mode (recommended for servers)
  stealth: true,           // puppeteer-extra stealth plugin (mask as real Chrome)
  stealthOpts: { ... },   // options for stealth plugin
  extraPlugins: [...],     // additional puppeteer-extra plugins
  logConnections: true,    // log all connections per thread
  defaultViewport: { width: 1280, height: 800 },
  args: ['--no-sandbox'],  // standard puppeteer launch args
  // ... all other standard puppeteer launch options
});
```

## API methods

```typescript
// Launch browser (use in init()):
browser = await this.puppeteer.launch(opts?);

// Bind page to A-Parser proxy (use in threadInit() after newPage()):
await this.puppeteer.setPageUseProxy(page);

// Close active connections after request (use in parse()):
await this.puppeteer.closeActiveConnections();       // closes for current thread's page
await this.puppeteer.closeActiveConnections(page);   // closes for specific page

// Log current page screenshot:
await this.puppeteer.logScreenshot();
```

## Common puppeteer patterns

### Wait for element / extract data
```typescript
await this.page.goto(url, { waitUntil: 'domcontentloaded' });

// Wait for selector:
await this.page.waitForSelector('.results', { timeout: 10000 });

// Extract text:
const title = await this.page.$eval('h1', el => el.textContent?.trim());

// Extract multiple elements:
const links = await this.page.$$eval('a.result', els =>
  els.map(el => ({ href: el.href, text: el.textContent?.trim() }))
);

// Execute JS in page context:
const data = await this.page.evaluate(() => {
  return window.__INITIAL_DATA__;
});
```

### Click and navigate
```typescript
await this.page.click('#submit-button');
await this.page.waitForNavigation({ waitUntil: 'networkidle2' });
```

### Fill form
```typescript
await this.page.type('#username', 'myuser');
await this.page.type('#password', 'mypass');
await this.page.click('#login-btn');
await this.page.waitForNavigation();
```

### Intercept network requests
```typescript
async threadInit() {
  this.page = await browser.newPage();
  await this.puppeteer.setPageUseProxy(this.page);
  
  await this.page.setRequestInterception(true);
  this.page.on('request', req => {
    // Block images and fonts to speed up loading:
    if (['image', 'font'].includes(req.resourceType())) {
      req.abort();
    } else {
      req.continue();
    }
  });
}
```

### Capture XHR/API responses
```typescript
async threadInit() {
  this.page = await browser.newPage();
  await this.puppeteer.setPageUseProxy(this.page);
  
  this.page.on('response', async response => {
    if (response.url().includes('/api/data')) {
      const json = await response.json();
      this.capturedData = json;
    }
  });
}
```

### Handle infinite scroll
```typescript
async parse(set, results) {
  await this.page.goto(set.query);
  
  let previousHeight = 0;
  for (let i = 0; i < 10; i++) {
    await this.page.evaluate(() => window.scrollTo(0, document.body.scrollHeight));
    await this.sleep(2);
    
    const newHeight = await this.page.evaluate(() => document.body.scrollHeight);
    if (newHeight === previousHeight) break;
    previousHeight = newHeight;
  }
  
  const items = await this.page.$$eval('.item', els => els.map(el => el.textContent));
  for (const item of items) results.items.push(item);
  results.success = 1;
  return results;
}
```

### Per-thread long-lived sessions (keep cookies between requests)
```typescript
// Pages keep cookies automatically — just don't close the page between requests
// To clear cookies for a new session:
const cookies = await this.page.cookies();
await this.page.deleteCookie(...cookies);
```

## Performance tips

1. Block unnecessary resources (images, fonts, media) via request interception
2. Use `waitUntil: 'domcontentloaded'` instead of `'networkidle2'` when JS isn't needed
3. Reuse pages across requests (don't create new page in `parse()`, use `threadInit()`)
4. Always call `closeActiveConnections()` to free proxy connections
5. Use `headless: true` on servers
6. Set `defaultViewport` to minimum needed size
