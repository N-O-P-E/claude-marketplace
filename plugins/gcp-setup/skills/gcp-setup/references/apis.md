# GCP APIs & Services

## Info to Collect
- **Which APIs to enable**: Exact API names. Always confirm with the user — don't guess.
- **Project ID**: APIs are enabled per-project.

## Common API Names by Use Case

### Web App / Cloud Run
- `run.googleapis.com` — Cloud Run
- `cloudbuild.googleapis.com` — Cloud Build
- `artifactregistry.googleapis.com` — Artifact Registry (container images)
- `secretmanager.googleapis.com` — Secret Manager

### Data / Analytics
- `bigquery.googleapis.com` — BigQuery
- `dataflow.googleapis.com` — Dataflow
- `pubsub.googleapis.com` — Pub/Sub
- `storage.googleapis.com` — Cloud Storage (usually auto-enabled)

### AI / ML
- `aiplatform.googleapis.com` — Vertex AI
- `translate.googleapis.com` — Cloud Translation
- `vision.googleapis.com` — Cloud Vision
- `speech.googleapis.com` — Speech-to-Text
- `language.googleapis.com` — Natural Language API

### Auth & Identity
- `iamcredentials.googleapis.com` — IAM Service Account Credentials
- `identitytoolkit.googleapis.com` — Identity Platform / Firebase Auth
- `cloudresourcemanager.googleapis.com` — Resource Manager

### Compute
- `compute.googleapis.com` — Compute Engine
- `container.googleapis.com` — Google Kubernetes Engine
- `appengine.googleapis.com` — App Engine
- `cloudfunctions.googleapis.com` — Cloud Functions

### Database
- `firestore.googleapis.com` — Firestore
- `sqladmin.googleapis.com` — Cloud SQL Admin
- `redis.googleapis.com` — Memorystore for Redis
- `spanner.googleapis.com` — Cloud Spanner

## Console Steps (per API)
1. Navigate to `https://console.cloud.google.com/apis/library/API_NAME?project=PROJECT_ID`
2. wait_for the API page to load (look for "Enable" button or API name heading)
3. take_snapshot to find the Enable button
4. click the Enable button
5. wait_for "Disable" or "Manage" text — this means the API is now enabled (can take 10-30 seconds)
6. take_screenshot to confirm

## Batch Enabling
If enabling many APIs, you can also use the search in the API Library:
1. Navigate to `https://console.cloud.google.com/apis/library?project=PROJECT_ID`
2. Search for the API name
3. Click into it and enable

But direct URL navigation per API is more reliable — search can be flaky.

## Common Issues
- **Billing required**: Some APIs require billing to be linked before they can be enabled
- **API not found**: Double-check the exact API service name
- **Already enabled**: If you see "Manage" or "Disable" instead of "Enable," it's already on — skip it
