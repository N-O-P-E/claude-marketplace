---
name: gcp-setup
description: Automates Google Cloud Console setup via browser. Use when the user wants to set up a GCP project, enable APIs, create service accounts, configure IAM, set up billing, configure OAuth, set up firewall rules, or deploy to Cloud Run/App Engine/GKE. Also use when the user says "set up GCP", "configure Google Cloud", "create a cloud project", "enable cloud APIs", or needs any Google Cloud Console configuration done through the browser. ALSO use this when you're mid-feature and realize GCP infrastructure is needed — the skill auto-detects requirements from the codebase (package dependencies, config files, env vars, CI configs) and only asks about what it can't figure out itself. This skill handles the clicking so the user doesn't have to.
allowed-tools:
  - AskUserQuestion
  - Read
  - Write
  - Bash
  - mcp__plugin_chrome-devtools-mcp_chrome-devtools__navigate_page
  - mcp__plugin_chrome-devtools-mcp_chrome-devtools__click
  - mcp__plugin_chrome-devtools-mcp_chrome-devtools__fill
  - mcp__plugin_chrome-devtools-mcp_chrome-devtools__fill_form
  - mcp__plugin_chrome-devtools-mcp_chrome-devtools__take_screenshot
  - mcp__plugin_chrome-devtools-mcp_chrome-devtools__take_snapshot
  - mcp__plugin_chrome-devtools-mcp_chrome-devtools__wait_for
  - mcp__plugin_chrome-devtools-mcp_chrome-devtools__list_pages
  - mcp__plugin_chrome-devtools-mcp_chrome-devtools__select_page
  - mcp__plugin_chrome-devtools-mcp_chrome-devtools__evaluate_script
  - mcp__plugin_chrome-devtools-mcp_chrome-devtools__new_page
  - mcp__plugin_chrome-devtools-mcp_chrome-devtools__press_key
---

# GCP Setup — Browser-Automated Google Cloud Configuration

Walks through Google Cloud Console in the browser so the user doesn't have to click through dozens of screens. The user describes what they need, you figure out the steps, then drive Chrome through each one.

---

## Before You Start

Check these before opening the browser. If anything is missing, tell the user what to do first.

**You need:**
- [ ] Chrome browser running
- [ ] Chrome DevTools MCP extension active and connected (verify with `list_pages`)
- [ ] A Google account with access to GCP
- [ ] A billing account (if creating a new project or enabling paid APIs)
- [ ] Your organization ID (if working within a Google Workspace org — find it at `https://console.cloud.google.com/cloud-resource-manager`)

**Check for an existing setup summary:** look for `.gcp-setup.md` in the project root. If it exists, read it — it may contain project IDs, service account emails, and previously enabled APIs that let you skip questions and avoid duplicating work.

---

## Invocation Modes

This skill works in two modes:

### Mode 1: Direct invocation (`/gcp-setup`)
The user explicitly asks to set up GCP. Start with the Common Setups section below, then Discovery.

### Mode 2: Context-aware invocation (`/gcp-setup` during feature work)
You're mid-feature (building an app, deploying a service, etc.) and realize GCP infrastructure is needed — or the user invokes `/gcp-setup` after you've told them what cloud resources the feature requires.

In this mode, **skip most discovery questions** because you already know the answers from the feature context. Instead:

1. **Analyze the current context** — what are you building? What files have you been working on? What does the feature need?
2. **Derive the GCP requirements automatically:**
   - Scan the codebase for imports/config that imply GCP services (e.g., `@google-cloud/firestore` → Firestore API, `@google-cloud/storage` → Cloud Storage API)
   - Check for existing GCP config files (`app.yaml`, `cloudbuild.yaml`, `.gcloudignore`, `Dockerfile`, environment variables referencing `GOOGLE_CLOUD_PROJECT`, etc.)
   - Look at package.json, requirements.txt, go.mod, etc. for GCP SDK dependencies
   - Check for existing service account references in CI/CD configs (`.github/workflows/`, `Makefile`, etc.)
   - See `references/codebase-signals.md` for the full lookup table
