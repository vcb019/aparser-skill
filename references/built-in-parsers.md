# A-Parser Built-in Parsers Reference

Complete reference for all 139+ built-in parsers: result fields, query format, key settings, and output examples.

Parser name format in UI and API: `Category::Name::SubName`  
Access results in templates: `$fieldName` or `$p1.fieldName` (multi-parser tasks)

---

## Search Engines

### SE::Google
Google search results parser.

**Query:** Any search phrase or operator (inurl:, intitle:, site:, etc.)

**Result fields:**
- `$serp` — array: `$link`, `$anchor`, `$snippet`, `$amp`, `$date`, `$flags` (Date/AMP/Image Preview/Video/Rich snippet/Featured snippet)
- `$totalcount` — estimated total result count
- `$ads` — array: `$link`, `$anchor`, `$text`, `$position` (top/bottom)
- `$related` — array: `$key` (related search keywords)
- `$rich` — array: `$name` (rich blocks: carousel, video, etc.)
- `$misspell` — 1 if query was auto-corrected
- `$detected_geo` — detected geo location

**Key settings:** `pagecount` (1–10), `linksperpage` (10/100), `gl` (country), `hl` (interface lang), `lr` (results lang), `domain`, `device` (desktop/mobile), `filter`, `useproxy`, `usesessions`

**Format examples:**
```
$serp.format('$link\n')
```
```
$serp.format('$link\t$anchor\t$snippet\n')
```
```
[% FOREACH i IN p1.serp; tools.CSVline(i.link, i.anchor, i.snippet); END %]
```
File: `$datefile.format().csv`  Prepend: `Link,Anchor,Snippet`

---

### SE::Google::Modern
Modern Google SERP parser (alternative implementation).

---

### SE::Google::Position
Check site ranking position for keywords in Google.

**Query:** `domain.com keyword phrase`  
Or use Query Format: `domain.com $query` to check one domain against a list of keywords.  
Bulk mode: `domain1.com,domain2.com,domain3.com keyword` — checks multiple domains at once.

**Result fields:**
- `$domain` — the domain checked
- `$key` — the keyword
- `$position` — position (0 = not found)
- `$positionurl` — URL of found page
- `$bulkcheck` — array (bulk mode): `$domain`, `$position`, `$positionurl`

**Key settings:** `pagecount`, `stopwhenfound`, `matchtype` (Exact domain / Top level domain / Exact URL)

**Format examples:**
```
$domain - $key: $position\n
```
```
$bulkcheck.format('$domain - $position\n')
```

---

### SE::Google::Suggest
Google search autocomplete suggestions.

**Query:** Search phrase (e.g., `buy essay`, `seo tools`)

**Result fields:**
- `$results` — array: `$suggest`, `$type` (0 = human, 1 = artificial)
- `$count` — total suggestions count

**Key settings:** `client` (Search page / Chrome omnibox), `followsuggests`, `maxlevel` (parse to level N)

**Format examples:**
```
$results.format('$suggest\n')
```
```
$query:\n$results.format('$suggest - $type\n')
```

---

### SE::Google::Images
Google Images search results.

**Query:** Search phrase

**Result fields:**
- `$images` — array: `$link`, `$thumb`, `$source`, `$width`, `$height`, `$caption`
- `$totalcount`

**Format example:**
```
$images.format('$link\t$source\n')
```

---

### SE::Google::Translate
Google Translate.

**Query:** `text|from_lang|to_lang` (e.g., `hello|en|ru`) or just text (auto-detect language)

**Result fields:**
- `$translation` — translated text
- `$alternatives` — array: `$source`, `$translation`
- `$detected_lang` — detected source language

**Format example:**
```
$query: $p1.translation\n
```

---

### SE::Google::Trends
Google Trends data for keywords.

**Query:** Keyword (e.g., `bitcoin`)

**Result fields:**
- `$interest` — array: `$date`, `$value` (interest over time, 0–100)
- `$regions` — array: `$region`, `$value`
- `$related` — array: `$key`, `$value`

---

### SE::Google::Trends::Suggest
Google Trends keyword suggestions.

**Query:** Search phrase

**Result fields:**
- `$results` — array: `$suggest`, `$type`

---

### SE::Google::KeywordPlanner
Google Keyword Planner — keyword ideas and search volumes.

**Query:** Keyword or URL

**Result fields:**
- `$keywords` — array: `$keyword`, `$avg_monthly_searches`, `$competition`, `$low_bid`, `$high_bid`

---

### SE::Google::KeywordPlanner::Ideas
Google Keyword Planner keyword ideas.

**Query:** Seed keyword

**Result fields:**
- `$keywords` — array: `$keyword`, `$avg_monthly_searches`, `$competition`

---

### SE::Google::KeywordPlanner::SearchVolume
Google Keyword Planner search volume for specific keywords.

**Query:** Keyword

**Result fields:**
- `$keyword`, `$avg_monthly_searches`, `$competition`, `$low_bid`, `$high_bid`

---

