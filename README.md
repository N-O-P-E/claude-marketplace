# Studio N.O.P.E. Claude Marketplace
*We are Not Of Planet Earth*

Claude Code plugins by [Studio N.O.P.E.](https://github.com/N-O-P-E).

## Plugins

| Plugin | Description |
|--------|-------------|
| **gcp-setup** | Automates Google Cloud Console setup via browser. Walks through project creation, API enabling, service accounts, IAM, billing, OAuth, Cloud Run, and firewall config. |

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

## How gcp-setup works

1. **Discovery:** Gathers all requirements before touching the browser. If invoked mid-feature, it auto-detects GCP needs from your codebase (package dependencies, config files, env vars).
2. **Planning:** Presents a numbered plan of every step, flagging which ones require manual action (auth, billing confirmation).
3. **Execution:** Opens Chrome and walks through GCP Console step by step, taking screenshots to verify each action.
4. **Manual handoffs:** Pauses for Google sign-in, billing confirmation, 2FA, and terms acceptance.

## About Studio N.O.P.E.

Creative Solution Engineers using AI's infinite possibilities to help humans realise their dreams.

We're [@tijsluitse](https://github.com/tijsluitse) and [@basfijneman](https://github.com/basfijneman), two guys on a mission to make the tools we use every day faster, smarter, and less annoying. This marketplace is where we ship the things that help us deliver and work faster, so they can do the same for you.

We automate the boring parts. The GCP setup clicks, the configuration loops, the browser handholding, so you can stay in flow and ship what actually matters. Every plugin here is something we use ourselves.

Want to work with us? We help teams build smarter workflows with AI-powered tooling, Shopify development, and creative engineering. Reach out at [info@studionope.nl](mailto:info@studionope.nl) or visit [studionope.nl](https://studionope.nl).

## License

MIT
