# GTM recipes — ready-to-apply tracking patterns

A catalog of common tracking setups. Each recipe is self-contained: goal, what to create, and a link to the matching procedure pattern in `modify.md` (Pattern A/B/C/D/E/F/G/H). Use these as templates — don't reinvent the setup per site.

## 1. Track a new dataLayer event in GA4

**Goal:** the theme (or app) pushes something like `dataLayer.push({ event: 'foo_bar', prop_a: ..., prop_b: ... })`, and you want it as a GA4 event with those props as parameters.

**What to create:**
1. Data-layer variables `dlv - prop_a` and `dlv - prop_b` — one per property you want as a GA4 param. See `modify.md` Pattern H.
2. Custom Event trigger `CE - foo_bar` matching event name `foo_bar` exactly. See Pattern A.
3. GA4 Event tag `GA4 - foo_bar`:
   - Configuration tag: your GA4 config / Google Tag
   - Event Name: `foo_bar`
   - Event Parameters: `prop_a` → `{{dlv - prop_a}}`, `prop_b` → `{{dlv - prop_b}}`
   - Triggering: `CE - foo_bar`
   - See Pattern B.
4. Preview, exercise the event, confirm `GA4 - foo_bar` fires exactly once per push. See `preview.md`.

## 2. Track outbound affiliate clicks

**Goal:** measure clicks to affiliate networks (awin1, daisycon, tradetracker, rakuten, impact, etc.). Two-layer setup works best.

**Layer 1 — URL-rule Link trigger (coarse, covers everything):**
- Trigger: Just Links, fires on clicks where `Click URL` matches RegEx `awin1\.com|daisycon|tradetracker|rakuten|impact\.com|...`.
- Tag: `GA4 - Affiliate Click All Partners`, event name `affiliate_click`, params include `{{Click URL}}`, `{{Click Text}}`, `{{Page Path}}`.

**Layer 2 — dataLayer event (fine-grained, includes product context):**
- Theme code pushes `dataLayer.push({ event: 'affiliate_outbound_click', product_name, product_handle, partner })` from the click handler BEFORE navigation.
- Custom Event trigger `CE - affiliate_outbound_click`.
- Tag `GA4 - affiliate_outbound_click` with params mapped from `dlv - product_name`, `dlv - product_handle`, `dlv - partner`.

Both layers are valuable. Layer 1 catches links you didn't instrument. Layer 2 gives you product-level attribution. Dropping one loses signal.

## 3. Scroll depth tracking

**Goal:** measure how far users scroll on key pages.

**Full recipe:**
1. Variables → **Configureren** → enable built-in `Scroll Depth Threshold`, `Scroll Depth Units`, `Scroll Depth Direction`. See `modify.md` Pattern D.
2. Trigger type **Scrolldiepte** (Scroll Depth). Percentages: `25,50,75,100`. Direction: vertical. Fires on: All Pages (or a subset page regex).
3. GA4 Event tag:
   - Event name: `scroll_depth` (NOT plain `scroll` — collides with GA4 Enhanced Measurement).
   - Param: `percent_scrolled` → `{{Scroll Depth Threshold}}`.
   - Trigger: the Scrolldiepte trigger from step 2.

**Warning:** GA4 Enhanced Measurement already tracks scroll at 90% as a `scroll` event. If you duplicate the event name, you'll get messy data. Always use a distinct name (`scroll_depth`) for custom GTM scroll tracking.

## 4. Meta Pixel custom events

**Goal:** fire a Meta Pixel custom event (`trackCustom`) from a dataLayer event.

Meta Pixel doesn't have a native GTM tag template — use a Custom HTML tag. See `modify.md` Pattern F.

**Recipe for `MoodboardHotspotClick`:**

```html
<script>
  if (typeof fbq !== 'undefined') {
    fbq('trackCustom', 'MoodboardHotspotClick', {
      content_name: {{dlv - product_name}},
      content_ids: [{{dlv - product_handle}}],
      content_type: 'product'
    });
  } else {
    console.warn('[gtm] fbq not loaded, skipping MoodboardHotspotClick');
  }
</script>
```

- Trigger: same Custom Event trigger as the matching GA4 tag (`CE - moodboard_hotspot_click`).
- Tag firing option: "Once per event".
- Requires Meta Pixel base code already present on the site — the tag assumes `fbq` exists.