### SE::Google::SafeBrowsing
Google Safe Browsing — check if URL is dangerous.

**Query:** URL

**Result fields:**
- `$status` — `safe` or `danger`
- `$threats` — array: `$type` (MALWARE, PHISHING, etc.)

**Format example:**
```
$query: $p1.status\n
```

---

### SE::Google::TrustCheck
Google trust/index check.

**Query:** URL

**Result fields:**
- `$indexed` — 1/0 (indexed by Google)
- `$cache_date`

---

### SE::Google::Compromised
Check if site appears in Google hacked sites list.

**Query:** Domain

**Result fields:**
- `$compromised` — 1/0

---

### SE::Google::ByImage
Google reverse image search.

**Query:** Image URL

**Result fields:**
- `$serp` — array: `$link`, `$anchor`, `$snippet`
- `$best_guess` — text description of image

---

### SE::Google::Cache
Fetch URL from Google Cache.

**Query:** URL

**Result fields:**
- `$data` — raw cached HTML content
- `$cache_date`

---

### SE::Google::SiteMapping
Collect all indexed pages for a domain via Google.

**Query:** Domain (e.g., `example.com`)

**Result fields:**
- `$serp` — array: `$link`, `$anchor`
- `$totalcount`

---

### SE::Yandex
Yandex search results parser.

**Query:** Search phrase

**Result fields:**
- `$serp` — array: `$link`, `$anchor`, `$snippet`, `$amp`, `$date`
- `$totalcount`
- `$related` — array: `$key`
- `$misspell`

**Key settings:** `pagecount`, `region` (geo region code), `family` (safe search), `device`

**Format examples:**
```
$serp.format('$link\n')
```
```
$serp.format('$link\t$anchor\t$snippet\n')
```

---

### SE::Yandex::Position
Check site ranking position in Yandex.

**Query:** `domain.com keyword phrase`

**Result fields:**
- `$domain`, `$key`, `$position`, `$positionurl`
- `$bulkcheck` — array (bulk mode): `$domain`, `$position`

**Format example:**
```
$domain - $key: $position\n
```

---

### SE::Yandex::Suggest
Yandex search autocomplete suggestions.

**Query:** Search phrase

**Result fields:**
- `$results` — array: `$suggest`

**Format example:**
```
$results.format('$suggest\n')
```

---

### SE::Yandex::Images
Yandex Images search.

**Query:** Search phrase

**Result fields:**
- `$images` — array: `$link`, `$thumb`, `$source`, `$width`, `$height`

---

### SE::Yandex::Video
Yandex Video search.

**Query:** Search phrase

**Result fields:**
- `$serp` — array: `$link`, `$title`, `$description`, `$duration`, `$views`

---

### SE::Yandex::Translate
Yandex Translate.

**Query:** `text|from_lang|to_lang` or just text

**Result fields:**
- `$translation`
- `$detected_lang`

---

### SE::Yandex::WordStat
Yandex Wordstat — keyword search volume.

**Query:** Keyword  
Bulk mode supported: pass multiple keywords to get volumes at once.

**Result fields:**
- `$wordstat_count` — monthly searches
- `$phrase_count` — exact phrase count
- `$wordstat` — array: `$keyword`, `$count` (related keywords list)
- `$related` — array: `$keyword`, `$count` (associated keywords)
- `$update_date` — statistics update date
- `$region` — region name (if regional search)

**Key settings:** `region`, `maxlevel` (parse to level)

**Format examples:**
```
"$query",$p1.wordstat_count,$p1.update_date\n
```
```
$wordstat.format('$keyword\t$count\n')
```

---

### SE::Yandex::WordStat::ByDate
Yandex Wordstat — keyword search volume by date (history).

**Query:** Keyword

**Result fields:**
- `$history` — array: `$date`, `$count`

**Format example:**
```
$history.format('$date\t$count\n')
```

---

### SE::Yandex::WordStat::ByRegion
Yandex Wordstat — keyword search volume by region.

**Query:** Keyword

**Result fields:**
- `$regions` — array: `$region`, `$count`, `$percent`

**Format example:**
```
$regions.format('$region\t$count\n')
```

---

### SE::Yandex::Direct::Frequency
Yandex Direct keyword frequency checker.

**Query:** Keyword (single or comma-separated list for bulk)  
Supports bulk mode: 10 keywords at once.

**Result fields:**
- `$frequency` — broad match frequency
- `$phrase_count` — phrase match count
- `$exact_count` — exact match count
- `$wordstat` — array: `$keyword`, `$count`, `$phrase_count`

**Key settings:** `region`, `phrase` (check phrase match), `broad` (broad match), `exact` (exact match), `wordstat` (include related keywords)

**Format examples:**
```
$query\t$p1.frequency\n
```
Bulk mode (10 keywords per request):
```typescript
// In JS parser, pass array of queries:
const response = await this.parser.request('SE::Yandex::Direct::Frequency', 'default', { bulkQueries: 10 }, queryArray);
response.bulkResults[i].frequency
```