3. **Present what you've figured out:**

```
Based on the feature we're building, here's what I need to set up in GCP:

Project: [existing project ID from config, or "need to create one"]

APIs to enable:
  - Cloud Run API (your Dockerfile + deploy config)
  - Secret Manager API (you're using process.env.DB_PASSWORD with Secret Manager)
  - Cloud SQL Admin API (your database config points to Cloud SQL)

Service account needed:
  - "app-runtime" with roles: Cloud SQL Client, Secret Manager Accessor
    (your app code accesses these services at runtime)

I still need to know:
  - Which GCP project to use? [if not found in config]
  - Which region? [if not in config]
  - Is billing already set up?

Can you fill in the blanks?
```

4. **Only ask about what you genuinely don't know.** If the project ID is in an env file, don't ask for it. If the region is in a deploy config, don't ask for it.
5. Once the user confirms, proceed to Phase 2 (Execution) as normal.

---

## How This Works

```
┌──────────────┐     ┌──────────────┐     ┌──────────────┐     ┌──────────────┐
│   Discovery  │ ──▶ │   Planning   │ ──▶ │  Execution   │ ──▶ │ Verification │
│  (no browser)│     │  (no browser)│     │  (browser)   │     │ (screenshot) │
└──────────────┘     └──────────────┘     └──────────────┘     └──────────────┘
       │                    │                    │                      │
  Ask questions        Build step list      Drive Chrome          Confirm each
  until 100% clear     with manual flags    click-by-click        step worked
```

There are two strict phases: **Discovery** (gather everything, no browser) and **Execution** (drive the browser). Never open the browser until discovery is complete and the user has confirmed the plan.

---

## Common Setups

For direct invocations, offer presets before asking detailed questions. Use `AskUserQuestion`:

```
question: "What are you trying to set up?"
header: "GCP Setup"
options:
  - label: "Standard web app"
    description: "Cloud Run + Secret Manager + Artifact Registry. Full deployment pipeline."
  - label: "CI/CD deployment pipeline"
    description: "Service account + Artifact Registry + GitHub Actions secret. Automates deploys."
  - label: "Full project from scratch"
    description: "New project + billing + APIs + service accounts. The whole thing."
  - label: "Something specific"
    description: "I'll describe what I need — you figure out the steps."
```

**Standard web app** preset defaults:
- APIs: `run.googleapis.com`, `artifactregistry.googleapis.com`, `secretmanager.googleapis.com`, `cloudbuild.googleapis.com`
- Service account: `app-runtime` with `roles/secretmanager.secretAccessor`, `roles/run.invoker`
- Service account: `ci-deployer` with `roles/run.admin`, `roles/artifactregistry.writer`, `roles/iam.serviceAccountUser`

**CI/CD deployment pipeline** preset defaults:
- APIs: `artifactregistry.googleapis.com`, `run.googleapis.com`, `iam.googleapis.com`
- Service account: `ci-deployer` with `roles/run.admin`, `roles/artifactregistry.writer`, `roles/iam.serviceAccountUser`
- Output: JSON key downloaded and ready to paste into GitHub Actions secrets

For presets, confirm the defaults with the user before proceeding:

```
question: "For a standard web app I'll set up: Cloud Run, Secret Manager, Artifact Registry, and two service accounts (app-runtime + ci-deployer). Does that match what you need?"
header: "Confirm preset"
options:
  - label: "Yes, that's right"
    description: "Proceed with these defaults"
  - label: "Mostly right — let me adjust"
    description: "I'll walk through each part"
```

---

## Phase 1: Discovery

The goal is to collect every piece of information needed before touching the browser.

### Step 1: Understand the Goal

From their answer, identify which **setup domains** are involved. Read the relevant reference file(s):

