---
name: gtm-workflow
description: Inspect, test, and modify Google Tag Manager containers via Chrome DevTools MCP. Use when user mentions "GTM", "Tag Manager", "Google Tag Manager", "dataLayer", "tag firing", "GA4 event", "Meta Pixel event", "scroll tracking", "tracking setup", "affiliate click tracking", "verify event", "Tag Assistant", "publish GTM", or needs to check if an event lands in Google Analytics / Meta / Pinterest / Clarity.
---

# gtm-workflow

Drive Google Tag Manager end-to-end through a real Chrome via chrome-devtools-mcp: inventory, verify, edit, preview, publish.

## First principles

1. **Reads before writes.** Always inventory the container before proposing changes — tags the user "forgot existed" are common.
2. **Workspaces stage; Publish ships.** Work in the GTM draft workspace. Nothing goes live until the explicit Publish step.
3. **Preview before Publish.** Use Tag Assistant to confirm tags fire on the real site before pushing to production.
4. **Events come from two places.** A tag fires because of (a) a dataLayer `push` from your site code or (b) an auto-tracked DOM event (click, scroll, visibility). Diagnose which source is missing before assuming the tag is broken.
5. **Every publish gets a version name and description.** Future-you needs to roll back.

## The loop

```
Inventory → Instrument → Verify → (optionally) Modify → Preview → Publish
```

For any GTM task, identify which phase you're starting in:

- **"What's set up?"** → Inventory only. See [references/inspect.md](references/inspect.md).
- **"Is X firing correctly?"** → Instrument + Verify. See [references/instrument.md](references/instrument.md).
- **"Add X / fix trigger on Y"** → Modify (+ Inventory first to know what exists). See [references/modify.md](references/modify.md).
- **"Ship today's changes"** → Preview + Publish. See [references/preview.md](references/preview.md) and [references/publish.md](references/publish.md).

## Phase 1 — Connect

Before any other work:

1. Confirm the user has chrome-devtools-mcp available. Invoke `list_pages`. If it errors with `browser already running`, follow [references/troubleshooting.md](references/troubleshooting.md).
2. Navigate to `https://tagmanager.google.com/#/home`.
3. Check auth: `evaluate_script` for `document.body.innerText`. If the page shows account listings, user is signed in. If it redirects to `accounts.google.com`, the user needs to sign in themselves — show them the tab and wait.
4. Read the account list. Ask the user which container they want if there's more than one. Each container row is a link to `/container/accounts/<account-id>/containers/<container-id>` — grab the real IDs from the href, never guess.

## Phase 2 — Inventory (always)

Navigate to the container's tags, triggers, and variables lists and capture every row. Detailed procedure + example output shapes in [references/inspect.md](references/inspect.md).

Output should be a table the user can read:

- Tags: name, type, trigger(s), last edited
- Triggers: name, event type, filter, tag count (orphans = tag count 0)
- Variables: name, type

Flag obvious smells:

- Tags with no trigger (orphan)
- Triggers with no tag using them (orphan)
- Two tags loading the same GA4 / Google Tag (duplicate `page_view`)
- Custom HTML tags on All Pages that duplicate native tag templates

## Phase 3 — Instrument & Verify

When the user asks "is X firing?" or "we added a dataLayer push but it doesn't show up":

1. **On the live site**: inject a dataLayer hook + a network request filter. See [references/instrument.md](references/instrument.md) for the exact code.
2. Reset the capture, then have the user (or the skill) perform the action.
3. Report:
   - DataLayer events that fired (name + full payload)
   - GA4 / Meta / Pinterest / Clarity / other beacons that landed (destination + event name + params)
   - Missing: expected events that did NOT fire

The most common diagnosis: dataLayer push happens but no GTM tag listens for that event name → need a Custom Event trigger.

## Phase 4 — Modify

All modifications happen in the draft workspace. Always:

1. Inventory first.
2. Explain to the user what will change and that it will NOT go live until Publish.
3. Drive the UI (full procedures in [references/modify.md](references/modify.md)).
4. Save each change.
5. Do NOT publish yet.

Common modifications documented in [references/patterns.md](references/patterns.md):

- Create a Custom Event trigger for a dataLayer event name
- Create a GA4 Event tag with parameters pulled from dataLayer variables
- Fix a tag that's wired to the wrong trigger
- Add scroll-depth tracking (enable built-in variables, create trigger, create GA4 tag)
- Add Meta Pixel custom event via Custom HTML tag
- Remove / pause an orphan or duplicate tag

## Phase 5 — Preview

Before Publish:

1. Click **Voorbeeld** (Preview) in the GTM UI. A new tab opens Tag Assistant.
2. Enter the live-site URL and Connect. Tag Assistant launches a second tab with `?gtm_debug=` appended.
3. Exercise the actions that should fire the new/changed tags.
4. Switch to the Tag Assistant tab and confirm the expected tags appear under **Geactiveerde tags** (Fired Tags) with the right count.

Full procedure, including switching between pages in chrome-devtools-mcp: [references/preview.md](references/preview.md).

## Phase 6 — Publish

Only after Preview confirms correct behaviour:

1. Click **Verzenden** (Submit).
2. Fill a version name and description. Both mandatory for traceability. Every change should be summarised in the description.
3. Publish to "Live" environment.

Full procedure + version naming conventions: [references/publish.md](references/publish.md).

## Edge cases and gotchas

- **Chrome profile locked** — stale lock files in `~/.cache/chrome-devtools-mcp/chrome-profile/`. See [references/troubleshooting.md](references/troubleshooting.md).
- **UI clicks time out** — GTM's SPA is sometimes slow to hydrate. Snapshot → wait 1.5s → re-snapshot → click.
- **"Already exists" on metafield-like ops** — Shopify-style TAKEN errors. Look up by key and update instead of create.
- **Duplicate `page_view` from GA4** — a common config error. Check if both `Google-tag G-XXXX` and a Custom HTML `gtag.js` loader coexist on All Pages.
- **Consent defaults to granted without CMP** — flag this for GDPR compliance.

## Anti-patterns

- ❌ **Publishing without Preview.** Never. Always preview unless the change is a pure rename.
- ❌ **Creating a tag when one already exists for the same event.** Always inventory first.
- ❌ **Using URL IDs you guessed.** Container and account IDs come from the home page's account list, not from memory.
- ❌ **Clicking "Continue to Site" in Tag Assistant reconnect dialogs** — if the debug tab closes, re-run Preview from the GTM UI.

## Available references

- [references/setup.md](references/setup.md) — connect to GTM, resolve profile locks, handle auth
- [references/inspect.md](references/inspect.md) — inventory tags / triggers / variables
- [references/instrument.md](references/instrument.md) — dataLayer hook + network sniffer
- [references/modify.md](references/modify.md) — UI automation for creating/editing tags and triggers
- [references/preview.md](references/preview.md) — Tag Assistant workflow
- [references/publish.md](references/publish.md) — safe publish with version naming
- [references/patterns.md](references/patterns.md) — ready-to-apply recipes (GA4 custom events, scroll, Meta Pixel, etc.)
- [references/troubleshooting.md](references/troubleshooting.md) — common errors and fixes
