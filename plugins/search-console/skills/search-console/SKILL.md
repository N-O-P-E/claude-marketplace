---
name: search-console
description: Audit Google Search Console via browser. Use when the user says "check Search Console", "what's wrong with my domain", "why aren't my pages indexed", "SEO audit", "sitemap issues", or wants to investigate indexing problems, Core Web Vitals, structured data errors, or merchant listing issues. Opens GSC in Chrome, reads all reports, and returns a prioritised fix list.
allowed-tools:
  - AskUserQuestion
  - Read
  - Write
  - Bash
  - mcp__plugin_chrome-devtools-mcp_chrome-devtools__navigate_page
  - mcp__plugin_chrome-devtools-mcp_chrome-devtools__click
  - mcp__plugin_chrome-devtools-mcp_chrome-devtools__fill
  - mcp__plugin_chrome-devtools-mcp_chrome-devtools__take_screenshot
  - mcp__plugin_chrome-devtools-mcp_chrome-devtools__take_snapshot
  - mcp__plugin_chrome-devtools-mcp_chrome-devtools__wait_for
  - mcp__plugin_chrome-devtools-mcp_chrome-devtools__list_pages
  - mcp__plugin_chrome-devtools-mcp_chrome-devtools__select_page
  - mcp__plugin_chrome-devtools-mcp_chrome-devtools__evaluate_script
  - mcp__plugin_chrome-devtools-mcp_chrome-devtools__new_page
---

# Google Search Console Audit — Browser-Automated

Opens Google Search Console in the browser and reads every report that matters for SEO health. Returns a prioritised, actionable fix list.

---

## Before You Start

1. Call `list_pages` to verify Chrome DevTools MCP is connected.
2. If it fails: "Please open Chrome and make sure the DevTools MCP extension is active."
3. Confirm the domain to audit. If not already known, ask:
   - "What domain should I audit? (e.g. https://example.com/)"
4. The user must already be signed in to Google Search Console in Chrome. If not, tell them to sign in first — you cannot handle Google authentication.

---

## The Audit Workflow

```
┌─────────────┐   ┌─────────────┐   ┌─────────────┐   ┌─────────────┐
│   Overview  │ → │  Indexing   │ → │  Sitemaps   │ → │  Shopping & │
│   (snapshot)│   │  (drill in) │   │  (status)   │   │  Structured │
└─────────────┘   └─────────────┘   └─────────────┘   └─────────────┘
                                                               │
                                                    ┌─────────────────┐
                                                    │  Produce Report │
                                                    └─────────────────┘
```

Work through each section below. Always `take_snapshot` after navigating — GSC is a React app that loads asynchronously.

---

## Step 1: Overview

Navigate to:
```
https://search.google.com/search-console?resource_id=DOMAIN
```
(Replace `DOMAIN` with the URL-encoded property, e.g. `https%3A%2F%2Fperfectpillows.nl%2F`)

`wait_for` text like the site name or "Overzicht" / "Overview", then `take_snapshot`.

**Capture from the overview:**
- Total indexed pages
- Total non-indexed pages
- Core Web Vitals summary (good / needs improvement / poor, mobile + desktop)
- Shopping section: product snippets valid/invalid, merchant listings valid/invalid
- Any "Recommendations" / "Aanbevelingen" cards — these flag recent anomalies
- HTTPS status

`take_screenshot` for the visual record.

---

## Step 2: Page Indexing Report

Navigate to:
```
https://search.google.com/search-console/index?resource_id=DOMAIN
```

`wait_for` "Niet geïndexeerd" / "Not indexed", then `take_snapshot`.

**Capture the full reasons table.** It typically shows up to 10 rows with:
- Reason (Dutch or English depending on account locale)
- Source (Website / Google systems)
- Validation status
- Page count

**Key reasons to watch for and their meaning:**

