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
   - "What domain should I audit? (e.g. https://perfectpillows.nl/)"
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

**Common sitemap issues for Shopify stores:**
- Only `sitemap.xml` submitted but not the sub-sitemaps (`sitemap_products_1.xml`, `sitemap_collections_1.xml`, `sitemap_pages_1.xml`, `sitemap_blogs_1.xml`)
- Sitemap contains redirected or 404 URLs
- Sitemap was never submitted (only auto-discovered)

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

**"Weergave van item verbeteren" / "Improve item display" warnings** are non-critical but worth fixing — items are valid but missing recommended fields. Common ones on Shopify:

| Missing field | Meaning | Fix |
|---|---|---|
| `priceValidUntil` (in `hasVariant.offers`) | No offer expiry date on product variants | Add to structured data snippet |
| `aggregateRating` | No review score on product | Wire to reviews app metafields |
| `review` | No individual reviews | Wire to reviews app |

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

## Shopify-Specific Fixes

For Shopify stores, these are the most common issues and fixes:

**Canonical tag problems**
- Shopify automatically adds canonical tags pointing to the product's main URL
- Variant URLs (`?variant=123`) should canonicalise to the base product URL
- Collection-scoped product URLs (`/collections/xxx/products/yyy`) should canonicalise to `/products/yyy`
- Check the theme's `meta-tags.liquid` snippet — if a `{{ canonical_url }}` is not being output correctly, all pages are affected

**Incomplete sitemap**
- Shopify's `/sitemap.xml` is an index file linking to sub-sitemaps
- Submit `sitemap.xml` to GSC — it will auto-discover `sitemap_products_1.xml`, `sitemap_collections_1.xml`, etc.
- Shopify excludes pages with `noindex` from the sitemap automatically

**404 pages**
- Use the Shopify Admin → Online Store → Navigation → URL Redirects to add 301s
- Or add redirects programmatically via the Admin API

**Redirect pages in index**
- Any URL in the sitemap that now redirects needs to be removed or the sitemap regenerated
- Old collection/product URLs that were changed are common culprits

**Noindex on important pages**
- Shopify adds `noindex` to search results pages, account pages, cart, checkout — this is correct
- If collection or product pages are noindexed, check: theme settings → search engine indexing, and the page's SEO settings in Shopify Admin

---

## Fixing Product Snippet Warnings on Shopify (Structured Data)

When GSC flags missing fields like `priceValidUntil`, `aggregateRating`, or `review`, the fix is to replace Shopify's built-in `structured_data` filter with a custom snippet you control.

### How Shopify outputs product structured data by default

Shopify themes (e.g. Horizon) use the built-in Liquid filter in `sections/product-information.liquid`:
```liquid
<script type="application/ld+json">
  {{ closest.product | structured_data }}
</script>
```

This outputs a valid `ProductGroup` schema but omits recommended fields. You can't extend it — you have to replace it.

### The fix: one snippet in the `<head>` for all page types

Create `snippets/structured-data.liquid` and render it from `layout/theme.liquid`:

```liquid
{%- render 'meta-tags' -%}
{%- render 'structured-data' -%}   {%- comment -%} ← add this line {%- endcomment -%}
```

The snippet uses `if/elsif` on `request.page_type` to output the right schema per page:

```liquid
{%- if request.page_type == 'index' -%}
  {%- comment -%} WebSite + SearchAction {%- endcomment -%}
{%- elsif request.page_type == 'product' -%}
  {%- comment -%} ProductGroup with priceValidUntil + aggregateRating {%- endcomment -%}
{%- elsif request.page_type == 'collection' or request.page_type == 'list-collections' -%}
  {%- comment -%} ItemList {%- endcomment -%}
{%- elsif request.page_type == 'article' -%}
  {%- comment -%} BlogPosting {%- endcomment -%}
{%- elsif request.page_type == 'blog' -%}
  {%- comment -%} Blog {%- endcomment -%}
{%- elsif request.page_type == 'page' -%}
  {%- comment -%} WebPage / ContactPage / AboutPage {%- endcomment -%}
{%- endif -%}
```

Then remove `{{ closest.product | structured_data }}` from `sections/product-information.liquid` and `{{ article | structured_data }}` from `sections/main-blog-post.liquid`.