---

### SE::Yandex::SQI
Yandex Site Quality Index (TIC/SQI).

**Query:** Domain or URL

**Result fields:**
- `$sqi` — site quality index value

---

### SE::Yandex::ByImage
Yandex reverse image search.

**Query:** Image URL

**Result fields:**
- `$serp` — array: `$link`, `$anchor`, `$snippet`
- `$best_guess` — image description

---

### SE::Yandex::Balaboba
Yandex Balaboba text generator.

**Query:** Text prompt

**Result fields:**
- `$text` — generated text

---

### SE::Yandex::Speller
Yandex Speller — spell check.

**Query:** Text to check

**Result fields:**
- `$errors` — array: `$word`, `$suggestions`
- `$corrected` — text with corrections applied

---

### SE::Yandex::SafeBrowsing
Yandex Safe Browsing check.

**Query:** URL

**Result fields:**
- `$status` — safe/danger
- `$threats` — array: `$type`

---

### SE::Yandex::Register
Check domain registration status in Yandex services.

**Query:** Domain

**Result fields:**
- `$registered` — 1/0

---

### SE::Yandex::WordCraft
Yandex WordCraft — text rewriting/paraphrasing tool.

**Query:** Text

**Result fields:**
- `$variants` — array: `$text`

---

### SE::Bing
Bing search results.

**Query:** Search phrase

**Result fields:**
- `$serp` — array: `$link`, `$anchor`, `$snippet`
- `$totalcount`
- `$related` — array: `$key`

**Key settings:** `pagecount`, `linksperpage`, `useproxy`

**Format example:**
```
$serp.format('$link\t$anchor\n')
```

---

### SE::Bing::Position
Check site ranking in Bing.

**Query:** `domain.com keyword`

**Result fields:**
- `$domain`, `$key`, `$position`, `$positionurl`

---

### SE::Bing::Suggest
Bing autocomplete suggestions.

**Query:** Search phrase

**Result fields:**
- `$results` — array: `$suggest`

---

### SE::Bing::Images
Bing Images search.

**Query:** Search phrase

**Result fields:**
- `$images` — array: `$link`, `$thumb`, `$source`, `$width`, `$height`

---

### SE::Bing::Video
Bing Video search.

**Query:** Search phrase

**Result fields:**
- `$serp` — array: `$link`, `$title`, `$description`, `$duration`

---

### SE::Bing::Translator
Bing / Microsoft Translator.

**Query:** `text|from_lang|to_lang`

**Result fields:**
- `$translation`
- `$detected_lang`

---

### SE::DuckDuckGo
DuckDuckGo search results.

**Query:** Search phrase

**Result fields:**
- `$serp` — array: `$link`, `$anchor`, `$snippet`

---

### SE::DuckDuckGo::Position
Check site ranking in DuckDuckGo.

**Query:** `domain.com keyword`

**Result fields:**
- `$domain`, `$key`, `$position`, `$positionurl`

---

### SE::DuckDuckGo::Images
DuckDuckGo Images search.

**Query:** Search phrase

**Result fields:**
- `$images` — array: `$link`, `$thumb`, `$source`

---

### SE::Yahoo
Yahoo search results.

**Query:** Search phrase

**Result fields:**
- `$serp` — array: `$link`, `$anchor`, `$snippet`
- `$totalcount`

---

### SE::Yahoo::Suggest
Yahoo autocomplete suggestions.

**Query:** Search phrase

**Result fields:**
- `$results` — array: `$suggest`

---

### SE::AOL
AOL search results.

**Query:** Search phrase

**Result fields:**
- `$serp` — array: `$link`, `$anchor`, `$snippet`

---

### SE::AOL::Suggest
AOL autocomplete suggestions.

**Query:** Search phrase

**Result fields:**
- `$results` — array: `$suggest`

---

### SE::Ask
Ask.com search results.

**Query:** Search phrase

**Result fields:**
- `$serp` — array: `$link`, `$anchor`, `$snippet`

---

### SE::Baidu
Baidu (Chinese) search results.

**Query:** Search phrase

**Result fields:**
- `$serp` — array: `$link`, `$anchor`, `$snippet`
- `$totalcount`

---

### SE::Rambler
Rambler (Russian) search results.

**Query:** Search phrase

**Result fields:**
- `$serp` — array: `$link`, `$anchor`, `$snippet`
- `$totalcount`

---

### SE::Seznam
Seznam (Czech) search results.

**Query:** Search phrase

**Result fields:**
- `$serp` — array: `$link`, `$anchor`, `$snippet`

---

### SE::Brave
Brave Search results.

**Query:** Search phrase

**Result fields:**
- `$serp` — array: `$link`, `$anchor`, `$snippet`

---

### SE::StartPage
StartPage search results (privacy-focused Google proxy).

**Query:** Search phrase

**Result fields:**
- `$serp` — array: `$link`, `$anchor`, `$snippet`

---

### SE::StartPage::Images
StartPage Images search.

**Query:** Search phrase

