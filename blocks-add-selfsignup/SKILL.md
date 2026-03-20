---
name: blocks-add-selfsignup
description: Enable users to register and access a Blocks app — invite users by email so they receive an activation link and set their own password. Use after the app is live and protected with Blocks auth.
argument-hint: "[app-domain]"
disable-model-invocation: true
---

You are setting up user access for a Blocks-protected app. Users cannot self-register on their own — an admin invites them via Blocks Cloud. Each invited user receives an email with an activation link and sets their own password on first login.

## What you need — tell the user this first

1. **Blocks Cloud admin access**
   The user must have an Admin role in their Blocks project.
   Where to check: [Blocks Cloud](https://cloud.seliseblocks.com) → select your project → Access Manager

2. **The app domain**
   e.g. `https://pbiwfs-abcd.seliseblocks.com`
   This is where the activation email will link to.

3. **Email addresses to invite**
   A list of emails for everyone who should have access.

---

## Step 1 — Invite Users via Blocks Cloud

For each user to invite:

1. Go to [Blocks Cloud](https://cloud.seliseblocks.com) → select your project
2. Open **Access Manager → Users**
3. Click **Invite User**
4. Enter the user's **email address**
5. Assign a **role** (at minimum: the default user role for your project)
6. Click **Send Invitation**

The user receives an email with an activation link in this format:
```
https://{app-domain}/activate?code={activation-code}&lang=en-US
```

They click the link, set a password, and can sign in immediately.

---

## Step 2 — Verify the Activation Flow Works

Before inviting everyone, test with one user:

1. Invite a test email address
2. Open the activation link from the email
3. Confirm the **Set Your Password** screen appears (powered by the auth overlay from `blocks-add-auth`)
4. Set a password and verify login works

If the activation screen does not appear:
- Check that `index.html` includes the activation overlay and `checkForActivation()` logic from `blocks-add-auth`
- Check that `nginx.conf` has `try_files $uri $uri/ /index.html` — without this, the `/activate` path returns 404

---

## Step 3 — Role and Permission Setup (optional)

If you need different access levels:

1. Go to **Access Manager → Roles**
2. Create roles (e.g. `viewer`, `editor`, `admin`)
3. Assign permissions to each role under **Roles → Edit Permissions**
4. When inviting users, assign the appropriate role

Roles are available to your app via the Blocks access token if you need them for frontend feature flags.

---

## Step 4 — Notify Users

After inviting, tell users:

> You'll receive an email from Blocks Cloud with a link to activate your account.
> Click the link, set a password, and you're in.
> The link expires after 24 hours — if it does, ask for a new invitation.

---

## How to Re-invite a User

If a user's activation link expired or they didn't receive the email:

1. Go to **Access Manager → Users**
2. Find the user (status will show `Pending`)
3. Click **Resend Invitation**

---

## Common Issues

| Issue | Cause | Fix |
|-------|-------|-----|
| Activation link goes to 404 | Missing `try_files` in nginx.conf | See `blocks-deploy-html` — add `try_files $uri $uri/ /index.html` |
| Password screen doesn't appear | Missing activation overlay in HTML | Run `blocks-add-auth` first |
| User exists but can't log in | Not yet activated | Check user status in Access Manager — resend if `Pending` |
| Invitation email not received | Spam filter or wrong email | Check spam, verify email address in Access Manager |
