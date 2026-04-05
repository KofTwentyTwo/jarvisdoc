# JarvisDoc — Open Source Interactive Design Document Platform

**Date**: 2026-04-04
**Status**: Design Spec
**Author**: James Maes, CTO — DMD Brands
**Org**: KofTwentyTwo (github.com/KofTwentyTwo)
**License**: MIT
**Package**: `npx create-jarvisdoc`

---

## Executive Summary

The Praesidium2 interactive design document site (praesidium2-design.web.app) was built as a one-off — a single-tenant, single-project application with 15+ hardcoded values across 10+ files. It works. The comment system, real-time AI assistant, search, authentication gate, change request pipeline, and dark-theme design system are all production-tested with real stakeholders.

JarvisDoc extracts this into a reusable open source product. Any engineering team can run `npx create-jarvisdoc my-project` and get a fully functional, AI-assisted interactive design document site deployed to their own Firebase project in under 30 minutes.

This is **Approach 1** from the [reusability analysis](reusability-analysis.md) — config-driven static site with a CLI generator — renamed from "Jarvis Page" to "JarvisDoc."

### What You Get

- **Authentication gate** — Google sign-in restricted to your domain
- **Threaded comments** — @mentions, resolve/reopen, typing indicators, real-time sync
- **AI design assistant** — Jarvis monitors comments, responds with contextual intelligence, files change requests
- **Automated change processor** — Claude Agent SDK implements approved changes, commits, deploys
- **Full-text search** — Cmd+K cross-document search with highlighted results and deep linking
- **Version tracking** — revision dropdown, version history, archived snapshots
- **Dark-theme design system** — cards, grids, stat boxes, badges, tables, responsive breakpoints
- **Starter templates** — Executive Summary, System Design, Implementation Plan, Business Review, Presentation

---

## 1. CLI Generator (`create-jarvisdoc`)

### 1.1 Package Identity

```
@koftwentytwo/create-jarvisdoc
```

Invoked via:
```bash
npx create-jarvisdoc my-project
# or
npm create jarvisdoc my-project
```

### 1.2 Interactive Prompts

The generator walks the user through project setup. Every prompt has a sensible default.

| Prompt | Default | Required | Notes |
|--------|---------|----------|-------|
| Project name | directory name | Yes | Used for branding, localStorage namespace, collection prefixes |
| Project slug | kebab-case of name | Yes | Firebase project ID, CSS class prefix |
| Firebase project ID | project slug | Yes | Must match an existing Firebase project |
| Firebase API key | — | Yes | From Firebase Console > Project Settings |
| Firebase auth domain | `{projectId}.firebaseapp.com` | Yes | |
| Firebase storage bucket | `{projectId}.firebasestorage.app` | Yes | |
| Firebase messaging sender ID | — | Yes | |
| Firebase app ID | — | Yes | |
| Allowed auth domain | — | Yes | e.g., `dmdbrands.com` |
| Owner email | — | Yes | The approver for change requests |
| Owner name | parsed from email | No | Display name in mention directory |
| AI assistant name | `Jarvis` | No | Name of the AI persona |
| AI assistant email | `{slug}@{assistantName}.ai` | No | |
| LLM provider | `anthropic` | No | v1: Anthropic only |
| LLM model (chat) | `claude-sonnet-4-6` | No | For comment responses |
| LLM model (agent) | `claude-sonnet-4-6` | No | For change processor |
| Starter templates | all | No | Multi-select: exec-summary, system-design, implementation, business-review, presentation |

### 1.3 Generated Output

After prompts, the generator scaffolds the full project:

```
my-project/
  jarvisdoc.config.js        # Single source of truth
  index.html                    # Executive Summary (starter)
  system-design.html            # System Design (starter)
  implementation.html           # Implementation Plan (starter)
  business-review.html          # Business Review (starter)
  presentation.html             # Presentation (starter)
  favicon.svg                   # Generated from project initial + accent color
  versions.json                 # Initial version entry
  docs-manifest.json            # Document registry
  firebase.json                 # Hosting + Firestore config
  .firebaserc                   # Project binding
  firestore.rules               # Parameterized with allowed domain + owner email
  firestore.indexes.json        # Required composite indexes
  js/
    auth-gate.js                # Authentication gate (reads window.JD)
    comments.js                 # Comment system (reads window.JD)
    jarvis-chat.js              # Chat assistant (reads window.JD)
    search.js                   # Full-text search (reads window.JD)
    version-loader.js           # Version dropdown (reads window.JD)
    built-with.js               # "Built with JarvisDoc" attribution
    config-loader.js            # Loads jarvisdoc.config.js → window.JD
  css/
    jarvisdoc.css             # Complete design system stylesheet
  jarvis/
    jarvis.js                   # Comment monitor + chat responder
    change-processor.js         # Claude Agent SDK auto-implementation
    jarvisdoc.config.json     # Node.js config (generated from .js config)
    PERSONA.md                  # AI persona definition (templated)
    package.json                # Dependencies: firebase-admin, @anthropic-ai/sdk
    .env.example                # ANTHROPIC_API_KEY=sk-...
  scripts/
    deploy.sh                   # firebase deploy --only hosting,firestore
    start-jarvis.sh             # Starts jarvis.js + change-processor.js
  .gitignore                    # service-account-key.json, node_modules, .env
```

### 1.4 Post-Generation Instructions

The CLI prints clear next steps:

```
Your JarvisDoc project is ready.

  cd my-project

1. Add your Firebase service account key:
   cp ~/Downloads/service-account-key.json jarvis/

2. Deploy Firestore rules + hosting:
   firebase deploy

3. Start the AI assistant:
   cd jarvis && npm install
   ANTHROPIC_API_KEY=sk-... npm start

4. Open your site:
   https://my-project.web.app
```

---

## 2. Configuration Schema

### 2.1 `jarvisdoc.config.js`

The single source of truth for the entire instance. This file is loaded by `config-loader.js` in the browser and by the backend services in Node.js.

