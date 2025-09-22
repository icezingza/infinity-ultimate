# Infinity AI Framework — Ultimate Edition (Starter)

GitHub-first + CI/CD → Google Cloud Run. This starter gives you:
- Minimal FastAPI app
- Docker build & push to Artifact Registry
- Deploy to Cloud Run via GitHub Actions using **Workload Identity Federation (no JSON keys in repo)**
- Terraform scaffold to provision core GCP resources

## Quickstart (Dev)

```bash
python -m venv .venv && source .venv/bin/activate
pip install -r requirements.txt
uvicorn app.main:app --reload --port 8000
```

## Deploy Flow (Overview)
1. Create a **private GitHub repo** and push this code.
2. Use Terraform to provision GCP resources (Artifact Registry, Service Account, Workload Identity Pool/Provider, Cloud Run).
3. Configure GitHub Actions OIDC to access GCP (no secrets.json).
4. Push to `main` → build, push, deploy to Cloud Run (dev).

> **Security:** Never commit secrets. Rotate any previously exposed keys (Firebase/API/SA). Use **Secret Manager** instead.

## Terraform (GCP bootstrap)
- Edit `terraform/envs/dev/terraform.tfvars` with your values (sample below).
- Run:

```bash
cd terraform/envs/dev
terraform init
terraform apply -var-file=terraform.tfvars
```

**Sample `terraform.tfvars`:**
```hcl
project_id = "YOUR_GCP_PROJECT_ID"
region     = "asia-southeast1"
repo_name  = "infinity-ultimate"
service_name = "infinity-ultimate-api"
wif_pool_id   = "github-oidc-pool"
wif_provider_id = "github-oidc-provider"
github_repo = "your-org/infinity-ultimate" # owner/repo
```

## GitHub Actions (CI/CD)
- After Terraform, go to your repo **Settings → Actions → OIDC** note the audience if needed.
- Set repository variables (Settings → Secrets and variables → **Actions** → **Variables**):
  - `GCP_PROJECT_ID`
  - `GCP_REGION`
  - `ARTIFACT_REPO` (e.g. "infinity-ultimate")
  - `SERVICE_NAME` (e.g. "infinity-ultimate-api")
  - `WIF_PROVIDER` (format: "projects/PROJECT_NUMBER/locations/global/workloadIdentityPools/POOL/providers/PROVIDER")
  - `WIF_SERVICE_ACCOUNT` (e.g. "github-deployer@PROJECT_ID.iam.gserviceaccount.com")

Push to `main` to trigger build & deploy.

## Structure
```
app/                    # FastAPI app
.github/workflows/      # CI/CD pipeline
terraform/              # IaC for GCP
```

## Next steps
- Wire your real Infinity modules under `app/` and expose `/metrics` for Prometheus.
- Add staging/prod envs by duplicating `terraform/envs/dev` and adjusting tfvars.
- Add policy checks (tfsec, trivy, codeql) into the workflow.