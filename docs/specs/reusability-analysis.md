# JarvisDoc System — Reusability Assessment

**Date**: 2026-04-03
**Status**: Analysis Complete
**Context**: Can the Praesidium2 interactive design document platform be turned into a reusable product?

---

## Executive Summary

The JarvisDoc system is a sophisticated interactive design document platform with genuine architectural bones worth preserving. However, it is currently a single-instance, single-tenant application with product-specific content, Firebase credentials, org-specific authorization, and persona/navigation hardcoded at 15+ distinct points across 10+ files. Making it reusable is achievable — the effort level depends on which reusability model you choose.

---

## Part 1: Full Hardcoding Inventory

### Firebase / Infrastructure Credentials (Critical)

Every instance needs its own Firebase project. Currently the credentials are duplicated across two files with no environment injection:

**`auth-gate.js` lines 11-17**
- `AUTH_GATE_CONFIG.projectId = "praesidium2-design"`
- `AUTH_GATE_CONFIG.authDomain = "praesidium2-design.firebaseapp.com"`
- `AUTH_GATE_CONFIG.apiKey` (live key, public-facing)
- `AUTH_GATE_CONFIG.messagingSenderId`
- `AUTH_GATE_CONFIG.appId`

**`comments.js` lines 18-25**
- Identical Firebase config object duplicated. Both files must match identically — a silent failure mode if they drift.

**`jarvis/jarvis.js` line 24**
- `const FIRESTORE_PROJECT_ID = "praesidium2-design"` — backend service uses this

**`jarvis/change-processor.js` line 58**
- `var FIRESTORE_PROJECT_ID = "praesidium2-design"` — duplicated again in the second backend service

**`.firebaserc` line 3**
- `"default": "praesidium2-design"` — controls which project `firebase deploy` targets

### Domain Authorization (Critical)

The allowed domain appears in 5 separate locations:

- `auth-gate.js` line 19: `const AUTH_ALLOWED_DOMAIN = "dmdbrands.com"`
- `comments.js` line 27: `var ALLOWED_DOMAIN = "dmdbrands.com"`
- `firestore.rules` lines 7, 31, 47, 57: `email.matches('.*@dmdbrands[.]com')` — four occurrences
- `firestore.rules` line 51: `jmaes@dmdbrands.com` — specific CTO email hardcoded as the change approver

### Product Name and Branding (Medium)

- `auth-gate.js` line 43: `title.textContent = 'Praesidium2'`
- `auth-gate.js` line 48: `subtitle.textContent = 'DMD Brands — Confidential Design Document'`
- `auth-gate.js` line 38: `logoDiv.textContent = 'P'` (single-letter logo)
- `comments.js` line 1: Module-level doc comment says "Praesidium2 Design Document"
- `jarvis/jarvis.js` line 199: System prompt embeds "Praesidium2 project at DMD Brands"
- `jarvis/change-processor.js` lines 110-133: `AGENT_SYSTEM_PROMPT` references "Praesidium2 documents"

### Jarvis Agent Identity (Medium)

- `jarvis-chat.js` line 23: `var JARVIS_EMAIL = "jarvis@praesidium2.ai"`
- `jarvis/jarvis.js` lines 29-33: `JARVIS_AUTHOR` object with same email
- `jarvis/change-processor.js` lines 96-100: `JARVIS_AUTHOR` duplicated a third time

### Document Structure and Navigation (Medium)

**`search.js` lines 11-16**: The `DOCS` array is fully hardcoded with Praesidium2-specific pages.

**`jarvis-chat.js` lines 34-45**: The `LINK_MAP` is fully hardcoded with Praesidium2-specific section names and anchors.

**`jarvis-chat.js` lines 990-1018**: `WELCOME_MESSAGES` and `ROLE_RESPONSES` reference specific Praesidium2 sections.

### The Change Processor's Document Map (Medium)

