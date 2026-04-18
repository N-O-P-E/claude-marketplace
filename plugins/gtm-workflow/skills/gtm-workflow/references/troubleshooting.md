# Common gotchas and how to fix them

Real issues encountered in sessions, with the fix that worked. Each section is Symptom → Cause → Fix.

## Chrome profile locked

**Symptom:** `mcp__plugin_chrome-devtools-mcp_chrome-devtools__new_page` (or `list_pages`) errors with something like:

```
The browser is already running for /Users/<you>/.cache/chrome-devtools-mcp/chrome-profile.
Use --isolated to run multiple browser instances.
```

**Cause:** Stale singleton files (`SingletonLock`, `SingletonSocket`, `SingletonCookie`, `RunningChromeVersion`) left behind in `~/.cache/chrome-devtools-mcp/chrome-profile/` by a previous session that crashed or got killed. No actual Chrome process is holding the profile — the files are just lying.

**Fix:**

1. Confirm no real Chrome-for-Testing process is running against that profile:

   ```bash
   ps aux | grep -iE "Chrome for Testing|chrome-for-testing|HeadlessChrome|google-chrome.*chrome-profile" | grep -v grep
   ```

   If the result is empty, proceed. If something is actually running, do not delete — that's a live session.

2. Remove the stale singletons:

   ```bash
   rm -f \
     ~/.cache/chrome-devtools-mcp/chrome-profile/SingletonLock \
     ~/.cache/chrome-devtools-mcp/chrome-profile/SingletonSocket \
     ~/.cache/chrome-devtools-mcp/chrome-profile/SingletonCookie \
     ~/.cache/chrome-devtools-mcp/chrome-profile/RunningChromeVersion
   ```

3. Retry `list_pages`. Usually works immediately; a fresh browser launches on the next MCP call.

**WARNING:** Do NOT kill `mcp-watchdog` processes. Those are per-session and killing them may break the active MCP connection. Only touch the singleton files listed above.

## Guessed URL IDs (404)

**Symptom:** Navigating to e.g. `/container/accounts/6000000000/containers/237242066/workspaces/5/tags` produces a 404 page, shows the wrong account, or dumps you on an unrelated container.

**Cause:** The account ID was invented or copied from another session. Every GTM account has a unique ~10-digit numeric ID that is only visible in the home-page account listing. You cannot derive it from the `GTM-XXXXXXX` public ID or from the container ID.

**Fix:** Navigate to `https://tagmanager.google.com/#/home`, `take_snapshot`, and extract the real `accounts/<id>/containers/<id>` pair from any container link's `href`. Use `evaluate_script` to pull them programmatically:

```js
Array.from(document.querySelectorAll('a[href*="/container/accounts/"]'))
  .map(a => a.getAttribute('href').match(/accounts\/(\d+)\/containers\/(\d+)/))
  .filter(Boolean)
  .map(m => ({ accountId: m[1], containerId: m[2] }));
```

Never hardcode account IDs across sessions.

## Workspace ID changed after publish

**Symptom:** A URL like `/workspaces/4/tags` that worked earlier in the day now redirects to overview, or loads an empty-looking tags list. Edits don't appear in the UI.

**Cause:** GTM creates a new default workspace every time you publish. Workspace 4 is now a historical snapshot (read-mostly), and the current live workspace is 5 (or 6, 7, etc.). Your hardcoded URL is pointing at the old one.

**Fix:** Navigate to the container root and let GTM auto-redirect to the current default workspace:

```text
navigate_page({ url: ".../containers/<C>" })
evaluate_script({ function: `() => location.hash` })
// Parse /workspaces/<N>/ from the hash — that's the current ID.
```

Store the new workspace ID and use it for the rest of the session. Do this again after every publish.

## UI clicks time out

**Symptom:** MCP's `click` tool returns `Element did not become interactive within the configured timeout` (typically 5000ms).

**Cause:** GTM is an SPA and rehydrates after modal open/close or route transition. The `uid` from a stale snapshot no longer exists in the live DOM, or the element is rendered but not yet clickable (still behind a loading overlay).

**Fix:**

1. `take_snapshot` immediately before the click — never click using uids from a snapshot older than the last navigation/modal.
2. If the fresh snapshot still fails to click, wait for the UI to settle:

   ```text
   evaluate_script({ function: `async () => { await new Promise(r => setTimeout(r, 1500)); }` })
   take_snapshot()
   click({ uid: "<uid from the new snapshot>" })
   ```

3. GTM's trigger picker, tag-type picker, and overflow menus especially need 1–2 seconds to finish populating after opening.

## Overflow menu items (Kopiëren / Verwijderen) missing from snapshot

**Symptom:** You click the three-dot overflow on a row. The menu visibly appears (you can see it in `take_screenshot`), but `take_snapshot` doesn't list `Kopiëren` or `Verwijderen` items as clickable nodes.

**Cause:** The overflow menu is portaled to the body root — it renders as a sibling `<menu>` element near the bottom of the DOM, far from the row that triggered it. Additionally, if focus was already on the overflow button (e.g. from a previous click), the first click can open-and-immediately-close the menu, leaving nothing for the snapshot to capture.

**Fix:**