For standard Meta events (`AddToCart`, `Purchase`, etc.), use `fbq('track', 'AddToCart', {...})` — no `Custom` prefix.

## 5. Pinterest Tag custom events

**Goal:** track a Pinterest conversion or custom event from a dataLayer push.

Requires Pinterest Tag already installed (`pintrk('load', '...')` present).

```html
<script>
  if (typeof pintrk !== 'undefined') {
    pintrk('track', 'custom', {
      event_id: 'moodboard_hotspot_click',
      product_name: {{dlv - product_name}},
      product_id: {{dlv - product_handle}}
    });
  }
</script>
```

Same trigger as the matching GA4 tag. Follow `modify.md` Pattern F.

## 6. TikTok Pixel event

**Goal:** TikTok Pixel custom or standard event from a dataLayer push.

Requires TikTok Pixel loaded (`ttq.load(...)`, `ttq.page()` in place).

```html
<script>
  if (typeof ttq !== 'undefined') {
    ttq.track('ViewContent', {
      content_id: {{dlv - product_handle}},
      content_name: {{dlv - product_name}},
      content_type: 'product'
    });
  }
</script>
```

Trigger: Custom Event for the matching dataLayer push. Uses the same pattern as Meta Pixel and Pinterest — wrap in a typeof-guard so the tag doesn't error when the pixel isn't loaded (e.g. on consent-denied sessions).

## 7. Remove / pause a tag

**Goal:** stop a tag from firing, either temporarily (investigation) or permanently (cleanup).

See `modify.md` Pattern G for click-path.

**Prefer Onderbreken (Pause) over Verwijderen (Delete) during investigation** — you can un-pause with one click if it turns out the tag was load-bearing. Only delete after you're sure nothing depends on it.

When pausing:
- The tag stays in the container, keeps its history, but stops firing on triggers.
- Pause is reversible; delete is not.
- Leave a note in the next version's description explaining why.

## 8. Consolidate two GA4 page_view loaders

**Symptom from inventory:** Custom HTML tag (often called `GA4 implementatie`, `GA4 page view`, or similar) loading `gtag/js?id=G-XXXXX` ALONGSIDE a native **Google-tag G-XXXXX**. Result: GA4 reports double `page_view` and inflated session counts.

**Fix:**
1. **Pause** (not delete) the Custom HTML tag. Run Preview — confirm the native Google Tag still loads `gtag/js` and fires `page_view` once.
2. If nothing in GA4 realtime breaks after a few sessions, delete the Custom HTML tag in a follow-up version.
3. Version description should call out the consolidation so future-you knows what happened.

## 9. Fix a tag wired to the wrong trigger

**Goal:** a tag fires on the wrong event (too broad, or completely wrong event name).

See `modify.md` Pattern C.

**Critical verification:** after swapping the trigger, Preview MUST show the event with the correct tag count (1× not 2×). The most common bug is leaving the OLD trigger attached while adding the new one — resulting in double-firing when both match, or spurious firing when either matches.

Check the tag's **Activerende triggers** list after the fix:
- Should contain only the new correct trigger.
- No "All Pages" bleed-through unless that's genuinely intentional.
- No duplicate entries.

## 10. Clean up orphans

**From inventory**, you'll often spot:
- **Orphan tags** — tags with no **Activerende triggers**. They never fire. Decide: wire to a correct trigger, or delete.
- **Orphan triggers** — triggers referenced by zero tags. They evaluate on every event but do nothing. Safe to delete.
- **Orphan variables** — variables referenced nowhere. Low-impact clutter; delete in a cleanup version.

Orphan cleanup is safe but should still go through Preview — occasionally a "variable referenced by zero tags" is actually referenced by a Custom HTML tag's inline code, which the GTM reference-counter doesn't see.

## 11. Add consent mode signals

**Goal:** implement Google Consent Mode v2 so GA4 / Ads honor consent choices.

**Best case:** the site uses a CMP (Cookiebot, OneTrust, Klaro, Usercentrics, Iubenda) — most of them already push `consent` commands to dataLayer. In that case do nothing in GTM; the CMP handles it.

