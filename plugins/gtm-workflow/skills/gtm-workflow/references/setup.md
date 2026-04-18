# Connect to Google Tag Manager via chrome-devtools-mcp

This is the bootstrapping sequence. Do it once at the start of any GTM session, then reuse the container/workspace IDs for the rest of the session.

## 1. Verify chrome-devtools-mcp is available

Call `list_pages`. If it returns a list (even empty-ish), the MCP is wired up. If it errors with something like "server not connected" or the tool itself is missing, stop — the plugin can't operate. Tell the user to install/enable `chrome-devtools-mcp` before retrying.

If `list_pages` errors with a profile-lock message (`SingletonLock`, "browser is already running for ..."), see `troubleshooting.md` — it's a stale lock, not a real running browser.

Expected happy-path response:

```json
[
  { "pageId": "A1B2...", "url": "about:blank", "title": "" }
]
```

## 2. Navigate to GTM

```text
navigate_page({ url: "https://tagmanager.google.com/#/home" })
```

GTM is an SPA — the hash fragment matters. Always include `#/home` on the initial navigation so you land on the account list, not a cached container.

Follow with `take_snapshot` so you can read the DOM labels.

## 3. Detect auth state

GTM redirects unauthenticated visitors to `accounts.google.com`. Check both URL and DOM:

```text
evaluate_script({
  function: `() => ({
    url: location.href,
    hostname: location.hostname,
    hasAccountList: !!document.querySelector('[data-account-id], [href*="/container/accounts/"]'),
    bodyText: document.body.innerText.slice(0, 400)
  })`
})
```

Interpret the result:

- `hostname === "accounts.google.com"` → user is signed out. Tell them to sign in manually in the MCP-controlled Chrome tab. **Do not** collect credentials, do not type into the password field. Wait for the user to confirm sign-in, then re-run the detection.
- `hostname === "tagmanager.google.com"` and `hasAccountList === true` → authenticated, proceed.
- `hostname === "tagmanager.google.com"` but no account list → probably loading. `wait_for` 1–2s, snapshot again.

## 4. Read the account + container list

The home page renders a vertical list of account cards. Each card has:

- An account name (visible text, e.g. "Studio Nope").
- One or more container rows, each a link with href like `/container/accounts/6001234567/containers/237242066`.

Grab the IDs directly from the href — never infer or reuse IDs from earlier sessions.

```text
evaluate_script({
  function: `() => {
    const links = Array.from(document.querySelectorAll('a[href*="/container/accounts/"]'));
    return links.map(a => {
      const m = a.getAttribute('href').match(/accounts\\/(\\d+)\\/containers\\/(\\d+)/);
      return m ? {
        accountId: m[1],
        containerId: m[2],
        label: a.innerText.trim(),
        href: a.getAttribute('href')
      } : null;
    }).filter(Boolean);
  }`
})
```

Example output:

```json
[
  { "accountId": "6001234567", "containerId": "237242066", "label": "ByTheMood GTM-ABC1234", "href": "/container/accounts/6001234567/containers/237242066" },
  { "accountId": "6001234567", "containerId": "298811102", "label": "Staging GTM-STG9999", "href": "/container/accounts/6001234567/containers/298811102" }
]
```

## 5. Disambiguate when there are multiple containers

If the list has more than one entry, ask the user which container to use. Quote the label exactly — the GTM-XXX public ID is usually in the label text and helps them pick.

Example prompt: "I see two containers: `ByTheMood (GTM-ABC1234)` and `Staging (GTM-STG9999)`. Which one?"

If there's exactly one, use it without asking.

Store the chosen `{ accountId, containerId }` in working memory for the rest of the session.

## 6. Navigate into the workspace

The root container URL is:

```
https://tagmanager.google.com/#/container/accounts/<ACCOUNT>/containers/<CONTAINER>
```

Navigating there lands on the container overview and GTM redirects to the current default workspace, e.g.:

```
#/container/accounts/6001234567/containers/237242066/workspaces/5
```

**Always discover the workspace ID from the redirect.** Do not hardcode `workspaces/4`. GTM creates a new default workspace after every publish, so the number increments over time.

```text
navigate_page({
  url: "https://tagmanager.google.com/#/container/accounts/6001234567/containers/237242066"
})
// then, after a brief wait:
evaluate_script({ function: `() => location.hash` })
// → "#/container/accounts/6001234567/containers/237242066/workspaces/5"
```

Parse the `workspaces/<N>` segment and save it.

## 7. Navigate straight to subsections when you have the workspace ID

Once you have a valid workspace ID, jump directly to whichever section you need — it's faster than clicking through the sidebar.

```text
// Tags list
navigate_page({ url: "https://tagmanager.google.com/#/container/accounts/<A>/containers/<C>/workspaces/<W>/tags" })

// Triggers
navigate_page({ url: ".../workspaces/<W>/triggers" })

// Variables (both built-in and user-defined live here)
navigate_page({ url: ".../workspaces/<W>/variables" })

// Folders
navigate_page({ url: ".../workspaces/<W>/folders" })

// Overview (pending changes)
navigate_page({ url: ".../workspaces/<W>/overview" })
```

If any of these 404 or redirect to the overview, the workspace ID is stale — go back to step 6 and rediscover.

## 8. Viewport

GTM's admin UI renders cleanly at 1440x900. You don't need to call `resize_page`. Modal dialogs (trigger picker, tag picker) are fine at that size. If you're on a smaller default and see overlapping layout, resize to 1440x900 once and leave it.

## 9. Session state to retain

By the end of setup you should have:

```json
{
  "accountId": "6001234567",
  "containerId": "237242066",
  "workspaceId": "5",
  "containerLabel": "ByTheMood (GTM-ABC1234)",
  "publicId": "GTM-ABC1234"
}
```

The `publicId` (`GTM-XXXXXXX`) is the one merchants paste into themes. You can extract it from the container label or from `evaluate_script` reading the sidebar.

## 10. Quick sanity snapshot

Before doing any mutation, run `take_snapshot` on the `/workspaces/<W>/tags` page. Confirm you see the tags table and the "Nieuw" button (Dutch UI) or "New" button (English UI). If the snapshot is empty or shows a login form, your workspace ID is wrong or the session expired — rerun from step 3.