| Reason | What it means | Typical fix |
|--------|--------------|-------------|
| Alternative page with correct canonical tag | Pages correctly pointing to a canonical — but validation failed | Canonical tag misconfigured or the canonical URL itself has issues |
| Crawled – currently not indexed | Google crawled but chose not to index | Thin content, duplicate content, low quality signals |
| Google chose different canonical than user | Google is overriding your canonical tag | Canonical points to a URL Google doesn't trust as the main URL |
| Duplicate without user-selected canonical | Missing canonical tags on duplicate pages | Add canonical tags |
| Redirect page | Redirect URLs appearing as discovered pages | Remove redirects from sitemap; fix internal links pointing to old URLs |
| Not found (404) | Broken pages Google has discovered | Add 301 redirects or fix internal links |
| Excluded by noindex tag | Pages explicitly excluded | Verify noindex is intentional |
| Blocked by robots.txt | Crawl blocked | Verify robots.txt allows important pages |
| Found – currently not indexed | Discovered but not yet crawled/indexed | May resolve itself; check priority |
| Soft 404 | Page returns 200 but looks like an error | Fix the page or return proper 404 |

`take_screenshot` of the full table.

---

## Step 3: Sitemaps

Navigate to:
```
https://search.google.com/search-console/sitemaps?resource_id=DOMAIN
```

`wait_for` "sitemap" or "Sitemap", then `take_snapshot`.

**Capture:**
- Which sitemaps are submitted
- Status (Successful / Has errors)
- Last read date — if >30 days ago, GSC may not be seeing updates
- Discovered pages count per sitemap
- Compare discovered pages in sitemap vs total indexed pages — a big gap means the sitemap is incomplete

**Common sitemap issues:**
- Only the root sitemap (`sitemap.xml`) submitted but sub-sitemaps (`sitemap_products.xml`, `sitemap_pages.xml`, etc.) aren't being discovered
- Sitemap contains redirected or 404 URLs
- Sitemap was never submitted (only auto-discovered)
- Stale sitemap — old URLs that were changed or removed still listed

`take_screenshot`.

---

## Step 4: Core Web Vitals (if issues flagged)

If the overview showed any "needs improvement" or "poor" pages, navigate to:
```
https://search.google.com/search-console/core-web-vitals?resource_id=DOMAIN
```

`wait_for` "Mobiel" / "Mobile", then `take_snapshot`.

**Capture:**
- Mobile: good / needs improvement / poor counts
- Desktop: good / needs improvement / poor counts
- Which metric is failing (LCP / CLS / INP) — click into the report if counts > 0

---

## Step 5: Shopping Reports (for e-commerce)

### Product Snippets
```
https://search.google.com/search-console/r/product?resource_id=DOMAIN
```

`wait_for` "Productfragmenten" / "Product snippets", `take_snapshot`.

Capture: valid count, invalid count, any error types listed.

**"Weergave van item verbeteren" / "Improve item display" warnings** are non-critical but worth fixing — items are valid but missing recommended fields. Common ones:

| Missing field | Meaning | Fix |
|---|---|---|
| `priceValidUntil` (in `offers` / `hasVariant.offers`) | No offer expiry date | Add to product structured data |
| `aggregateRating` | No review score on product | Wire to your reviews source |
| `review` | No individual reviews | Wire to your reviews source |
| `availability` | Missing in-stock / out-of-stock signal | Add to each `offer` |
| `shippingDetails` / `hasMerchantReturnPolicy` | Missing merchant info | Add once at template level |

**To see which URLs are affected:** click the warning row in the table — it drills into a list of exact URLs and item names. Always check this before assuming all products are affected (often just 2–5 pages).

**After fixing**, click "Oplossing valideren" / "Validate Fix" to trigger an immediate re-crawl of the affected URLs. GSC will report back within a few days.

### Merchant Listings
```
https://search.google.com/search-console/r/merchant-listings?resource_id=DOMAIN
```

`wait_for` "Verkopersvermeldingen" / "Merchant listings", `take_snapshot`.

Capture: valid count, invalid count, any missing fields (price, availability, image, etc.).

---

## Step 6: Security & Manual Actions

Click the "Beveiligingsproblemen en handmatige acties" / "Security & Manual Actions" section in the sidebar, or navigate to:
```
https://search.google.com/search-console/manual-actions?resource_id=DOMAIN
```

