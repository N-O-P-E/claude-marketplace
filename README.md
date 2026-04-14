# Studio N.O.P.E. Claude Marketplace
*We are Not Of Planet Earth*

Claude Code plugins by [Studio N.O.P.E.](https://github.com/N-O-P-E).

## Plugins

| Plugin | Description |
|--------|-------------|
| **gcp-setup** | Automates Google Cloud Console setup via browser. Walks through project creation, API enabling, service accounts, IAM, billing, OAuth, Cloud Run, and firewall config. |
| **search-console** | Audits Google Search Console via browser. Surfaces indexing issues, sitemap gaps, Core Web Vitals, structured data warnings, and merchant listing problems — then gives actionable fixes including Shopify-specific structured data implementation. |

## Install

Add the marketplace:

```
/plugin marketplace add n-o-p-e/claude-marketplace
```

Install a plugin:

```
/plugin install gcp-setup@nope-marketplace
```

## Requirements

The `gcp-setup` plugin requires:
- Chrome DevTools MCP plugin installed and connected
- Chrome browser running
- A Google account with access to GCP

The `search-console` plugin requires:
- Chrome DevTools MCP plugin installed and connected
- Chrome browser running
- Already signed in to Google Search Console in Chrome

## How gcp-setup works

1. **Discovery:** Gathers all requirements before touching the browser. If invoked mid-feature, it auto-detects GCP needs from your codebase (package dependencies, config files, env vars).
2. **Planning:** Presents a numbered plan of every step, flagging which ones require manual action (auth, billing confirmation).
3. **Execution:** Opens Chrome and walks through GCP Console step by step, taking screenshots to verify each action.
4. **Manual handoffs:** Pauses for Google sign-in, billing confirmation, 2FA, and terms acceptance.

## How search-console works

1. **Audit:** Opens GSC in Chrome and reads every report — indexing, sitemaps, Core Web Vitals, product snippets, merchant listings, security.
2. **Report:** Returns a prioritised P0→P3 fix list with root causes and verification steps.
3. **Fix guidance:** Includes Shopify-specific fixes for structured data warnings, including a ready-to-use `structured-data.liquid` pattern that covers all page types from a single snippet.

## About Studio N.O.P.E.

Creative Solution Engineers using AI's infinite possibilities to help humans realise their dreams.

We're [@tijsluitse](https://github.com/tijsluitse) and [@basfijneman](https://github.com/basfijneman), two guys on a mission to make the tools we use every day faster, smarter, and less annoying. This marketplace is where we ship the things that help us deliver and work faster, so they can do the same for you.

We automate the boring parts. The GCP setup clicks, the configuration loops, the browser handholding, so you can stay in flow and ship what actually matters. Every plugin here is something we use ourselves.

Want to work with us? We help teams build smarter workflows with AI-powered tooling, Shopify development, and creative engineering. Reach out at [info@studionope.nl](mailto:info@studionope.nl) or visit [studionope.nl](https://studionope.nl).

## License

MIT
