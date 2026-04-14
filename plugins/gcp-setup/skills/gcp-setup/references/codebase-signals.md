# Codebase Signals → GCP Requirements

When invoked in context-aware mode (mid-feature), scan the codebase for these signals to auto-detect what GCP services are needed. This avoids asking the user questions you can answer yourself.

## Package Dependencies → APIs

### Node.js (package.json)
| Package | GCP API | Service Account Role |
|---------|---------|---------------------|
| `@google-cloud/firestore` | `firestore.googleapis.com` | `roles/datastore.user` |
| `@google-cloud/storage` | `storage.googleapis.com` | `roles/storage.objectAdmin` |
| `@google-cloud/bigquery` | `bigquery.googleapis.com` | `roles/bigquery.dataEditor` + `roles/bigquery.jobUser` |
| `@google-cloud/pubsub` | `pubsub.googleapis.com` | `roles/pubsub.editor` |
| `@google-cloud/secret-manager` | `secretmanager.googleapis.com` | `roles/secretmanager.secretAccessor` |
| `@google-cloud/run` | `run.googleapis.com` | `roles/run.invoker` |
| `@google-cloud/translate` | `translate.googleapis.com` | `roles/cloudtranslate.user` |
| `@google-cloud/vision` | `vision.googleapis.com` | `roles/aiplatform.user` |
| `@google-cloud/text-to-speech` | `texttospeech.googleapis.com` | — |
| `@google-cloud/speech` | `speech.googleapis.com` | — |
| `firebase-admin` | `firestore.googleapis.com` + `identitytoolkit.googleapis.com` | `roles/firebase.admin` |
| `@google-cloud/tasks` | `cloudtasks.googleapis.com` | `roles/cloudtasks.enqueuer` |
| `@google-cloud/scheduler` | `cloudscheduler.googleapis.com` | `roles/cloudscheduler.admin` |

### Python (requirements.txt / pyproject.toml)
| Package | GCP API | Service Account Role |
|---------|---------|---------------------|
| `google-cloud-firestore` | `firestore.googleapis.com` | `roles/datastore.user` |
| `google-cloud-storage` | `storage.googleapis.com` | `roles/storage.objectAdmin` |
| `google-cloud-bigquery` | `bigquery.googleapis.com` | `roles/bigquery.dataEditor` |
| `google-cloud-pubsub` | `pubsub.googleapis.com` | `roles/pubsub.editor` |
| `google-cloud-secret-manager` | `secretmanager.googleapis.com` | `roles/secretmanager.secretAccessor` |
| `google-cloud-aiplatform` | `aiplatform.googleapis.com` | `roles/aiplatform.user` |
| `google-cloud-translate` | `translate.googleapis.com` | `roles/cloudtranslate.user` |
| `google-cloud-vision` | `vision.googleapis.com` | — |
| `google-cloud-logging` | `logging.googleapis.com` | `roles/logging.logWriter` |
| `firebase-admin` | `firestore.googleapis.com` | `roles/firebase.admin` |

### Go (go.mod)
| Module | GCP API |
|--------|---------|
| `cloud.google.com/go/firestore` | `firestore.googleapis.com` |
| `cloud.google.com/go/storage` | `storage.googleapis.com` |
| `cloud.google.com/go/bigquery` | `bigquery.googleapis.com` |
| `cloud.google.com/go/pubsub` | `pubsub.googleapis.com` |
| `cloud.google.com/go/secretmanager` | `secretmanager.googleapis.com` |

## Config Files → Deployment Target

| File | Implies |
|------|---------|
| `Dockerfile` | Likely Cloud Run or GKE |
| `app.yaml` | App Engine |
| `cloudbuild.yaml` / `cloudbuild.json` | Cloud Build (`cloudbuild.googleapis.com`) |
| `.gcloudignore` | Some GCP deployment |
| `firebase.json` | Firebase / Firestore |
| `firestore.rules` | Firestore |
| `storage.rules` | Cloud Storage via Firebase |

## Environment Variables → Project/Config

Look for these in `.env`, `.env.example`, `docker-compose.yml`, CI configs:

| Variable | What it tells you |
|----------|------------------|
| `GOOGLE_CLOUD_PROJECT` / `GCLOUD_PROJECT` / `GCP_PROJECT_ID` | Project ID |
| `GOOGLE_APPLICATION_CREDENTIALS` | Service account key path |
| `GOOGLE_CLOUD_REGION` / `GCP_REGION` | Deployment region |
| `FIREBASE_PROJECT_ID` | Firebase/GCP project |
| `CLOUDSQL_CONNECTION_NAME` | Cloud SQL instance |
| `GOOGLE_CLOUD_BUCKET` / `GCS_BUCKET` | Storage bucket name |

## CI/CD Config → Service Account Needs

| CI System | Config File | What to look for |
|-----------|-------------|------------------|
| GitHub Actions | `.github/workflows/*.yml` | `google-github-actions/auth`, `google-github-actions/deploy-cloudrun`, workload identity |
| GitLab CI | `.gitlab-ci.yml` | `gcloud` commands, `GOOGLE_APPLICATION_CREDENTIALS` |
| Cloud Build | `cloudbuild.yaml` | Build steps, substitutions |
| Terraform | `*.tf` | `google` provider, resource definitions |

## What to Do with Signals

1. **Collect all signals** — scan dependency files, config files, env files, and CI configs
2. **Deduplicate** — multiple signals may point to the same API
3. **Determine roles** — combine all roles needed into the minimum set of service accounts
4. **Check what's already set up** — if you find a project ID, that project might already have some APIs enabled
5. **Present findings** — show the user what you detected and what you'd set up, clearly marking what was auto-detected vs. what you're assuming
