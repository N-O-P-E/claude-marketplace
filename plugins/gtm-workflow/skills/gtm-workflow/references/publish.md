# Publishing a GTM version safely

Use this reference when running the **Verzenden** (Submit/Publish) flow. This is the ONLY step in the `gtm-workflow` plugin with production impact — everything else is sandboxed in a workspace. The skill MUST ask the user for explicit approval before clicking Publiceren.

## When to publish

Only publish after ALL of the following are true:

1. All intended workspace changes are **Opgeslagen** (saved). The workspace banner should show "Wijzigingen in werkruimte: N" with N matching the number of edits you expect.
2. **Voorbeeld** (Preview mode) confirms each changed/new tag fires exactly as expected — see `preview.md` for the verification loop.
3. The user has **explicitly approved** going live. Never auto-publish. The skill should always surface the change list and wait for confirmation before clicking **Verzenden**.

If any of the three is missing, stop and return to the prior step.

## Click-by-click publish flow

1. Navigate to any workspace URL (tags, triggers, variables — all work).
2. The header banner shows "Wijzigingen in werkruimte: N". Confirm N matches your expected change count. If N is higher than expected, investigate before publishing — stray edits live in the same workspace.
3. Click **Verzenden** (top-right). A full-screen dialog opens titled "Wijzigingen verzenden".
4. The dialog shows two large radio buttons:
   - **Versie publiceren en maken** (Publish and create version) — the default. Creates a version AND pushes it live. This is what you want for a normal publish.
   - **Versie maken** (Create version only) — snapshots the workspace as a non-live version. Use this for staging without going live.
5. Two text fields sit below:
   - **Versienaam** (Version name) — concise, ideally <60 chars. Subject-line style.
   - **Versiebeschrijving** (Description) — multi-line. Numbered list of each change with intent.
6. **Publiceren naar omgeving** (Publish to environment) — defaults to "Live". Leave it on Live unless the user has set up staging environments and explicitly requests one.
7. Below the form, the full **Wijzigingen in werkruimte** table lists every changed/added/deleted entity (tags, triggers, variables, folders). Scan it. Anything unexpected = stop.
8. Click the big **Publiceren** button at the top-right of the dialog.
9. The URL redirects to `/versions/.../versions/<new-version-id>`. A green banner appears: "Versie N is live" (or similar). Note the version number.

## Version naming conventions

The name and description live forever in the container's version history — they are your audit trail.

**Versienaam (title):**
- Concise, subject-line style
- Present-tense verb + object
- Examples:
  - `Fix affiliate trigger`
  - `Add moodboard_view tracking`
  - `Scroll depth + Meta custom events`
  - `Consolidate GA4 page_view loaders`
  - `Rollback: restore Versie 47`

**Versiebeschrijving (description):**
- Numbered list if more than one change
- Each item: WHAT changed + WHY (one sentence each)
- Mention every trigger/tag/variable created, modified, or deleted
- Call out rollback-worthy notes if anything is risky

Example for a multi-change publish:

```
1) GA4 - affiliate_outbound_click trigger fixed (was CE - moodboard_hotspot_click,
   now CE - affiliate_outbound_click) so it fires only on real outbound clicks,
   not every hotspot open.
2) New CE - moodboard_view trigger + GA4 - moodboard_view tag so moodboard
   impressions land in GA4 events.
3) Added dlv - moodboard_id variable used by both tags above.
```

Example for a single-change publish:

```
Scroll Depth Threshold / Units / Direction enabled in Variables. New trigger
Scroll Depth - 25/50/75/100 and GA4 tag GA4 - scroll_depth. Event name is
scroll_depth (not scroll) to avoid collision with GA4 Enhanced Measurement.
```

## Partial publish / staging a version only

If the changes aren't ready to go live but you want to snapshot progress:

1. Click **Verzenden**.
2. Select **Versie maken** (NOT "Versie publiceren en maken").
3. Fill name/description (same conventions as above).
4. Click the **Maken** (Create) button.

This creates a numbered, non-live version you can publish later from the **Versies** tab. The workspace is then reset to match the latest LIVE version, so you start fresh.

## Rollback

GTM versioning is linear. Every publish mints a new `Versie N`. To roll back to a prior good version:

1. Click the **Versies** tab in the container's left nav.
2. Click the version you want to restore — open its detail page.
3. Click the overflow menu (•••) at the top → **Publiceren als laatste versie** (Publish as most recent).
4. Fill Versienaam/Versiebeschrijving mentioning the rollback reason and the target version number.
5. Click **Publiceren**.

This creates a NEW version (Versie N+1) whose content matches the old one and immediately pushes it live. The buggy version stays in history for forensics.

Alternative: if you have a clean workspace and want to import a prior version's state before publishing, use **Werkruimte → Importeren vanuit versie**, then Verzenden normally. Gives you a chance to tweak before going live.

## Safety checks before hitting Publiceren

The skill should surface these to the user and wait for "go" before clicking:

- [ ] Count of workspace changes matches expectation (Wijzigingen in werkruimte: N)
- [ ] Every change was run through Preview and passes verification
- [ ] No **Waarschuwingen** (warnings) shown in the Verzenden dialog
- [ ] Versienaam is filled (not left blank)
- [ ] Versiebeschrijving is filled with meaningful content (not the default empty state, not just "update")
- [ ] Publiceren naar omgeving is "Live" (or the user explicitly chose another env)
- [ ] User has explicitly said "publish" / "ship it" / equivalent

If ANY check fails: stop, report to user, do not click Publiceren.

## Post-publish verification

Right after publishing:

1. Confirm the "Versie N is live" success banner appeared.
2. Record the new version number in the skill output for traceability.
3. Optionally: revisit the live site with NO `?gtm_debug=...` in the URL and confirm at least one of the new events fires naturally in production. Use a dataLayer hook:

```js
() => {
  window.dataLayer = window.dataLayer || [];
  const origPush = window.dataLayer.push.bind(window.dataLayer);
  window.__capturedEvents = [];
  window.dataLayer.push = function(...args) {
    window.__capturedEvents.push(args[0]);
    return origPush(...args);
  };
  return 'hook installed';
}
```

Then exercise the change, then read `window.__capturedEvents` to confirm the expected event actually pushes in production.

4. If something is wrong in production: immediately roll back (see Rollback section above). Don't try to fix-forward under pressure — restore the last known good version first, then fix in workspace, then re-publish.

## What NOT to do

- Don't click **Verzenden** without Preview verification. Workspaces can look correct and still be broken.
- Don't use "update" as a Versienaam. It tells future-you nothing.
- Don't publish with an empty Versiebeschrijving. Audit trail matters.
- Don't publish while another teammate is mid-edit in the same workspace — they'll lose context on which changes went live. Coordinate.
- Don't skip the Wijzigingen in werkruimte count check — it's your last chance to catch stray edits.
- Don't publish to the Live environment when the user asked for staging.
