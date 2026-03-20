# Blocks Agent Skills — Deploy Real Apps with AI Coding Agents

**You found this page. That's already the first step.**

Hey — welcome. Whether you got here from a YouTube video, a Reddit thread, or you just asked ChatGPT "how do I vibe code my own app" — you're in the right place.

This is a collection of **skills for AI coding agents** (like Claude Code) that let you deploy real, working web apps with authentication, user management, and a live domain — without needing to know how to code from scratch.

No CS degree. No years of experience. Just you, an AI agent, and a clear set of steps.

---

## What is vibe coding?

Vibe coding means telling an AI what you want to build, and the AI writes the code, sets up the infrastructure, and deploys it — while you stay in control of the decisions. You describe the outcome. The agent figures out the how.

It's not magic. It's a new skill. And it's much easier to learn than traditional programming — because you're working *with* the AI as a partner, not learning syntax by yourself.

---

## What is an agentic engineer?

An agentic engineer is someone who uses AI agents as their primary tool to build software. Instead of writing every line of code manually, they:

- Give the agent clear instructions
- Review and guide what the agent produces
- Make decisions about what to build and why
- Ship things — fast

You don't need to be a programmer to become an agentic engineer. You need curiosity, clear thinking, and the right tools. This repo is one of those tools.

---

## What you can build with this

Using these skills, your AI coding agent can:

- Take any HTML file and wrap it in a password-protected web app
- Deploy it to a real domain (`*.seliseblocks.com`) on production infrastructure
- Set up login, MFA, and account activation for your users
- Invite users by email so they can sign up and get access

All of this runs on [SELISE Blocks Cloud](https://cloud.seliseblocks.com) — a platform that handles authentication, user management, and deployment infrastructure so you don't have to build it yourself.

---

## Skills in this repo

| Skill | What it does |
|-------|-------------|
| [`blocks-get-started`](./blocks-get-started/SKILL.md) | Set up GitHub, Git, and a Blocks project from zero — perfect if this is your first time |
| [`blocks-add-auth`](./blocks-add-auth/SKILL.md) | Add Blocks login, MFA, and account activation to any HTML file |
| [`blocks-deploy-html`](./blocks-deploy-html/SKILL.md) | Deploy an HTML app to Blocks Cloud using Docker and GitHub Actions |
| [`blocks-add-selfsignup`](./blocks-add-selfsignup/SKILL.md) | Invite users by email so they can activate their accounts and sign in |

---

## How to use these skills with Claude Code

### 1. Install Claude Code

Claude Code is Anthropic's AI coding agent that runs in your terminal. Install it with:

```bash
npm install -g @anthropic-ai/claude-code
```

You need [Node.js](https://nodejs.org) installed first. If you don't have it, download it from nodejs.org — it takes 2 minutes.

### 2. Add these skills to Claude Code

Clone this repo into your Claude Code skills directory:

```bash
git clone https://github.com/julianweber/blocks-agent-skills ~/.claude/skills/blocks-agent-skills
```

Or add it via the Claude Code settings. Skills in `~/.claude/skills/` are automatically available in every project.

### 3. Tell the agent to use a skill

Open your project folder in the terminal, start Claude Code, and just say:

> "Use the blocks-get-started skill to help me set everything up"

or

> "Add Blocks auth to my index.html using the blocks-add-auth skill"

The agent reads the skill, asks you for the things it needs, and does the work.

---

## Start here if you are brand new

Run through these in order:

1. **[blocks-get-started](./blocks-get-started/SKILL.md)** — GitHub account, Git in your terminal, Blocks Cloud project
2. **[blocks-add-auth](./blocks-add-auth/SKILL.md)** — Add login to your HTML app
3. **[blocks-deploy-html](./blocks-deploy-html/SKILL.md)** — Put it live on the internet
4. **[blocks-add-selfsignup](./blocks-add-selfsignup/SKILL.md)** — Invite your first users

Each skill is a plain text file. Your AI agent reads it and knows exactly what to do. You just answer its questions.

---

## Frequently asked questions

**Do I need to know how to code?**
No. These skills are designed for people who want to build real things without needing to understand every line of code. The AI handles the code. You handle the decisions.

**What is SELISE Blocks?**
SELISE Blocks is a cloud platform that gives your app authentication, user management, and a live domain out of the box. It's what runs behind the scenes when your app goes live.

**What is an x-blocks-key?**
It's a short identifier that links your app to your Blocks Cloud project. You get it from Blocks Cloud → Project Settings. Your AI agent will ask for it when it needs it.

**Can I use this for commercial projects?**
Yes. The skills in this repo are MIT licensed. SELISE Blocks Cloud has its own terms of service.

**What if I get stuck?**
Ask your AI agent. Paste the error message and say "what's wrong?" — it will diagnose and fix it. If you're still stuck, open an issue in this repo.

---

## Why this exists

Setting up authentication, deployment pipelines, and user management used to require a backend engineer, a DevOps person, and weeks of work.

Now it takes an afternoon — if you have the right skills plugged into your AI agent.

This repo exists to make that accessible to anyone. The barrier to building real software is lower than it has ever been. These skills are a starting point.

---

## Contributing

Found a bug? Have a skill idea? Pull requests are welcome.

If you're new to contributing to GitHub repos — that's fine. Ask your AI agent: "How do I open a pull request?" It will walk you through it.

---

## License

MIT — use it, fork it, build on it.

---

*Built for vibe coders, agentic engineers, and everyone who decided to stop waiting and start building.*
