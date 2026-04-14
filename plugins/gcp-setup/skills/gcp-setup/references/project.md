# GCP Project Creation

## Info to Collect
- **Project name**: Display name (e.g., "My App Production")
- **Project ID**: Unique identifier, auto-generated from name but user may want custom (e.g., "my-app-prod-2024"). Must be 6-30 chars, lowercase letters/digits/hyphens, start with letter.
- **Organization**: Optional. If user is in a Google Workspace org, projects can live under it.
- **Folder**: Optional. Organizational folders within an org.
- **Labels**: Optional key-value pairs for cost tracking.

## Console Steps
1. Navigate to `https://console.cloud.google.com/projectcreate`
2. wait_for "New Project" heading
3. take_snapshot to find the project name input
4. fill the project name field
5. If org/folder needed: click the Organization dropdown, select the right org
6. If custom project ID: click "Edit" next to the auto-generated ID, fill custom ID
7. Click "Create"
8. wait_for the notification that project was created (can take 10-30 seconds)
9. Navigate to the new project's dashboard to confirm

## Common Issues
- **Quota exceeded**: Orgs have project creation limits. User needs to request increase or delete unused projects.
- **Organization policy**: Some orgs restrict who can create projects. Manual step if policy blocks it.
- **Project ID taken**: IDs are globally unique across all of GCP. Suggest adding a suffix.

## After Creation
- The new project has no billing account linked — billing setup is usually the next step
- No APIs are enabled by default (except a few core ones)
- The creator gets Owner role automatically
