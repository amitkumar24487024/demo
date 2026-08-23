# demo

## Run locally

```bash
cd /home/runner/work/demo/demo
./gradlew bootRun
```

The app is available at `http://localhost:8080/api`.

## Deploy to Google Cloud Run

### Prerequisites

- A Google Cloud project
- Billing enabled for the project
- `gcloud` installed and authenticated
- Cloud Run, Cloud Build, and Artifact Registry APIs enabled

### Deploy from source

```bash
cd /home/runner/work/demo/demo
gcloud run deploy demo \
  --source . \
  --region us-central1 \
  --allow-unauthenticated
```

Cloud Run builds and deploys the application directly from this repository.

### Deploy with Docker

```bash
cd /home/runner/work/demo/demo
gcloud run deploy demo \
  --source . \
  --region us-central1 \
  --allow-unauthenticated
```

If you prefer to build the image yourself:

```bash
cd /home/runner/work/demo/demo
gcloud builds submit --tag REGION-docker.pkg.dev/PROJECT_ID/REPOSITORY/demo
gcloud run deploy demo \
  --image REGION-docker.pkg.dev/PROJECT_ID/REPOSITORY/demo \
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
