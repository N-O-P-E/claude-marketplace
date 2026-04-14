# Studio N.O.P.E. Claude Marketplace
*We are Not Of Planet Earth*

![Claude Code × Google Search & Cloud — open-source integration for AI, SEO, and infrastructure](assets/banner.png)

## About Studio N.O.P.E.

Creative Solution Engineers using AI's infinite possibilities to help humans realise their dreams.

We're [@tijsluitse](https://github.com/tijsluitse) and [@basfijneman](https://github.com/basfijneman) — two guys who believe the best tools are the ones that get out of your way. We built this marketplace because setup shouldn't require clicking through a dozen consoles and copy-pasting from stale docs. It should be one command, some sensible defaults, done — with tools you can read, fork, and shape.

We made this open source because we think everybody deserves useful tools, not just the people who can afford them. When you can automate the boring parts without friction, you ship more of what actually matters. Open source means the community can shape this into exactly what they need.

Want to work with us? We help teams build smarter workflows with AI-powered tooling, Shopify development, and creative engineering. Reach out at [info@studionope.nl](mailto:info@studionope.nl) or visit [studionope.nl](https://studionope.nl).

## Plugins

Both plugins drive a real Chrome browser through [chrome-devtools-mcp](https://github.com/ChromeDevTools/chrome-devtools-mcp) — they click, type, and screenshot the actual Google consoles so you don't have to. They never handle your credentials; you sign in once in Chrome and the plugins take it from there.

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

## Install

Add the marketplace to Claude Code:

```
/plugin marketplace add n-o-p-e/nope-marketplace
```

Install a plugin:

```
/plugin install gcp-setup@nope-marketplace
/plugin install search-console@nope-marketplace
```

## Chrome DevTools MCP

Both plugins drive Chrome via [chrome-devtools-mcp](https://github.com/ChromeDevTools/chrome-devtools-mcp). Set it up once:

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
