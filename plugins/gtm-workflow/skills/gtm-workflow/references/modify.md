# modify.md — creating and editing tags, triggers, variables via the UI

All modifications live in the **draft workspace** until Publish. Every action here is safe to test — nothing hits production.

## Golden rule before modifying

**Inventory first.** Then ask: is there already a tag/trigger that does this? Editing an existing one is always preferred over creating a duplicate.

## Navigating tabs

All workspace work lives under the same URL pattern:

```
https://tagmanager.google.com/#/container/accounts/<ACCOUNT_ID>/containers/<CONTAINER_ID>/workspaces/<WORKSPACE_ID>/{tags|triggers|variables|folders}
```

- `ACCOUNT_ID` and `CONTAINER_ID` come from the home page. Never guess.
- `WORKSPACE_ID` changes after each publish (GTM creates a new default workspace). If a hardcoded workspace URL 404s, navigate to the container root first and grab the current workspace ID from the redirect.

## Pattern A — Create a Custom Event trigger

Use when a dataLayer `push({ event: 'X', ... })` is firing but no GTM tag listens for it.

1. Navigate to `/workspaces/<ws>/triggers`.
2. Click **Nieuw** (New).
3. In the right-side panel:
   - Fill the trigger name. Convention: `CE - <event_name>` (e.g. `CE - moodboard_view`).
   - Click the trigger-type placeholder ("Kies een type trigger om het instellen te starten..."); pick **Aangepaste gebeurtenis** (Custom Event).
   - Fill **Naam van gebeurtenis** with the exact dataLayer event name (e.g. `moodboard_view`). Case-sensitive.
   - Leave "Deze trigger wordt geactiveerd voor" on **Alle aangepaste gebeurtenissen** unless you need a filter.
4. Click **Opslaan** (Save).

Success criteria: the trigger appears in the triggers list with "0" tags.

## Pattern B — Create a GA4 Event tag bound to a trigger

The fastest path is to **duplicate an existing GA4 tag** rather than create from scratch, because duplication copies the Measurement ID + Google-tag config + the `{{dlv - *}}` variables you need.

1. Navigate to `/workspaces/<ws>/tags/<some-existing-ga4-tag-id>`.
2. Click the overflow menu (three-dot button next to Save) → **Kopiëren** (Duplicate).
3. A new tag opens named `Kopie van <original>`. Rename it: convention `GA4 - <event_name>`.
4. Click the edit icon on the **Tagconfiguratie** section:
   - Change **Gebeurtenisnaam** to the new event name.
   - Remove params you don't need (click each row's trash icon).
   - Add params you need (click **Parameter toevoegen** — name = GA4 param key, value = `{{dlv - <name>}}` from existing variables or a new one you create).
5. Exit the config section. Click the edit icon on the **Triggers** section:
   - Click the `x` on the inherited trigger to remove it.
   - Click the placeholder "Kies een trigger om deze tag te activeren...".
   - Pick the Custom Event trigger created in Pattern A.
6. Click **Opslaan**.

Success criteria: tag list shows it with the correct trigger in the "Activerende triggers" column.

## Pattern C — Fix a tag wired to the wrong trigger

Very common bug: an event tag was originally bound to the wrong Custom Event trigger and is firing on the wrong dataLayer event.

1. Open the tag's edit page directly via its URL (`/tags/<id>`).
2. Click the edit icon on the **Triggers** section.
3. Click the `x` on the current (wrong) trigger chip.
4. Click the placeholder "Kies een trigger..." and select the correct one.
5. Click **Opslaan**.

No need to touch Tagconfiguratie if only the trigger is wrong.

## Pattern D — Enable built-in variables (Scroll Depth, Click data, etc.)

Many trigger types (scroll, visibility, video) require their matching built-in variables to be enabled first.

1. Navigate to `/workspaces/<ws>/variables`.
2. In the "Ingebouwde variabelen" section at the top, click **Configureren**.
3. Check the variables you need in the right-side panel. Typical groups:
   - **Scrollen** — Scroll Depth Threshold, Scroll Depth Units, Scroll Direction
   - **Klikken** — Click Element, Click Target (Click Classes/ID/URL/Text already enabled in most containers)
   - **Zichtbaarheid** — Percent Visible, On-Screen Duration
   - **Video's** — Video Provider, Video Status, Video URL, Video Title, Video Percent