**If no CMP:** add a Custom HTML tag on the **Initialisatie - All Pages** trigger (fires before All Pages, so before any GA4/Ads tag):

```html
<script>
  window.dataLayer = window.dataLayer || [];
  function gtag(){dataLayer.push(arguments);}
  gtag('consent', 'default', {
    'ad_storage': 'denied',
    'analytics_storage': 'denied',
    'ad_user_data': 'denied',
    'ad_personalization': 'denied',
    'wait_for_update': 500
  });
</script>
```

Then your CMP (or custom banner) must `gtag('consent', 'update', {...})` when the user accepts. Without a CMP, the default should **never** be `granted` for GDPR compliance — always start denied, upgrade on explicit consent.

Firing priority: set this tag's firing priority HIGH (e.g. 100) so it runs before every GA4 / Ads tag on the same Initialisatie trigger.

## 12. Debug: tag fires twice

**Symptom in Preview:** a single user action produces "N keer geactiveerd: 2" on one tag.

**Two root causes:**

**A) Two triggers on the same tag.** Open the tag, check **Activerende triggers** — should be 1 entry (unless you intentionally bound it to multiple events). Remove the extra.

**B) Two tags bound to the same trigger.** Inventory tags and grep for duplicates. Common when the same tag was re-created instead of edited. Pause one and verify in Preview; delete the duplicate in a follow-up version.

**Fix:** Pattern C from `modify.md` to swap or remove a trigger. Pattern G to pause/delete the duplicate tag.

**Verification:** Preview after the fix, exercise the action, confirm tag fires exactly once per action.

## 13. Exclude internal / office traffic

**Goal:** don't count your own team's visits in GA4 or Ads.

**Option A — IP-based (preferred):** configure in GA4 admin → Data Streams → Internal Traffic. Not a GTM job.

**Option B — Query-parameter override (when IP is dynamic):** set a cookie when visitors arrive with a special URL like `?internal=1`. Then:
1. Variable `cookie - internal` (1st-party Cookie type, name `internal`).
2. Exception trigger: Custom Event / All Pages where `{{cookie - internal}}` equals `1`.
3. Add the exception as **Trigger voor uitzondering** (Exception Trigger) on every GA4/Ads tag.

**Option C — User agent / hostname filter:** exception trigger on `{{Page Hostname}}` = `staging.example.com` so staging traffic is excluded from production GA4.

## 14. Cross-domain tracking

**Goal:** preserve session across two domains (e.g. `example.com` → `shop.example.com` or `example.com` → `checkout.stripe.com` if under your control).

GA4's built-in Google Tag config supports this: **Google Tag → Configure → Configure your domains**. Add every domain you control. GA4 auto-decorates outbound links with `_gl=...`.

GTM doesn't need a special tag for this — it's configured on the Google Tag itself. But verify in Preview: clicking an outbound link to a linked domain should show `_gl` in the destination URL.

## 15. Enhanced Ecommerce (GA4)

**Goal:** fire `view_item_list`, `view_item`, `add_to_cart`, `begin_checkout`, `purchase` events to GA4.

Standard GA4 ecommerce dataLayer schema: `items` array with `item_id`, `item_name`, `price`, `quantity`, etc. See Google's GA4 ecommerce reference.

**Per-event setup:**
- Custom Event trigger for each event name (`CE - view_item`, `CE - add_to_cart`, etc.).
- GA4 Event tag per event name.
- CRITICAL: map the `items` parameter as `{{dlv - items}}` — data-layer variable with key `items`. GA4 will auto-parse the array.
- Other params (`currency`, `value`) mapped from their own dlv variables.

Don't try to construct `items` inside a Custom HTML tag — always push it from the theme / app with the correct schema. GTM's role is relay, not construction.

## 16. Variable scoping: dlv vs. Custom JS

- Use **Data Layer Variable (dlv)** when the value is pushed onto `dataLayer` by the theme. 1st choice, reliable.
- Use **Custom JavaScript** variable when you need to derive a value at read-time (e.g. reformat a date, combine two fields, read a DOM attribute not on dataLayer).
- Never use a Custom JavaScript variable to READ from dataLayer — that's what dlv is for.

Keep Custom JS variables small, pure, and defensive (`try/catch`, null-safe). They run synchronously every time the variable is referenced.