```javascript
// jarvisdoc.config.js
window.JD = {
  // ── Project Identity ──────────────────────────────────────────
  project: {
    name: "Praesidium2",                         // Display name
    slug: "praesidium2-design",                  // Used for localStorage keys, collection prefixes
    subtitle: "DMD Brands — Confidential Design Document",
    logoLetter: "P",                             // Single character for the logo icon
    version: "v1.7",                             // Current version label
  },

  // ── Firebase ──────────────────────────────────────────────────
  firebase: {
    apiKey: "YOUR_FIREBASE_API_KEY",
    authDomain: "your-project.firebaseapp.com",
    projectId: "your-project",
    storageBucket: "your-project.firebasestorage.app",
    messagingSenderId: "000000000000",
    appId: "1:000000000000:web:abcdef1234567890",
  },

  // ── Authorization ─────────────────────────────────────────────
  auth: {
    allowedDomain: "yourcompany.com",            // Google Workspace domain
    ownerEmail: "owner@yourcompany.com",         // Change request approver
  },

  // ── Documents ─────────────────────────────────────────────────
  docs: [
    { id: "index",           file: "index.html",           name: "Executive Summary", color: "#f59e0b", shortName: "EXEC SUMMARY" },
    { id: "system-design",   file: "system-design.html",   name: "System Design",     color: "#60a5fa", shortName: "SYSTEM DESIGN" },
    { id: "implementation",  file: "implementation.html",  name: "Implementation",    color: "#22d3ee", shortName: "IMPLEMENTATION" },
    { id: "business-review", file: "business-review.html", name: "Business Review",   color: "#f87171", shortName: "BUSINESS REVIEW" },
  ],

  // ── Team Directory ────────────────────────────────────────────
  team: [
    { handle: "jmaes",      name: "James Maes",      title: "CTO",                    type: "user" },
    { handle: "bchupp",     name: "Bryan Chupp",     title: "CMO",                    type: "user" },
    { handle: "cchupp",     name: "Chris Chupp",     title: "CEO",                    type: "user" },
    { handle: "bpotter",    name: "Bryan Potter",    title: "VP Me Health",           type: "user" },
    { handle: "mcarpenter", name: "Matt Carpenter",  title: "VP Product Dev",         type: "user" },
    { handle: "jarvis",     name: "Jarvis",          title: "Design review assistant", type: "agent" },
    { handle: "friday",     name: "F.R.I.D.A.Y.",   title: "Code review agent",      type: "agent" },
  ],

  // ── AI Assistant (Jarvis) ─────────────────────────────────────
  assistant: {
    name: "Jarvis",
    email: "jarvis@praesidium2.ai",
    avatarUrl: "data:image/svg+xml,...",         // Inline SVG or URL
    personality: "British, dry wit, technically brilliant, subtly protective of quality",
    personaFile: "jarvis/PERSONA.md",            // Full persona definition
    model: "claude-sonnet-4-6",                  // Chat response model
    agentModel: "claude-sonnet-4-6",             // Change processor model
    maxReplyTokens: 200,
    replyDelayMs: 15000,                         // Delay before posting a reply
  },

  // ── Navigation Links ──────────────────────────────────────────
  // Maps display names to URLs for auto-linking in chat messages
  linkMap: [
    { pattern: "Executive Summary",  href: "index.html" },
    { pattern: "Implementation Plan", href: "implementation.html" },
    { pattern: "Business Review",    href: "business-review.html" },
    { pattern: "System Design",      href: "system-design.html" },
  ],

  // ── Welcome Flow ──────────────────────────────────────────────
  welcome: {
    messages: [
      { content: "Good day. I'm Jarvis, your guide through the {{PROJECT_NAME}} design documents." },
      { content: "A few things that might help:\n* Press **Cmd+K** to search across all documents\n* Hover any card and click the comment icon to leave a note\n* Use the nav bar at top to switch between documents" },
      { content: "What's your role? I can suggest where to start.", roleButtons: [
        { key: "executive", label: "Executive / C-Suite" },
        { key: "technical", label: "Technical / Engineering" },
        { key: "product",   label: "Product / Operations" },
        { key: "browsing",  label: "Just Browsing" },
      ]},
    ],
    roleResponses: {
      executive: "Start with the Executive Summary, it is the landing page.",
      technical: "Head to System Design for the full technical architecture.",
      product:   "The Business Review has deployment examples and ROI analysis.",
      browsing:  "No problem. Start with the Executive Summary and explore from there.",
    },
  },

  // ── Theme ─────────────────────────────────────────────────────
  theme: {
    primaryColor: "#f59e0b",                     // Amber — used for buttons, accents, logo gradient start
    accentColor: "#ef4444",                      // Red — used for logo gradient end, error states
    highlightColor: "#22d3ee",                   // Cyan — used for agent badges, code highlights
  },
};
```

### 2.2 Configuration Loading

**Browser (client-side)**: A small `config-loader.js` script is included before all other scripts. It loads `jarvisdoc.config.js` which sets `window.JD`. All framework JS reads from `window.JD` at runtime.

```html
<!-- In every HTML page <head> -->
<script src="jarvisdoc.config.js"></script>
<script src="js/auth-gate.js"></script>
```

**Node.js (backend)**: The CLI generates a parallel `jarvis/jarvisdoc.config.json` from the same source values. Backend services read this JSON file at startup.

```javascript
// jarvis/jarvis.js
const config = require("./jarvisdoc.config.json");
const FIRESTORE_PROJECT_ID = config.firebase.projectId;
const JARVIS_AUTHOR = {
  authorName: config.assistant.name,
  authorEmail: config.assistant.email,
  // ...
};
```

### 2.3 `docs-manifest.json`

A standalone document registry that can be edited independently of the config. Used by the change processor to validate document IDs and map them to file paths.

```json
{
  "documents": [
    {
      "id": "index",
      "file": "index.html",
      "name": "Executive Summary",
      "color": "#f59e0b",
      "shortName": "EXEC SUMMARY",
      "editable": true
    },
    {
      "id": "system-design",
      "file": "system-design.html",
      "name": "System Design",
      "color": "#60a5fa",
      "shortName": "SYSTEM DESIGN",
      "editable": true
    }
  ],
  "protectedFiles": [
    "js/auth-gate.js",
    "js/comments.js",
    "jarvis/jarvis.js",
    "jarvis/change-processor.js",
    "firestore.rules",
    "firebase.json"
  ]
}
```

---

## 3. Framework Components

These are the client-side JavaScript files extracted from the current Praesidium2 implementation. Each is refactored to read all project-specific values from `window.JD` instead of hardcoded constants.

### 3.1 `auth-gate.js`

**Current state**: Firebase config (6 values), allowed domain, project name, subtitle, and logo letter are all hardcoded.

