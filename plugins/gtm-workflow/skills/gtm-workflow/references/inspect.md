# inspect.md — Inventory tags, triggers, and variables in a GTM container

This is the **read phase**: you land in a container, take stock of what exists, and flag smells before proposing any change. No writes, no publishes. Output is a concise inventory the user can scan in 10 seconds.

## Prerequisites

- Chrome is already attached via `chrome-devtools-mcp` (see `setup.md`).
- You are on `tagmanager.google.com` and inside a container workspace. The URL looks like:
  `https://tagmanager.google.com/#/container/accounts/<account-id>/containers/<container-id>/workspaces/<workspace-id>/tags`
- Dutch UI: labels are **Tags / Triggers / Variabelen / Mappen / Sjablonen**.

## Navigate & scrape the three lists

For each of `/tags`, `/triggers`, `/variables` under the current workspace, do the same three-step dance:

1. `navigate_page` with `type=url` to the list URL.
2. Wait ~1500–2500 ms via `evaluate_script` — the GTM SPA hydrates lazily and the table is empty for the first second.
3. `evaluate_script` to extract structured data from the DOM.

### Tag list extraction pattern

```js
async () => {
  await new Promise(r => setTimeout(r, 2000));
  const anchors = Array.from(document.querySelectorAll('a[href*="/tags/"]'));
  const rows = anchors.map(a => ({
    href: a.href,
    text: a.textContent.trim(),
  }));
  return {
    rows,
    // full text of the table body — useful as a fallback for parsing type + triggers
    bodyPreview: document.body.innerText.slice(0, 6000),
  };
}
```

GTM's list view is a table where each tag row contains:

- `<a href=".../tags/<id>">` — the tag name
- A StaticText with the tag type (e.g. `Google Analytics: GA4-gebeurtenis`, `Aangepaste HTML`, `Google-tag`, `Google Ads-conversietracking`)
- One or more `<a href=".../triggers/<id>">` links for triggering triggers
- A `X maanden geleden` relative timestamp ("months ago")

For a complete inventory you have two options:

- **Fast**: scrape `bodyPreview` text and parse by row boundaries (tag name appears first in each row, timestamp last).
- **Structured**: query `a[href*="/tags/"]` for tag rows, then `a[href*="/triggers/"]` for trigger links, then merge by sibling/ancestor position in the DOM.

In practice the text-parse is faster and more reliable than walking the DOM — GTM's table classes are minified and change between deploys.

### Triggers list structure

```js
async () => {
  await new Promise(r => setTimeout(r, 2000));
  return {
    rows: Array.from(document.querySelectorAll('a[href*="/triggers/"]')).map(a => ({
      href: a.href,
      text: a.textContent.trim(),
    })),
    bodyPreview: document.body.innerText.slice(0, 6000),
  };
}
```

Each row carries:

- Name link
- Event type StaticText — common values:
  - `Aangepaste gebeurtenis` (Custom Event)
  - `Paginaweergave` (Page View)
  - `Alleen links` (Link Click — Just Links)
  - `Alle elementen` (All Elements click)
  - `Zichtbaarheid van element` (Element Visibility)
  - `Scrolldiepte` (Scroll Depth)
  - `Timer`
  - `YouTube-video`
- Filter conditions for link/URL triggers, e.g. `Click URL bevat awin1.com`
- **Tag count** — a small number at the end of the row. `0` means **orphan trigger** (defined but never fires a tag).

### Variables list

Two sections, top-to-bottom:

- **Ingebouwde variabelen** (built-in): `Page URL`, `Click Element`, `Form Element`, etc. Don't inventory these unless the user asks — they are standard and uninteresting.
- **Gebruikersgedefinieerde variabelen** (user-defined): this is where the interesting stuff lives. URL pattern: `/variables/<id>`.

Common naming prefixes you'll see in Dutch-run containers:

- `dlv - *` — dataLayer variable (reads a key out of the dataLayer push)
- `cjs - *` — custom JavaScript variable (returns a value from a function)
- `cookie - *` — 1st-party cookie variable
- `const - *` — constant
- URL-based: `URL - path`, `URL - host`

Focus your inventory on user-defined. For each: capture name, type (DataLayer Var / Custom JS / Constant / URL / Cookie), and any obvious description or default value visible in the row.

## What to report back to the user

Produce a concise inventory in this shape — one block per entity type, with smells called out at the end:

```
Container: GTM-XXXX (workspace 5 "Default Workspace")

Tags (11):
- GA4 - affiliate_outbound_click       | GA4 Event      | trigger: CE - affiliate_outbound_click
- GA4 - moodboard_hotspot_click        | GA4 Event      | trigger: CE - moodboard_hotspot_click
- GA4 - moodboard_view                 | GA4 Event      | trigger: CE - moodboard_view
- GA4 – Product Click                  | GA4 Event      | trigger: NONE  ⚠ orphan
- Google-tag - G-XXXXXXXX              | Google-tag     | trigger: Initialisatie — alle pagina's
- GA4 implementatie                    | Aangepaste HTML| trigger: Alle pagina's     ⚠ duplicates Google-tag
- Meta Pixel - Base                    | Aangepaste HTML| trigger: Alle pagina's
- Meta Pixel - ViewContent             | Aangepaste HTML| trigger: CE - view_product
- Pinterest Base Code                  | Aangepaste HTML| trigger: Alle pagina's
- Microsoft Clarity                    | Aangepaste HTML| trigger: Alle pagina's
- Awin Mastertag                       | Aangepaste HTML| trigger: Alle pagina's

Triggers (7):
- Alle pagina's                        | Page View       | (built-in, 4 tags)
- Initialisatie — alle pagina's        | Initialization  | 1 tag
- CE - moodboard_view                  | Custom Event    | event=moodboard_view            | 1 tag
- CE - moodboard_hotspot_click         | Custom Event    | event=moodboard_hotspot_click   | 1 tag
- CE - affiliate_outbound_click        | Custom Event    | event=affiliate_outbound_click  | 1 tag
- CE - view_product                    | Custom Event    | event=view_product              | 1 tag
- Product Click - Alle producten       | Click Link      | Click URL bevat /products/      | 0 tags  ⚠ orphan

Variables (user-defined, 9):
- dlv - moodboard_name                 | DataLayer Var   | key=moodboard_name
- dlv - hotspot_product_id             | DataLayer Var   | key=hotspot.product_id
- dlv - outbound_url                   | DataLayer Var   | key=outbound_url
- dlv - product_id                     | DataLayer Var   | key=ecommerce.items.0.item_id
- cjs - resolve utm                    | Custom JS       | returns URL utm_* params
- const - GA4 Measurement ID           | Constant        | G-XXXXXXXX
- const - Meta Pixel ID                | Constant        | 1234567890
- URL - path                           | URL             | component=path
- Cookie - _ga                         | 1st-Party Cookie| name=_ga

Smells to investigate:
- `GA4 – Product Click` has no trigger (orphan — dead tag, will never fire)
- `Product Click - Alle producten` is defined but 0 tags use it (cleanup candidate)
- `GA4 implementatie` (Aangepaste HTML, All Pages) likely duplicates the native `Google-tag`
  → probable double page_view on every page load
- `Meta Pixel - Base` fires on All Pages (OK), but no CE trigger for `add_to_cart` → Meta only
  captures auto-tracked PageView
- No consent trigger visible — check whether `consent update` dataLayer event is handled
```

## Flag common smells

Walk the inventory looking for these patterns. Report each with a one-sentence "why this matters":

- **Tags with no trigger (orphans)** — dead tags, will never fire; either wire them or delete.
- **Triggers with 0 tags** — cleanup candidates; either a refactor left them behind or someone started wiring and never finished.
- **Two tags loading GA4** — one native `Google-tag`, one `Aangepaste HTML` running `gtag('config', ...)` manually. This almost always double-fires `page_view` and inflates session counts. Keep the Google-tag, delete the HTML one.
- **Custom HTML tags running `fbq('init', ...)` on All Pages** — check whether Meta Pixel is also loaded by another tag or by the theme directly.
- **Custom HTML tags running `gtag('js', ...)` on All Pages** when a Google-tag also exists — double gtag bootstrap.
- **Event-name mismatch**: a tag named `GA4 - add_to_cart` whose `Event name` parameter is `addToCart` (camelCase) while the Custom Event trigger listens for `add_to_cart`. The tag will fire but report the wrong event to GA4.
- **Tags with trigger = "Alle pagina's" that read dataLayer params** — the dataLayer may not be populated on page load yet; these usually need a Custom Event trigger or a window-loaded trigger.
- **Missing `Initialisatie — alle pagina's` trigger on the Google-tag** — means consent mode may not initialize early enough.
- **`Aangepaste HTML` tag with `<script src="https://...">`** — blocks or delays the rest of GTM until the external script loads; usually better moved into theme `<head>`.

## Reading container metadata

To get container ID, account ID, and container name:

- Navigate to `#/home`. The container picker lists each account; inside each account, each container row shows container name, type (**Web** / **Server**), and container ID (`GTM-XXXXX`).
- Once inside a container, the banner shows `GTM-XXXXX` and the workspace name (default: `Default Workspace`, Dutch UI shows it as `Standaardwerkruimte` on some tenants).
- Workspace ID is in the URL path: `.../workspaces/<id>/...`. Most shops only have the default workspace (id `5` or similar).

Example:

```js
() => ({
  url: location.href,
  title: document.title,
  // the banner at the top shows "GTM-XXXXX · <workspace-name>"
  banner: document.querySelector('[class*="header"]')?.innerText?.slice(0, 200),
})
```

## Output contract

After running the three scrapes, produce:

1. A one-line container header (`Container: GTM-XXXX (workspace N "<name>")`).
2. Three labelled sections: Tags, Triggers, Variables (user-defined). One entity per line, aligned.
3. A "Smells to investigate" section. Bullet one smell per line with the entity name it applies to.

Keep it under ~40 lines total for small containers, ~80 for larger ones. The caller will decide what to act on next — your job here is only to surface.
