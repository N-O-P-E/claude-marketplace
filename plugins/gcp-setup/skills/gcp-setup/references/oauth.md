# GCP OAuth & Credentials

## Info to Collect
- **App type**: Internal (org users only) or External (any Google user)
- **App name**: Shown on the consent screen
- **Support email**: Displayed on consent screen
- **Scopes**: Which Google APIs the app needs access to (e.g., email, profile, Drive)
- **Authorized domains**: Domains allowed for redirect URIs
- **Client type**: Web application, Desktop app, iOS, Android, or TV
- **Redirect URIs**: Where Google sends users after auth (e.g., `http://localhost:3000/callback`)
- **JavaScript origins**: Allowed origins for browser requests

## Console Steps

### Step 1: Configure OAuth Consent Screen
1. Navigate to `https://console.cloud.google.com/apis/credentials/consent?project=PROJECT_ID`
2. wait_for the consent screen configuration page
3. If first time: choose Internal or External user type, click "Create"
4. take_snapshot to find form fields
5. Fill: App name, User support email, Developer contact email
6. Add authorized domains if needed
7. Click "Save and Continue"

### Step 2: Configure Scopes
8. Click "Add or Remove Scopes"
9. take_snapshot to find the scope selector
10. Search for and select required scopes
11. Click "Update"
12. Click "Save and Continue"

### Step 3: Test Users (External apps only)
13. If External: add test users (email addresses) while app is in "Testing" status
14. Click "Save and Continue"

### Step 4: Create OAuth Client ID
15. Navigate to `https://console.cloud.google.com/apis/credentials?project=PROJECT_ID`
16. Click "Create Credentials" → "OAuth client ID"
17. take_snapshot for the form
18. Select application type (Web application, Desktop, etc.)
19. Fill name
20. For web apps: add authorized redirect URIs and JavaScript origins
21. Click "Create"
22. **IMPORTANT**: A modal appears with the Client ID and Client Secret
23. take_snapshot to capture the credentials
24. Tell the user to copy these now — the secret won't be shown again in full

## Common Scopes
- `openid` — OpenID Connect
- `email` — User's email address
- `profile` — User's basic profile info
- `https://www.googleapis.com/auth/drive.readonly` — Read Google Drive
- `https://www.googleapis.com/auth/calendar.readonly` — Read Google Calendar
- `https://www.googleapis.com/auth/spreadsheets.readonly` — Read Google Sheets

## Common Issues
- **Consent screen not configured**: Must set up consent screen before creating OAuth clients
- **Unverified app warning**: External apps in "Testing" mode show a warning screen. Verification required for production.
- **Redirect URI mismatch**: Most common OAuth error. The redirect URI in the request must exactly match one registered in the console (including trailing slashes and http vs https).
- **Localhost exception**: `http://localhost` redirect URIs work without adding the domain to authorized domains