**Refactored behavior**:
```javascript
// Before (hardcoded)
const AUTH_GATE_CONFIG = {
  apiKey: "AIzaSy..._YOUR_HARDCODED_KEY",
  projectId: "praesidium2-design",
  // ...
};
const AUTH_ALLOWED_DOMAIN = "dmdbrands.com";
logoDiv.textContent = 'P';
title.textContent = 'Praesidium2';

// After (config-driven)
const AUTH_GATE_CONFIG = window.JD.firebase;
const AUTH_ALLOWED_DOMAIN = window.JD.auth.allowedDomain;
logoDiv.textContent = window.JD.project.logoLetter;
title.textContent = window.JD.project.name;
subtitle.textContent = window.JD.project.subtitle;
```

Primary color for the sign-in button and logo gradient reads from `window.JD.theme.primaryColor` and `window.JD.theme.accentColor`.

### 3.2 `comments.js`

**Current state**: Firebase config duplicated (6 values), allowed domain, and 7-person team directory hardcoded.

**Refactored behavior**:
```javascript
// Before
var FIREBASE_CONFIG = { apiKey: "...", projectId: "praesidium2-design", ... };
var ALLOWED_DOMAIN = "dmdbrands.com";
var MENTION_USERS = [
  { handle: "jmaes", name: "James Maes", title: "CTO", type: "user" },
  // ...
];

// After
var FIREBASE_CONFIG = window.JD.firebase;
var ALLOWED_DOMAIN = window.JD.auth.allowedDomain;
var MENTION_USERS = window.JD.team;
```

Firestore collection names get a project-slug prefix to prevent collision if two instances ever share a Firebase project (not recommended, but defensive):
```javascript
var COMMENTS_COLLECTION = window.JD.project.slug + "_comments";
var TYPING_COLLECTION = window.JD.project.slug + "_typing_indicators";
```

localStorage keys are namespaced:
```javascript
var STORAGE_PREFIX = "jf_" + window.JD.project.slug + "_";
```

### 3.3 `jarvis-chat.js`

**Current state**: Jarvis email, chat collection name, link map (10 entries), welcome messages (3 messages), role responses (4 entries), and localStorage keys are all hardcoded.

**Refactored behavior**:
```javascript
// Before
var JARVIS_EMAIL = "jarvis@praesidium2.ai";
var CHAT_COLLECTION = "jarvis_chats";
var LINK_MAP = [
  { re: /Executive Summary/g, href: "index.html" },
  // ...
];

// After
var JARVIS_EMAIL = window.JD.assistant.email;
var CHAT_COLLECTION = window.JD.project.slug + "_jarvis_chats";

// Link map built from config (regex compiled at init time, not per-message)
var LINK_MAP = window.JD.linkMap.map(function(entry) {
  return { re: new RegExp(escapeRegExp(entry.pattern), 'g'), href: entry.href };
});

var WELCOME_MESSAGES = window.JD.welcome.messages.map(function(msg) {
  return {
    content: msg.content.replace(/\{\{PROJECT_NAME\}\}/g, window.JD.project.name),
    roleButtons: msg.roleButtons || null
  };
});

var ROLE_RESPONSES = window.JD.welcome.roleResponses;
```

Session and chat history localStorage keys use the project slug prefix:
```javascript
sessionStorage.getItem("jf_" + window.JD.project.slug + "_session_id");
localStorage.getItem("jf_" + window.JD.project.slug + "_chat_history");
```

### 3.4 `search.js`

**Current state**: DOCS array hardcoded with 4 Praesidium2-specific pages. Empty-state message says "Praesidium2 documents."

**Refactored behavior**:
```javascript
// Before
var DOCS = [
  { id: 'index', file: 'index.html', name: 'Executive Summary', color: '#f59e0b', shortName: 'EXEC SUMMARY' },
  // ...
];

// After
var DOCS = window.JD.docs;
```

Empty-state message:
```javascript
// Before: "Type to search across all Praesidium2 documents"
// After:
"Type to search across all " + window.JD.project.name + " documents"
```

### 3.5 `version-loader.js`

**Already generic.** Reads `versions.json` dynamically, determines current version from filename. No changes needed — this is the one component that was already reusable from day one.

### 3.6 `built-with.js`

**Current state**: Hardcoded "Built with Praesidium1" narrative about how the design doc was created.

**Refactored behavior**: Becomes a generic "Built with JarvisDoc" attribution. The modal content is simplified to describe JarvisDoc itself (the platform), not any specific project.

```javascript
var MODAL_CONTENT = {
  title: "Built with JarvisDoc",
  subtitle: "An open source platform for interactive AI-assisted design documents.",
  sections: [
    {
      heading: "What is JarvisDoc?",
      text: "JarvisDoc is an open source toolkit by KofTwentyTwo that turns static design documents into interactive, AI-assisted review environments. Comments, search, version tracking, and a real-time AI assistant — all deployed to your own Firebase project."
    },
    {
      heading: "Features",
      items: [
        { name: "AI Design Assistant", role: "Monitors comments, answers questions, files change requests" },
        { name: "Threaded Comments", role: "Real-time, @mentions, resolve/reopen, typing indicators" },
        { name: "Full-Text Search", role: "Cmd+K cross-document search with highlighted results" },
        { name: "Change Processor", role: "Claude Agent SDK auto-implements approved changes" },
        { name: "Auth Gate", role: "Google sign-in restricted to your domain" },
      ]
    },
    {
      heading: "Learn More",
      text: "github.com/KofTwentyTwo/jarvisdoc"
    }
  ]
};
```

The footer link text reads "Built with JarvisDoc" (always, not configurable — this is the attribution for the open source project).

---

## 4. Jarvis Backend

Two Node.js services run alongside the static site, watching Firestore for events and responding autonomously.

### 4.1 `jarvis.js` — Comment Monitor + Chat Responder

**Responsibilities**:
1. Listen for new top-level comments in the `{slug}_comments` collection
2. Load relevant design spec context from the HTML file
3. Generate a contextual reply using Claude (model from config)
4. Post the reply back to Firestore
5. Listen for chat messages in `{slug}_jarvis_chats` collection
6. Generate conversational responses for the chat window
7. File change requests when appropriate

