# gcp-setup

Automates Google Cloud Console setup via browser — so you never have to click through project creation, API enabling, service accounts, IAM, billing, OAuth, or Cloud Run config again.

Invoke it directly when setting up a new project, or mid-feature when you realise you need cloud infrastructure. It reads your codebase to figure out what needs to be set up and only asks about what it can't determine itself.

## Install

```
/plugin install gcp-setup@nope-marketplace
```

Requires the [N-O-P-E marketplace](https://github.com/N-O-P-E/nope-marketplace):
```
/plugin marketplace add n-o-p-e/nope-marketplace
```

## Requirements

- [Chrome DevTools MCP](https://github.com/ChromeDevTools/chrome-devtools-mcp) configured
- Chrome browser running
- A Google account with GCP access
- A billing account (if creating projects or enabling paid APIs)

## How it works

```
Discovery → Planning → Execution → Verification
```

**Discovery** — Gathers everything it needs before touching the browser. When invoked mid-feature, scans your codebase (package dependencies, config files, env vars, CI configs) to auto-detect which GCP services you need. Only asks about what it can't figure out itself.

**Planning** — Presents a numbered list of every step. Flags which ones require manual action (auth, billing, 2FA) upfront so there are no surprises.

**Execution** — Drives Chrome through GCP Console step by step. Uses direct URLs rather than clicking through menus. Takes screenshots to verify each step worked.

**Manual handoffs** — Pauses and waits for you to complete: Google sign-in, billing confirmation, 2FA prompts, Terms of Service. Never handles credentials.

## Common setups

**Standard web app** — Cloud Run + Secret Manager + Artifact Registry, two service accounts (`app-runtime` + `ci-deployer`).

**CI/CD pipeline** — Service account + Artifact Registry + GitHub Actions secret. Outputs the JSON key ready to paste.

**Full project from scratch** — New project + billing link + APIs + service accounts. Everything.

**Something specific** — Describe what you need, it figures out the steps.

## After setup

Writes a `.gcp-setup.md` summary to your project root with project ID, enabled APIs, service account emails, and any follow-up notes. Subsequent invocations read this file to avoid duplicating work.

## Troubleshooting

- **"Service account key creation is disabled"** — Your org has Secure by Default policies. The skill includes `gcloud` commands to resolve this.
- **Browser not responding** — See the [Chrome DevTools MCP troubleshooting guide](https://github.com/ChromeDevTools/chrome-devtools-mcp/blob/main/docs/troubleshooting.md).
- **GCP Console in wrong language** — The skill handles localised UI automatically.