**`change-processor.js` lines 80-86**: `DOCUMENT_FILE_MAP` hardcodes every valid document ID-to-filename mapping (also serves as the security boundary preventing path traversal).

**`change-processor.js` line 89**: `MARKDOWN_SPEC_PATH` — absolute repo-relative path to the design spec file.

### Other Hardcoded Values (Low)

- `comments.js` lines 32-40: `MENTION_USERS` — hardcoded 7-person team directory
- `built-with.js` lines 9-51: `MODAL_CONTENT` — Praesidium2-specific narrative
- `versions.json`: Completely project-specific version history
- `jarvis/JARVIS_PERSONA.md`: References "Praesidium2 project at DMD Brands"

---

## Part 2: What Is Already Reusable

Zero product-specific logic — works unchanged for any instance:

- The entire CSS design system (dark theme, cards, typography, color system)
- `version-loader.js` — reads `versions.json` dynamically
- Comment panel UI mechanics (threading, resolve/reopen, typing indicators, real-time listeners)
- `auth-gate.js` login screen mechanism (flow is generic, only config/copy changes)
- `search.js` fetch + parse + index pipeline (if `DOCS` array is configurable)
- Change request Firestore data model (schema is product-agnostic)
- Claude integration in `jarvis.js` (persona loading, reply posting)
- Claude Agent SDK integration in `change-processor.js` (agent spawning, tool allowlist)

---

## Part 3: Architecture Options

### Option A — Template/Generator Approach (Recommended for MVP)

A CLI generates a new instance directory from the framework, prompting for project-specific values. Each instance is standalone, deployed to its own Firebase project.

**Effort**: 3-5 days
**Pros**: Low cost, full isolation, each instance customizable after generation
**Cons**: Manual sync if framework improves, not a true product

### Option B — Multi-Tenant SaaS (Not Recommended)

Single hosted app, Firestore partitioned by `projectId`.

**Effort**: 6-10 weeks
**Pros**: Single deployment, feature updates propagate
**Cons**: Massive over-engineering. Auth model, Firestore rules, change processor all need re-architecture.

### Option C — Config-Driven with Shared Codebase (Recommended Long-Term)

JS files read config from `jarvisdoc.config.json` at runtime. The "template" IS the source. Generated instances are just config files + content.

**Effort**: 1-2 weeks refactor, then ~1 hour per new instance
**Pros**: Single source of truth, per-instance isolation, config changes without code changes
**Cons**: Slightly more complex local dev setup

---

## Part 4: Quick Instance Spin-Up (Manual)

For a me-health-portal instance with the current codebase: ~15 files to create/modify, 4-8 hours, non-trivial risk of missed occurrences.

---

## Part 5: Full Productization

**Phase 1**: Config externalization — `jarvisdoc.config.js` → `window.JPC` global (1 week)
**Phase 2**: Firestore rules templating — build step fills tokens (2 days)
**Phase 3**: Instance generator CLI — `npx create-jarvisdoc my-project` (2-3 days)
**Phase 4**: Document schema definition — `docs-manifest.json` (1 day)
**Phase 5**: Persona templating — `{{PROJECT_NAME}}` tokens (1 day)

**Total**: 2-3 weeks of focused engineering.

---

## Part 6: Critical Risks

- **localStorage key collision**: Keys like `jarvis_chat_history` have no namespace. Need project-slug prefix.
- **Firestore collection names**: `jarvis_chats` etc. are hardcoded. Two instances against same Firebase project would mix data.
- **Change processor repo root**: Relies on `__dirname` being exactly 3 levels deep.
- **`processedIds` in-memory Set**: Jarvis loses dedup state on restart.
- **Firebase API key**: Public by design, but each instance needs its own Firebase project.

---

## Recommendation

- **Immediate need** (me-health-portal): Manual copy-paste, 4-8 hours
- **Real kit** (3-5 instances): Config externalization + shared framework, 2 weeks
- **Do NOT attempt multi-tenant SaaS** — per-Firebase-project isolation is the right model