**Result fields:**
- `$images` — array: `$link`, `$thumb`, `$source`

---

### SE::StartPage::Videos
StartPage Videos search.

**Query:** Search phrase

**Result fields:**
- `$serp` — array: `$link`, `$title`, `$description`

---

### SE::Dogpile
Dogpile search results.

**Query:** Search phrase

**Result fields:**
- `$serp` — array: `$link`, `$anchor`, `$snippet`

---

### SE::Dogpile::Images
Dogpile Images search.

**Query:** Search phrase

**Result fields:**
- `$images` — array: `$link`, `$thumb`, `$source`

---

### SE::Quora
Quora question search.

**Query:** Search phrase

**Result fields:**
- `$serp` — array: `$link`, `$title`, `$snippet`

---

### SE::You
You.com AI-powered search.

**Query:** Search phrase

**Result fields:**
- `$serp` — array: `$link`, `$anchor`, `$snippet`
- `$ai_answer` — AI-generated summary

---

### SE::YouTube
YouTube video search.

**Query:** Search phrase

**Result fields:**
- `$serp` — array: `$link`, `$title`, `$description`, `$views`, `$channel`, `$date`, `$duration`, `$thumbnail`
- `$totalcount`

**Format example:**
```
$serp.format('$link\t$title\t$views\t$channel\n')
```

---

### SE::YouTube::Suggest
YouTube autocomplete suggestions.

**Query:** Search phrase

**Result fields:**
- `$results` — array: `$suggest`

---

### SE::YouTube::Video
YouTube video details by URL or ID.

**Query:** YouTube video URL or ID

**Result fields:**
- `$title`, `$description`, `$views`, `$likes`, `$channel`, `$date`, `$duration`, `$thumbnail`
- `$comments` — array: `$author`, `$text`, `$likes`, `$date`

---

### SE::Pinterest
Pinterest search results.

**Query:** Search phrase

**Result fields:**
- `$serp` — array: `$link`, `$title`, `$thumbnail`

---

### SE::Pinterest::Suggest
Pinterest autocomplete suggestions.

**Query:** Search phrase

**Result fields:**
- `$results` — array: `$suggest`

---

## SEO / Rank

### Rank::Ahrefs
Ahrefs domain/URL metrics.

**Query:** Domain or URL

**Result fields:**
- `$domain_rating` — Domain Rating (0–100)
- `$url_rating` — URL Rating
- `$backlinks` — total backlinks count
- `$refdomains` — referring domains count
- `$organic_traffic` — estimated monthly organic traffic
- `$organic_keywords` — organic keywords count

**Key settings:** `mode` (Domain / Subdomain / Exact URL / Prefix)

**Format example:**
```
$query\t$p1.domain_rating\t$p1.backlinks\t$p1.refdomains\n
```

---

### Rank::Ahrefs::BrokenLinks
Check broken outbound links via Ahrefs.

**Query:** Domain

**Result fields:**
- `$links` — array: `$url`, `$target`, `$anchor`

---

### Rank::Ahrefs::KeywordDifficulty
Ahrefs keyword difficulty score.

**Query:** Keyword

**Result fields:**
- `$difficulty` — keyword difficulty (0–100)
- `$volume` — monthly search volume
- `$cpc`

---

### Rank::Ahrefs::KeywordGenerator
Ahrefs keyword ideas generator.

**Query:** Seed keyword

**Result fields:**
- `$keywords` — array: `$keyword`, `$volume`, `$difficulty`, `$cpc`

---

### Rank::Ahrefs::TrafficChecker
Ahrefs traffic checker for URL.

**Query:** URL

**Result fields:**
- `$traffic` — estimated monthly traffic
- `$keywords_count`

---

### Rank::Moz
Moz domain/URL SEO metrics.

**Query:** Domain or URL

**Result fields:**
- `$pa` — Page Authority (0–100)
- `$da` — Domain Authority (0–100)
- `$spam_score` — spam score (0–17)
- `$external_links` — external links count
- `$root_domains` — linking root domains

**Format example:**
```
$query\t$p1.da\t$p1.pa\t$p1.spam_score\n
```

---

### Rank::MajesticSEO
Majestic SEO metrics.

**Query:** Domain or URL

**Result fields:**
- `$trust_flow` — Trust Flow (0–100)
- `$citation_flow` — Citation Flow (0–100)
- `$ref_domains` — referring domains
- `$backlinks` — backlinks count
- `$indexed_urls` — indexed URLs

**Format example:**
```
$query\t$p1.trust_flow\t$p1.citation_flow\t$p1.ref_domains\n
```

---

### Rank::CMS
CMS and technology detection.

**Query:** URL or domain

**Result fields:**
- `$cms` — detected CMS name (WordPress, Joomla, etc.)
- `$cms_version`
- `$plugins` — array: `$name`, `$version`
- `$technologies` — array: `$name`

**Format example:**
```
$query\t$p1.cms\n
```

---

### Rank::Archive
Wayback Machine (archive.org) data.

**Query:** URL

