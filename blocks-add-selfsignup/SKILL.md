---
name: blocks-add-selfsignup
description: Add self-registration to a Blocks HTML app so users can sign themselves up instead of waiting for an admin invitation. Covers enabling self-signup in Blocks Cloud, configuring Google reCAPTCHA v2, and adding the signup UI to an existing HTML tool. Use after blocks-add-auth is already in place.
argument-hint: "[x-blocks-key] [app-domain] [recaptcha-site-key]"
disable-model-invocation: true
---

You are adding self-registration to a Blocks HTML app. Users will be able to sign up with their own email, solve a reCAPTCHA challenge, and receive an activation link — without needing an admin to invite them first.

This skill requires `blocks-add-auth` to already be in place. The activation flow (`/activate?code=`) must exist before self-registration is useful.

## What you need — tell the user this first

Three things to gather before starting:

1. **x-blocks-key**
   Where to get it: [Blocks Cloud](https://cloud.seliseblocks.com) → select your project → **Project Settings** → copy the value labeled `x-blocks-key`

2. **App domain** (without `https://`)
   e.g. `myapp-abc.seliseblocks.com`
   You will need to register this exact value in the Google reCAPTCHA console.

3. **Google account**
   Needed to create a reCAPTCHA key at [google.com/recaptcha/admin](https://www.google.com/recaptcha/admin)

---

## Step 1 — Enable Self-Registration in Blocks Cloud

The user must do this manually in the browser before anything else — without it the API returns `403` on every signup attempt.

1. Open [Blocks Cloud](https://cloud.seliseblocks.com) → select your project
2. Go to **Access Manager → Settings**
3. Find the **Self Registration** or **Allow Self Signup** toggle and enable it
4. Save

---

## Step 2 — Create a Google reCAPTCHA v2 Key

The user must do this manually:

1. Go to [google.com/recaptcha/admin](https://www.google.com/recaptcha/admin) and sign in
2. Click **Create** (or the **+** button)
3. Fill in:
   - **Label**: any name you'll recognize (e.g. `my-app signup`)
   - **reCAPTCHA type**: **Challenge (v2) → "I'm not a robot" Checkbox**
     — This is the only type Blocks supports. v3 and Enterprise keys fail with "Invalid key type".
   - **Domains**: the app domain without `https://` (e.g. `myapp-abc.seliseblocks.com`)
4. Accept Terms and click **Submit**
5. Copy both the **Site Key** and **Secret Key** — you need both

---

## Step 3 — Configure Captcha in Blocks Cloud

The user must do this manually:

1. Open [Blocks Cloud](https://cloud.seliseblocks.com) → select your project
2. Go to **Project Settings → Captcha** (or **Authentication → Captcha Config**)
3. Create or edit the captcha configuration for the **Signup** action
4. Set:
   - **Provider**: Google reCAPTCHA
   - **Site Key**: the site key from Step 2
   - **Secret Key**: the secret key from Step 2
5. Save and confirm the config is **enabled**

The backend uses the secret key to verify tokens with Google. A mismatch between the site key used in the frontend and the secret key saved here causes `400 {"CaptchaCode": "Captcha doesn't match"}`.

---

## Step 4 — Add Signup UI to index.html

Ask the user for the **reCAPTCHA site key** from Step 2, if not already provided.

### 4a — Add the reCAPTCHA script

Add before any `<script>` blocks, just inside `</body>`:

```html
<script src="https://www.google.com/recaptcha/api.js" async defer></script>
```

### 4b — Add tab buttons to the login overlay

Inside the existing login overlay (from `blocks-add-auth`), add tab buttons above the `credentials-step` div:

```html
<div style="display:flex;gap:0.5rem;margin-bottom:1rem">
  <button onclick="showLoginTab()" id="tab-login" style="flex:1;padding:0.5rem;cursor:pointer;font-weight:bold">Sign In</button>
  <button onclick="showRegisterTab()" id="tab-register" style="flex:1;padding:0.5rem;cursor:pointer">Sign Up</button>
</div>
```

### 4c — Add the signup form

Add inside the login overlay, after the `mfa-step` div:

```html
<!-- Signup form — hidden by default -->
<form id="register-form" style="display:none" onsubmit="handleRegister(event)">
  <p style="margin-top:0;font-size:0.9rem">Enter your email and we'll send you a link to activate your account.</p>
  <input type="email" id="register-email" placeholder="your@email.com" required
    style="display:block;width:100%;margin-bottom:1rem;padding:0.5rem;box-sizing:border-box">
  <div class="g-recaptcha" data-sitekey="{recaptcha-site-key}" style="margin-bottom:1rem"></div>
  <div id="register-error" style="color:red;font-size:0.9rem;margin-bottom:0.5rem"></div>
  <div id="register-success" style="color:green;font-size:0.9rem;margin-bottom:0.5rem"></div>
  <button type="submit" id="register-submit" style="width:100%;padding:0.75rem;cursor:pointer">Send Activation Link</button>
</form>
```

### 4d — Add the Identifier API constant

Alongside the existing `BLOCKS_API` constant:

```javascript
const IDENTIFIER_API = 'https://api.seliseblocks.com/identifier/v1';
```

### 4e — Add tab-switching helpers

```javascript
function showLoginTab() {
  document.getElementById('credentials-step').style.display = 'block';
  document.getElementById('mfa-step').style.display = 'none';
  document.getElementById('register-form').style.display = 'none';
  document.getElementById('tab-login').style.fontWeight = 'bold';
  document.getElementById('tab-register').style.fontWeight = 'normal';
}

function showRegisterTab() {
  document.getElementById('credentials-step').style.display = 'none';
  document.getElementById('mfa-step').style.display = 'none';
  document.getElementById('register-form').style.display = 'block';
  document.getElementById('tab-login').style.fontWeight = 'normal';
  document.getElementById('tab-register').style.fontWeight = 'bold';
}
```

### 4f — Add the signup handler

```javascript
async function handleRegister(event) {
  event.preventDefault();
  const email     = document.getElementById('register-email').value.trim().toLowerCase();
  const errorEl   = document.getElementById('register-error');
  const successEl = document.getElementById('register-success');
  const submitBtn = document.getElementById('register-submit');

  errorEl.textContent   = '';
  successEl.textContent = '';

  const captchaToken = grecaptcha.getResponse();
  if (!captchaToken) {
    errorEl.textContent = 'Please complete the captcha.';
    return;
  }

  submitBtn.disabled    = true;
  submitBtn.textContent = 'Sending...';

  try {
    const res = await fetch(`${IDENTIFIER_API}/People/Signup`, {
      method: 'POST',
      headers: {
        'Content-Type': 'application/json',
        'x-blocks-key': BLOCKS_KEY
      },
      credentials: 'include',
      body: JSON.stringify({
        email:       email,
        captchaCode: captchaToken,
        projectKey:  BLOCKS_KEY   // required — 500 NullReferenceException if omitted
      })
    });

    if (res.ok) {
      successEl.textContent = 'Check your email for an activation link.';
      document.getElementById('register-form').reset();
      grecaptcha.reset();
    } else {
      const text = await res.text().catch(() => '');
      let message = 'Signup failed. Please try again.';
      try { const j = JSON.parse(text); message = j.message || j.title || message; } catch {}
      errorEl.textContent = message;
      grecaptcha.reset();
    }
  } catch {
    errorEl.textContent = 'Connection error. Please try again.';
    grecaptcha.reset();
  } finally {
    submitBtn.disabled    = false;
    submitBtn.textContent = 'Send Activation Link';
  }
}
```

---

## Step 5 — Test the Full Flow

Walk through a complete test before going live:

1. Open the app and click **Sign Up**
2. Enter a valid email and solve the reCAPTCHA
3. Click **Send Activation Link** — expect the success message
4. Check the inbox for an activation email from Blocks
5. Click the link → lands on `https://{app-domain}/activate?code=...`
6. Set a password → account activated, user can log in immediately

---

## Common Mistakes

| Mistake | Symptom | Fix |
|---------|---------|-----|
| Self-registration not enabled in Blocks Cloud | `403` on `People/Signup` | Enable in Access Manager → Settings |
| reCAPTCHA key type is v3 or Enterprise | "Invalid key type" shown in widget | Recreate key as **v2 Checkbox** only |
| App domain not listed in reCAPTCHA console | `400 {"CaptchaCode": "Captcha doesn't match"}` | Add domain in Google reCAPTCHA admin |
| Site key / secret key from different reCAPTCHA entries | `400 {"CaptchaCode": "Captcha doesn't match"}` | Both keys must come from the same entry |
| `projectKey` missing from request body | `500 NullReferenceException` | Always include `projectKey: BLOCKS_KEY` |
| `credentials: 'include'` missing | CORS / cookie errors | Required on all Blocks fetch calls |
| Activation flow not yet in place | User receives email but cannot activate | Run `blocks-add-auth` first |