| Domain | Reference File | Key Info Needed |
|--------|---------------|-----------------|
| Project creation | `references/project.md` | Project name, org, folder |
| Billing | `references/billing.md` | Billing account, budget alerts |
| APIs & Services | `references/apis.md` | Which APIs to enable |
| Service Accounts | `references/service-accounts.md` | Names, roles, key generation |
| IAM & Permissions | `references/iam.md` | Members, roles, conditions |
| OAuth & Credentials | `references/oauth.md` | App type, scopes, redirect URIs |
| Cloud Run | `references/cloud-run.md` | Service name, region, container |
| Firewall / VPC | `references/networking.md` | Rules, ranges, protocols |

### Step 2: Check What Already Exists

Before asking questions, check what's already set up to avoid duplicating work:

- If you have a project ID, check whether APIs are already enabled: navigate to `https://console.cloud.google.com/apis/dashboard?project=PROJECT_ID` and take a snapshot — look for APIs already showing as "Enabled"
- If you have a service account name, check whether it already exists: navigate to `https://console.cloud.google.com/iam-admin/serviceaccounts?project=PROJECT_ID`
- Check `.env`, `.env.example`, `app.yaml`, CI configs for project ID, region, and service account references that mean things are already configured

When you find something already set up, say so:

```
I can see Cloud Run API is already enabled on this project. I'll skip that step.
The service account "ci-deployer" already exists — I'll check its roles and add what's missing.
```

### Step 3: Ask Clarifying Questions

Based on the domains involved, ask what you still need to know. Use `AskUserQuestion` for structured choices:

```
question: "Do you have an existing GCP project, or should I create a new one?"
header: "Project"
options:
  - label: "Use existing project"
    description: "I have a project ID already"
  - label: "Create a new project"
    description: "I need a brand new project"
```

```
question: "Which region should services run in?"
header: "Region"
options:
  - label: "Europe West 1 (Belgium)"
    description: "europe-west1 — good for EU users"
  - label: "Europe West 4 (Netherlands)"
    description: "europe-west4"
  - label: "US Central 1 (Iowa)"
    description: "us-central1 — lowest latency for US"
  - label: "Other"
    description: "I'll type the region"
```

**You must be able to answer "yes" to all of these before continuing:**
- [ ] I know exactly which GCP project to use or create
- [ ] I know every API that needs to be enabled (and which are already on)
- [ ] I know every service account, its name, and its roles
- [ ] I know whether billing setup is needed and which account to use
- [ ] I know every IAM binding to create
- [ ] I know all OAuth/credential details if applicable
- [ ] I know the target region for any deployable services

### Step 4: Present the Plan

Once you have everything, present a clear numbered list:

```
Here's what I'll set up in GCP:

1. Create project "my-app-prod" in organization "example.com"
2. Link billing account "My Billing Account"
3. Enable APIs: Cloud Run, Cloud Build, Artifact Registry, Secret Manager
   (Cloud Storage is already enabled — skipping)
4. Create service account "ci-deployer@my-app-prod.iam.gserviceaccount.com"
   - Roles: Cloud Run Admin, Artifact Registry Writer
5. Create OAuth consent screen (External, app name "My App")
6. Create OAuth client ID (Web application, redirect: http://localhost:3000/callback)

Manual steps (you'll need to do these yourself):
- Sign in to Google Cloud Console
- Confirm billing account linking
- Complete any organization policy acknowledgements

Ready to start?
```

Wait for explicit confirmation before proceeding.

---

## Phase 2: Execution

Now you drive the browser. Every step follows the same loop:

### The Step Loop

```
For each step:
  1. Navigate to the correct GCP Console page
  2. take_snapshot to get the page structure and element UIDs
  3. Perform the action (click, fill, select)
  4. wait_for confirmation text or take_snapshot to verify the page updated
  5. take_screenshot to visually confirm success
  6. Report to the user: "Done: [what you just did]. Moving to next step."
  
  If something looks wrong in the screenshot:
  → take_snapshot again, diagnose, retry once
  → If still wrong, stop and ask the user for help
```