**Result fields:**
- `$snapshots` — array: `$date`, `$url`
- `$first_date` — first archive date
- `$last_date` — last archive date
- `$total_snapshots`

---

### Rank::BingAnalytics
Bing indexed pages count and analytics.

**Query:** Domain

**Result fields:**
- `$indexed_pages` — pages indexed by Bing

---

### Rank::Bukvarix::Domain
Bukvarix — Russian SEO tool, domain keywords.

**Query:** Domain

**Result fields:**
- `$keywords` — array: `$keyword`, `$position`, `$url`, `$frequency`

---

### Rank::Bukvarix::Keyword
Bukvarix — sites ranked for keyword.

**Query:** Keyword

**Result fields:**
- `$domains` — array: `$domain`, `$position`, `$url`

---

### Rank::Curlie
Curlie (DMOZ successor) category check.

**Query:** Domain

**Result fields:**
- `$category`, `$url`, `$title`

---

### Rank::KeySSO
KeySSO keyword data.

**Query:** Keyword

**Result fields:**
- `$domains` — array: `$domain`, `$position`

---

### Rank::Mustat
Mustat website statistics.

**Query:** Domain

**Result fields:**
- `$rank`, `$visitors`, `$pageviews`

---

### Rank::SocialSignal
Social signals count for URL.

**Query:** URL

**Result fields:**
- `$facebook_shares`, `$pinterest`, `$reddit`

---

### Check::Backlink
Check if a page links to a target domain.

**Query:** `source_url target_domain`

**Result fields:**
- `$found` — 1/0
- `$anchor` — anchor text of found link
- `$link` — exact link found

---

### SEO::Ping
Ping URLs to search engine update services.

**Query:** URL to ping

**Result fields:**
- `$status` — success/error

---

### SecurityTrails::Domain
SecurityTrails domain intelligence.

**Query:** Domain

**Result fields:**
- `$dns` — DNS records history
- `$whois` — WHOIS data
- `$subdomains` — array: `$subdomain`
- `$associated_domains` — array: `$domain`

---

### SecurityTrails::IP
SecurityTrails IP intelligence.

**Query:** IP address

**Result fields:**
- `$hostnames` — array: `$hostname`
- `$whois`

---

### Cloudflare::Radar
Cloudflare Radar domain statistics.

**Query:** Domain

**Result fields:**
- `$traffic` — traffic ranking
- `$categories` — array: `$name`
- `$protocols` — HTTP version breakdown

---

## HTML / Network

### HTML::LinkExtractor
Extract all links from a page.

**Query:** URL

**Result fields:**
- `$links` — array: `$link`, `$anchor`, `$type` (internal/external)
- `$count` — total links found

**Key settings:** `parse_to_level` (recursive crawl, enables site mapping), `onlyinternal`, `onlyexternal`, `useproxy`

**Format examples:**
```
$links.format('$link\n')
```
```
$links.format('$link\t$anchor\n')
```
For site mapping — enable "Parse to level" and "Unique queries" in task settings.

---

### HTML::TextExtractor
Extract visible text content from a page.

**Query:** URL

**Result fields:**
- `$text` — page visible text (cleaned)
- `$title` — page title
- `$description` — meta description
- `$h1` — H1 content

**Format example:**
```
$query\t$p1.title\t$p1.description\n
```

---

### HTML::EmailExtractor
Extract email addresses from a page.

**Query:** URL

**Result fields:**
- `$emails` — array: `$email`
- `$count`

**Format example:**
```
$emails.format('$email\n')
```

---

### HTML::ArticleExtractor
Extract article text from a page (news/blog articles).

**Query:** URL

**Result fields:**
- `$title`, `$author`, `$date`, `$text`, `$images` — array: `$link`

---

### HTML::TextExtractor::LangDetect
Extract text + detect language.

**Query:** URL

**Result fields:**
- `$text`, `$title`, `$lang` (detected language code)

---

### Net::HTTP
Raw HTTP request — fetch page content.

**Query:** URL

**Result fields:**
- `$data` — raw response body (HTML, JSON, XML, etc.)
- `$headers` — response headers
- `$status_code` — HTTP status code
- `$redirect_url` — final URL after redirects

**Key settings:** `method` (GET/POST), `params`, `addheaders`, `timeout`, `useproxy`, `check_content`, `parsecodes`, `use_pages`, `check_next_page`

**Format example:**
```
$query\t$p1.status_code\n
```

---

### Net::DNS
DNS lookup.

**Query:** Domain (e.g., `example.com`)

**Result fields:**
- `$records` — array: `$type`, `$value`, `$ttl`
- `$a` — array: `$value` (A records / IPv4)
- `$aaaa` — array: `$value` (IPv6)
- `$mx` — array: `$value`, `$priority`
- `$cname` — array: `$value`
- `$ns` — array: `$value`
- `$txt` — array: `$value`
- `$spf`, `$dmarc`

**Format example:**
```
$query\t$p1.a.0.value\t$p1.mx.0.value\n
```