Check for any manual penalties. If any are present, they are the highest priority fix.

---

## Producing the Report

After completing all sections, output a structured report:

```markdown
## Search Console Audit: [domain] — [date]

### Summary
- Indexed: X pages
- Not indexed: Y pages (Z reasons)
- Core Web Vitals: X good / Y needs improvement / Z poor (mobile)
- Manual actions: none / [list]

### Issues by Priority

#### P0 — Fix immediately
[Any manual actions, security issues, or >50% drop in impressions]

#### P1 — High impact
[Large non-indexing buckets with fixable causes: 404s, redirects in sitemap, missing canonicals]

#### P2 — Medium impact
[Canonical misconfigurations, Google overriding canonicals, thin content]

#### P3 — Monitor
[Noindex pages to verify intentional, robots.txt blocks, small counts]

### Fixes

**[Issue name]** — X pages affected
- Root cause: ...
- Fix: ...
- How to verify: ...

### What's Healthy
[List things that are working well — don't just report problems]
```

---

## GSC Navigation Reference

All URLs use `resource_id=DOMAIN` where DOMAIN is the URL-encoded property URL.

```
Overview:            /search-console?resource_id=DOMAIN
Page indexing:       /search-console/index?resource_id=DOMAIN
Sitemaps:            /search-console/sitemaps?resource_id=DOMAIN
Core Web Vitals:     /search-console/core-web-vitals?resource_id=DOMAIN
HTTPS:               /search-console/https?resource_id=DOMAIN
Product snippets:    /search-console/r/product?resource_id=DOMAIN
Merchant listings:   /search-console/r/merchant-listings?resource_id=DOMAIN
Merchant opps:       /search-console/merchant-opportunities?resource_id=DOMAIN
Manual actions:      /search-console/manual-actions?resource_id=DOMAIN
Links:               /search-console/links?resource_id=DOMAIN
Performance:         /search-console/performance/search-analytics?resource_id=DOMAIN
URL inspection:      /search-console/inspect?resource_id=DOMAIN&url=URL_TO_INSPECT
```

Base URL: `https://search.google.com`

---

## GSC Quirks to Know

- **UI is localised.** The account language affects all button and label text. Dutch accounts show "Niet geïndexeerd", "Pagina's", "Verzonden sitemaps" etc. Use the snapshot as-is — the a11y tree will show the localised strings.
- **Data lag.** GSC data is typically 2–3 days behind. Don't panic if a fix you made yesterday isn't reflected.
- **Indexing report only shows ~1000 URLs** in the drilldown. For large sites, use the Export button to get the full list.
- **Validation failures** on canonical/duplicate issues mean GSC ran a re-crawl after you asked it to validate and the issue was still present — not that validation is in progress.
- **"Crawled – currently not indexed"** is the hardest to fix. Google has no obligation to index every page. Common causes: thin product variant pages, faceted navigation URLs, low E-E-A-T signals, content too similar to already-indexed pages.
- **Sitemap discovered pages ≠ indexed pages.** The sitemap number is URLs Google found in the sitemap file. It doesn't mean those pages are indexed.

---

## Fixing Structured Data Warnings

When GSC flags missing fields, the fix is always the same shape regardless of platform: locate where the JSON-LD is emitted, add or correct the field, verify.

### 1. Find where structured data is emitted

Structured data lives in a `<script type="application/ld+json">` block. Depending on the stack, it may come from:

- **CMS / theme layer** — a template, partial, or snippet that wraps product/article/page data (Shopify Liquid, WordPress PHP, Hugo/Jekyll templates, etc.)
- **Framework** — a component or meta-handler in Next.js, Nuxt, Remix, Astro, SvelteKit
- **Plugin or app** — an SEO plugin (Yoast, Rank Math) or platform built-in (Shopify's `structured_data` filter, WooCommerce schema, Magento's SD module)
- **Custom backend** — SSR output assembled in the server

Fetch a product/article URL and grep for `application/ld+json` to confirm the current output before touching anything. If multiple blocks exist, note all of them — duplicates cause validation conflicts.

