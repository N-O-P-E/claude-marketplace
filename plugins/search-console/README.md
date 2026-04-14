# search-console

Audits Google Search Console via browser automation. Reads every report that matters, returns a prioritised fix list, and helps you implement fixes for common structured data issues on any stack.

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

## Fixing structured data warnings

When GSC flags missing fields like `priceValidUntil`, `aggregateRating`, or `availability`, the skill walks a platform-agnostic fix loop:

- **Locate the emitter** — finds where JSON-LD is currently output (CMS template, framework component, SEO plugin, or custom SSR) and whether to extend or replace it
- **Add the missing fields** — `priceValidUntil` (rolling), `availability`, `aggregateRating` (only when real reviews exist), merchant-level fields like `shippingDetails`
- **Verify before claiming done** — view-source for duplicates, Rich Results Test on sample URLs, then trigger GSC validation so Google re-crawls

Platform-specific gotchas are flagged as they come up — Shopify's locked `structured_data` filter, WordPress plugins that emit their own graph, Next.js/Nuxt hydration pitfalls — but the core workflow works for any stack that outputs HTML.

## Troubleshooting

- **GSC UI in Dutch** — The skill handles localised strings automatically (`Niet geïndexeerd`, `Pagina's`, etc.).
- **"No items detected" in Rich Results Test** — Often a CDN cache hit. Test with a different URL on the same template, or wait a few minutes.
- **Duplicate schemas after a theme/plugin update** — Check view-source for multiple JSON-LD blocks of the same `@type`. Usually a built-in emitter got re-enabled alongside your custom one; disable whichever is redundant.
- **Browser not responding** — See the [Chrome DevTools MCP troubleshooting guide](https://github.com/ChromeDevTools/chrome-devtools-mcp/blob/main/docs/troubleshooting.md).
