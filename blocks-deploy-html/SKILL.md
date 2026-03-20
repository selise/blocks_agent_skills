---
name: blocks-deploy-html
description: Wrap a single-file HTML app in Docker + nginx and deploy it to Blocks Cloud via GitHub Actions on Azure AKS. Use when an HTML file is ready and needs to go live at a *.seliseblocks.com domain.
argument-hint: "[github-org/repo] [app-domain] [service-name]"
disable-model-invocation: true
---

You are setting up the deployment pipeline for an HTML app on Blocks Cloud. This creates the Docker container, nginx config, and GitHub Actions workflow. Done in about 10 minutes (plus infra team setup).

## What you need — tell the user this first

Ask the user to have these ready:

1. **GitHub org and repo name**
   The repo where the HTML file lives, e.g. `selise/my-tool`
   The repo must already exist on GitHub.

2. **App domain**
   The target URL, e.g. `https://pbiwfs-abcd.seliseblocks.com`
   Get this from your SELISE infra admin — they provision the domain.

3. **Service name**
   A short lowercase slug used for container + namespace naming, e.g. `my-tool`
   Convention: matches the repo name if possible.

Tell the user: **You will also need a SELISE infra admin** to complete Step 5. This is unavoidable — they hold the Azure and GitHub org secrets.

If the user already provided these, extract them and proceed.

---

## Step 1 — Create `Dockerfile`

In the root of the repo:

```dockerfile
FROM nginx:alpine
COPY index.html /usr/share/nginx/html/index.html
COPY nginx.conf /etc/nginx/conf.d/default.conf
EXPOSE 80
```

---

## Step 2 — Create `nginx.conf`

```nginx
server {
    listen 80;
    server_name _;
    root /usr/share/nginx/html;
    index index.html;
    location / {
        try_files $uri $uri/ /index.html;
    }
}
```

`try_files` is required. Without it, paths like `/activate?code=...` return 404 and the hosting layer strips them to `/?code=...`, breaking activation links.

---

## Step 3 — Create `.gitignore`

```
.env
*.local
node_modules/
```

---

## Step 4 — Create GitHub Actions Pipeline

### `.github/variables/vars.env`
Replace `{repo-name}` and `{service-name}` with the actual values:

```
REPO_NAME={repo-name}
SERVICE_NAME={service-name}
DOCKERFILE=Dockerfile
```

### `.github/actions/setvars/action.yml`

```yaml
name: 'Set environment variables'
description: 'Configures environment variables for a workflow'
inputs:
  varFilePath:
    required: false
    default: ./.github/variables/*
runs:
  using: "composite"
  steps:
    - run: sed "" ${{ inputs.varFilePath }} >> $GITHUB_ENV
      shell: bash
```

### `.github/workflows/main.yml`

```yaml
name: Build (main)
on:
  push:
    branches: [main]
jobs:
  cd-job:
    permissions:
      contents: read
      id-token: write
    uses: ./.github/workflows/3_web.yml
    with:
      CONTAINER_NAME: 'prod-{service-name}-webclient'
      NAMESPACE: 'prod-{repo-name}'
      SERVICE_NAME: '{service-name}'
      CI_BUILD: 'prod'
    secrets:
      SELISE_GITHUB_PAT: ${{ secrets.SELISE_GITHUB_PAT }}
      AZURE_CREDENTIALS: ${{ secrets.AZURE_AKS_BLOCKS_PROD_CREDENTIALS }}
      AZURE_CONTAINER_REGISTRY: ${{ secrets.AZURE_BLOCKS_PROD_CONTAINER_REGISTRY }}
      ClUSTER_RESOURCE_GROUP: ${{ secrets.ClUSTER_AKS_BLOCKS_PROD_RESOURCE_GROUP }}
      CLUSTER_NAME: ${{ secrets.AKS_BLOCKS_PROD_CLUSTER }}
      ACR_RESOURCE_GROUP: ${{ secrets.ClUSTER_AKS_BLOCKS_PROD_RESOURCE_GROUP }}
```

### `.github/workflows/3_web.yml`

Copy verbatim from:
```
SELISEdigitalplatforms/l3-react-blocks-construct/.github/workflows/3_web.yml
```

---

## Step 5 — Push to GitHub

```bash
git init
git branch -m main
git remote add origin https://github.com/{org}/{repo-name}.git
git add .gitignore Dockerfile nginx.conf index.html .github/
git commit -m "feat: add Docker deployment for Blocks Cloud"
git push -u origin main
```

---

## Step 6 — Infra Team Actions (user must request this)

Tell the user to contact their **SELISE infra admin** and ask them to:

1. **Grant org secrets to the new repo** in GitHub → Org Settings → Actions → Secrets → Repositories:
   - `SELISE_GITHUB_PAT`
   - `AZURE_AKS_BLOCKS_PROD_CREDENTIALS`
   - `AZURE_BLOCKS_PROD_CONTAINER_REGISTRY`
   - `ClUSTER_AKS_BLOCKS_PROD_RESOURCE_GROUP` ← the `l` is lowercase — this is intentional
   - `AKS_BLOCKS_PROD_CLUSTER`

2. **Add a Helm values file** to the private `l0-yml-infrastructure-helm` repo:
   - Path: `{cluster-name}/{service-name}-webclient.values.yaml`
   - Must contain the ingress hostname mapped to the app domain

Once the infra admin completes this, any push to `main` triggers: build → push to ACR → deploy to AKS automatically.

---

## Verification

After the first successful pipeline run, the app should be live at:
```
https://{app-domain}
```

Check GitHub Actions → the `cd-job` workflow for build logs if anything fails.
