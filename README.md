# JarvisDoc

Interactive AI-assisted design document platform. Create beautiful, collaborative design docs with threaded comments, an AI assistant, cross-document search, and automated change requests.

## What is JarvisDoc?

JarvisDoc is an open source framework for creating interactive design document websites. Run one command, answer a few questions, and get a fully working site with:

- **Authentication gate** — Google sign-in restricted to your domain
- **Threaded comments** — @mentions, resolve/reopen, typing indicators, real-time sync via Firebase
- **AI design assistant (Jarvis)** — monitors comments, responds with contextual intelligence, files change requests
- **Automated change processor** — Claude Agent SDK implements approved changes, commits, deploys
- **Full-text search** — Cmd+K cross-document search with highlighted results and deep linking
- **Version tracking** — revision dropdown, version history, archived snapshots
- **Dark-theme design system** — cards, grids, stat boxes, badges, tables, responsive breakpoints
- **Starter templates** — Executive Summary, System Design, Implementation Plan, Business Review, Presentation

## Quick Start

```bash
npx create-jarvisdoc my-project
cd my-project
# Add your Firebase service account key
npx firebase deploy
```

## How It Works

Each JarvisDoc instance is a standalone static site deployed to its own Firebase project. A single `jarvisdoc.config.js` file controls everything — project name, auth domain, document registry, team directory, AI persona, and theme.

The framework JS files (`auth-gate.js`, `comments.js`, `jarvis-chat.js`, `search.js`) read from the config at runtime. No build step required.

### The Change Request Pipeline

1. Someone posts a comment with feedback (e.g., "The timeline section should be wider")
2. Jarvis detects actionable language, replies, and files a change request
3. The project owner approves the change request
4. The Change Processor (a full Claude Agent SDK agent) reads the request, edits the files, commits, and deploys
5. Jarvis posts a completion notice and resolves the comment thread

## Prerequisites

- [Node.js](https://nodejs.org/) 18+
- A [Firebase](https://firebase.google.com/) project (free Spark plan works)
- An [Anthropic API key](https://console.anthropic.com/) (for Jarvis AI features)

## Documentation

- [Design Spec](docs/specs/design.md) — Full product design with architecture, config schema, and implementation phases
- [Reusability Analysis](docs/specs/reusability-analysis.md) — How JarvisDoc was extracted from the Praesidium2 design system

## Origin Story

JarvisDoc was extracted from the [Praesidium2](https://praesidium2-design.web.app) interactive design document — a 35,000+ line design system built by AI agents in a single Claude Code session. The site you see there is the reference implementation: threaded comments, Jarvis AI assistant, change request pipeline, cross-document search — all running in production with real stakeholders.

We realized the framework was more valuable than the one-off. JarvisDoc makes it reusable.

## License

MIT

## Contributing

Contributions welcome. See [docs/specs/design.md](docs/specs/design.md) for the architecture and implementation plan.

---

Built by [KofTwentyTwo](https://github.com/KofTwentyTwo)