1. Click the overflow button.
2. `take_snapshot` right after — scroll the snapshot output to the end and look for a `menu` element with `Kopiëren`, `Verwijderen`, etc.
3. If the items are absent, click the overflow again (which opens it fresh), `take_snapshot`, then click the target item from the new snapshot. The second-click pattern reliably populates the menu.

## Tag Assistant Preview tab doesn't open

**Symptom:** You click the `Voorbeeld` button in GTM. Nothing observably changes in the current tab. No preview banner appears, no navigation.

**Cause:** Tag Assistant deliberately opens in a **new tab**. The current tab is unaffected.

**Fix:** After clicking `Voorbeeld`:

```text
list_pages()  // should now show a new entry
select_page({ pageId: "<the new one>" })
```

Expected URL pattern on the new tab:

```
https://tagassistant.google.com/?hl=nl#/?source=TAG_MANAGER&id=GTM-XXXXXXX&gtm_auth=...&gtm_preview=env-...
```

Once selected, you can drive the Tag Assistant connect flow (enter site URL, click `Verbinden`, etc.) from that tab.

## Preview debug tab closes prematurely

**Symptom:** The site tab that received the `?gtm_debug=...` query string navigates away before you can verify events, or the tab closes entirely. Tag Assistant shows "Disconnected".

**Cause:** A script on the merchant site (redirect rule, aggressive analytics, router-based full-page navigation), an accidental back-navigation, or the site dropping the `gtm_debug` query parameter on internal links.

**Fix:** Re-run Preview from Tag Manager to get a fresh debug session. Click `Voorbeeld` again, enter the site URL again, accept the connection again. **Do not** try to reuse a closed debug session — the token is bound to that specific connection. If the site keeps dropping the param, add `gtm_debug` to the allowed query params in the site's router, or preview from a page that doesn't redirect.

## "Already exists" / TAKEN errors on create operations

**Symptom:** Shopify-style error payload with `code: TAKEN` when creating a definition. (Not a GTM-side error — this happens when the plugin is invoked against Shopify workflows to back GTM tracking with metafields/metaobjects.)

**Cause:** The definition you're trying to create already exists in the Shopify shop. Shopify rejects duplicates instead of upserting.

**Fix:** On `TAKEN`, don't treat it as fatal. Look up the existing definition by namespace/key, grab its `id`, and call the update mutation instead. See the plugin's `ensureMetafieldDefinitions` pattern for the canonical guard:

```js
// pseudo
try { create(...) }
catch (e) {
  if (hasTaken(e)) {
    const existing = await findByNamespaceKey(namespace, key);
    await update(existing.id, ...);
  } else throw e;
}
```

## Consent defaults missing or wrong

**Symptom:** GA4 tags fire on a cold page load with no CMP interaction. `page_view` hits flow to Google before the visitor has accepted cookies.

**Cause:** The container has no Consent Mode defaults set up, or `consent default` is set to `granted` for everything. Without the consent model, GA4 ignores consent state and always sends.

**Fix:** Check the dataLayer at load:

```js
// in devtools or via evaluate_script on the merchant site
window.dataLayer.filter(x => x[0] === 'consent');
```

You want to see a `consent default` entry with `analytics_storage: 'denied'`, `ad_storage: 'denied'`, etc. — pushed **before** GTM loads — and a `consent update` entry pushed after the user accepts.

**Flag and warn** if defaults are already `granted` without a CMP. That's a GDPR violation for EU traffic. Either add a CMP-driven consent default push or pause the GA4 tag until one is in place.

## Google Tag loading twice

**Symptom:** DevTools Network tab shows two `gtag/js?id=G-XXXXXXXXXX` requests per page load, or Tag Assistant reports `page_view` firing twice, or GA4 realtime shows inflated numbers.

**Cause:** Both a legacy Custom HTML tag (often named `GA4 implementatie`, `GA4 loader`, or similar — contains a hand-rolled `<script async src="...gtag/js?id=G-..."></script>` block) AND a native `Google-tag G-XXX` tag are configured to fire on All Pages / Initialization - All Pages.

**Fix:** Keep the native Google Tag; pause or delete the Custom HTML. Modern GTM handles consent mode, enhanced measurement, and parameter merging correctly only via the native tag. Custom HTML loaders bypass this.

Verify in Tag Assistant after the change: you should see exactly one `Google Tag` fire per page load, followed by `page_view` once.

## Workspace has changes but Publish button won't enable

**Symptom:** Sidebar badge shows `Wijzigingen in werkruimte: N` (N > 0), but the `Verzenden` button in the top-right is greyed out or the submit modal complains about unvalidated content.

**Cause:** A prior save left a tag, trigger, or variable in edit mode without explicitly exiting. GTM flags that section as "dirty but unvalidated" and blocks publish until you clean it.

**Fix:**

1. Open the workspace overview (`.../workspaces/<W>/overview`). Look at the changes list.
2. Click into each changed item. If you see an `Opslaan` or `Annuleren` button in the top-right (meaning you're in edit mode), click `Opslaan` (or `Annuleren` if you meant to discard).
3. Return to overview. The publish button should now enable.

If it still won't enable after visiting every changed item, try a hard refresh (`navigate_page` to the overview URL again). Occasionally GTM's dirty-state flag gets stuck client-side and a reload clears it.