4. Click the X to close the panel. Variables are saved automatically.

Each enabled variable counts as a workspace change.

## Pattern E — Scroll Depth trigger + GA4 tag

1. Enable built-in scroll variables (Pattern D).
2. Create trigger:
   - Name: `Scroll Depth - All Pages`
   - Type: **Scrolldiepte**
   - Check **Diepten voor verticaal scrollen**; pick **Percentages**; enter `25,50,75,100`.
   - Trigger on **Window Load (gtm.load)** (default; leave it).
   - Fires on **Alle pagina's**.
   - Save.
3. Create GA4 tag (duplicate pattern):
   - Name: `GA4 - scroll_depth`
   - Event name: `scroll_depth` (use `page_scroll` if `scroll` is already reserved by GA4 Enhanced Measurement)
   - Param: `percent_scrolled` = `{{Scroll Depth Threshold}}`
   - Trigger: `Scroll Depth - All Pages`
   - Save.

**Note on duplication with GA4 default scroll tracking:** GA4 Enhanced Measurement tracks a `scroll` event at 90% by default. Don't name your event `scroll` — use `scroll_depth` or `page_scroll` to avoid collisions.

## Pattern F — Meta Pixel custom event via Custom HTML

GA4 uses the native template; Meta Pixel doesn't. Use a Custom HTML tag firing `fbq('trackCustom', ...)`.

1. Tags → Nieuw.
2. Name: `Meta Pixel - <EventName>` (e.g. `Meta Pixel - MoodboardHotspotClick`).
3. Tag type: **Aangepaste HTML**.
4. HTML:
   ```html
   <script>
     if (typeof fbq !== 'undefined') {
       fbq('trackCustom', 'MoodboardHotspotClick', {
         content_name: {{dlv - product_name}},
         content_ids: [{{dlv - product_handle}}],
         content_type: 'product'
       });
     }
   </script>
   ```
5. Trigger: the relevant Custom Event trigger (`CE - moodboard_hotspot_click`).
6. Save.

## Pattern G — Pause / delete a tag

1. Open the tag's edit page.
2. Overflow menu → **Onderbreken** (Pause) to disable without deleting, or **Verwijderen** (Delete).

Prefer **pause** during investigation — easier to restore. Delete only after confirming no dependencies.

## Pattern H — Create a dataLayer variable (`dlv - *`)

Used as a param value in GA4/Meta tags to pull data from the dataLayer push.

1. Navigate to `/workspaces/<ws>/variables`.
2. Scroll to "Door de gebruiker gedefinieerde variabelen" → **Nieuw**.
3. Name: `dlv - <key>` (e.g. `dlv - product_name`).
4. Type: **Variabele voor gegevenslaag**.
5. **Naam van gegevenslaagvariabele**: the exact dataLayer key (e.g. `product_name`).
6. Save.

Now usable as `{{dlv - product_name}}` anywhere a variable is accepted.

## UI automation recipes (chrome-devtools-mcp)

GTM's SPA is occasionally slow. This sequence works reliably:

```
navigate_page → wait 1500ms → take_snapshot → click (returns new snapshot) → ...
```

- **Element IDs rotate per snapshot.** Always snapshot immediately before clicking.
- **Overflow menu items (Kopiëren, Verwijderen) are in a popup** that opens as a sibling. If a `click` on an item times out, re-click the overflow button to re-open the menu, then immediately click the item from the fresh snapshot.
- **Edit-mode buttons** on each Section (Tagconfiguratie, Triggers) are labelled `"Section edit button, click to enable edit mode for this section"`. They toggle to "Exit edit mode button..." when active. Clicking them opens the inline editor for that section.
- **Trigger picker** is a modal with a list of all triggers — click the trigger's row button to select (no separate "OK").
- **Fill textboxes** with `fill` (Dutch label "textbox") — the current value shows in the `value="..."` attribute of the snapshot's `generic` or `textbox` element.

## After modifying — workspace counter

The header shows **Wijzigingen in werkruimte: N**. After a save, this increments. If it doesn't, the save didn't take — re-check the UI state.

Every workspace change is listed under the container Overzicht (Overview) page with timestamps and user emails, and the full list reappears in the Publish dialog.

## Next steps

After modifying, go to [preview.md](preview.md) to verify, then [publish.md](publish.md) to ship.
