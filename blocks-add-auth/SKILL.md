---
name: blocks-add-auth
description: Add Blocks login, MFA, and account activation to any single-file HTML app. Use when you want to password-protect an HTML tool using SELISE Blocks Cloud authentication.
argument-hint: "[x-blocks-key] [app-domain] [tool-name]"
disable-model-invocation: true
---

You are adding Blocks authentication to an HTML file. This protects the app with login, optional MFA, and email-based account activation. Done in about 5 minutes.

## What you need — tell the user this first

Ask the user to have these three things ready:

1. **x-blocks-key**
   Where to get it: [Blocks Cloud](https://cloud.seliseblocks.com) → select your project → **Project Settings** → copy the value labeled `x-blocks-key`

2. **App domain**
   The full URL where this app is hosted, e.g. `https://pbiwfs-abcd.seliseblocks.com`
   This is the frontend URL only — never used as the API URL.

3. **Tool name**
   A short slug used for localStorage, e.g. `my-dashboard`. No spaces, lowercase.

If the user already provided these in their message, extract them and skip asking.

---

## Step 1 — Add Constants

At the top of the `<script>` section in `index.html`:

```javascript
const BLOCKS_KEY = '{x-blocks-key}';
const BLOCKS_API = 'https://api.seliseblocks.com';  // always this — never the app domain
const AUTH_KEY   = '{tool-name}-auth';
```

---

## Step 2 — Add Overlay HTML

Paste before the closing `</body>` tag. Both screens start hidden — JS shows them:

```html
<!-- Login overlay -->
<div id="login-screen" style="position:fixed;inset:0;z-index:2000;display:none;align-items:center;justify-content:center;background:rgba(0,0,0,0.8)">
  <div style="background:#fff;padding:2rem;border-radius:8px;width:320px;font-family:sans-serif">
    <h2 style="margin-top:0">Sign In</h2>
    <div id="credentials-step">
      <input id="login-email" type="email" placeholder="Email" style="display:block;width:100%;margin-bottom:0.5rem;padding:0.5rem;box-sizing:border-box">
      <input id="login-password" type="password" placeholder="Password" style="display:block;width:100%;margin-bottom:1rem;padding:0.5rem;box-sizing:border-box">
      <button onclick="doLogin()" style="width:100%;padding:0.75rem;cursor:pointer">Sign In</button>
    </div>
    <div id="mfa-step" style="display:none">
      <p style="margin-top:0">Enter your MFA code</p>
      <input id="mfa-code" type="text" placeholder="123456" style="display:block;width:100%;margin-bottom:1rem;padding:0.5rem;box-sizing:border-box">
      <button onclick="doMfa()" style="width:100%;padding:0.75rem;cursor:pointer">Verify</button>
    </div>
    <div id="login-error" style="color:red;margin-top:0.75rem;font-size:0.9rem"></div>
  </div>
</div>

<!-- Activation overlay (shown via activation link from email) -->
<div id="activation-screen" style="position:fixed;inset:0;z-index:2000;display:none;align-items:center;justify-content:center;background:rgba(0,0,0,0.8)">
  <div style="background:#fff;padding:2rem;border-radius:8px;width:320px;font-family:sans-serif">
    <h2 style="margin-top:0">Set Your Password</h2>
    <input id="new-password" type="password" placeholder="New password" style="display:block;width:100%;margin-bottom:0.5rem;padding:0.5rem;box-sizing:border-box">
    <input id="confirm-password" type="password" placeholder="Confirm password" style="display:block;width:100%;margin-bottom:1rem;padding:0.5rem;box-sizing:border-box">
    <button onclick="doActivate()" style="width:100%;padding:0.75rem;cursor:pointer">Activate Account</button>
    <div id="activation-error" style="color:red;margin-top:0.75rem;font-size:0.9rem"></div>
  </div>
</div>
```

---

## Step 3 — Add Auth JavaScript

Add inside the `<script>` tag (after the constants from Step 1):

```javascript
let activationCode = null;

// Storage helpers
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
function showLogin() {
  document.getElementById('login-screen').style.display = 'flex';
}

// Login (step 1 of 2 — credentials)
async function doLogin() {
  const username = document.getElementById('login-email').value;
  const password = document.getElementById('login-password').value;
  document.getElementById('login-error').textContent = '';
  try {
    const res = await fetch(`${BLOCKS_API}/idp/v1/Authentication/Token`, {
      method: 'POST',
      headers: { 'Content-Type': 'application/x-www-form-urlencoded', 'x-blocks-key': BLOCKS_KEY },
      credentials: 'include',
      body: new URLSearchParams({ grant_type: 'password', username, password })
    });
    const data = await res.json();
    if (data.mfaRequired) {
      document.getElementById('credentials-step').style.display = 'none';
      document.getElementById('mfa-step').style.display = 'block';
    } else if (data.accessToken) {
      storeAuth(data);
      document.getElementById('login-screen').style.display = 'none';
      bootApp();
    } else {
      document.getElementById('login-error').textContent = data.message || 'Login failed';
    }
  } catch {
    document.getElementById('login-error').textContent = 'Network error — check your connection';
  }
}

// Login (step 2 of 2 — MFA code)
async function doMfa() {
  const username = document.getElementById('login-email').value;
  const password = document.getElementById('login-password').value;
  const mfaCode  = document.getElementById('mfa-code').value;
  const res = await fetch(`${BLOCKS_API}/idp/v1/Authentication/Token`, {
    method: 'POST',
    headers: { 'Content-Type': 'application/x-www-form-urlencoded', 'x-blocks-key': BLOCKS_KEY },
    credentials: 'include',
    body: new URLSearchParams({ grant_type: 'password', username, password, mfaCode })
  });
  const data = await res.json();
  if (data.accessToken) {
    storeAuth(data);
    document.getElementById('login-screen').style.display = 'none';
    bootApp();
  } else {
    document.getElementById('login-error').textContent = data.message || 'MFA code incorrect';
  }
}

// Activation (triggered by email link with ?code=)
function checkForActivation() {
  const code = new URLSearchParams(window.location.search).get('code');
  if (!code) return false;
  activationCode = code;
  document.getElementById('activation-screen').style.display = 'flex';
  return true;
}

async function doActivate() {
  const newPassword = document.getElementById('new-password').value;
  const confirm     = document.getElementById('confirm-password').value;
  if (newPassword !== confirm) {
    document.getElementById('activation-error').textContent = 'Passwords do not match';
    return;
  }
  const res = await fetch(`${BLOCKS_API}/idp/v1/Iam/Activate`, {
    method: 'POST',
    headers: { 'Content-Type': 'application/json', 'x-blocks-key': BLOCKS_KEY },
    credentials: 'include',
    body: JSON.stringify({
      code: activationCode,
      password: newPassword,
      captchaCode: '',
      projectKey: BLOCKS_KEY,
      preventPostEvent: true   // required — activation fails silently without this
    })
  });
  const data = await res.json();
  if (data.accessToken) {
    storeAuth(data);
    document.getElementById('activation-screen').style.display = 'none';
    window.history.replaceState({}, '', window.location.pathname);
    bootApp();
  } else {
    document.getElementById('activation-error').textContent = data.message || 'Activation failed';
  }
}

// Boot sequence — this order is critical
window.addEventListener('load', () => {
  if (checkForActivation()) return;   // activation link always takes priority
  const auth = getStoredAuth();
  if (!auth?.accessToken) { showLogin(); return; }
  bootApp();
});
```

Make sure the existing app has a `bootApp()` function (or rename the call to match whatever initializes the app).

---

## Common Mistakes

| Mistake | Symptom | Fix |
|---------|---------|-----|
| `BLOCKS_API` set to app domain | 405 on POST — nginx receives the API call | Always `https://api.seliseblocks.com` |
| Testing from `localhost` | CORS blocked on preflight | Auth only works from `*.seliseblocks.com` |
| `credentials: 'include'` missing | Auth cookies not sent | Required on every Blocks fetch call |
| `preventPostEvent: true` missing | Activation fails silently | Always include in activate body |
| Login/activation overlay `display` not `none` in HTML | Flash of login UI before JS runs | Set `style="display:none"` in the HTML |