**Configuration loading**:
```javascript
const config = require("./jarvisdoc.config.json");

const FIRESTORE_PROJECT_ID = config.firebase.projectId;
const JARVIS_AUTHOR = {
  authorName: config.assistant.name,
  authorEmail: config.assistant.email,
  authorPhoto: config.assistant.avatarUrl,
};
const COMMENTS_COLLECTION = config.project.slug + "_comments";
const CHAT_COLLECTION = config.project.slug + "_jarvis_chats";
const MODEL = config.assistant.model;
const MAX_REPLY_TOKENS = config.assistant.maxReplyTokens;
const REPLY_DELAY_MS = config.assistant.replyDelayMs;
```

**Persona loading**: Reads `PERSONA.md` (generated from template during `create-jarvisdoc`). The persona file uses `{{PROJECT_NAME}}` and `{{ASSISTANT_NAME}}` tokens that are replaced at generation time.

### 4.2 `change-processor.js` — Automated Change Implementation

**Responsibilities**:
1. Watch the `{slug}_change_requests` collection for approved changes
2. Spawn a Claude Code agent (via `@anthropic-ai/claude-agent-sdk`) to implement the change
3. Read the target HTML file, make the minimum edit, verify
4. Commit the change to git
5. Deploy via `firebase deploy --only hosting`
6. Post a summary comment back to Firestore

**Configuration loading**:
```javascript
const config = require("./jarvisdoc.config.json");

const FIRESTORE_PROJECT_ID = config.firebase.projectId;
const MODEL = config.assistant.agentModel;

// Document file map built from docs-manifest.json
const manifest = require("../docs-manifest.json");
const DOCUMENT_FILE_MAP = {};
manifest.documents.forEach(function(doc) {
  DOCUMENT_FILE_MAP[doc.id] = doc.file;
});

const PROTECTED_FILES = new Set(manifest.protectedFiles);
```

**Agent system prompt**: Templated with project name and document list from config, not hardcoded.

```javascript
const AGENT_SYSTEM_PROMPT = [
  "You are the " + config.project.name + " Change Request Processor.",
  "You implement approved design changes to the " + config.project.name + " documents.",
  "",
  "Your workspace is " + DESIGN_DIR,
  "",
  "Available documents:",
  ...manifest.documents.map(d => "- " + d.file + " (" + d.name + ")"),
  "",
  "Rules:",
  "- Read the target file first to understand current content",
  "- Make the minimum edit necessary to fulfill the change request",
  "- Do NOT modify any files listed in protectedFiles",
  // ... (remaining rules are project-agnostic)
].join("\n");
```

### 4.3 Service Account

Each instance requires its own Firebase service account key at `jarvis/service-account-key.json`. This file is gitignored. The CLI reminds the user to download it from Firebase Console.

### 4.4 Process Management

The generated `scripts/start-jarvis.sh`:

```bash
#!/bin/bash
cd "$(dirname "$0")/../jarvis"

if [ ! -f service-account-key.json ]; then
  echo "ERROR: jarvis/service-account-key.json not found."
  echo "Download from: Firebase Console > Project Settings > Service Accounts"
  exit 1
fi

if [ -z "$ANTHROPIC_API_KEY" ]; then
  echo "ERROR: ANTHROPIC_API_KEY environment variable required."
  exit 1
fi

echo "Starting Jarvis comment monitor..."
node jarvis.js &
JARVIS_PID=$!

echo "Starting change processor..."
node change-processor.js &
PROCESSOR_PID=$!

echo "Jarvis is running. PID: $JARVIS_PID, $PROCESSOR_PID"
echo "Press Ctrl+C to stop."

trap "kill $JARVIS_PID $PROCESSOR_PID 2>/dev/null" EXIT
wait
```

---

## 5. Starter Templates

### 5.1 Template Design

Each starter template is a complete, valid HTML file with:
- Proper `<head>` with meta tags, favicon, config loader, auth gate
- Header with logo, project name, doc switcher nav, mobile hamburger menu
- Content area with placeholder sections
- Script tags for comments, search, version loader, chat, built-with
- The full design system CSS (linked stylesheet, not inline)

### 5.2 Available Templates

| Template | File | Purpose |
|----------|------|---------|
| Executive Summary | `index.html` | C-suite one-pager with hero, stats grid, team cards |
| System Design | `system-design.html` | Tabbed technical architecture with tab navigation groups |
| Implementation Plan | `implementation.html` | Phased build plan with task cards, acceptance criteria |
| Business Review | `business-review.html` | Independent assessment, findings table, recommendations |
| Presentation | `presentation.html` | Slide deck with keyboard navigation, progress bar |

### 5.3 Template Structure

Every template follows the same HTML skeleton:

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <link rel="icon" type="image/svg+xml" href="favicon.svg">
  <title><!-- Filled from config --></title>
  <script src="jarvisdoc.config.js"></script>
  <script src="js/auth-gate.js"></script>
  <link rel="stylesheet" href="css/jarvisdoc.css">
</head>
<body>

  <!-- Header -->
  <header class="header">
    <div class="header-top">
      <div class="logo-area">
        <div class="logo-icon" id="jf-logo"></div>
        <div class="logo-text">
          <h1 id="jf-project-name"></h1>
          <span id="jf-doc-name">Executive Summary</span>
        </div>
      </div>
      <nav class="doc-switcher" id="jf-doc-nav"></nav>
      <!-- Mobile hamburger injected by JS -->
    </div>
  </header>

  <!-- Content -->
  <div class="content">
    <section class="hero">
      <h2>Executive Summary</h2>
      <p class="section-subtitle">Add your project overview here.</p>
    </section>

    <!-- Add your content sections here -->
    <section>
      <h2>Section Title</h2>
      <p>Your content goes here. Use the design system's card, grid, stat-box,
         and table classes to structure your document.</p>
    </section>
  </div>

  <!-- Framework scripts -->
  <script src="js/comments.js"></script>
  <script src="js/search.js"></script>
  <script src="js/version-loader.js"></script>
  <script src="js/jarvis-chat.js"></script>
  <script src="js/built-with.js"></script>

  <!-- Dynamic elements populated from config -->
  <script>
    document.getElementById('jf-logo').textContent = window.JD.project.logoLetter;
    document.getElementById('jf-project-name').textContent = window.JD.project.name;
    // Doc nav built from window.JD.docs
  </script>
</body>
</html>
```

### 5.4 Content Patterns

Templates include commented examples of design system patterns:

```html
<!-- Stat Grid -->
<div class="stat-grid">
  <div class="stat-card">
    <div class="stat-number">42</div>
    <div class="stat-label">Your Metric</div>
  </div>
</div>

<!-- Info Card -->
<div class="card">
  <h3>Card Title</h3>
  <p>Card content with the dark theme design system.</p>
