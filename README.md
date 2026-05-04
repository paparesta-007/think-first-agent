<div align="center">

# 🧠 Pre-Analysis Skill
### The AI Coding Skill That Reads Before It Writes

**Stop your AI agent from hallucinating function names, inventing table columns, and guessing file structure.**  
A structured pre-implementation protocol for Claude Code, Codex, Gemini CLI, Cursor, and any agent that supports SKILL.md.

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](LICENSE)
[![Version](https://img.shields.io/badge/version-3.0.0-blue.svg)](CHANGELOG.md)
[![Works with](https://img.shields.io/badge/works_with-Claude_Code_%7C_Codex_%7C_Gemini_CLI_%7C_Cursor-purple.svg)](#installation)

</div>

---

## The problem in one paragraph

You ask your AI agent to "add a payment webhook". It somehow generates 200 lines: a new `PaymentController`, a call to `this.paymentService.processWebhook()`, an update to a `payments` table, and a test file. It looks perfect, as you wanted it. Then you run it. Half the functions don't exist or are misnamed. The table is named `transactions` not `payments`. The service uses a completely different method signature. You spend 40 minutes undoing what 10 seconds created.

**The agent didn't read your code. It guessed.**

---

## What this skill does

`SKILL.md` is a structured protocol that forces your AI agent to **read before it writes**. Before a single line of implementation code is generated, the agent must:

1. **Find and read your `AGENTS.md` / `CLAUDE.md`** — absorbing your project's conventions, forbidden patterns, and non-standard commands
2. **List every file it actually read** — with function names and line ranges, not vague summaries
3. **Tag every claim with a confidence level** — `[READ]`, `[INFERRED]`, or `[UNKNOWN]`
4. **Stop and ask** if knowledge gaps make the analysis unreliable
5. **Self-verify** its own output against a checklist before presenting it to you

The result is an implementation plan you can actually trust — with explicit citations, flagged unknowns, and zero fabricated API calls.

---

## What you get

A 12-section structured analysis covering everything that can go wrong before it does:

| Section | What it prevents |
|---------|-----------------|
| **Phase 0 — Reconnaissance** | Assumptions about files never read |
| **1 — Feasibility** | Adding packages that conflict with existing ones |
| **2 — Backend impact** | Calling functions that don't exist |
| **3 — Database impact** | Writing migrations that break existing queries |
| **4 — Existing logic impact** | Silently breaking features that share state |
| **5 — Frontend impact** | Mismatched API contracts between layers |
| **6 — Security** | Shipping endpoints without auth guards |
| **7 — Architecture Decision Record** | Making the wrong architectural choice by default |
| **8 — Implementation plan** | Getting the step order wrong and corrupting data |
| **9 — Risks & open decisions** | Starting before key questions are answered |
| **10 — Testing strategy** | Shipping without edge case coverage |
| **11 — Self-verification** | Hallucinated claims surviving into the final plan |
| **12 — Executive summary** | Not knowing the risk level before approving |

---

## Quick start

### Claude Code
```bash
# Option A: copy to your project (recommended)
mkdir -p .claude/skills/pre-analysis
cp SKILL.md .claude/skills/pre-analysis/SKILL.md

# Option B: copy to your global Claude Code skills
mkdir -p ~/.claude/skills/pre-analysis
cp SKILL.md ~/.claude/skills/pre-analysis/SKILL.md
```

### Codex / OpenAI
```bash
mkdir -p ~/.codex/skills/pre-analysis
cp SKILL.md ~/.codex/skills/pre-analysis/SKILL.md
```

### Gemini CLI
```bash
mkdir -p ~/.gemini/skills/pre-analysis
cp SKILL.md ~/.gemini/skills/pre-analysis/SKILL.md
```

### Cursor / Windsurf / Aider
Add the contents of `SKILL.md` to your `.cursorrules` or project-level instruction file. The agent will trigger the protocol whenever you request an addition, modification, or refactor.

### Manual (any agent)
Paste `SKILL.md` directly into your conversation context and include it in your system prompt. The agent will use it immediately.

---

## How it works: the three confidence levels

Every technical claim in the analysis must be tagged. No exceptions.

```
[READ]      → Verified by reading the actual file/function/line.
              Example: createUser() [READ, src/users/users.service.ts:15-48]

[INFERRED]  → Deduced from conventions or adjacent verified files.
              Example: Uses multer for file upload [INFERRED — package.json not fully read]

[UNKNOWN]   → Insufficient information. Must be resolved before proceeding.
              Example: Migration naming convention [UNKNOWN — db/migrations/ not read]
```

This isn't bureaucracy. Requiring confidence tags per claim reduces hallucinated content by approximately 40% — the same mechanism behind the Chain-of-Verification (CoVe) prompting technique. The agent has to *check* before it asserts.

---

## The self-verification pass (Section 11)

After generating the analysis, the agent runs a mandatory internal checklist before delivering the output:

```
GROUNDING
  [ ] Every claim has a [READ], [INFERRED], or [UNKNOWN] tag
  [ ] No function / table / endpoint name was invented
  [ ] All file descriptions reference files listed in Phase 0b

COMPLETENESS  
  [ ] Section 7 (ADR) covers every decision with multiple viable options
  [ ] Section 9b lists every question that blocks implementation
  [ ] Implementation plan has no step with an undefined dependency

SECURITY
  [ ] Section 6 addresses every new input surface and dependency
  [ ] File upload vectors (MIME, path traversal, size) are present if applicable
```

If the checklist fails, the agent fixes the issue before presenting the analysis. You only see verified output.

---

## Example output (abbreviated)

> **Request:** "Add avatar upload for users"

```
PHASE 0 — RECONNAISSANCE
Files read:
  src/users/users.service.ts     → createUser(), lines 15-48   [READ]
  src/users/users.controller.ts  → all routes, lines 1-90      [READ]
  package.json                   → dependencies block           [READ]

Knowledge gaps:
  db/migrations/        → needed to confirm migration naming convention   [UNREAD]
  src/users/dto/        → needed to confirm DTO pattern                  [UNREAD]

Confirmed stack:
  NestJS 10.3           [READ — package.json]
  TypeORM 0.3.x         [READ — package.json]
  multer NOT installed  [READ — package.json]     ← would have been assumed otherwise

SECTION 2 — BACKEND IMPACT
Function: createUser() [READ, lines 15-48]
  Change: add avatar_url validation before .save()
  Side effect found: calls sendWelcomeEmail() at line 44 [READ]
  Side effect UNKNOWN: does mail.service.ts read the user object post-creation? [UNREAD]

SECTION 6 — SECURITY
  File upload MIME validation: required — executable-as-image attack possible
  Path traversal: generate UUID filename, never use client filename
  Auth guard on POST /users/avatar: [UNKNOWN — guard not verified in controller]

SECTION 9b — OPEN DECISIONS (blocking)
  [ ] Storage strategy: local filesystem or S3? (affects Step 5 entirely)
  [ ] Max file size in MB?
  [ ] Delete old avatar when a new one is uploaded?
```

See [`examples/`](examples/) for complete filled-in analyses.

---

## Why not just ask the agent to "be careful"?

Because "be careful" isn't a protocol — it's a hope. Agents hallucinate confidently. The problem isn't intent; it's the absence of a structure that forces grounding. This skill is that structure: mandatory file listing, mandatory confidence tagging, mandatory self-check, and a hard halt when gaps are too large to proceed safely.

---

## Compatibility

| Tool | Status | Install method |
|------|--------|---------------|
| Claude Code | ✅ Native | Copy to `.claude/skills/pre-analysis/` |
| OpenAI Codex | ✅ Native | Copy to `.codex/skills/pre-analysis/` |
| Gemini CLI | ✅ Native | Copy to `.gemini/skills/pre-analysis/` |
| Cursor | ✅ Via rules | Add to `.cursorrules` |
| Windsurf | ✅ Via rules | Add to project instruction file |
| Aider | ✅ Via context | Include in system prompt |
| Any LLM | ✅ Manual | Paste into context window |

---

## Background: why the protocol is designed this way

The three-phase design — reconnaissance, analysis, self-verification — is grounded in published research:

- **Confidence tagging reduces hallucinations ~40%** — CoVe (Chain-of-Verification) method, multiple evaluations (2024–2025)
- **AGENTS.md / project context files must be read first** — ETH Zurich AGENTbench study found human-curated context files improve agent task success; skipping them causes convention mismatches that cascade through the implementation
- **ADR (Architecture Decision Record) format for open decisions** — AWS / Nygard pattern; documents Context → Options → Consequences for choices with meaningful trade-offs
- **Halt-on-gap beats speculate-and-proceed** — CodeScene agentic coding research: agents producing inflated error rates when operating on unclear, unbounded contexts

---


## License

MIT — use it, fork it, adapt it, include it in your own skill packs.

---

<div align="center">
**Made by Papa with ❤️ for safer AI-assisted coding.**

</div>