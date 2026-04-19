# CLAUDE.md — stats / Running Analytics Dashboard

> This file governs AI-assisted development on this project. Both layers of the workflow must treat the rules below as non-negotiable. When in doubt, escalate rather than assume.

---

## Project Overview

A Vite + React running analytics dashboard that consumes Strava API data. Purpose-built for running only — no rides, walks, or other sports. The project delivers personal training insights, performance trends, and session-level breakdowns for a single athlete.

**Stack:** Vite, React, JavaScript  
**Design System:** Ferrari (see `DESIGN.md`)  
**Folder structure:**

```
/
├── planning/          # Spec files produced by Claude Pro (reasoning layer)
├── api/
│   └── token.js       # Vercel Edge Function — Strava OAuth proxy
├── src/
│   ├── components/
│   ├── utils/
│   ├── App.jsx
│   ├── main.jsx
│   └── index.css
├── public/
├── dist/              # Build output — never commit
├── DESIGN.md          # Ferrari design system — reference before writing any UI
├── .env.local         # Local secrets — never touched by AI
└── CLAUDE.md
```

---

## Two-Layer AI Workflow

This project uses a strict separation between a **reasoning layer** (Claude Pro) and an **execution layer** (OpenCode Go). Neither layer may operate outside its designated role.

### Layer 1 — Reasoning (Claude Pro, $20/month)

Claude Pro is responsible exclusively for:

- Ingesting project context and existing code before making any plan
- Writing feature specs and saving them to `planning/<feature>.md`
- Making architectural decisions (component structure, data flow, API design, state management patterns)
- Reviewing completed work for correctness against spec acceptance criteria

**Claude Pro must never write implementation code directly into source files.**  
All output from this layer is a spec file. Code generation belongs to Layer 2.

### Layer 2 — Execution (OpenCode Go, ~$60/month credit budget)

OpenCode Go consumes spec files from `planning/` and implements the diff. It must:

- Read the relevant `planning/<feature>.md` in full before starting any task
- Read `DESIGN.md` in full before writing any UI component
- Implement exactly what the spec describes — no additions, no refactors beyond scope
- Never make architectural decisions; if a spec is ambiguous on architecture, stop and flag it rather than deciding independently
- Never touch `.env.local` or any secrets file

**A spec file must exist in `planning/` before any OpenCode Go task runs.** No exceptions.

---

## Model Assignment (OpenCode Go)

Model selection is credit-sensitive. Assign models by task complexity as follows.

### Complex tasks — multi-file changes, new features, API integration, data transformation logic

Use **Kimi K2.5** or **GLM-5.1** as primary models.

### Routine / high-volume tasks — styling fixes, copy changes, minor component edits, type corrections, test boilerplate

Use **MiniMax M2.5** or **Qwen3.5 Plus**.

| Task type | Preferred model |
|---|---|
| New feature implementation (multi-file) | Kimi K2.5 |
| Complex refactor or architectural migration | GLM-5.1 |
| Single-component edits, style fixes | MiniMax M2.5 |
| High-volume repetitive tasks (tests, stubs) | Qwen3.5 Plus |

---

## Spec File Format (`planning/<feature>.md`)

```markdown
# Feature: <name>

## Goal
One sentence stating what this feature accomplishes and why it exists.

## Context
Relevant background: which existing files are affected, what external APIs or data shapes are involved, and any constraints inherited from prior decisions.

## Steps
Numbered implementation steps. Each step names the file(s) to modify and describes the change in enough detail that the executing model requires no architectural judgment.

## File Targets
Explicit list of files to be created or modified.

## Acceptance Criteria
Verifiable conditions that define "done."
```

---

## Non-Negotiable Rules

1. **Layer separation is absolute.** OpenCode Go must never make architectural decisions. Claude Pro must never write implementation code into source files.
2. **Spec-first execution.** No OpenCode Go task may begin without a corresponding spec in `planning/`.
3. **No secrets, ever.** Neither AI layer may read from, write to, or reference `.env.local` or any secrets file.
4. **No direct commits to `main`.** All changes on a feature branch, merged via pull request.
5. **No file deletions without explicit spec instruction.**
6. **No scope creep.** OpenCode implements the spec as written.
7. **Read DESIGN.md before any UI work.** The Ferrari design system is non-negotiable.