</div>

<!-- Data Table -->
<table class="data-table">
  <thead><tr><th>Column</th><th>Column</th></tr></thead>
  <tbody><tr><td>Value</td><td>Value</td></tr></tbody>
</table>
```

---

## 6. Firestore Rules Template

### 6.1 Parameterization

The CLI generates `firestore.rules` from a template, replacing two tokens:

| Token | Source | Example |
|-------|--------|---------|
| `{{ALLOWED_DOMAIN}}` | `auth.allowedDomain` from config | `dmdbrands.com` |
| `{{OWNER_EMAIL}}` | `auth.ownerEmail` from config | `jmaes@dmdbrands.com` |
| `{{SLUG}}` | `project.slug` from config | `praesidium2-design` |

### 6.2 Template

```
rules_version = '2';
service cloud.firestore {
  match /databases/{database}/documents {

    // Comments — restricted to @{{ALLOWED_DOMAIN}}
    match /{{SLUG}}_comments/{commentId} {
      allow read: if request.auth != null
                  && request.auth.token.email.matches('.*@{{ALLOWED_DOMAIN_ESCAPED}}');

      allow create: if request.auth != null
                    && request.auth.token.email.matches('.*@{{ALLOWED_DOMAIN_ESCAPED}}')
                    && request.resource.data.authorEmail == request.auth.token.email
                    && request.resource.data.keys().hasAll([
                         'documentId', 'pageId', 'sectionId', 'content',
                         'authorName', 'authorEmail', 'parentId', 'status',
                         'docVersion', 'createdAt'
                       ]);

      allow update: if request.auth != null
                    && request.auth.token.email.matches('.*@{{ALLOWED_DOMAIN_ESCAPED}}')
                    && request.resource.data.diff(resource.data).affectedKeys()
                       .hasOnly(['status', 'resolvedBy', 'resolvedAt'])
                    && request.resource.data.status in ['open', 'resolved'];

      allow delete: if false;
    }

    // Chat messages — restricted to @{{ALLOWED_DOMAIN}}
    match /{{SLUG}}_jarvis_chats/{chatId} {
      allow read: if request.auth != null
                  && request.auth.token.email.matches('.*@{{ALLOWED_DOMAIN_ESCAPED}}');

      allow create: if request.auth != null
                    && request.auth.token.email.matches('.*@{{ALLOWED_DOMAIN_ESCAPED}}')
                    && request.resource.data.keys().hasAll([
                         'sessionId', 'role', 'content',
                         'authorName', 'authorEmail', 'createdAt'
                       ]);

      allow update: if false;
      allow delete: if false;
    }

    // Change requests — owner-approved workflow
    match /{{SLUG}}_change_requests/{crId} {
      allow read: if request.auth != null
                  && request.auth.token.email.matches('.*@{{ALLOWED_DOMAIN_ESCAPED}}');

      allow update: if request.auth != null
                    && request.auth.token.email == '{{OWNER_EMAIL}}'
                    && request.resource.data.diff(resource.data).affectedKeys()
                       .hasOnly(['status', 'updatedAt']);
    }

    // Typing indicators — ephemeral
    match /{{SLUG}}_typing_indicators/{indicatorId} {
      allow read, write: if request.auth != null
                         && request.auth.token.email.matches('.*@{{ALLOWED_DOMAIN_ESCAPED}}');
    }
  }
}
```

`{{ALLOWED_DOMAIN_ESCAPED}}` is the regex-escaped domain (dots escaped: `dmdbrands[.]com`).

Collection names are prefixed with the project slug to prevent data collision in the (unlikely) scenario of shared Firebase projects.

---

## 7. Design System

### 7.1 Extracted Stylesheet: `jarvisdoc.css`

The complete dark-theme CSS is extracted from the inline `<style>` blocks currently duplicated across every HTML file. It becomes a single linked stylesheet.

### 7.2 CSS Custom Properties

Theme customization via CSS custom properties, set by `config-loader.js` from `window.JD.theme`:

```css
:root {
  /* Set dynamically from window.JD.theme */
  --jf-primary: #f59e0b;
  --jf-accent: #ef4444;
  --jf-highlight: #22d3ee;

  /* Fixed design system tokens */
  --jf-bg-base: #0f172a;
  --jf-bg-surface: #1e293b;
  --jf-bg-elevated: #334155;
  --jf-border: #334155;
  --jf-border-hover: #475569;
  --jf-text-primary: #f8fafc;
  --jf-text-secondary: #e2e8f0;
  --jf-text-muted: #94a3b8;
  --jf-text-dim: #64748b;
  --jf-text-faint: #475569;
  --jf-font-sans: 'Inter', -apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto, sans-serif;
  --jf-font-mono: 'JetBrains Mono', 'Fira Code', monospace;
  --jf-radius-sm: 6px;
  --jf-radius-md: 10px;
  --jf-radius-lg: 16px;
}
```

### 7.3 Component Classes

| Component | Class | Description |
|-----------|-------|-------------|
| Card | `.jf-card` | Dark surface with border, border-radius, padding |
| Stat Card | `.jf-stat-card` | Number + label layout for KPI displays |
| Stat Grid | `.jf-stat-grid` | Responsive grid for stat cards (auto-fit, min 200px) |
| Badge | `.jf-badge` | Small colored label (status, category, version) |
| Table | `.jf-table` | Striped dark table with hover states |
| Code Block | `.jf-code` | Monospace block with syntax-appropriate background |
| Tab Nav | `.jf-tabs` | Horizontal tab navigation with active indicator |
| Tab Panel | `.jf-tab-panel` | Content panel toggled by tab selection |
| Section | `.jf-section` | Content section with bottom margin |
| Hero | `.jf-hero` | Full-width section with gradient background |
| Grid | `.jf-grid-2`, `.jf-grid-3`, `.jf-grid-4` | Responsive column grids |
| Alert | `.jf-alert`, `.jf-alert--warn`, `.jf-alert--info` | Callout boxes |
| List | `.jf-list` | Styled unordered list with custom markers |
| Divider | `.jf-divider` | Subtle horizontal rule |

### 7.4 Responsive Breakpoints

```css
/* Mobile first */
@media (max-width: 768px) {
  .header { padding: 1rem; }
  .content { padding: 1rem; }
  .jf-stat-grid { grid-template-columns: 1fr 1fr; }
  .jf-grid-3, .jf-grid-4 { grid-template-columns: 1fr; }
  .doc-switcher { display: none; }        /* Hidden, hamburger menu shown */
  .mobile-nav { display: block; }
}

