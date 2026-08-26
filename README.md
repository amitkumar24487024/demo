# demo

## Run locally

```bash
cd /path/to/demo
./gradlew bootRun
```

The app is available at `http://localhost:8080/api`.

## Deploy to Google Cloud Run

### Prerequisites

- A Google Cloud project
- Billing enabled for the project
- `gcloud` installed and authenticated
- Cloud Run, Cloud Build, and Artifact Registry APIs enabled

```bash
gcloud config set project PROJECT_ID
gcloud services enable run.googleapis.com cloudbuild.googleapis.com artifactregistry.googleapis.com
```

### Deploy from source

```bash
cd /path/to/demo
gcloud run deploy demo \
  --source . \
  --region us-central1 \
  --allow-unauthenticated
```

Cloud Run builds and deploys the application directly from this repository.

### Build and deploy an image manually

```bash
cd /path/to/demo
gcloud builds submit --tag us-central1-docker.pkg.dev/PROJECT_ID/demo/demo
gcloud run deploy demo \
  --image us-central1-docker.pkg.dev/PROJECT_ID/demo/demo \
  --region us-central1 \
  --allow-unauthenticated
```

### Verify

After deployment, open the Cloud Run service URL and request:

```text
/api
```

The service returns `Hello, World!`.

## Notes

- The app reads the Cloud Run `PORT` environment variable and falls back to `8080` locally.
- Adjust CPU, memory, concurrency, and min/max instances in the Cloud Run service settings as needed.

## GitHub Actions CI and Docker Hub publish

Workflow file: `.github/workflows/docker-image.yml`

- On pull requests to `main`, it runs `./gradlew test bootJar --no-daemon` and verifies Docker image build.
- On pushes to `main`, it does the same checks and then publishes the image to Docker Hub.

### Required GitHub repository configuration for Docker Hub push

Set these in GitHub repository settings:

- Repository **Secrets**:
  - `DOCKERHUB_USERNAME` = your Docker Hub username
  - `DOCKERHUB_TOKEN` = Docker Hub access token (not your password)
- Repository **Variables**:
  - `DOCKERHUB_IMAGE` = full Docker image name, e.g. `yourusername/demo`

## GitHub Actions CD to Google Cloud Run

Workflow file: `.github/workflows/deploy-cloud-run.yml`

- Trigger: after **CI and Docker Publish** completes successfully for a `push` to `main`.
- Deploys immutable image tag: `${DOCKERHUB_IMAGE}:${GITHUB_SHA}` (source SHA from the CI workflow run).
- Uses GitHub OIDC + GCP Workload Identity Federation (no JSON key file).
- Uses `production` GitHub Environment and workflow concurrency lock `cloud-run-production`.

### Required GitHub repository configuration for Cloud Run deploy

Set these in GitHub repository settings:

- Repository **Secrets**:
  - `GCP_PROJECT_ID` = Google Cloud project ID
  - `GCP_WIF_PROVIDER` = Workload Identity Provider resource name
    - Example: `projects/123456789/locations/global/workloadIdentityPools/github/providers/my-provider`
  - `GCP_SERVICE_ACCOUNT` = deployer service account email
    - Example: `github-deployer@PROJECT_ID.iam.gserviceaccount.com`
- Repository **Variables**:
  - `GCP_REGION` = Cloud Run region (example: `us-central1`)
  - `CLOUD_RUN_SERVICE` = Cloud Run service name (example: `demo`)
  - `DOCKERHUB_IMAGE` = full Docker image name (already used by CI publish)

### Optional repository variables for runtime flags

- `CLOUD_RUN_ALLOW_UNAUTHENTICATED` = `false` (default) or `true`
- `CLOUD_RUN_PORT` = container port (example: `8080`)
- `CLOUD_RUN_MEMORY` = memory limit (example: `512Mi`)
- `CLOUD_RUN_MIN_INSTANCES` = minimum instances (example: `0`)
- `CLOUD_RUN_MAX_INSTANCES` = maximum instances (example: `3`)

### GCP setup checklist

1. Enable required APIs in your GCP project:
   - Cloud Run API (`run.googleapis.com`)
   - IAM Service Account Credentials API (`iamcredentials.googleapis.com`)
   - Security Token Service API (`sts.googleapis.com`)
2. Create a deploy service account for GitHub Actions.
3. Grant least-privilege IAM roles:
   - `roles/run.admin`
   - `roles/iam.serviceAccountUser` on the Cloud Run runtime service account
4. Configure Workload Identity Federation trust from GitHub to the deploy service account.
