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

## What kind of apps are we talking about?

Everything here is a **single HTML file** — which sounds limiting but really isn't. Here are ideas to get your brain going:

### Browser games
- A word guessing game (like Wordle) for your friend group, with a leaderboard
- A trivia quiz app where each user logs in and tracks their own score
- A multiplayer rock-paper-scissors game with a live scoreboard
- A memory card matching game with difficulty levels
- A typing speed test with personal best tracking

### Dashboards and data tools
- A personal finance tracker where family members log in and track shared expenses
- A team KPI dashboard that pulls from a public API and visualizes trends
- A stock or crypto price watcher with your own watchlist
- A habit tracker with streaks, charts, and weekly summaries
- A website uptime monitor that checks your links and shows a status board

### Content and knowledge tools
- A password-protected team wiki or internal FAQ page
- A recipe collection app where your household logs in and adds meals
- A book or movie tracker — log what you've read/watched, rate it, share with friends
- A travel planning board where a group can pin destinations and vote
- A private journal or mood diary, locked behind your Blocks login

### Business and productivity tools
- A client-facing project status page with login for each client
- A simple CRM — track leads, add notes, filter by stage
- A meeting notes archive for your team, searchable and organized by date
- An employee onboarding checklist app with progress tracking
- A small inventory tracker for a physical store or warehouse

### Financial calculators and tools
- A mortgage or loan repayment calculator with amortization breakdown
- A freelance rate calculator — input hours, expenses, taxes, get your ideal day rate
- A SaaS pricing model simulator to test revenue projections
- A currency converter with historical comparison charts
- A retirement savings calculator with scenario modeling

### Fun and community tools
- A wishlist app you share with family before the holidays
- A bracket tournament app for your office fantasy league
- A secret Santa organizer — participants log in and see their assignment
- A poll or voting tool for teams that want something lighter than Slack polls
- A shared playlist curator where friends submit songs and vote on them

The pattern: **one HTML file + Blocks auth + a deployed domain = a real working app with real user accounts.** Your AI agent builds the whole thing. You just describe what you want.

---

## This repo is for fast, fun building — here's the honest caveat

Single-file HTML apps are great for getting things shipped quickly. They are **not** the right choice for apps that handle sensitive personal data, financial transactions, or anything that needs serious security auditing. The auth here is real, but the architecture is intentionally simple.

**Once you've built a few things and caught the bug**, you'll naturally want more structure — proper component architecture, better state management, mobile apps, and so on. That's the natural progression.

When you're ready to level up, SELISE Blocks has production-ready templates that use the same authentication infrastructure but with proper frameworks:

- **React** → [blocks-construct-react](https://github.com/SELISEdigitalplatforms/blocks-construct-react) — a full React starter with Blocks auth pre-wired, routing, and component structure
- **Angular** — a full Angular template with the same Blocks integration
- **Flutter** — build iOS and Android apps backed by Blocks auth
- **NativeScript** — cross-platform native mobile with JavaScript

These give you everything this repo gives you, plus proper security patterns, testable components, and the structure needed for production-grade software.

**But don't start there.** Start here. Build something small. Ship it. Show someone. Then level up when you're ready. The best way to learn is to have something real to show for it.

---

## Skills in this repo

| Skill | What it does |
|-------|-------------|
| [`blocks-get-started`](./blocks-get-started/SKILL.md) | Set up GitHub, Git, and a Blocks project from zero — perfect if this is your first time |
| [`blocks-add-auth`](./blocks-add-auth/SKILL.md) | Add Blocks login, MFA, account activation, and admin user invitations to any HTML file |
| [`blocks-deploy-html`](./blocks-deploy-html/SKILL.md) | Deploy an HTML app to Blocks Cloud using Docker and GitHub Actions |
| [`blocks-add-selfsignup`](./blocks-add-selfsignup/SKILL.md) | Enable self-registration with Google reCAPTCHA so users can sign up themselves |

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
4. **[blocks-add-selfsignup](./blocks-add-selfsignup/SKILL.md)** — Enable self-registration so users can sign up themselves

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
