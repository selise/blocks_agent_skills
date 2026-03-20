---
name: blocks-get-started
description: Guide an absolute beginner through creating a GitHub account, setting up Git in the terminal, creating a SELISE Blocks Cloud project, and connecting everything together. Use when someone is starting from zero and wants to deploy their first app with an AI coding agent.
argument-hint: ""
disable-model-invocation: true
---

You are guiding a complete beginner through the full setup — from zero to having a GitHub account, a working terminal, and a live Blocks Cloud project. Go step by step. Be encouraging. Explain why each thing matters, not just what to do.

---

## What this setup gives you

Once done, you will have:
- A **GitHub account** — where your code lives in the cloud, versioned and backed up
- **Git** in your terminal — so you can push code from your computer to GitHub with one command
- A **Blocks Cloud project** — a backend with authentication, user management, and a *.seliseblocks.com domain, all set up for you
- The ability to tell an AI coding agent like Claude: *"deploy my app to Blocks"* — and it actually works

---

## Part 1 — GitHub Account

### Step 1.1 — Create your account

1. Go to [github.com](https://github.com)
2. Click **Sign up**
3. Enter your email, create a password, and pick a username
   - Your username becomes part of your URLs, e.g. `github.com/yourname` — choose something you'd be happy to share
4. Verify your email address when GitHub sends you a confirmation

That's it. You now have a GitHub account.

### Step 1.2 — What GitHub is for

Think of GitHub like Google Drive but for code. Every project lives in a **repository** (repo for short). Instead of just saving files, you make **commits** — snapshots of your code with a message explaining what changed. This means you can always go back to any previous version, and your AI coding agent can push code directly to GitHub on your behalf.

---

## Part 2 — Git in Your Terminal

### Step 2.1 — Open your terminal

- **Mac**: Press `Cmd + Space`, type `Terminal`, press Enter
- **Windows**: Press `Win + R`, type `cmd`, press Enter (or install [Windows Terminal](https://apps.microsoft.com/detail/9n0dx20hk701) for a better experience)

### Step 2.2 — Check if Git is already installed

Type this and press Enter:
```bash
git --version
```

If you see something like `git version 2.x.x` — you already have Git. Skip to Step 2.4.

If you see `command not found` — continue to Step 2.3.

### Step 2.3 — Install Git

**Mac:**
```bash
xcode-select --install
```
A dialog will pop up — click Install. Wait for it to finish (takes a few minutes).

**Windows:**
Download and install from [git-scm.com/download/win](https://git-scm.com/download/win). Use all default settings during installation.

After installing, close your terminal and reopen it, then run `git --version` again to confirm.

### Step 2.4 — Tell Git who you are

Git needs your name and email so that every change you make is labeled with your identity. These should match your GitHub account:

```bash
git config --global user.name "Your Name"
git config --global user.email "your@email.com"
```

Replace `Your Name` and `your@email.com` with the name and email you used for GitHub.

### Step 2.5 — Connect your terminal to GitHub (authentication)

The easiest way is to install the **GitHub CLI** — a tool that lets you log in to GitHub directly from the terminal.

**Mac:**
```bash
brew install gh
```

If `brew` is not found, first install Homebrew:
```bash
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
```

**Windows:**
Download the installer from [cli.github.com](https://cli.github.com) and run it.

After installing, log in:
```bash
gh auth login
```

Follow the prompts:
- Select **GitHub.com**
- Select **HTTPS**
- Type `Y` to authenticate with your GitHub credentials
- Select **Login with a web browser** and follow the steps

When it says `Logged in as yourname` — you are connected. Your terminal can now create repos, push code, and more on your behalf.

---

## Part 3 — SELISE Blocks Cloud Project

### Step 3.1 — Create a Blocks account

1. Go to [cloud.seliseblocks.com](https://cloud.seliseblocks.com)
2. Click **Sign Up** and create an account
3. Verify your email

### Step 3.2 — Create a new project

1. After logging in, click **Create Project**
2. Give it a name — this will become part of your app's URL, e.g. `my-app` → `https://my-app-xxxx.seliseblocks.com`
3. Select a region closest to your users
4. Click **Create**

Blocks will set up your project — this takes about a minute.

### Step 3.3 — Get your project key

Your project key (`x-blocks-key`) is how your app identifies itself to Blocks Cloud.

1. Inside your project, click **Project Settings**
2. Copy the value labeled **x-blocks-key**

Save this somewhere safe — you will need it when your AI agent sets up authentication for your app.

### Step 3.4 — What Blocks gives you (for free)

Out of the box, every Blocks project includes:
- **User authentication** — login, password, MFA
- **User management** — invite users, manage roles and permissions
- **A live domain** — `https://your-project-xxxx.seliseblocks.com`
- **API infrastructure** — your app can call `api.seliseblocks.com` to handle login and user data

You do not need to build any of this yourself. Your AI coding agent does it for you using these skills.

---

## Part 4 — Connect Blocks to GitHub

This step needs a **SELISE infra admin** — someone at your organization who manages the Azure and GitHub infrastructure. If you are working alone or at a company without a dedicated infra person, ask the Blocks Cloud support team.

Tell your infra admin you need:
1. A new GitHub repository created in your org (or create one yourself at [github.com/new](https://github.com/new))
2. The standard org secrets granted to your new repo (they will know what this means — it's the five Azure + GitHub secrets used in Blocks deployments)
3. A Helm values file added to the infrastructure repo for your app domain

Once they confirm this is done — you are ready to deploy.

---

## Part 5 — What to do next

You now have everything in place. Here is what your AI coding agent can do for you:

| I want to... | Use this skill |
|---|---|
| Add login, MFA, and invite users by email | `blocks-add-auth` |
| Deploy my HTML app to my Blocks domain | `blocks-deploy-html` |
| Let users sign themselves up (with reCAPTCHA) | `blocks-add-selfsignup` |

To use a skill, just tell your AI coding agent:

> "Use the `blocks-add-auth` skill to add login to my index.html"

The agent will walk you through the rest.

---

## You are ready

You have:
- A GitHub account and Git connected in your terminal
- A Blocks Cloud project with your `x-blocks-key`
- An understanding of how these pieces connect

From here, you and your AI coding agent can build and ship real apps — without needing a team of engineers, a DevOps setup, or years of experience.
