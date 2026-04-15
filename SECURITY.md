# Security Policy

Thanks for helping keep Studio N.O.P.E. and its users safe.

## Reporting a Vulnerability

**Please do not open a public GitHub issue for security vulnerabilities.**

If you believe you've found a security issue in any plugin in this marketplace, report it privately:

- **Email:** [info@studionope.nl](mailto:info@studionope.nl) with subject line `SECURITY: <short description>`
- **GitHub:** Use [private vulnerability reporting](https://github.com/N-O-P-E/claude-marketplace/security/advisories/new) on this repository

Please include:

- A description of the issue and its impact
- Steps to reproduce (proof-of-concept welcome)
- The affected plugin(s) and version(s)
- Any suggested mitigation, if you have one

We'll acknowledge your report within 3 business days and aim to provide a status update within 10 business days. Once the issue is confirmed, we'll work on a fix and coordinate disclosure with you.

## Scope

This policy covers the plugins published from this repository:

- `gcp-setup`
- `search-console`

Both plugins drive a local browser via [chrome-devtools-mcp](https://github.com/ChromeDevTools/chrome-devtools-mcp) and operate against your own Google accounts and projects. They do not ship or run remote services.

**Out of scope:**

- Vulnerabilities in Claude Code itself — report those to [Anthropic](https://github.com/anthropics/claude-code/security)
- Vulnerabilities in `chrome-devtools-mcp` — report those to the [ChromeDevTools project](https://github.com/ChromeDevTools/chrome-devtools-mcp/security)
- Vulnerabilities in Google Cloud, Google Search Console, or other third-party services the plugins interact with — report those to the respective vendor
- Issues that require an already-compromised local machine or attacker-controlled Claude Code configuration

## Safe Harbor

We support good-faith security research. If you make a reasonable effort to comply with this policy, we will:

- Not pursue or support legal action against you for your research
- Work with you to understand and resolve the issue quickly
- Credit you in the fix announcement if you'd like

Please avoid privacy violations, destruction of data, and interruption or degradation of services during your testing. Only test against your own accounts and projects.

## Supported Versions

Security fixes are applied to the latest released version of each plugin. Older versions are not maintained.