### 2. Choose your insertion point

Decide whether to **extend** the existing block or **replace** it.

- Extend when the emitter is under your control (custom template, your own component) — just add the missing fields.
- Replace when the emitter is a black-box filter/plugin that can't be extended. Disable or remove it, then render your own JSON-LD block at template level.

Centralise where possible: one snippet / component that dispatches by page type (`home`, `product`, `collection/category`, `article/post`, `page`) is easier to maintain than schema scattered across templates.

### 3. Add the commonly missing fields

**`priceValidUntil`** — typically one year out, rolling. Expose a date variable in the template and inject it into every `offer`:

```json
"priceValidUntil": "2027-12-31"
```

**`availability`** — map to schema.org values:

```json
"availability": "https://schema.org/InStock"
```
(or `OutOfStock`, `PreOrder`, `Discontinued`)

**`aggregateRating`** — only emit when real reviews exist. Pull from whatever reviews source the project uses (CMS metafield, reviews API, database column). Skip the field entirely when count is 0 — empty ratings are a validation error.

```json
"aggregateRating": {
  "@type": "AggregateRating",
  "ratingValue": 4.7,
  "reviewCount": 128
}
```

**`shippingDetails` / `hasMerchantReturnPolicy`** — merchant-level fields. Usually added once at template level, not per-product, unless policies differ by SKU.

### 4. Verify the fix

Three checks, in this order:

1. **View-source the page.** Confirm exactly one JSON-LD block per `@type`. Duplicates (two `Product` blocks, two `BlogPosting` blocks) are the #1 cause of GSC flagging "conflicting" schema.
2. **Run the [Rich Results Test](https://search.google.com/test/rich-results)** on a sample URL from each template type. It parses schema the way Google does and reports missing fields immediately.
3. **Trigger GSC validation.** In the GSC warning row, click "Oplossing valideren" / "Validate Fix". Google will re-crawl the affected URLs and report back within a few days.

Quick duplicate check from the terminal:

```python
import urllib.request, re, json

def check(url):
    html = urllib.request.urlopen(urllib.request.Request(url, headers={'User-Agent': 'Mozilla/5.0'})).read().decode()
    blocks = re.findall(r'<script[^>]*ld\+json[^>]*>(.*?)</script>', html, re.DOTALL)
    return [json.loads(b).get('@type') for b in blocks]
```

Each page type should output exactly the schemas it needs (e.g. `['WebSite', 'Organization']` on homepage, `['Product']` or `['ProductGroup']` on a PDP). Unexpected duplicates mean an old emitter is still active alongside the new one.

### Platform-specific gotchas worth knowing

- **Shopify** — the built-in `| structured_data` Liquid filter cannot be extended. Replace it by rendering a custom snippet from `layout/theme.liquid` and removing the filter calls from the theme sections. Variables assigned in `theme.liquid` are NOT accessible inside sections (isolated scope).
- **WordPress** — Yoast and Rank Math both emit their own graph. Disable the one you're not using, or use their filter hooks (`wpseo_schema_*`) to extend rather than duplicate.
- **Next.js / Remix / Nuxt** — emit JSON-LD via a `<script>` element in the page component or layout. Beware hydration: use `dangerouslySetInnerHTML` (or framework equivalent) and render server-side only.
- **Headless builds** — if the CMS ships schema AND the frontend also renders it, you'll get duplicates. Pick one source of truth.

---

## Important Principles

1. **Never log in.** If the user isn't already signed in to GSC, stop and ask them to sign in first. You cannot handle Google authentication.
2. **Read before concluding.** Take a snapshot of every report before making claims. GSC numbers change frequently.
3. **Prioritise by page count.** A 78-page canonical failure beats a 1-page robots.txt issue.
4. **Separate fixable from "monitor".** Some issues (crawled-not-indexed) are Google's choice. Be honest about what can and can't be forced.
5. **Screenshot each section.** Gives the user a visual record and lets you spot things the a11y snapshot misses.
6. **Check both mobile and desktop** for Core Web Vitals — they often differ.
