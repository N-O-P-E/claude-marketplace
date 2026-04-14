# GCP Cloud Run Setup

## Info to Collect
- **Service name**: Name for the Cloud Run service
- **Region**: Where to deploy (e.g., `us-central1`, `europe-west1`)
- **Container image**: Full image path (e.g., `gcr.io/PROJECT/image:tag` or Artifact Registry path)
- **Port**: Container port (default 8080)
- **Environment variables**: Key-value pairs
- **Secrets**: Secret Manager references to mount
- **Memory/CPU**: Resource limits
- **Min/Max instances**: Scaling configuration
- **Authentication**: Allow unauthenticated (public) or require IAM auth
- **Service account**: Which SA the service runs as

## Prerequisites
- APIs enabled: `run.googleapis.com`, `cloudbuild.googleapis.com` (if building), `artifactregistry.googleapis.com` (if using AR)
- Container image must exist (either pre-built or set up Cloud Build)
- Billing must be linked

## Console Steps

### Creating a Cloud Run Service
1. Navigate to `https://console.cloud.google.com/run/create?project=PROJECT_ID`
2. wait_for "Create service" page
3. take_snapshot to find form elements

### Container Configuration
4. Select "Deploy one revision from an existing container image"
5. fill the container image URL
6. fill the service name
7. Select the region from dropdown

### Authentication
8. Choose "Allow unauthenticated invocations" (public) or "Require authentication" (IAM)

### Advanced Settings
9. Click to expand "Container, Networking, Security" section
10. Under Container tab:
    - Set container port if not 8080
    - Set memory and CPU limits
    - Add environment variables
    - Mount secrets from Secret Manager
11. Under Networking tab: configure VPC connector if needed
12. Under Security tab: select the service account

### Scaling
13. Set minimum instances (0 for scale-to-zero, 1+ for always-on)
14. Set maximum instances

### Deploy
15. Click "Create"
16. wait_for deployment to complete (watch for "Deploying..." status)
17. take_screenshot to verify the service is active
18. The service URL is displayed at the top — capture it for the user

## Common Issues
- **Image not found**: Container image path is wrong or permissions are missing
- **Port mismatch**: Container listens on different port than configured
- **Cold start timeout**: If min instances is 0, first request may timeout — increase timeout or set min instances to 1
- **VPC connector required**: If connecting to internal resources (Cloud SQL, Redis), need a Serverless VPC Access connector