---

### Net::Whois
WHOIS domain registration data.

**Query:** Domain (e.g., `example.com`)

**Result fields:**
- `$registered` — 1/0
- `$creation_date` — registration date (DD.MM.YYYY)
- `$expire_date` — expiry date
- `$free_date` — date becomes available
- `$registrar` — registrar name
- `$ns` — array: `$server` (nameservers)
- `$statuses` — array: `$status`
- `$data` — raw WHOIS text

**Format examples:**
```
$query: $p1.expire_date [% FOREACH ns IN p1.ns %]$ns.server [% END %]\n
```
```
$query\t$p1.expire_date\t$p1.registrar\n
```

---

### IP::Geo
IP geolocation.

**Query:** IP address

**Result fields:**
- `$country`, `$country_code`, `$city`, `$region`
- `$lat`, `$lon` — coordinates
- `$isp`, `$org` — internet provider
- `$timezone`

**Format example:**
```
$query\t$p1.country\t$p1.city\t$p1.isp\n
```

---

### IP::Info
Detailed IP information (ipinfo.io).

**Query:** IP address

**Result fields:**
- `$hostname`, `$org`, `$country`, `$city`, `$region`, `$postal`, `$lat`, `$lon`, `$timezone`

---

### Check::Roskomnadzor
Check if domain/URL is blocked by Roskomnadzor (Russian internet regulator).

**Query:** Domain or URL

**Result fields:**
- `$blocked` — 1/0
- `$reason`

---

## Maps & Local

### Maps::Google
Google Maps business information.

**Query:** Business name, address, or "business name city"

**Result fields:**
- `$name`, `$address`, `$phone`
- `$rating` — rating (0–5)
- `$reviews_count`
- `$lat`, `$lon`
- `$url` — website
- `$type` — business category
- `$hours` — array: `$day`, `$hours`
- `$place_id`

**Format example:**
```
$query\t$p1.name\t$p1.address\t$p1.phone\t$p1.rating\n
```

---

### Maps::Google::Reviews
Google Maps business reviews.

**Query:** Business name or Place ID

**Result fields:**
- `$reviews` — array: `$author`, `$rating`, `$text`, `$date`
- `$rating`, `$reviews_count`

---

### Maps::Yandex
Yandex Maps business information.

**Query:** Business name or address

**Result fields:**
- `$name`, `$address`, `$phone`
- `$rating`, `$reviews_count`
- `$lat`, `$lon`
- `$url`
- `$categories` — array: `$name`

**Format example:**
```
$query\t$p1.name\t$p1.address\t$p1.phone\t$p1.rating\n
```

---

## E-commerce

### Shop::Amazon
Amazon product information.

**Query:** Product search phrase or ASIN (e.g., `B08N5WRWNW`)

**Result fields:**
- `$title`, `$asin`, `$price`, `$currency`
- `$rating`, `$reviews_count`
- `$description`
- `$features` — array: `$text` (bullet points)
- `$images` — array: `$link`
- `$seller`, `$brand`

**Format example:**
```
$query\t$p1.title\t$p1.price\t$p1.rating\n
```

---

### Shop::AliExpress
AliExpress product information.

**Query:** Product search phrase or URL

**Result fields:**
- `$title`, `$price`, `$currency`
- `$rating`, `$orders`
- `$seller`, `$seller_rating`
- `$images` — array: `$link`

---

### Shop::Ebay
eBay product search results.

**Query:** Search phrase

**Result fields:**
- `$items` — array: `$title`, `$price`, `$link`, `$condition`, `$seller`, `$bids`

---

### Shop::Wildberries::ProductInfo
Wildberries (Russian marketplace) product information.

**Query:** Product URL or article number

**Result fields:**
- `$title`, `$brand`, `$price`, `$sale_price`
- `$rating`, `$reviews_count`
- `$colors` — array: `$color`, `$sizes`
- `$description`

---

### Shop::Wildberries::ProductsList
Wildberries products list by category or search.

**Query:** Category URL or search phrase

**Result fields:**
- `$products` — array: `$title`, `$link`, `$price`, `$brand`, `$rating`

---

### Shop::Wildberries::Suggest
Wildberries search autocomplete.

**Query:** Search phrase

**Result fields:**
- `$results` — array: `$suggest`

---

### Shop::Yandex::Market
Yandex Market product search.

**Query:** Search phrase or product URL

**Result fields:**
- `$products` — array: `$title`, `$price`, `$shop`, `$rating`
- `$title`, `$price`, `$specs` (for product page)

---

### Shop::GooglePlay::Apps
Google Play app search.

**Query:** App name or keyword

**Result fields:**
- `$apps` — array: `$title`, `$developer`, `$rating`, `$installs`, `$link`, `$price`

---

### CoinMarketCap::LastPrice
CoinMarketCap cryptocurrency price.

**Query:** Coin symbol (e.g., `BTC`, `ETH`)

**Result fields:**
- `$price`, `$change_24h`, `$volume_24h`, `$market_cap`, `$rank`