@media (max-width: 480px) {
  .jf-stat-grid { grid-template-columns: 1fr; }
  .jf-grid-2 { grid-template-columns: 1fr; }
}
```

### 7.5 Dark Theme Only (v1)

JarvisDoc v1 ships with the dark theme only. A light theme is a v2 consideration. The dark theme is the product's visual identity.

---

## 8. Hardcoded-to-Configurable Map

Complete inventory of every hardcoded value in the current Praesidium2 codebase and its migration path. Referenced from the [reusability analysis](reusability-analysis.md).

| # | Current Value | File(s) | Config Path |
|---|---------------|---------|-------------|
| 1 | Firebase API key | `auth-gate.js:11`, `comments.js:19` | `window.JD.firebase.apiKey` |
| 2 | Firebase project ID | `auth-gate.js:13`, `comments.js:21`, `jarvis.js:24`, `change-processor.js:58` | `window.JD.firebase.projectId` |
| 3 | Firebase auth domain | `auth-gate.js:12`, `comments.js:20` | `window.JD.firebase.authDomain` |
| 4 | Firebase storage bucket | `auth-gate.js:14`, `comments.js:22` | `window.JD.firebase.storageBucket` |
| 5 | Firebase messaging sender ID | `auth-gate.js:15`, `comments.js:23` | `window.JD.firebase.messagingSenderId` |
| 6 | Firebase app ID | `auth-gate.js:16`, `comments.js:24` | `window.JD.firebase.appId` |
| 7 | Allowed domain (`dmdbrands.com`) | `auth-gate.js:19`, `comments.js:27`, `firestore.rules` (x6) | `window.JD.auth.allowedDomain` |
| 8 | Owner email (`jmaes@dmdbrands.com`) | `firestore.rules:51` | `window.JD.auth.ownerEmail` |
| 9 | Project name (`Praesidium2`) | `auth-gate.js:43`, `comments.js:1`, `search.js:211,266,413` | `window.JD.project.name` |
| 10 | Subtitle (`DMD Brands — Confidential...`) | `auth-gate.js:48` | `window.JD.project.subtitle` |
| 11 | Logo letter (`P`) | `auth-gate.js:38` | `window.JD.project.logoLetter` |
| 12 | Jarvis email | `jarvis-chat.js:23`, `jarvis.js:31`, `change-processor.js:98` | `window.JD.assistant.email` |
| 13 | Jarvis author name + photo | `jarvis.js:29-33`, `change-processor.js:96-100` | `window.JD.assistant.{name,avatarUrl}` |
| 14 | DOCS array (search) | `search.js:11-16` | `window.JD.docs` |
| 15 | LINK_MAP (chat links) | `jarvis-chat.js:34-45` | `window.JD.linkMap` |
| 16 | WELCOME_MESSAGES | `jarvis-chat.js:989-1007` | `window.JD.welcome.messages` |
| 17 | ROLE_RESPONSES | `jarvis-chat.js:1009-1018` | `window.JD.welcome.roleResponses` |
| 18 | MENTION_USERS (team directory) | `comments.js:32-40` | `window.JD.team` |
| 19 | DOCUMENT_FILE_MAP | `change-processor.js:80-86` | `docs-manifest.json` |
| 20 | MODAL_CONTENT (built-with) | `built-with.js:9-51` | Static "Built with JarvisDoc" |
| 21 | localStorage keys (`jarvis_chat_history`) | `jarvis-chat.js:961,973,984` | Prefixed: `jf_{slug}_chat_history` |
| 22 | sessionStorage keys (`jarvis_session_id`) | `jarvis-chat.js:70` | Prefixed: `jf_{slug}_session_id` |
| 23 | Firestore collection names | `jarvis-chat.js:27`, `jarvis.js`, `change-processor.js` | Prefixed: `{slug}_comments`, etc. |
| 24 | `.firebaserc` default project | `.firebaserc:3` | Generated from `firebase.projectId` |
| 25 | MARKDOWN_SPEC_PATH | `change-processor.js:89` | Optional in `jarvisdoc.config.json` |
| 26 | Agent system prompts | `jarvis.js:199`, `change-processor.js:109-137` | Templated from config values |

---

## 9. Repository Structure

```
jarvisdoc/
  packages/
    create-jarvisdoc/              # CLI generator (npm package)
      bin/
        create-jarvisdoc.js        # Entry point (#!/usr/bin/env node)
      lib/
        prompts.js                   # Interactive prompt definitions
        generator.js                 # File scaffolding logic
        config-builder.js            # Builds config from prompt answers
        rules-compiler.js            # Firestore rules template → output
        persona-compiler.js          # Persona markdown template → output
      templates/
        jarvisdoc.config.js.tmpl   # Config file template
        jarvisdoc.config.json.tmpl # Node.js config template
        .firebaserc.tmpl             # Firebase RC template
        .gitignore.tmpl              # Gitignore template
        favicon.svg.tmpl             # Favicon with dynamic color
      package.json
      README.md

    jarvisdoc-core/                # Framework JS (browser)
      js/
        auth-gate.js                 # Authentication gate
        comments.js                  # Threaded comment system
        jarvis-chat.js               # AI chat assistant
        search.js                    # Full-text cross-document search
        version-loader.js            # Version dropdown
        built-with.js                # Attribution footer + modal
        config-loader.js             # Loads config → window.JD
      css/
        jarvisdoc.css              # Complete design system
      package.json

    jarvisdoc-backend/             # Jarvis + change processor (Node.js)
      jarvis.js                      # Comment monitor + chat responder
      change-processor.js            # Claude Agent SDK auto-implementation
      lib/
        config.js                    # Config loader for Node.js
        firestore.js                 # Firestore helpers
        anthropic.js                 # Claude API client wrapper
        context-reader.js            # HTML section text extractor
      package.json

  templates/
    starter/                         # HTML starter templates
      index.html                     # Executive Summary
      system-design.html             # System Design (tabbed)
      implementation.html            # Implementation Plan
      business-review.html           # Business Review
      presentation.html              # Slide Presentation
    firestore.rules.tmpl             # Parameterized Firestore rules
    firestore.indexes.json           # Required composite indexes
    firebase.json.tmpl               # Firebase hosting config
    PERSONA.md.tmpl                  # AI persona template
    docs-manifest.json.tmpl          # Document registry template
    versions.json.tmpl               # Initial version entry
    scripts/
      deploy.sh.tmpl                 # Deploy script
      start-jarvis.sh.tmpl           # Jarvis process manager

  docs/                              # Usage documentation
    getting-started.md               # Quick start guide
    configuration.md                 # Full config reference
    design-system.md                 # CSS component guide
    jarvis-backend.md                # AI backend setup
    templates.md                     # Template customization guide
    faq.md                           # Common questions

  examples/
    praesidium2/                     # The original as reference implementation
      jarvisdoc.config.js          # Real config (sanitized credentials)
      README.md                      # "This is the project that started it all"

  .github/
    workflows/
      ci.yml                         # Lint + test on PR
      publish.yml                    # npm publish on release tag

  package.json                       # Monorepo root (workspaces)
  LICENSE                            # MIT
  README.md                          # Project README
