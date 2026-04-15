# Studio N.O.P.E. Claude Marketplace
*We are Not Of Planet Earth*

## About Studio N.O.P.E.

Creative Solution Engineers using AI's infinite possibilities to help humans realise their dreams.

We're [@tijsluitse](https://github.com/tijsluitse) and [@basfijneman](https://github.com/basfijneman) — two guys who believe the best tools are the ones that get out of your way. We built this marketplace because setup shouldn't require clicking through a dozen consoles and copy-pasting from stale docs. It should be one command, some sensible defaults, done — with tools you can read, fork, and shape.

We made this open source because we think everybody deserves useful tools, not just the people who can afford them. When you can automate the boring parts without friction, you ship more of what actually matters. Open source means the community can shape this into exactly what they need.

Want to work with us? We help teams build smarter workflows with AI-powered tooling, Shopify development, and creative engineering. Reach out at [info@studionope.nl](mailto:info@studionope.nl) or visit [studionope.nl](https://studionope.nl).

## Plugins

| Plugin | Description |
|--------|-------------|
| [**gcp-setup**](plugins/gcp-setup) | Drives Chrome through Google Cloud Console — project creation, APIs, service accounts, IAM, billing, OAuth, Cloud Run. Auto-detects what your project needs from the codebase. |
| [**search-console**](plugins/search-console) | Audits Google Search Console via browser. Surfaces indexing issues, sitemap gaps, Core Web Vitals, and structured data warnings — then implements the fixes. |

## Install

Add the marketplace to Claude Code:

```
/plugin marketplace add n-o-p-e/claude-marketplace
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

## Security

Found a vulnerability? See [SECURITY.md](SECURITY.md) for how to report it.

## License

[MIT](LICENSE)
