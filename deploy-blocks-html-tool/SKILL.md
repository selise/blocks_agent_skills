---
name: deploy-blocks-html-tool
description: Deploy a password-protected single-file HTML tool to SELISE Blocks Cloud using Docker and GitHub Actions. Use when the user wants to add Blocks authentication (login, MFA, account activation) to an HTML tool and deploy it to a *.seliseblocks.com domain via Azure AKS.
argument-hint: "[x-blocks-key] [app-domain]"
disable-model-invocation: true
---

You are helping the user deploy a password-protected HTML tool on SELISE Blocks Cloud. Follow these steps in order. At each step, confirm with the user before proceeding to the next.

## What you will build

- A single `index.html` with Blocks authentication (login + MFA + account activation)
- A Docker + nginx container with SPA routing
- A GitHub Actions pipeline deploying to Azure AKS via the standard Blocks CI/CD pattern
- Live at `https://{project}-{suffix}.seliseblocks.com`

---

## Step 1 — Collect Required Values

Ask the user for:
1. Their **x-blocks-key** (from Blocks Cloud → Project Settings)
2. Their **app domain** (e.g. `https://pbiwfs-abcd.seliseblocks.com`)
3. Their **GitHub org and repo name** (e.g. `selise/my-tool`)
4. A short **tool name** for the localStorage key (e.g. `my-tool`)

If the user has already provided these in their message, extract them and proceed.

**Critical constant** — always hardcode this regardless of what the user provides:
```
BLOCKS_API = 'https://api.seliseblocks.com'
```
Never use the app domain as the API URL. The app domain is frontend-only.

---

## Step 2 — Add Auth to index.html

Add the following to the user's `index.html`. All screens must be `position:fixed; inset:0; z-index:2000; display:none` in HTML and shown via JS only.

### Constants
```javascript
const BLOCKS_KEY = '{x-blocks-key}';
const BLOCKS_API = 'https://api.seliseblocks.com';
const AUTH_KEY   = '{tool-name}-auth';
```

### Login screen
Two-step overlay: credentials step + MFA step (hidden by default, shown when `mfaRequired` is returned).

### Login API call
```javascript
const res = await fetch(`${BLOCKS_API}/idp/v1/Authentication/Token`, {
  method: 'POST',
  headers: {
    'Content-Type': 'application/x-www-form-urlencoded',
    'x-blocks-key': BLOCKS_KEY
  },
  credentials: 'include',
  body: new URLSearchParams({ grant_type: 'password', username, password })
});
const data = await res.json();
if (data.mfaRequired) { /* show MFA step */ }
else if (data.accessToken) { storeAuth(data); bootApp(); }
```

### MFA API call — same endpoint, add `mfaCode` field to body

### Activation screen
Shown when URL contains `?code=`. Must also handle the `/activate` path (nginx routes it to `index.html`).
```javascript
function checkForActivation() {
  const code = new URLSearchParams(window.location.search).get('code');
  if (!code) return false;
  activationCode = code;
  document.getElementById('activation-screen').style.display = 'flex';
  return true;
}
```

### Activation API call
```javascript
await fetch(`${BLOCKS_API}/idp/v1/Iam/Activate`, {
  method: 'POST',
  headers: { 'Content-Type': 'application/json', 'x-blocks-key': BLOCKS_KEY },
  credentials: 'include',
  body: JSON.stringify({
    code: activationCode,
    password: newPassword,
    captchaCode: '',
    projectKey: BLOCKS_KEY,
    preventPostEvent: true   // required — do not omit
  })
});
```

### Auth storage
```javascript
function storeAuth(data) {
  localStorage.setItem(AUTH_KEY, JSON.stringify({
    accessToken: data.accessToken,
    expiresAt: Date.now() + (data.expiresIn || 3600) * 1000
  }));
}
function getStoredAuth() {
  try {
    const d = JSON.parse(localStorage.getItem(AUTH_KEY));
    if (d?.expiresAt > Date.now()) return d;
  } catch {}
  return null;
}
```

### Boot sequence — order matters
```javascript
window.addEventListener('load', () => {
  if (checkForActivation()) return;  // activation link takes priority
  const auth = getStoredAuth();
  if (!auth?.accessToken) { showLogin(); return; }
  bootApp();
});
```

---

## Step 3 — Create Docker Files

### `Dockerfile`
```dockerfile
FROM nginx:alpine
COPY index.html /usr/share/nginx/html/index.html
COPY nginx.conf /etc/nginx/conf.d/default.conf
EXPOSE 80
```

### `nginx.conf`
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

`try_files` is required — without it, `/activate?code=...` returns 404 and the hosting layer strips the path to `/?code=...`, breaking activation links.

---

## Step 4 — Create GitHub Actions Pipeline

### `.github/variables/vars.env`
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
Copy verbatim from `SELISEdigitalplatforms/l3-react-blocks-construct/.github/workflows/3_web.yml`.

---

## Step 5 — Push to GitHub

```bash
git init
git branch -m main
git remote add origin https://github.com/{org}/{repo-name}.git
git add .gitignore Dockerfile nginx.conf index.html .github/
git commit -m "feat: add Blocks auth and Docker deployment"
git push -u origin main
```

Ensure `.gitignore` excludes any local secret files (`.env`, etc.) before the first push.

---

## Step 6 — Infra Team Actions (inform the user)

Tell the user they need a SELISE infra admin to:

1. **Grant org secrets** to the new repo in GitHub → Org Settings → Secrets:
   - `SELISE_GITHUB_PAT`
   - `AZURE_AKS_BLOCKS_PROD_CREDENTIALS`
   - `AZURE_BLOCKS_PROD_CONTAINER_REGISTRY`
   - `ClUSTER_AKS_BLOCKS_PROD_RESOURCE_GROUP` ← lowercase `l` is intentional
   - `AKS_BLOCKS_PROD_CLUSTER`

2. **Add Helm values file** to the private `l0-yml-infrastructure-helm` repo:
   - Path: `{cluster-name}/{service-name}-webclient.values.yaml`
   - Must contain the ingress hostname mapping to the app domain

Once done, any push to `main` triggers a full build → ACR push → AKS deploy automatically.

---

## Step 7 — Invite Users (inform the user)

After the tool is live:

1. Open **Blocks Cloud → Access Manager → Users → Invite User**
2. Enter the user's email
3. They receive an activation email linking to:
   ```
   https://{app-domain}/activate?code={code}&lang=en-US
   ```
4. They set their password and can sign in immediately

---

## Common Mistakes — Check for These

| Mistake | Symptom | Fix |
|---------|---------|-----|
| `BLOCKS_API` set to app domain | 405 on POST — nginx receives the API call | Always `https://api.seliseblocks.com` |
| Missing `try_files` in nginx.conf | `/activate?code=...` → 404, path stripped | Add `try_files $uri $uri/ /index.html` |
| Testing auth from `localhost` | CORS blocked on preflight | Auth only works from `*.seliseblocks.com` |
| `credentials: 'include'` missing | Auth cookies not sent | Required for Blocks deployments |
| `preventPostEvent: true` missing | Activation fails silently | Always include in activate body |
| `activation-screen` visible on load | Flash of UI before JS | Set `style="display:none"` in HTML |