```

### 9.1 Monorepo Management

The repo uses npm workspaces. All three packages share a root `package.json`:

```json
{
  "name": "jarvisdoc",
  "private": true,
  "workspaces": [
    "packages/create-jarvisdoc",
    "packages/jarvisdoc-core",
    "packages/jarvisdoc-backend"
  ]
}
```

### 9.2 Published Packages

| Package | npm Name | Purpose |
|---------|----------|---------|
| `create-jarvisdoc` | `@koftwentytwo/create-jarvisdoc` | CLI generator |
| `jarvisdoc-core` | `@koftwentytwo/jarvisdoc-core` | Browser framework JS + CSS |
| `jarvisdoc-backend` | `@koftwentytwo/jarvisdoc-backend` | Node.js AI services |

The CLI copies files from `jarvisdoc-core` and `jarvisdoc-backend` into the generated project. Users do NOT install these packages as dependencies — they get standalone copies they can modify.

---

## 10. Implementation Phases

### Phase 1: Config Externalization (1 week)

Extract all hardcoded values into `jarvisdoc.config.js` and refactor framework JS to read from `window.JD`.

**Tasks**:
1. Create `jarvisdoc.config.js` with the full schema (Section 2.1)
2. Create `config-loader.js` that reads config and sets CSS custom properties
3. Refactor `auth-gate.js` — replace 6 Firebase config values, domain, name, subtitle, logo letter
4. Refactor `comments.js` — replace Firebase config, domain, team directory, collection names, localStorage keys
5. Refactor `jarvis-chat.js` — replace email, collection name, link map, welcome messages, role responses, localStorage/sessionStorage keys
6. Refactor `search.js` — replace DOCS array and empty-state message
7. Refactor `built-with.js` — replace with generic JarvisDoc attribution
8. Extract CSS into `jarvisdoc.css` from inline `<style>` blocks; convert hardcoded colors to CSS custom properties
9. Refactor `jarvis.js` — read from `jarvisdoc.config.json`, template system prompt
10. Refactor `change-processor.js` — read from `jarvisdoc.config.json`, build document map from manifest
11. Namespace all Firestore collections with project slug
12. Namespace all localStorage/sessionStorage keys with project slug
13. Verify the Praesidium2 instance still works identically after refactoring

**Exit criteria**: Praesidium2 design site deploys and functions with zero behavioral changes, all config read from `jarvisdoc.config.js`.

### Phase 2: CLI Generator (3-4 days)

Build `create-jarvisdoc` as an interactive Node.js CLI.

**Tasks**:
1. Set up `packages/create-jarvisdoc` with `bin` entry point
2. Implement interactive prompts using `inquirer` (or `prompts` for zero-dep)
3. Implement config builder (prompt answers to config object)
4. Implement file generator (config to directory of files)
5. Implement Firestore rules compiler (template to rules file)
6. Implement persona compiler (template to PERSONA.md)
7. Generate `.firebaserc`, `firebase.json`, `.gitignore`
8. Generate `docs-manifest.json` from selected templates
9. Generate `versions.json` with initial entry
10. Print post-generation instructions
11. Test end-to-end: run generator, deploy to a fresh Firebase project, verify all features work

**Exit criteria**: `npx create-jarvisdoc test-project` produces a deployable site with working auth, comments, search, and AI assistant.

### Phase 3: Starter Templates + Design System (2-3 days)

Create clean, content-free versions of each document type.

**Tasks**:
1. Create blank Executive Summary template with hero, stats grid, team cards
2. Create blank System Design template with tab navigation scaffold
3. Create blank Implementation Plan template with phase/task card scaffold
4. Create blank Business Review template with findings table, recommendations
5. Create blank Presentation template with slide navigation
6. Extract and consolidate CSS into `jarvisdoc.css` with CSS custom properties
7. Document all available CSS classes and their usage
8. Add commented HTML examples of each design pattern in templates

**Exit criteria**: Each template is a complete, valid HTML file that renders correctly with placeholder content and all framework features functional.

### Phase 4: Documentation + Examples (2-3 days)

Write user-facing documentation and prepare the Praesidium2 reference implementation.

**Tasks**:
1. Write `getting-started.md` — zero-to-deployed walkthrough
2. Write `configuration.md` — full config schema reference with all options
3. Write `design-system.md` — visual guide to CSS classes and patterns
4. Write `jarvis-backend.md` — AI backend setup, persona customization, troubleshooting
5. Write `templates.md` — how to customize and create new templates
6. Write `faq.md` — Firebase setup, auth issues, cost estimation
7. Write root `README.md` with badges, screenshot, quick start
8. Prepare `examples/praesidium2/` with sanitized config

**Exit criteria**: A new user can go from zero to deployed by following `getting-started.md` without additional help.

### Phase 5: Publish + Repo Setup (1-2 days)

Open source release.

**Tasks**:
1. Create `github.com/KofTwentyTwo/jarvisdoc` repository
2. Set up GitHub Actions CI (lint, test)
3. Set up GitHub Actions publish workflow (npm publish on release tag)
4. Publish `@koftwentytwo/create-jarvisdoc` to npm
5. Publish `@koftwentytwo/jarvisdoc-core` to npm
6. Publish `@koftwentytwo/jarvisdoc-backend` to npm
7. Create GitHub release with changelog
8. Add repo topics, description, social preview image
9. Test `npx @koftwentytwo/create-jarvisdoc` from a clean machine

**Exit criteria**: `npx @koftwentytwo/create-jarvisdoc my-project` works from any machine with Node.js 18+.

### Total Estimated Duration

| Phase | Duration | Dependencies |
|-------|----------|-------------|
| Phase 1: Config externalization | 5 days | None |
| Phase 2: CLI generator | 3-4 days | Phase 1 |
| Phase 3: Templates + design system | 2-3 days | Phase 1 |
| Phase 4: Documentation | 2-3 days | Phases 1-3 |
| Phase 5: Publish | 1-2 days | Phases 1-4 |
| **Total** | **13-17 days** | |

Phases 2 and 3 can run in parallel after Phase 1 completes, compressing the critical path to **10-13 days**.

---

## 11. What Is NOT in Scope

These items are explicitly deferred. They are not forgotten — they are conscious deferrals.

### Not in v1

| Item | Reason | Future Version |
|------|--------|---------------|
| **Multi-tenancy / SaaS hosting** | Per-Firebase-project isolation is the correct model. Multi-tenancy requires re-architecting auth, Firestore rules, and the change processor. Massive over-engineering for the use case. | Not planned |
| **Non-Firebase hosting** | Firebase Hosting + Firestore + Auth is a cohesive stack. Supporting Vercel, Netlify, or AWS would require abstracting the auth layer, database layer, and real-time listeners — essentially rewriting the backend. | v3+ |
| **Custom LLM provider plugins** | v1 supports Anthropic Claude only. The backend architecture allows swapping the API client, but a formal plugin interface adds complexity without demand signal. | v2 |
| **Markdown/MDX authoring** | The documents are HTML. Markdown-to-HTML compilation would require a build step, breaking the "static files on Firebase Hosting" simplicity. | v2 |
| **Light theme** | The dark theme is the product identity. A light theme requires auditing every color value in 800+ lines of CSS. | v2 |
| **Real-time collaborative editing** | Comments are real-time. Document editing is done by the change processor or manually. Google Docs-style co-editing is a different product. | Not planned |
| **Custom authentication providers** | Google sign-in via Firebase Auth is the only supported method. SAML, OIDC, email/password would require significant auth-gate refactoring. | v2 |
| **Internationalization** | All UI strings are English. i18n would require extracting 50+ strings across 6 JS files. | v2 |
| **Analytics / usage tracking** | No telemetry, no analytics. Users can add their own Firebase Analytics or Google Analytics. | User's choice |

---

## 12. Security Considerations

### 12.1 Firebase API Key

The Firebase API key is public by design — it identifies the Firebase project for client-side SDK initialization. It does NOT grant administrative access. Firebase security is enforced by Firestore rules, not API key secrecy.

Each JarvisDoc instance gets its own Firebase project, providing complete data isolation.

### 12.2 Service Account Key

The `service-account-key.json` is the only true secret. It grants administrative Firestore access to the backend services. The CLI:
- Adds it to `.gitignore` immediately
- Never generates a placeholder file (avoids accidental commits)
- Prints a warning if the user tries to commit it

### 12.3 Firestore Rules

The generated rules enforce:
- **Domain restriction**: Only `@{allowedDomain}` emails can read or write
- **Author integrity**: Comments can only be created with the authenticated user's email
- **Immutability**: Chat messages and comments cannot be deleted; comments can only have status fields updated
- **Approval gate**: Change requests can only be updated by the owner email
- **Schema validation**: Required fields enforced on create

### 12.4 XSS Prevention

All framework JS uses DOM API methods exclusively — no `innerHTML` assignments anywhere. User-generated content (comments, chat messages) is rendered via `document.createTextNode()`. DOMPurify is loaded as a defense-in-depth measure for the comment system.

### 12.5 localStorage Namespace Collision

The current Praesidium2 implementation uses global localStorage keys like `jarvis_chat_history`. If two JarvisDoc instances run on the same domain (e.g., during local dev on localhost), they would overwrite each other's state. The fix is project-slug prefixing on all storage keys: `jf_{slug}_chat_history`.

---

## 13. Cost Model

### Per-Instance Costs

| Component | Cost | Notes |
|-----------|------|-------|
| Firebase Hosting | Free tier: 10 GB/month | More than enough for static HTML |
| Firestore | Free tier: 50K reads/day, 20K writes/day | Sufficient for ~50 active reviewers |
| Firebase Auth | Free tier: unlimited Google sign-in | |
| Anthropic API (Jarvis chat) | ~$0.50-5/day | Depends on comment volume, model choice |
| Anthropic API (change processor) | ~$1-10/change request | Agent SDK sessions can be expensive |
| Custom domain (optional) | ~$12/year | Firebase supports custom domains |

**Typical total for a 10-person review team**: $0-15/month (mostly within free tiers).

---

## 14. Open Source Strategy

### 14.1 License

MIT. No restrictions on commercial use, modification, or redistribution.

### 14.2 Attribution

The `built-with.js` component renders a "Built with JarvisDoc" footer on every page. This is the only attribution requirement, and it is not enforced — users can remove it. The footer links to the GitHub repo.

### 14.3 Contribution Model

- Issues and PRs welcome on `github.com/KofTwentyTwo/jarvisdoc`
- Code of Conduct: Contributor Covenant
- PR requirements: passing CI, one approval from a maintainer
- Release cadence: semantic versioning, npm publish on GitHub release

### 14.4 Branding

- **Name**: JarvisDoc
- **Tagline**: "Interactive AI-assisted design documents"
- **Visual identity**: Dark theme, amber primary (#f59e0b), cyan highlight (#22d3ee)
- **Logo**: The Jarvis targeting reticle SVG (circles + crosshairs, cyan on dark)

---

## 15. Success Criteria

The product is ready for release when:

1. `npx @koftwentytwo/create-jarvisdoc test-project` works on a clean machine with Node.js 18+
2. Generated project deploys to a fresh Firebase project with `firebase deploy`
3. Auth gate restricts access to the configured domain
4. Comments system works: create, reply, resolve, reopen, @mention, real-time sync
5. Jarvis chat works: welcome flow, role-based suggestions, Firestore-backed conversation
6. Jarvis backend works: comment monitoring, contextual replies, change request filing
7. Change processor works: approved change request triggers agent, implements change, deploys
8. Search works: Cmd+K indexes all documents, returns highlighted results
9. Version loader works: dropdown populated from `versions.json`
10. The Praesidium2 reference implementation runs on the refactored framework with zero behavioral regressions
11. Documentation is complete: a new user can deploy without additional help
12. All three npm packages published and installable
