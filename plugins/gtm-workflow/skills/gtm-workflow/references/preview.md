# Preview mode and Tag Assistant verification

Use this reference when running GTM **Voorbeeld** (Preview) mode and verifying events in Tag Assistant. This is the single most important feedback loop for the `gtm-workflow` plugin — every workspace change should be run through Preview before `Verzenden`.

## What Preview mode does

Clicking **Voorbeeld** in the GTM UI (top-right of any container page) opens a second tab at `tagassistant.google.com`. Tag Assistant connects to the container in debug mode and, once you provide a target URL, appends `?gtm_debug=<timestamp>` to it. Opening that debug URL launches a THIRD tab with the live site running against the workspace's **unpublished** changes.

From that point forward, Tag Assistant shows, in real time:
- Every `dataLayer.push` event, numbered in the order they fired
- Every tag that fired (**Geactiveerde tags**) with "N keer geactiveerd" counts
- Every tag that did **not** fire (**Tags niet geactiveerd**) with the reason
- Variable values captured at trigger time
- Full firing context: trigger that matched, hit destination (for Google tags)

Preview mode does NOT publish anything — it only runs the workspace version in your debug session.

## Starting Preview — click-by-click

1. On any workspace page under `tagmanager.google.com/...` (tags, triggers, variables all work), click the **Voorbeeld** button in the header.
2. A new browser tab opens showing Tag Assistant. Use `list_pages` then `select_page` to switch to it.
3. Tag Assistant shows a modal titled "Tag Assistant koppelen aan uw site" with a URL input field.
4. Fill the live-site URL (e.g. `https://bythemood.com`) using `fill` or `evaluate_script`, then click the button labelled **Koppelen** (sometimes also shown as "Opent uw site in een nieuw venster").
5. A third tab opens with the live site loaded in debug mode. The URL pattern is `https://<site>/?gtm_debug=<timestamp>`.

## Switching between tabs in chrome-devtools-mcp

The MCP keeps numbered pages across tabs. After Preview is fully started you typically have:

- **Page 2** — `tagmanager.google.com/#/container/...` (the GTM edit UI where you made changes)
- **Page 3** — an earlier tab, often `osmo.supply/...` or similar (irrelevant, leave alone)
- **Page 4** — `tagassistant.google.com/...` (Tag Assistant debug panel)
- **Page 5** — `https://<site>/?gtm_debug=...` (live site in debug mode)

Switch between them with `select_page` by pageId. **Don't close any of them** — closing the debug site (Page 5) ends Preview; closing Tag Assistant (Page 4) breaks the event feed.

Typical flow:
1. `select_page` → Page 2 (GTM UI) — make a change, save.
2. `click` the **Voorbeeld** button.
3. `list_pages` to find the new Tag Assistant tab.
4. `select_page` → Tag Assistant → fill URL → click **Koppelen**.
5. `list_pages` again to find the debug site tab.
6. `select_page` → debug site → exercise events.
7. `select_page` → Tag Assistant → read results.
8. `select_page` → GTM UI for next change.

## Exercising events on the debug site

On Page 5 (the debug site), use `evaluate_script` or `click` to:

- **Scroll** to sections to trigger scroll / visibility / view events
- **Click** buttons, links, and interactive elements
- Use `e.preventDefault()` on outbound links if you want to stay on the page to watch the event fire
- **Simulate navigations** with `window.location.href = '...'` or `navigate_page`

Example — exercising a hotspot click without navigating away:

```js
async () => {
  const beforeCount = window.__btmCapturedEvents?.length || 0;
  const hotspot = document.querySelector('.moodboard-hotspot[data-product-name]');
  hotspot.addEventListener('click', (e) => e.preventDefault(), { capture: true, once: true });
  hotspot.click();
  await new Promise(r => setTimeout(r, 600));
  return window.__btmCapturedEvents?.slice(beforeCount) || [];
}
```

For page-level events (scroll, view), just scroll the page with `evaluate_script`:

```js
() => { window.scrollTo({ top: document.body.scrollHeight, behavior: 'instant' }); }
```

For outbound link tracking, clicking the link naturally will navigate the tab away and you lose the Tag Assistant feed for that click. Use the `preventDefault` pattern above, or use a capturing listener that pushes the click to dataLayer first then cancels navigation.

## Reading Tag Assistant (Page 4)

Tag Assistant's UI is a React SPA. `evaluate_script` on Page 4 and parse `document.body.innerText` for the sections you care about.

**Key sections in the DOM:**

