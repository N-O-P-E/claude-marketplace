# GCP Service Accounts

## Info to Collect
- **Service account name**: e.g., "ci-deployer" (becomes `ci-deployer@PROJECT_ID.iam.gserviceaccount.com`)
- **Display name**: Human-readable name shown in console
- **Description**: What this service account is for
- **Roles**: Which IAM roles to grant (e.g., `roles/run.admin`, `roles/storage.objectViewer`)
- **Key generation**: Whether to create and download a JSON key (for external use like CI/CD)

## Console Steps

### Creating a Service Account
1. Navigate to `https://console.cloud.google.com/iam-admin/serviceaccounts/create?project=PROJECT_ID`
2. wait_for "Create service account" heading
3. take_snapshot to find the form fields
4. fill the service account name (ID auto-generates from this)
5. fill the display name and description if provided
6. Click "Create and Continue"
7. wait_for the roles section to appear

### Granting Roles
8. take_snapshot to find the role selector
9. Click "Select a role" dropdown
10. Search/filter for the desired role
11. click the correct role
12. If multiple roles: click "Add another role" and repeat
13. Click "Continue"

### Creating a Key (if needed)
14. On the "Grant users access" step, click "Done" (or skip)
15. Navigate to the service account's detail page
16. Click "Keys" tab
17. Click "Add Key" → "Create new key"
18. Select "JSON" format
19. Click "Create"
20. **The key file auto-downloads** — tell the user where it was saved
21. take_screenshot to confirm

## Common Roles by Use Case

### CI/CD Deployment
- `roles/run.admin` — Deploy to Cloud Run
- `roles/cloudbuild.builds.editor` — Trigger Cloud Build
- `roles/artifactregistry.writer` — Push container images
- `roles/iam.serviceAccountUser` — Act as service account

### Data Access
- `roles/bigquery.dataViewer` — Read BigQuery data
- `roles/bigquery.jobUser` — Run BigQuery queries
- `roles/storage.objectViewer` — Read Cloud Storage objects
- `roles/storage.objectAdmin` — Read/write Cloud Storage

### App Backend
- `roles/secretmanager.secretAccessor` — Read secrets
- `roles/firestore.user` — Read/write Firestore
- `roles/pubsub.publisher` — Publish to Pub/Sub
- `roles/pubsub.subscriber` — Subscribe to Pub/Sub

## Common Issues
- **Permission denied creating SA**: User needs `iam.serviceAccounts.create` permission
- **Key download blocked**: Org policy may disable key creation — user must use Workload Identity instead
- **Too many keys**: Limit of 10 keys per service account
