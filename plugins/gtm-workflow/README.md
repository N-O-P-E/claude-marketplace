# gtm-workflow — Google Tag Manager, automated

Read, verify, edit, preview and publish Google Tag Manager containers without clicking through the GTM UI yourself. Drives a real Chrome via [chrome-devtools-mcp](https://github.com/ChromeDevTools/chrome-devtools-mcp) — no API keys, no service accounts. You sign in to Google once; the skill does everything inside your existing session.

## What it does

- **Inventory a container** — lists every tag, trigger, and variable with its wiring so you can see in one go what's set up and what's missing.
- **Verify events fire** — injects a dataLayer hook + network sniffer on your live site, walks through user actions, and reports which GTM tags fired, with what parameters, to which destinations (GA4, Meta, Pinterest, Clarity, Klaviyo, etc.).
- **Add / edit tags and triggers** — drives the GTM UI click-by-click to create new Custom Event triggers, GA4 event tags, scroll-depth tracking, Meta Pixel custom events, etc.
- **Preview before publish** — runs Tag Manager Preview mode, loads the site in the debug window, confirms the changes trigger correctly in Tag Assistant, then publishes with a version name and description for traceability.
- **Catches the usual gotchas** — tag-firing on the wrong trigger, orphan tags without triggers, duplicate page_view loaders, missing consent signals, redundant Custom HTML overlaps.

## When to use it

- "Is the Meta Pixel actually firing a `Purchase` event on checkout?"
- "We added a dataLayer push for `product_click` but nothing shows up in GA4 — what's wrong?"
- "Add a scroll-depth trigger and wire it to GA4."
- "Rename every GA4 event param that's still using old snake_case."
- "Show me what tags are in our container and which ones have no trigger."
- "Test our changes in Preview, then publish version 5 with a proper description."

## How it works

The skill runs everything through chrome-devtools-mcp against two tabs:

1. **tagmanager.google.com** — reads/writes the container via the UI.
2. **the live site under Preview** (auto-launched by Tag Manager) — where it simulates clicks, scroll, and navigation to verify events.

It never stores your credentials. If your Chrome session is already signed into Google, the skill uses that session. If not, you'll get a sign-in screen in the MCP-controlled browser and you handle it yourself.

## Safety

- **Reads before writes.** Every run opens with an inventory pass so you know what exists before anything is touched.
- **Stages in a workspace.** Changes accumulate in the GTM draft workspace. Nothing is live until the explicit Publish (Verzenden) step.
- **Always previews first.** Preview runs before Publish by default. You can skip it for trivial changes (renames) but the skill will warn.
- **Version descriptions required.** Every published version gets a human-readable name and summary of changes, so rollback via "Versies" is trivial if something breaks.

## Not a replacement for

- **Server-side GTM** — this drives the web container. SGTM works via API.
- **Google Analytics 4 config** — the skill wires tags to GA4 but doesn't create GA4 properties or edit Enhanced Measurement.
- **Publishing without review** — if you want fully unattended publishes, use the [Tag Manager API](https://developers.google.com/tag-platform/tag-manager/api/v2) instead.

## Author

Built by [Studio N.O.P.E.](https://studionope.nl) for teams that need analytics to Just Work™ without a week of tab-switching.