### Adding `priceValidUntil`

In the product branch, generate a date 1 year out:

```liquid
{%- assign price_valid_until = 'now' | date: '%Y' | plus: 1 | append: '-12-31' -%}
```

Add to each offer in the `hasVariant` loop:
```liquid
"priceValidUntil": {{ price_valid_until | json }},
```

Price formatting without currency symbol quirks:
```liquid
"price": "{{ variant.price | divided_by: 100 }}.{{ variant.price | modulo: 100 | prepend: '00' | slice: -2, 2 }}"
```

### Adding `aggregateRating` (requires reviews app)

`aggregateRating` only makes sense when there are real reviews. Wire it conditionally to two sources:

```liquid
{%- liquid
  assign rating_value = nil
  assign rating_count = 0

  # Source 1: standard Shopify reviews namespace (Judge.me, Loox, Okendo when sync enabled)
  if product.metafields.reviews.rating != blank
    assign rating_value = product.metafields.reviews.rating.value.rating
    assign rating_count = product.metafields.reviews.rating_count.value | default: 0

  # Source 2: Judge.me native metafield (type: json, always written when Judge.me is installed)
  elsif product.metafields.judgeme.review_widget_data != blank
    assign jdgm = product.metafields.judgeme.review_widget_data.value
    assign rating_count = jdgm.number_of_reviews | default: 0
    if rating_count > 0
      assign rating_value = jdgm.average_rating | times: 1.0
    endif
  endif
-%}

{%- if rating_value != nil and rating_count > 0 -%}
,"aggregateRating": {
  "@type": "AggregateRating",
  "ratingValue": {{ rating_value }},
  "reviewCount": {{ rating_count }}
}
{%- endif -%}
```

**Judge.me note:** Judge.me writes to `judgeme.review_widget_data` (type: `json`) automatically. To also write to the standard `reviews` namespace, enable it in Judge.me → Settings → Integrations → Shopify Product Ratings.

### Important: Shopify section scope vs layout scope

Variables assigned in `layout/theme.liquid` are **NOT accessible in sections** rendered via `{{ content_for_layout }}`. Sections run in an isolated scope. This means you cannot use a Liquid flag set in `theme.liquid` to suppress built-in structured data in a section. The only reliable fix is to **remove** the built-in filter calls from the sections entirely.

### On Horizon theme updates

`layout/theme.liquid` and `snippets/structured-data.liquid` are not Horizon-owned files — theme updates never touch them. The only post-update task is removing the built-in `structured_data` calls from the two updated sections if Horizon adds them back. Two lines, two deletes.

### Verifying the fix

Check that each page type outputs exactly one schema (no duplicates):

```python
import urllib.request, re, json

def check(url):
    html = urllib.request.urlopen(urllib.request.Request(url, headers={'User-Agent': 'Mozilla/5.0'})).read().decode()
    blocks = re.findall(r'<script[^>]*ld\+json[^>]*>(.*?)</script>', html, re.DOTALL)
    return [json.loads(b).get('@type') for b in blocks]
```

Expected output per page type:
- Homepage: `['WebSite', 'Organization']`
- Product: `['ProductGroup', 'Organization']`
- Collection: `['ItemList', 'Organization']`
- Article: `['BlogPosting', 'Organization']`
- Contact page: `['ContactPage', 'Organization']`

`Organization` is always present — injected by Shopify via `content_for_header`. Any duplicate `@type` values indicate a built-in filter is still active alongside the custom snippet.

---

## Important Principles

1. **Never log in.** If the user isn't already signed in to GSC, stop and ask them to sign in first. You cannot handle Google authentication.
2. **Read before concluding.** Take a snapshot of every report before making claims. GSC numbers change frequently.
3. **Prioritise by page count.** A 78-page canonical failure beats a 1-page robots.txt issue.
4. **Separate fixable from "monitor".** Some issues (crawled-not-indexed) are Google's choice. Be honest about what can and can't be forced.
5. **Screenshot each section.** Gives the user a visual record and lets you spot things the a11y snapshot misses.
6. **Check both mobile and desktop** for Core Web Vitals — they often differ.