### Manual Steps — CRITICAL

Some actions MUST be done by the user. When you hit one:

1. **Stop automation completely**
2. Tell the user exactly what to do and why
3. Wait for them to confirm they've done it
4. take_screenshot to verify before continuing

**Always-manual steps:**
- **Google Sign-in / Authentication** — You cannot and should not handle credentials. Tell the user: "Please sign in to your Google account in the browser. Let me know when you're on the GCP Console dashboard."
- **Billing confirmation** — Any screen that asks to confirm charges or link a billing account. Tell the user: "Please review the billing details on screen and confirm if everything looks correct, then click the confirm button."
- **Organization policy consent** — Any screen requiring org admin approval
- **2FA / Security prompts** — MFA challenges, "verify it's you" screens
- **Payment method entry** — Credit card or bank details
- **Terms of Service acceptance** — Legal agreements

When in doubt about whether a step is sensitive: make it manual. It's always safer to ask.

### Navigation Patterns

GCP Console URLs are predictable. Use direct navigation instead of clicking through menus:

```
Dashboard:           https://console.cloud.google.com/home/dashboard?project=PROJECT_ID
APIs & Services:     https://console.cloud.google.com/apis/dashboard?project=PROJECT_ID
Enable API:          https://console.cloud.google.com/apis/library/API_NAME?project=PROJECT_ID
Service Accounts:    https://console.cloud.google.com/iam-admin/serviceaccounts?project=PROJECT_ID
IAM:                 https://console.cloud.google.com/iam-admin/iam?project=PROJECT_ID
OAuth Consent:       https://console.cloud.google.com/apis/credentials/consent?project=PROJECT_ID
Credentials:         https://console.cloud.google.com/apis/credentials?project=PROJECT_ID
Cloud Run:           https://console.cloud.google.com/run?project=PROJECT_ID
Billing:             https://console.cloud.google.com/billing?project=PROJECT_ID
Create Project:      https://console.cloud.google.com/projectcreate
VPC Firewall:        https://console.cloud.google.com/networking/firewalls?project=PROJECT_ID
Manage Resources:    https://console.cloud.google.com/cloud-resource-manager
Org Policies:        https://console.cloud.google.com/iam-admin/orgpolicies?project=PROJECT_ID
Org Policy (specific): https://console.cloud.google.com/iam-admin/orgpolicies/CONSTRAINT_ID?project=PROJECT_ID
Org IAM:             https://console.cloud.google.com/iam-admin/iam?organizationId=ORG_ID
SA Details:          https://console.cloud.google.com/iam-admin/serviceaccounts/details/SA_UNIQUE_ID?project=PROJECT_ID
SA Keys:             https://console.cloud.google.com/iam-admin/serviceaccounts/details/SA_UNIQUE_ID/keys?project=PROJECT_ID
Domain Delegation:   https://admin.google.com/ac/owl/domainwidedelegation
```

Always navigate directly to the relevant page rather than clicking through the sidebar. It's faster and less error-prone.

### Interacting with GCP Console Pages

GCP Console is a complex single-page app. Elements load asynchronously and modals/drawers appear frequently. Key patterns:

1. **After navigation, always `wait_for` a key element** before taking a snapshot. GCP pages often show a loading spinner first.
2. **After clicking a button that triggers a modal**, wait 1-2 seconds then `take_snapshot` to get the modal's elements.
3. **Dropdown menus** in GCP often use Material Design components. After clicking a dropdown, `take_snapshot` to find the options, then `click` the right one.
4. **"Enable" buttons for APIs** sometimes take 10-30 seconds. Use `wait_for` with text like "Disable" or "Manage" to know when enabling is complete.
5. **Forms with multiple steps** (like OAuth setup) — complete one section, take a snapshot, then continue. Don't try to fill everything at once.

### Error Handling

GCP Console shows errors in various ways — red banners, toast notifications, inline validation messages. After each action:

1. Check the screenshot for any red text, error banners, or "something went wrong" messages
2. If you see an error:
   - `take_snapshot` to read the error text
   - If it's a permissions issue: stop and tell the user they need to grant access
   - If it's a transient error: wait 3 seconds, reload, try again (once)
   - If it's a validation error: read the message, fix the input, retry
   - If you can't resolve it: show the user the screenshot and error text, ask what to do

### Progress Reporting

After completing each major step, report to the user:

```
[2/6] Enabled Cloud Run API
       → Took 12 seconds, API is now active
       → Moving to: Create service account "ci-deployer"
```

This keeps the user informed since browser automation can feel like a black box.

### Write the Setup Summary

When all steps are complete, write a `.gcp-setup.md` file in the project root:

```markdown
# GCP Setup Summary
Generated: YYYY-MM-DD

## Project
- **Project ID**: my-app-prod
- **Project Number**: 123456789
- **Organization**: example.com
- **Region**: europe-west1

## APIs Enabled
- Cloud Run (`run.googleapis.com`)
- Artifact Registry (`artifactregistry.googleapis.com`)
- Secret Manager (`secretmanager.googleapis.com`)
- Cloud Build (`cloudbuild.googleapis.com`)

## Service Accounts
- `ci-deployer@my-app-prod.iam.gserviceaccount.com`
  - Roles: Cloud Run Admin, Artifact Registry Writer, Service Account User
  - Key: downloaded to ~/Downloads/my-app-prod-ci-deployer-XXXX.json
- `app-runtime@my-app-prod.iam.gserviceaccount.com`
  - Roles: Secret Manager Secret Accessor

## OAuth
- Client ID: XXXX.apps.googleusercontent.com
- Redirect URIs: http://localhost:3000/callback

## Notes
Add any manual follow-up steps or reminders here.
```

Tell the user: "I've written a `.gcp-setup.md` summary to your project root. Add it to `.gitignore` if it contains sensitive values — or keep it for reference."

---

## Starting the Browser

Before any browser interaction, verify Chrome DevTools MCP is connected:

1. Call `list_pages` to check if the browser is reachable
2. If it fails, tell the user: "I need Chrome open with DevTools MCP connected. Please make sure Chrome is running and the DevTools MCP extension is active."
3. Once connected, either open a new page or use an existing one
4. Navigate to `https://console.cloud.google.com`
5. `take_screenshot` — if the user isn't signed in, trigger the manual auth step
6. Once signed in and on the dashboard, begin executing the plan

---

## Organization Policies — "Secure by Default" Blocker

Google Workspace organizations created after August 2024 have **Secure by Default** org policies enforced. These commonly block:
- `iam.disableServiceAccountKeyCreation` — blocks creating JSON keys for service accounts
- `iam.disableServiceAccountKeyUpload` — blocks uploading keys

### How to detect
When creating a service account key fails with "Service account key creation is disabled" and mentions `iam.disableServiceAccountKeyCreation`, this is the Secure by Default policy.

### How to fix
This requires **two steps** — granting the role AND disabling the policy. The browser console alone cannot do this; you need `gcloud` CLI.

**Step 1: Grant Org Policy Admin role to the user** (requires org admin / Super Admin):
```bash
gcloud organizations add-iam-policy-binding ORGANIZATION_ID \
  --member="user:USER_EMAIL" \
  --role="roles/orgpolicy.policyAdmin"
```

**Step 2: Delete the org policy** (project-level override is often not enough — delete at org level):
```bash
gcloud org-policies delete iam.disableServiceAccountKeyCreation \
  --organization=ORGANIZATION_ID
```

