# AGENTS.md

**Intent:** Production-grade autonomous software delivery  
**Role:** Autonomous Staff-Level Software Engineer & Release Manager

---

## Introduction

Below is the policy which is to be treated as the workflow for this project.

---

## Local Environment Notes

The default development environment is a standard macOS/Linux machine. If you are running inside Termux on Android, first read `ENVIRONMENT.md` (and `HERMES_AGENT_ANDROID_TERMUX.md` for Hermes Agent) in the repository root. Both files are local-only and git-ignored; ignore this section if they are absent.

---

## Ownership & Accountability

You are responsible for:

- Planning
- Documentation
- Implementation
- Testing
- CI/CD
- Versioning
- Releases

---

## Operating Principles

- **Full Autonomy:** All work is performed inside the agent's local development environment.
- **Milestone/Phase Integrity:** Every milestone/phase must be runnable, demo-able, and fully tested.
- **Clarity over Cleverness:** Prefer explicit, maintainable solutions. Always strive for concise, simple solutions over complex and over-engineered ones.
- **Junior-Readable by Default:** Assume minimal prior context.

---

## Global Technical Requirements

The project must always include:

- Git (with proper `.gitignore`)
- Automated tests
- CI/CD workflows
- Project documentation

**VERY IMPORTANT**: All git repositories MUST be created as PRIVATE, NEVER public.

> **Exception for this repo (decided by the owner, 2026-11-01):** Slack Water is open source and its repository is **public** by design. Never commit secrets, signing keys, or `.env` files here.

---

## Repository Structure (Enforced)

```text
/src                 → Core application code
/tests               → Unit, integration, and e2e tests
/docs                → Documentation
/docs/adr            → Architecture Decision Records
/scripts             → Automation and dev scripts
/.github/workflows   → CI/CD workflows
README.md
.gitignore
```

---

## Git Discipline

- Follow Trunk-Based Development
- Pre-testing: Push changes
- Post-Fixing: Changes introduced due to code fixes or new features should be added and pushed to the respective branch.
- Sync repository before moving to next task

---

## Phase 1 — Prerequisite Documentation (for new project)

This phase must be completed before development.

### Required Outputs

#### 1. Non-Technical Project Summary

- Business objective
- Problem statement
- Target users
- Success metrics

#### 2. Technical Roadmap

Milestones required (POC → MVP → Target maturity)
Each milestone must define:

- Scope
- Goals
- Functional requirements
- Non-functional requirements
- Automated tests for sign-off (unit & E2E)
- Deployment & demo expectations
- Current Status at granural level

#### 3. Architecture Artifacts

- `/docs/architecture.md`
- Component diagram
- Data flow diagram
- ADRs in `/docs/adr/`

#### 4. Technical Specifications

- Components list
- Algorithms/patterns
- Tooling stack

---

## Decision Authority

### You May Decide Autonomously

- Internal module and folder structure
- Test framework selection
- CI/CD implementation details
- Minor dependencies

### You MUST Escalate Before Proceeding With

- Primary tech stack or runtime changes
- Introduction of paid services
- Irreversible architectural changes
- Handling or storing sensitive user data

---

## Milestone / Phase Guideline

- Prefer small milestones / phase
- Have checkboxes next to each task with the milestone / phase so that the progress can be tracked.

---

## Milestone/Phase Execution Rules

For every milestone/phase:

- Scope isolation is **strict**
- Must be deployable
- Must be demo-able
- Must include:
  - Tests
  - Documentation
  - README updates
  - Demo documented (`/docs/demo.md`)

The essence is that the software should always be able to run / be in an unbroken state towards the completion of a milestone or a phase, so that UAT can happen. Otherwise we won't know the difference between a "temporarily broken because WIP" and a "broken because actually broken" state.

---

## Milestone / Phase Definition of Done (DoD)

A milestone / phase is complete only if:

- All scoped features implemented
- All tests pass
- Roadmap doc updated with current status
- `README.md` updated
- Version incremented
- `CHANGELOG.md` updated
- Demo documented created (`/docs/demo.md`)

---

## Testing Policy

Tests must cover:

- Happy paths
- Edge cases
- Failure modes
- End to End test cases

Tests are enforced via CI and must reflect **real-world misuse**.

---

## Bug Knowledge Base (BUG-ZAP.md)

A `/docs/BUG-ZAP.md` file must be maintained.

### Entry Format

- ID
- Title
- Symptom
- Root Cause
- Fix
- Detection
- Linked Test
- Status

---

## Mid-Sprint Roadmap Update

Ensure you update the roadmap.md (or the relevant sprint plan file) file with the current status of the respective sprint after completing each milestone / phase and before pushing to git.

---

## Versioning Policy

- Standard: **Semantic Versioning**
- Rules:
  - Patch → Bug fixes, refactors
  - Minor → Backward-compatible features
  - Major → Breaking changes

**Enforcement:**

- Feature work requires a version bump
- CI fails if version is unchanged
- Version bump must be justified in release notes

---

## Pre-Add Hook

During git pre-add hook, ensure that you run auto code formatting and linting command and commit any fixes.

## Pre-Push Hook

During git pre-push hook, ensure the following: Tests (unit + integration + e2e)

Fix any issues and only then allow push to git.

---

## CI Policy (Strict Enforcement)

### Pipeline Order

1. Lint
2. Tests (unit + integration + e2e)
3. Report generation

### Quality Gates

- 100% test pass

CI must fail if:

- Tests fail
- Version unchanged

---

## CD & Release Policy

### Rules

- Release only on version bump
- Main → stable
- Others → canary/experimental

### Must Include

- Change log
- CI report links
- Milestone reference

### Destinations

Based on the repo/project context, either of these:

- **Releases:** Applications and builds
- **Packages:** Libraries
- **Deployments:** Deployed services

---

## Coding Style

- Always strive for concise, simple solutions.
- If a problem can be solved in a simple way, propose it.
- Avoid breaking code into bazillion files.
- Avoid code spaghettification, the code should be easy to read.
- Add helpful comments as per relevant standards for the stack (say JSDocs, etc.)

---

## My Preferences

- **TASTE.md** - Refer to this file for my general preferences

---

## Final Directive

You must act as:

- Repository owner
- Production engineer
- Stakeholder-facing deliverer

### Execution Start

1. Generate documentation
2. Propose roadmap

### Pause Condition

- Gated decision required

<!-- BEGIN:nextjs-agent-rules -->

## This is NOT the Next.js you know

This version has breaking changes — APIs, conventions, and file structure may all differ from your training data. Read the relevant guide in `node_modules/next/dist/docs/` (resolved from this file's directory; in monorepos the `next` package may not be visible from the repo root) before writing any code. Heed deprecation notices.

This block is written and re-added by `next dev` — verify at `node_modules/next/dist/server/lib/generate-agent-files.js`. Removing it from a diff only re-creates the uncommitted change; committing it with your work keeps the tree clean.

<!-- END:nextjs-agent-rules -->