- **Left rail: event list** — ordered list of everything that hit dataLayer. Starts with `gtm.js`, `consent`, `Initialisatie`, then numbered custom events: `19 moodboard_view`, `20 moodboard_hotspot_click`, etc.
- **Overzicht** (Overview) — summary of the selected event: what triggered it, what variables were available.
- **Geactiveerde tags** — tags that fired for the selected event, each with a firing count.
- **Tags niet geactiveerd** — tags that did NOT fire, with the reason (trigger didn't match, blocked, etc.).

Example extraction helper:

```js
async () => {
  await new Promise(r => setTimeout(r, 1500)); // let Tag Assistant catch up
  const text = document.body.innerText;
  const firedSection = (text.match(/Geactiveerde tags([\s\S]*?)Tags niet geactiveerd/) || [])[1] || '';
  const notFiredSection = (text.match(/Tags niet geactiveerd([\s\S]*?)(?=Variabelen|$)/) || [])[1] || '';
  const eventSection = (text.match(/Overzicht\s*\n[\s\S]{0,1500}/) || [])[0] || '';
  return {
    firedSection: firedSection.trim(),
    notFiredSection: notFiredSection.trim(),
    eventSection: eventSection.trim()
  };
}
```

To click a specific event in the left rail (e.g. `moodboard_hotspot_click` to inspect what fired for it):

```js
() => {
  const target = [...document.querySelectorAll('[role="button"], li, div')]
    .find(el => /moodboard_hotspot_click/i.test(el.innerText || ''));
  target?.click();
  return !!target;
}
```

## Verification matrix

For every change made in the workspace, write down expectations BEFORE opening Preview, then verify each row:

| Action on Page 5 | Expected dataLayer event | Expected fired tag | Expected destinations |
|---|---|---|---|
| Scroll into moodboard | `moodboard_view` | `GA4 - moodboard_view` | GA4 g/collect with `en=moodboard_view` |
| Click hotspot | `moodboard_hotspot_click` (+ `affiliate_outbound_click` if outbound) | GA4 event tags + Meta `SubscribedButtonClick` (auto) | GA4 + Meta |
| Slide carousel | `moodboard_slide_change` | `GA4 - moodboard_slide_change` | GA4 |
| Page view | `gtm.js` → `Initialisatie` | `Google-tag G-XXX` (once) | GA4 page_view |

Document the expected matrix in your notes before you verify. Walk through, tick off each row. Any miss = stop and investigate.

## When Preview shows the WRONG result

Most common failure modes and where to look:

- **Tag not firing when expected** — open the tag's detail page in the main GTM tab (`/tags/<id>`). Check the **Activerende triggers** list. Then open the trigger and confirm the dataLayer event name matches EXACTLY (case-sensitive, no trailing spaces).
- **Tag firing but parameters are wrong / empty** — the GA4 event parameter mapping uses `dlv - X` variables. Open the variable definition and confirm the dataLayer key matches what the theme actually pushes. Check the **Variabelen** section of the Tag Assistant event overview to see what keys WERE available.
- **Tag firing too often** — either two triggers fire the same tag per event, or there are duplicate tags bound to the same trigger. Check the tag's **Activerende triggers** list (should usually be 1) and grep the inventory for duplicates.
- **Tag fires twice per event** — nearly always two triggers on the same tag (e.g. both `CE - moodboard_hotspot_click` AND `All Pages` are listed). Remove the extra trigger.
- **Nothing shows in Tag Assistant** — the debug site may have lost the `gtm_debug` parameter (client-side routers sometimes strip it). Navigate back to a URL that includes `?gtm_debug=...` or re-click **Koppelen**.
- **Event shows in dataLayer feed but no tag fires** — the trigger condition string doesn't match. Events are case-sensitive. `Moodboard_View` ≠ `moodboard_view`.

## Re-entering Preview after making more changes

After additional workspace edits, click **Voorbeeld** again in the GTM UI. Tag Assistant picks up the new workspace state on its existing tab — you don't need to restart the whole session. The debug site reloads with a new `gtm_debug` timestamp, and the event feed resets.

If Tag Assistant seems stuck on stale state, close Page 4 (Tag Assistant) and click **Voorbeeld** again to start fresh.

## Closing Preview

Two ways to end Preview cleanly:
- Click **Foutopsporing stoppen** (Stop debugging) in Tag Assistant.
- Simply close the Tag Assistant tab, or move on — Preview mode auto-expires after inactivity (~90 minutes).

The debug site (Page 5) continues to load the debug-flagged version until you remove `?gtm_debug=...` from its URL or navigate away.

## Checklist before leaving Preview

Before calling a change "verified and ready to `Verzenden`":

- [ ] Every intended dataLayer event appears in the left rail
- [ ] Every intended tag shows under **Geactiveerde tags** with the expected count
- [ ] No unexpected tag appears under **Geactiveerde tags**
- [ ] Variable values (product name, product handle, percent_scrolled, etc.) look correct in the event overview
- [ ] No duplicate firings (1× per user action, unless intentionally batched)
- [ ] GA4 tag Destination IDs hit (look for GA4 rows under the event, with 200 status)

If any box is unchecked, do NOT publish — go back to the workspace and fix.
