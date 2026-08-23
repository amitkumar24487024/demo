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