---

## Social & Messengers

### Social::Instagram::Profile
Instagram profile data.

**Query:** Username (without @)

**Result fields:**
- `$followers`, `$following`, `$posts_count`
- `$bio`, `$name`, `$link`
- `$is_private`, `$is_verified`

---

### Social::Instagram::Post
Instagram post data by URL.

**Query:** Post URL

**Result fields:**
- `$likes`, `$comments_count`, `$date`
- `$caption`, `$author`
- `$images` — array: `$link`

---

### Social::Instagram::Search
Instagram hashtag/user search.

**Query:** Hashtag or username

**Result fields:**
- `$results` — array: `$link`, `$type`, `$likes`, `$thumbnail`

---

### Social::Instagram::Tag
Instagram tag (hashtag) information.

**Query:** Hashtag (without #)

**Result fields:**
- `$posts_count`
- `$posts` — array: `$link`, `$thumbnail`, `$likes`

---

### Social::Instagram::Geo
Instagram posts by geolocation.

**Query:** Location ID or name

**Result fields:**
- `$posts` — array: `$link`, `$thumbnail`

---

### Social::TikTok::Profile
TikTok profile data.

**Query:** Username

**Result fields:**
- `$followers`, `$following`, `$likes`, `$videos_count`
- `$bio`, `$name`

---

### Reddit::Posts
Reddit post search.

**Query:** Search phrase (optionally with `subreddit:name`)

**Result fields:**
- `$posts` — array: `$title`, `$link`, `$score`, `$comments`, `$author`, `$date`, `$subreddit`, `$thumbnail`

**Format example:**
```
$posts.format('$title\t$link\t$score\n')
```

---

### Reddit::PostInfo
Reddit post details by URL.

**Query:** Reddit post URL

**Result fields:**
- `$title`, `$text`, `$score`, `$author`, `$date`, `$subreddit`
- `$comments` — array: `$author`, `$text`, `$score`

---

### Reddit::Comments
Reddit comments for a post.

**Query:** Reddit post URL

**Result fields:**
- `$comments` — array: `$author`, `$text`, `$score`, `$date`

---

### Telegram::GroupScraper
Telegram group member scraper.

**Query:** Group link (t.me/groupname) or username

**Result fields:**
- `$members` — array: `$user_id`, `$name`, `$username`, `$phone`, `$bio`
- `$members_count`

---

## AI & Translation

### OpenAI::ChatGPT
OpenAI ChatGPT responses via API.

**Query:** Prompt/question

**Result fields:**
- `$answer` — model response text
- `$tokens_used`

**Key settings:** `api_key`, `model` (gpt-4, gpt-3.5-turbo, etc.), `system_prompt`, `temperature`, `max_tokens`

**Format example:**
```
$query\n$p1.answer\n---\n
```

---

### OpenAI::Completions
OpenAI text completions (legacy API).

**Query:** Prompt text

**Result fields:**
- `$text` — completed text
- `$tokens_used`

**Key settings:** `api_key`, `model`, `temperature`, `max_tokens`

---

### FreeAI::ChatGPT
Free ChatGPT (web scraping, no API key required).

**Query:** Prompt/question

**Result fields:**
- `$answer`

---

### FreeAI::Copilot
Microsoft Copilot (Bing Chat).

**Query:** Prompt/question

**Result fields:**
- `$answer`

---

### FreeAI::DeepAI
DeepAI text generation.

**Query:** Prompt/question

**Result fields:**
- `$answer`

---

### FreeAI::GoogleAI
Google AI (Bard/Gemini).

**Query:** Prompt/question

**Result fields:**
- `$answer`

---

### FreeAI::Kimi
Kimi AI assistant.

**Query:** Prompt/question

**Result fields:**
- `$answer`

---

### FreeAI::Perplexity
Perplexity AI search.

**Query:** Question

**Result fields:**
- `$answer`, `$sources` — array: `$link`, `$title`

---

### FreeAI::Server::OpenAI
Custom OpenAI-compatible API server.

**Query:** Prompt/question

**Key settings:** `api_url`, `api_key`, `model`

**Result fields:**
- `$answer`

---

### DeepL::Translator
DeepL professional translation.

**Query:** `text|from_lang|to_lang` (e.g., `Hello world|EN|DE`) or just text (auto-detect source)

**Result fields:**
- `$translation` — translated text
- `$detected_lang`

**Key settings:** `api_key`, `formality` (default/more/less)

**Format example:**
```
$query\t$p1.translation\n
```

---

### DeepL::Write
DeepL Write — text improvement and paraphrasing.

**Query:** Text to improve (English or German)

**Result fields:**
- `$text` — improved text
- `$improvements` — array: `$original`, `$suggestion`

---

## Captcha & Utilities

### Util::AntiGate
Solve image captchas via AntiGate/Anti-Captcha service.

**Query:** Base64-encoded captcha image or URL to image

**Result fields:**
- `$answer` — solved captcha text
- `$id` — captcha task ID

**Key settings:** `apikey`, `type` (image/text/custom)

**Usage in JS parser:**
```typescript
const { answer, id } = await this.captcha.recognize('presetName', imageBuffer, 'png');
```

---

### Util::ReCaptcha2
Solve Google ReCaptcha v2 tokens.

**Query:** `siteKey|pageUrl`

**Result fields:**
- `$token` — solved g-recaptcha-response token

**Usage in JS parser:**
```typescript
const r = await this.parser.request('Util::ReCaptcha2', 'default', {}, `${siteKey}|${pageUrl}`);
const token = r.token;
```

---

### Util::ReCaptcha3
Solve Google ReCaptcha v3 tokens.

**Query:** `siteKey|pageUrl|action`

**Result fields:**
- `$token`

---

### Util::hCaptcha
Solve hCaptcha tokens.

**Query:** `siteKey|pageUrl`

**Result fields:**
- `$token`

---

### Util::Turnstile
Solve Cloudflare Turnstile tokens.

**Query:** `siteKey|pageUrl`

**Result fields:**
- `$token`

---

### Util::RotateCaptcha
Solve rotation captchas (rotate image to correct angle).

**Query:** Base64 image or URL

**Result fields:**
- `$angle` — rotation angle in degrees

---

### Util::SMS
SMS verification number service integration.

**Query:** Service + phone number (depends on service)

**Result fields:**
- `$phone`, `$code`

---

### Util::YandexRecognize
Yandex image recognition / captcha solving.

**Query:** Base64 image

**Result fields:**
- `$answer`

---

## Other

### Browser::ScreenshotsMaker
Take screenshots of websites via browser (Puppeteer).

**Query:** URL

**Result fields:**
- `$screenshot` — path to saved screenshot file

**Key settings:** `viewport_width`, `viewport_height`, `wait_for`, `full_page`, `proxy`

**Note:** This is a built-in wrapper. For custom screenshot behavior with JS parsers, use `this.puppeteer` API (see `references/puppeteer.md`).

---

### API::Server::Redis
Redis-based API server — processes queries from a Redis queue.

**Not used as a regular parser.** Run as a task to expose A-Parser as an async request processor.

**Setup:**
1. Create preset: set Redis Host, Port, Queue Key (default: `aparser_redis_api`), Result Expire TTL
2. Add task with `{num:1:N}` queries (N = thread count)

**Query format (Redis lpush):**
```
["unique_id", "SE::Google", "default", "keyword"]
```
or with overrides:
```
["uid", "SE::Google", "default", "keyword", {"pagecount": 1}]
```

**Get result:**
```
blpop aparser_redis_api:unique_id 0
```

See `references/http-api.md` → Redis API section for full details.

---

## Quick Reference: Result Fields by Parser

| Parser | Key flat fields | Key array fields |
|---|---|---|
| SE::Google | totalcount, misspell, detected_geo | serp(link,anchor,snippet,flags), ads, related, rich |
| SE::Google::Position | domain, key, position, positionurl | bulkcheck(domain,position,positionurl) |
| SE::Google::Suggest | count | results(suggest,type) |
| SE::Yandex | totalcount, misspell | serp(link,anchor,snippet), related |
| SE::Yandex::WordStat | wordstat_count, phrase_count, update_date | wordstat(keyword,count), related(keyword,count) |
| SE::Yandex::Direct::Frequency | frequency, phrase_count, exact_count | wordstat(keyword,count,phrase_count) |
| SE::Bing | totalcount | serp(link,anchor,snippet), related |
| SE::YouTube | totalcount | serp(link,title,description,views,channel,date,duration,thumbnail) |
| Rank::Ahrefs | domain_rating, url_rating, backlinks, refdomains, organic_traffic | — |
| Rank::Moz | pa, da, spam_score, external_links, root_domains | — |
| Rank::MajesticSEO | trust_flow, citation_flow, ref_domains, backlinks | — |
| Rank::CMS | cms, cms_version | plugins(name,version), technologies |
| HTML::LinkExtractor | count | links(link,anchor,type) |
| HTML::TextExtractor | text, title, description, h1 | — |
| HTML::EmailExtractor | count | emails(email) |
| Net::HTTP | data, status_code, redirect_url | headers |
| Net::DNS | — | records(type,value,ttl), a, mx, ns, txt |
| Net::Whois | registered, creation_date, expire_date, registrar | ns(server), statuses(status) |
| IP::Geo | country, country_code, city, region, lat, lon, isp | — |
| Maps::Google | name, address, phone, rating, reviews_count, lat, lon, url | hours(day,hours) |
| Maps::Yandex | name, address, phone, rating, lat, lon | categories(name) |
| Shop::Amazon | title, asin, price, rating, reviews_count, brand | features(text), images(link) |
| DeepL::Translator | translation, detected_lang | — |
| OpenAI::ChatGPT | answer, tokens_used | — |
| Util::ReCaptcha2 | token | — |
| Util::hCaptcha | token | — |
| Util::Turnstile | token | — |
