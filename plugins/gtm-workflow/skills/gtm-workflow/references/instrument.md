# instrument.md — Inject a dataLayer hook and network sniffer to verify tag firing on a live site

This is the **verify phase**: you already know (from `inspect.md`) what the GTM container *says* should fire. Now you open the live site in a second Chrome tab, hook `window.dataLayer`, reproduce the user flow, and confirm that:

1. The dataLayer push actually happens with the expected event name and payload.
2. The destination beacons (GA4, Meta, Pinterest, ...) actually leave the browser with matching params.
3. Nothing fires twice, nothing fires wrong, nothing fires that shouldn't.

All injection here happens on the **site tab**, not the GTM tab. Use `select_page` (or target by URL) to make sure you're running `evaluate_script` against the right page.

## The dataLayer hook pattern

Inject once per session. The guard (`window.__btmHookInstalled`) makes re-injection safe.

```js
() => {
  if (!window.__btmHookInstalled) {
    window.__btmCapturedEvents = [];
    const dl = window.dataLayer = window.dataLayer || [];
    const origPush = dl.push.bind(dl);
    dl.push = function(...args) {
      for (const arg of args) {
        try {
          window.__btmCapturedEvents.push({
            t: Date.now(),
            event: (arg && (arg.event || arg[0])) || null,
            payload: JSON.parse(JSON.stringify(arg)),
          });
        } catch (e) {
          window.__btmCapturedEvents.push({
            t: Date.now(),
            event: 'unserializable',
            err: String(e),
          });
        }
      }
      return origPush(...args);
    };
    window.__btmHookInstalled = true;
  }
  return { installed: true, existing: window.__btmCapturedEvents.length };
}
```

Two helpers you'll run often:

**Reset** (call before each user action so captures are scoped to that action):

```js
() => { window.__btmCapturedEvents = []; return 'reset'; }
```

**Read** (call after the action settles):

```js
() => ({
  total: window.__btmCapturedEvents.length,
  events: window.__btmCapturedEvents.map(e => ({
    event: e.event,
    payload: e.payload,
  })),
})
```

Gotchas:

- Hook must be installed **before** the dataLayer receives events you care about. If the theme pushes `consent default` during `<head>` execution, you missed it — reload the page *with the hook already installed* by re-running the injection after navigation.
- If GTM itself loads after your hook, GTM replaces `dataLayer.push` with its own wrapped version. Re-install the hook once `google_tag_manager` is defined:
  ```js
  () => {
    const ready = !!window.google_tag_manager;
    // if ready, re-install on top of GTM's wrapper
  }
  ```
- Circular refs in payloads (rare but happens with DOM nodes) will throw in `JSON.parse(JSON.stringify(...))`; the try/catch around `unserializable` handles it.

## Detecting tracker destinations

After the hook captures dataLayer, use `list_network_requests` (chrome-devtools-mcp) to see which beacons actually fired. Filter by URL substring. Most destinations expose event name + params directly in the URL's query string so you don't need response bodies.

### Google Analytics 4

- Collect endpoints: `region1.google-analytics.com/g/collect`, `www.google-analytics.com/g/collect` (region depends on user).
- Event name: `en=<event_name>` query param.
- Measurement ID: `tid=G-XXXXXXXX`.
- Event params:
  - String params: `ep.<key>=<value>` (e.g. `ep.moodboard_name=summer-2024`)
  - Number params: `epn.<key>=<value>`
  - User props: `up.<key>=<value>` / `upn.<key>=<value>`
- Session params: `sid`, `sct` (session count), `seg` (engagement), `_et` (engagement time ms).
- Debug mode: `_dbg=1` (from Tag Assistant preview).

### Google Ads

- Conversions: `googleads.g.doubleclick.net/pagead/viewthroughconversion/<AW-ID>/` or `www.google.com/rmkt/collect/<AW-ID>`.
- Dynamic remarketing: includes `data.item_id`, `data.value`.
- Conversion label: `label=<label>` param on the conversion URL.

### Meta Pixel

- Browser beacon: `www.facebook.com/tr/?id=<pixel-id>&ev=<event-name>&...`
  - Standard events: `PageView`, `ViewContent`, `AddToCart`, `InitiateCheckout`, `Purchase`, `Lead`.
  - Custom data: `cd[content_ids]`, `cd[value]`, `cd[currency]`, `cd[content_type]`.