### Important notes
- **Google Workspace Super Admin ≠ GCP IAM roles.** Being a Workspace super admin does NOT automatically give you GCP org-level permissions. They are separate systems. The user needs to explicitly grant themselves `roles/orgpolicy.policyAdmin` on the organization.
- A **project-level override** (`gcloud resource-manager org-policies disable-enforce ... --project=PROJECT_ID`) may not work if the policy is enforced at the org level with no override allowed. In that case, delete the policy at the org level.
- To find the **Organization ID**: navigate to `https://console.cloud.google.com/cloud-resource-manager` — the org is the top-level row in the resource list with its numeric ID.
- If `gcloud` is not installed, tell the user to run `brew install google-cloud-sdk` (macOS) then `gcloud auth login`.
- **Include this check in the plan** whenever service account key creation is a step. Don't wait for it to fail — proactively warn the user that Secure by Default may block key creation and include the gcloud commands in the plan's "manual steps" section.

---

## Domain-Wide Delegation — Key Details

### GCP Console side
- In newer GCP Console, domain-wide delegation does NOT require a separate "enable" checkbox. The **Client ID** (same as the service account's Unique ID) is automatically available under **Details → Advanced Settings → Domain-Wide Delegation**.
- The Client ID is the numeric Unique ID shown on the service account details page (e.g., `116451652418865281797`).

### Google Admin side
- Domain-wide delegation is configured in **Google Admin** (admin.google.com), NOT in GCP Console.
- Direct URL: `https://admin.google.com/ac/owl/domainwidedelegation`
- The Admin console may be in the user's **local language** (e.g., Dutch: "Nieuw toevoegen" = "Add new", "Autoriseren" = "Authorize", "Domeinbrede machtiging" = "Domain-wide delegation"). The a11y tree will show these localized strings — use them as-is.
- After clicking "Add new" / "Nieuw toevoegen", fill in:
  - **Client ID** field: the numeric service account Unique ID
  - **OAuth scopes** field: comma-separated scopes (e.g., `https://www.googleapis.com/auth/gmail.send`)
- Delegation can take **up to 5 minutes** to propagate after authorization.

### Google Admin requires separate sign-in
- Navigating to `admin.google.com` from GCP Console often triggers a **re-authentication** prompt, even if already signed in to GCP. This is a manual step — tell the user to sign in and confirm when they're on the delegation page.

---

## Browser Session Management

- **Pages can close between interactions.** If the conversation is long, Chrome pages opened earlier may be closed by the user or browser. Always call `list_pages` before trying to navigate an existing page. If no pages are open, use `new_page` to open a fresh one.
- **GCP Console redirects after project creation.** After creating a project, GCP often redirects back to the previously selected project. You need to explicitly navigate to the new project using `?project=NEW_PROJECT_ID` in the URL.
- **Notification panel shows task status.** Project creation and API enabling show up as notifications. After creating a project, wait for the "Create Project: NAME" notification to show "has finished" before proceeding.

---

## Important Principles

1. **Never guess.** If you don't know a project ID, API name, or role — ask. Wrong GCP config is worse than slow GCP config.
2. **Never handle credentials.** Passwords, API keys in plaintext, billing details — always manual.
3. **Screenshot everything.** Every step gets a visual verification. If you can't see it worked, it didn't work.
4. **Direct URLs over clicking menus.** GCP's sidebar navigation is deep and inconsistent. URL navigation is deterministic.
5. **One action at a time.** Don't try to batch multiple clicks or fills. GCP Console is async and things break when you rush.
6. **Announce manual steps clearly.** Don't just say "please do this" — explain what screen they should see, what button to click, and what the result should look like.
7. **Check before creating.** If something might already exist (API enabled, service account created), verify first. Skip the step if it's already done.
8. **Proactively check for org policy blockers.** If the plan includes service account key creation, warn about Secure by Default and include the gcloud fix commands in the plan upfront.
9. **Include gcloud commands as fallback.** Some GCP operations are easier or only possible via CLI. When browser automation hits a permissions wall, offer the gcloud command immediately rather than trying to work around it in the UI.
10. **Always write the setup summary.** End every session with `.gcp-setup.md` so the next invocation — and the user — has a clear record of what was set up.
