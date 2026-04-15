# search-console

Audits Google Search Console via browser automation. Reads every report that matters, returns a prioritised fix list, and implements fixes for common structured data issues — including a complete Shopify implementation pattern.

## Install

```
/plugin install search-console@nope-marketplace
```

Requires the [N-O-P-E marketplace](https://github.com/N-O-P-E/nope-marketplace):
```
/plugin marketplace add n-o-p-e/nope-marketplace
```

## Requirements

- [Chrome DevTools MCP](https://github.com/ChromeDevTools/chrome-devtools-mcp) configured
- Chrome browser running
- Already signed in to Google Search Console in Chrome (sign-in is always manual)

## How it works

```
Overview → Indexing → Sitemaps → Shopping → Security → Report
```

Opens GSC in Chrome and reads each report in sequence:

**Overview** — Indexed vs non-indexed page counts, Core Web Vitals summary, shopping report health, recent anomaly cards.

**Indexing** — Full breakdown of why pages aren't indexed (canonical issues, crawled-not-indexed, redirects, 404s, noindex, robots.txt). Drills into each reason to get the exact affected URLs.

**Sitemaps** — Submitted sitemaps, status, last read date, discovered page count. Flags gaps between sitemap coverage and total known pages.

**Shopping** — Product snippet valid/invalid counts, merchant listing issues, missing recommended fields (`priceValidUntil`, `aggregateRating`, `review`).

**Security** — Manual actions and security issues. If any are present, they're flagged as P0.

**Report** — Prioritised P0→P3 fix list with root causes, fixes, and verification steps.

## What you get

A structured audit report:

```
P0 — Fix immediately   (manual actions, security, >50% impression drops)
P1 — High impact       (404s, redirects in sitemap, missing canonicals)
P2 — Medium impact     (canonical misconfigs, thin content)
P3 — Monitor           (intentional noindex, small counts)
```

## Fixing product structured data on Shopify

When GSC flags missing fields like `priceValidUntil` or `aggregateRating`, the skill implements the fix directly in your Shopify theme:

- Replaces Shopify's built-in `structured_data` filter with a single custom `snippets/structured-data.liquid` that handles all page types (`index`, `product`, `collection`, `article`, `blog`, `page`) from one `if/elsif` dispatcher in the `<head>`
- Adds `priceValidUntil` (1 year rolling) to all product offers
- Wires `aggregateRating` to Judge.me's native metafield (`judgeme.review_widget_data`) and the standard Shopify `reviews` namespace — appears automatically once reviews exist
- Triggers GSC validation after deploying the fix

After fixing, the only maintenance on Horizon theme updates is removing two built-in `structured_data` filter calls from sections — everything else lives outside Horizon's files.

## Troubleshooting

- **GSC UI in Dutch** — The skill handles localised strings automatically (`Niet geïndexeerd`, `Pagina's`, etc.).
- **"No items detected" in Rich Results Test** — Often a CDN cache hit. Test with a different URL on the same template, or wait a few minutes.
- **Duplicate schemas after Shopify theme update** — Remove `{{ closest.product | structured_data }}` from `sections/product-information.liquid` and `{{ article | structured_data }}` from `sections/main-blog-post.liquid`.
- **Browser not responding** — See the [Chrome DevTools MCP troubleshooting guide](https://github.com/ChromeDevTools/chrome-devtools-mcp/blob/main/docs/troubleshooting.md).