- Server-side register (Privacy Sandbox): `www.facebook.com/privacy_sandbox/pixel/register/trigger/`.
- If you see only `/tr/?id=...&ev=PageView` and nothing else, Meta is running default tracking only — no custom events wired.

### Pinterest

- Identity: `ct.pinterest.com/user/` (fires once per session).
- Events: `ct.pinterest.com/v3/?event=<name>&tid=<tag-id>` (e.g. `event=pagevisit`, `event=addtocart`, `event=checkout`).
- Enhanced match: `em=<sha256-of-email>`.

### TikTok

- `analytics.tiktok.com/api/v2/pixel/track/` (POST with JSON body, params in body not query string — you'll need to check request body, not just URL).
- Events: `ViewContent`, `AddToCart`, `InitiateCheckout`, `CompletePayment`, or custom.

### Microsoft Clarity

- Session ingest: `y.clarity.ms/collect` (high volume — one per interaction).
- Tag loader: `www.clarity.ms/tag/<project-id>`.
- Clarity doesn't report custom "events" the way GA4 does; it records sessions. If a Clarity tag is wired to a Custom Event trigger, it fires a `clarity('event', '<name>')` which shows up as a separate small request.

### Klaviyo

- Identify: `a.klaviyo.com/onsite/ajax?a=<company-id>&...`
- Track: `a.klaviyo.com/onsite/ajax` with `action=track`.
- Loader: `static.klaviyo.com/onsite/js/<company-id>/klaviyo.js`.

### Shopify Web Pixels (Customer Events)

- Endpoint: `<shop-domain>/.well-known/shopify/monorail/unstable/produce_batch`.
- This is Shopify's own event bus. If you see this firing even when GTM tags are disabled, it's Shopify Customer Events running independently — they are sandboxed and **don't see `window.dataLayer`**, so anything you want to track from a GTM-driven dataLayer push has to go through a *custom* Web Pixel inside Shopify admin, not GTM.

### Merchant Center Analytics

- `www.merchant-center-analytics.goog/mc/collect?tid=MC-XXXXXX` — Google's Shopping/MC tracking. Fires alongside GA4 when Google-tag has MC configured.

### Google Tag Assistant / preview mode signals

- `www.googletagmanager.com/gtm.js?id=GTM-XXX&gtm_debug=<token>` — preview-mode loader.
- `www.googletagmanager.com/gtag/js?id=G-XXX&l=dataLayer` — base gtag loader.
- If the user ran `Voorbeeld` (Preview) in GTM and then landed on the site, `gtm_debug` proves it.

### Consent Mode signals

- `consent=<state>` query param on collect requests. `G111` means both `ad_storage` and `analytics_storage` granted; `G100` means both denied; `G101`/`G110` are partial.
- `gcs=G1--` in the URL is the *default* state frozen at load time; `gcd` encodes more detailed consent.

## Extracting params from URLs

```js
(reqUrl) => {
  const url = new URL(reqUrl);
  return {
    host: url.host,
    path: url.pathname,
    params: Object.fromEntries(url.searchParams),
  };
}
```

Pair with `get_network_request` if you need headers or the POST body (TikTok, server-side pixels) — the query-string approach only covers GET beacons.

## Verifying expected events — the flow

For each event the inventory said should fire:

1. **Reset** the dataLayer capture (`window.__btmCapturedEvents = []`).
2. (Optional) Clear network log in DevTools; chrome-devtools-mcp doesn't clear it but you can record the last-seen request ID as a baseline.
3. **Simulate** the user action:
   - Scroll: `window.scrollTo({ top: ..., behavior: 'instant' })`
   - Click: `document.querySelector('<sel>').click()` or use the `click` tool with a snapshot ref
   - Navigate: `navigate_page`
   - Form fill: `fill` / `fill_form`
4. **Wait** ~500–1500 ms for async pushes (500 for direct clicks, 1000–1500 for beacons/fetches to leave the pipe).
5. **Read** captured dataLayer events.
6. Run `list_network_requests` and filter for destination host substrings (see list above).
7. **Cross-reference**: for each expected event X, did
   - (a) a dataLayer push happen with `event === 'X'`?
   - (b) a GA4/Meta/... beacon fire with matching event name and expected params?
8. Report as a 2-column table: **dataLayer** vs. **network**.

## Common diagnoses

| Observation | Likely cause |
|---|---|
| dataLayer push happens, no GA4 beacon | No tag listens for that Custom Event trigger. Create one — see `modify.md` Patterns A+B. |
| dataLayer push with wrong event name | Theme code bug; check the `event` key in the pushed object. |
| GA4 beacon fires with wrong params | Tag config has stale `{{dlv - ...}}` variables, or dataLayer key path mismatches (e.g. `ecommerce.items.0.item_id` vs `items[0].id`). |
| Same event fires twice | Two tags bound to the same trigger; or tag + Custom HTML both fire. Also: SPA double-navigation pushing the same event. |
| Meta Pixel `PageView` fires but no custom event | Meta has auto-tracking only. Add a Custom HTML tag running `fbq('trackCustom', '<Name>', {...})` bound to the Custom Event trigger. |
| Pinterest fires `PageVisit` but nothing else | Pinterest default tracking only; add a Pinterest event tag with the matching event name. |
| GA4 `scroll` event fires at 90% without a trigger | GA4 Enhanced Measurement — disable in GA4 admin if you want to control it from GTM, or accept it. |
| TikTok fires but has no body params | TikTok events are POST with JSON body; query string is empty. Use `get_network_request` to inspect the body. |
| GA4 `tid` is wrong (points to test property) | Constant variable `const - GA4 Measurement ID` not updated, or there are two Google-tags with different IDs. |
| `consent=G100` on every beacon | User denied consent in the banner; not a bug. If no banner is present, default consent is stuck at denied. |
| Beacon fires from wrong host (`analytics.google.com` instead of `region1.*`) | Ad-blocker or uBlock rewriting — test in a clean profile. |

## Privacy / consent checks

Inspect `window.__btmCapturedEvents` for `consent` events:

- `push({ event: 'consent', action: 'default', ad_storage: 'denied', analytics_storage: 'denied', ... })` — sets initial state before any tag runs.
- `push({ event: 'consent', action: 'update', ad_storage: 'granted', analytics_storage: 'granted', ... })` — user accepted on the banner.

Flag these if you see them:

- **Default state = granted, no consent banner visible on the site** → GDPR risk. EU visitors are being tracked without opt-in.
- **No `consent default` event at all** → GTM is initializing without Consent Mode. Depending on destinations this may or may not be allowed.
- **`consent update` fires but GA4 `gcs=G100` in the beacon URL** → the update came too late; the tag fired before consent resolved. Fix order-of-ops: make tags wait on `Initialisatie — alle pagina's` + a consent update trigger.

## Example: capture scroll events

```js
async () => {
  window.__btmCapturedEvents = [];
  window.scrollTo({ top: document.body.scrollHeight * 0.8, behavior: 'instant' });
  await new Promise(r => setTimeout(r, 1500));
  return window.__btmCapturedEvents.map(e => e.event);
}
```

Expected output on a site with GA4 Enhanced Measurement enabled: `['scroll']` at 90% threshold. If you wanted a custom `scroll_to_footer` event, it would appear here too — and should be followed by a GA4 beacon with `en=scroll_to_footer`.

## Example: capture an outbound affiliate click

```js
async () => {
  window.__btmCapturedEvents = [];
  // intercept the click so we don't actually navigate away from the test page
  document.addEventListener('click', (e) => {
    const a = e.target.closest('a[href*="awin1.com"]');
    if (a) e.preventDefault();
  }, { capture: true, once: true });
  document.querySelector('a[href*="awin1.com"]').click();
  await new Promise(r => setTimeout(r, 800));
  return window.__btmCapturedEvents;
}
```

Then immediately follow with `list_network_requests` filtered on `google-analytics.com/g/collect` and confirm the last one has `en=affiliate_outbound_click` and `ep.outbound_url=https://awin1.com/...`.

## Output contract

When you're done verifying, report:

1. A table of **expected event → dataLayer push? → destination beacon?** with checkmarks and failure details.
2. A short "Findings" block listing each mismatch with the diagnostic from the table above.
3. A recommended next step — usually a hand-off to `modify.md` with a specific patch (e.g. "Add a Custom Event trigger for `view_product` and bind `GA4 - view_product` to it").

Do not propose changes in this phase. Just surface the facts.
