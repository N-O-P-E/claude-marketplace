# Studio N.O.P.E. Claude Marketplace
*We are Not Of Planet Earth*

![Claude Code × Google Search & Cloud — open-source integration for AI, SEO, and infrastructure](assets/banner.png)

## About Studio N.O.P.E.

Creative Solution Engineers using AI's infinite possibilities to help humans realise their dreams.

We're [@tijsluitse](https://github.com/tijsluitse) and [@basfijneman](https://github.com/basfijneman) — two guys who believe the best tools are the ones that get out of your way. We built this marketplace because setup shouldn't require clicking through a dozen consoles and copy-pasting from stale docs. It should be one command, some sensible defaults, done — with tools you can read, fork, and shape.

We made this open source because we think everybody deserves useful tools, not just the people who can afford them. When you can automate the boring parts without friction, you ship more of what actually matters. Open source means the community can shape this into exactly what they need.

Want to work with us? We help teams build smarter workflows with AI-powered tooling, Shopify development, and creative engineering. Reach out at [info@studionope.nl](mailto:info@studionope.nl) or visit [studionope.nl](https://studionope.nl).

## Plugins

All plugins drive a real Chrome browser through [chrome-devtools-mcp](https://github.com/ChromeDevTools/chrome-devtools-mcp) — they click, type, and screenshot the actual dashboards so you don't have to. They never handle your credentials; you sign in once in Chrome and the plugins take it from there.

### [gcp-setup](plugins/gcp-setup) — Google Cloud Console, automated

Setting up a GCP project means clicking through a dozen consoles: create project, enable APIs, make service accounts, grant IAM roles, link billing, configure OAuth, deploy to Cloud Run. This plugin does all of that for you.

- **Auto-detects what you need** — Scans your codebase (dependencies, config, env vars, CI files) to figure out which GCP services your project requires. Only asks about what it can't determine itself.
- **Plans before acting** — Presents every step upfront, flags which ones need manual input (sign-in, billing, 2FA), then drives Chrome through the console step by step.
- **Writes a summary** — Drops a `.gcp-setup.md` in your project with project ID, enabled APIs, service account emails, and follow-up notes. Future runs read this to avoid duplicate work.

Good for: spinning up a new project from scratch, adding Cloud Run + Secret Manager to an existing app, creating CI/CD service accounts with GitHub Actions keys, or anything else that normally involves tab-switching between six GCP pages.

### [search-console](plugins/search-console) — GSC audits and fixes

Reads every report in Google Search Console that actually matters, returns a prioritised fix list, and helps you implement fixes for common structured data issues on any stack.

- **Full audit** — Walks through Indexing, Sitemaps, Shopping, Core Web Vitals, and Security. Drills into each issue to get the exact affected URLs, not just counts.
- **Prioritised report** — P0 → P3 list with root causes, fixes, and verification steps. Manual actions and security issues always surface as P0.
- **Structured data fixes** — When GSC flags missing fields like `priceValidUntil`, `aggregateRating`, or `availability`, finds where your JSON-LD is emitted (CMS template, framework component, SEO plugin, custom SSR), adds the fields, and verifies with the Rich Results Test before triggering GSC validation.

Good for: diagnosing why pages aren't ranking, catching sitemap/indexing gaps before they hurt traffic, fixing merchant listing errors, or just understanding what's actually happening in GSC without clicking through 15 reports.

### [cloudflare-connect](plugins/cloudflare-connect) — Subdomains wired to Railway, Vercel, and more

Pointing a subdomain at a hosting platform means jumping between three tabs and copy-pasting CNAME targets. This plugin drives both sides — the platform's custom-domain form AND the Cloudflare DNS editor — end-to-end.

- **Platform-first flow** — Opens Railway (or Vercel) first, attaches the subdomain, reads the CNAME target the platform reveals. Then opens Cloudflare and adds the DNS record with the captured target. No copy-paste.
- **TXT verification handled** — When Vercel asks for a `_vercel` TXT challenge, the plugin adds it automatically, waits for verification, then continues with the CNAME.
- **Safe defaults** — Defaults to DNS-only (grey cloud) because Railway and Vercel issue their own Let's Encrypt certs. Won't silently overwrite existing DNS records — always prompts.
- **Writes a summary** — Drops a `.cloudflare-setup.md` in your project with links to both dashboards and the live URL.

Good for: wiring a new subdomain to a Railway service, attaching a custom domain to a Vercel project, or any time you'd otherwise be alt-tabbing between Cloudflare and a deploy dashboard with a CNAME in your clipboard.

### [gtm-workflow](plugins/gtm-workflow) — Google Tag Manager, automated

Inspect, verify, edit, preview and publish GTM containers without clicking through the UI yourself. Reads the full container state so you can see what tags/triggers are wired where, injects a dataLayer hook on the live site to prove which events actually fire through to GA4 / Meta / Pinterest / Clarity, then drives the GTM UI end-to-end to add or fix tags with a Preview-then-Publish workflow.

- **Inventory with smells flagged** — lists every tag, trigger, and variable; flags orphan tags, unused triggers, duplicate `page_view` loaders, and consent defaults that would fail GDPR.
- **Proves events fire end-to-end** — captures every `dataLayer.push`, cross-refs with live network requests (GA4, Meta, Pinterest, TikTok, Clarity, Klaviyo, Shopify web pixels), and tells you which expected events didn't make it through.
- **Ready-to-apply recipes** — GA4 custom events, scroll-depth, Meta Pixel custom events, Pinterest/TikTok tracking, consent mode, double-fire debugging — all click-by-click procedures the skill can execute or hand to you.
- **Safe publish** — always stages in a workspace, always runs Preview + Tag Assistant validation first, always requires a version name + description so rollback via Versies is trivial.

Good for: diagnosing why a tracked event isn't showing up in GA4, wiring new dataLayer events into analytics, adding scroll/Meta Pixel/Pinterest tracking across a batch of sites, cleaning up years of accumulated GTM cruft, or just understanding what's actually live in your container.

## Install

Add the marketplace to Claude Code:

```
/plugin marketplace add n-o-p-e/nope-marketplace
```

Install a plugin:

```
/plugin install gcp-setup@nope-marketplace
/plugin install search-console@nope-marketplace
/plugin install cloudflare-connect@nope-marketplace
/plugin install gtm-workflow@nope-marketplace
```

## Chrome DevTools MCP

All plugins drive Chrome via [chrome-devtools-mcp](https://github.com/ChromeDevTools/chrome-devtools-mcp). Set it up once:

**Add to `~/.claude/settings.json`:**
```json
{
  "mcpServers": {
    "chrome-devtools-mcp": {
      "command": "npx",
      "args": ["chrome-devtools-mcp@latest"]
    }
  }
}
```

Or via Claude Code:
```
/mcp add chrome-devtools-mcp -- npx chrome-devtools-mcp@latest
```

Chrome launches automatically on first use. If something goes wrong, see the [troubleshooting guide](https://github.com/ChromeDevTools/chrome-devtools-mcp/blob/main/docs/troubleshooting.md).

## Star History

<a href="https://www.star-history.com/?repos=N-O-P-E%2Fnope-marketplace&type=date&legend=top-left">
 <picture>
   <source media="(prefers-color-scheme: dark)" srcset="https://api.star-history.com/chart?repos=N-O-P-E/nope-marketplace&type=date&theme=dark&legend=top-left" />
   <source media="(prefers-color-scheme: light)" srcset="https://api.star-history.com/chart?repos=N-O-P-E/nope-marketplace&type=date&legend=top-left" />
   <img alt="Star History Chart" src="https://api.star-history.com/chart?repos=N-O-P-E/nope-marketplace&type=date&legend=top-left" />
 </picture>
</a>

## Security

Found a vulnerability? See [SECURITY.md](SECURITY.md) for how to report it.

## License

[MIT](LICENSE)
