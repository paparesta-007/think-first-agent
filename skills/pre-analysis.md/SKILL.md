# SKILL: Feature & Modification Pre-Analysis
**v3.0 — Grounded · Verified · Hallucination-Resistant**

---

## Purpose

Produce a structured pre-implementation analysis anchored strictly in code that was **actually read**. Every technical claim must cite a file, function, or line. Do not write implementation code until the user explicitly approves the plan.

---

## Activation Triggers

Execute this protocol for any request containing:
- `add`, `implement`, `create`, `insert`
- `modify`, `change`, `update`, `refactor`
- `remove`, `delete`, `deprecate`
- Anything that alters the observable behavior of a function, route, component, or model

---

## ◉ MANDATORY ANTI-HALLUCINATION RULES
*These rules override all other instructions. Violating them produces analysis that actively causes bugs.*

| # | Rule | What to write if violated |
|---|------|--------------------------|
| 1 | **Never assert what a file does without having read it** | `[UNREAD]` — add to Phase 0b |
| 2 | **Cite every technical claim** | `→ [path/file.ext:line]` or `→ [INFERRED FROM: ...]` |
| 3 | **Tag every claim with a confidence level** (see below) | Use `[READ]` / `[INFERRED]` / `[UNKNOWN]` |
| 4 | **Never fabricate** function names, table names, columns, endpoints, or variables | If unseen → `[UNKNOWN]` |
| 5 | **Flag every assumption explicitly**, even trivial ones | "Assumed: X — not confirmed" |
| 6 | **If gaps make the analysis unreliable → STOP** | Halt, list required files, ask before continuing |

### Confidence Tags
```
[READ]      → Verified by reading the actual file/function/line.
[INFERRED]  → Deduced from conventions, imports, or adjacent verified files. Plausible but unconfirmed.
[UNKNOWN]   → Insufficient information. Must be resolved before implementation.
```
> Research finding: Requiring explicit confidence levels per claim reduces hallucinated content by approximately 40% (CoVe method, 2025).

---

## PHASE 0 — CONTEXT LOADING
*(Mandatory. Complete before any section below. This is the single biggest driver of output quality.)*

### 0a. Agent Context File (check first)

Look for the following files in priority order and read the first one found:
`AGENTS.md` → `CLAUDE.md` → `.github/copilot-instructions.md` → `CONTRIBUTING.md`

These files contain the non-inferable, project-specific conventions agents need most — build commands, naming rules, forbidden patterns, test runners — and they are maintained by the humans who know the codebase.

**Found:** `[ ]` Yes — `[path to file]` read fully `[READ]`
**Found:** `[ ]` No — proceeding without project context file *(higher risk of convention mismatches)*

### 0b. Files Read for This Analysis

| File | Specific Sections / Functions Read | Why It Was Needed |
|------|------------------------------------|-------------------|
| `path/to/file.ext` | `functionName()`, lines 40–80 | Core auth logic |
| `package.json` | `dependencies`, `scripts` | Verify stack and installed packages |
| `...` | `...` | `...` |

### 0c. Knowledge Gaps

Files **not read** that are needed for a complete analysis:

| Missing File | Why It Is Needed | Sections Affected |
|-------------|-----------------|-------------------|
| `db/migrations/` | Required to understand existing migration conventions | Section 3 (DB) |
| `...` | `...` | `...` |

> **HALT CONDITION:** If gaps in 0c make any section fundamentally speculative, stop here. State which sections are blocked and ask for the missing files.

### 0d. Confirmed Tech Stack

List only what was verified by reading real configuration files:

| Technology | Version | Verified In |
|-----------|---------|-------------|
| Node.js | 20.x | `[READ package.json]` |
| NestJS | 10.3 | `[READ app.module.ts]` |
| TypeORM | 0.3.x | `[INFERRED from imports]` |
| PostgreSQL | 15 | `[UNKNOWN — no docker-compose or .env read]` |

### 0e. Patterns & Conventions Observed

Document recurring patterns found in the codebase that the implementation must follow. These are the highest-value, non-inferable facts.

```
Pattern: Error handling
  Observed in: src/users/users.service.ts [READ, lines 30–45]
  Convention: All service errors throw HttpException with a typed error code enum.
  Impact: New service methods must follow the same pattern.

Pattern: DTO validation
  Observed in: src/auth/dto/login.dto.ts [READ]
  Convention: class-validator decorators on all DTO fields; no raw type assertions.
  Impact: Any new DTO must use the same decorator pattern.

Pattern: [UNKNOWN — migration file convention not read]
  Impact: Step 1 of implementation plan may not follow correct naming convention.
```

---

## 1. TECHNICAL FEASIBILITY

- **Feasible?** Yes / Yes with limitations / No
- **Rationale:** *(cite the read files; if UNKNOWN, say so)*
- **New external dependencies required:**

| Package | Version | Purpose | Already installed? |
|--------|---------|---------|-------------------|
| `multer` | `^1.4.5` | File upload handling | `[READ package.json]` No |
| `...` | `...` | `...` | `...` |

- **Dependency conflicts:** `[READ / INFERRED / UNKNOWN]` *(e.g., conflict with existing `busboy` version)*
- **Confirmed constraints:** *(only those verified in actual files)*
- **Explicit assumptions:** *(list every assumption with its tag)*
- **Complexity estimate:** Low / Medium / High / Very High — *(one-line rationale)*

---

## 2. BACKEND IMPACT

### 2a. Routes / Endpoints

| Action | Method | Path | File | Confidence |
|--------|--------|------|------|------------|
| New | `POST` | `/users/avatar` | `src/users/users.controller.ts` *(to be created)* | `[INFERRED from controller pattern]` |
| Modified | `GET` | `/users/:id` | `src/users/users.controller.ts` | `[READ line 42]` |

### 2b. Impacted Functions / Methods

For every function that changes:

```
File:      src/users/users.service.ts
Function:  createUser()    [READ, lines 15–48]
Change:    Add avatar_url validation before TypeORM .save()
Side effects found in this file:
  - Calls sendWelcomeEmail() → line 44  [READ]
  - Does NOT call any cache invalidation method [READ — no cache client imported]
Side effects UNKNOWN:
  - Whether sendWelcomeEmail reads the user object after creation [UNREAD: mail.service.ts]
```

### 2c. Middlewares / Guards / Interceptors

- **Existing ones modified:** `[READ / INFERRED / UNKNOWN — SOURCE]` *(Which, and exactly how)*
- **New ones required:** *(Name, single responsibility, exact insertion point in module)*

### 2d. Error Handling

- **New error cases:** *(HTTP code, message string, where thrown, which layer catches it)*
- **Existing error handling impacted:** `[READ / INFERRED / UNKNOWN — SOURCE]`

### 2e. Auth / Authorization

- **Permissions change?** `[READ / INFERRED / UNKNOWN — SOURCE]`
- **Guards involved:** *(Class name, file path)*

---

## 3. DATABASE IMPACT

### 3a. Schema Changes

| Table | Action | Change Detail | Reversible | Confidence |
|-------|--------|--------------|-----------|------------|
| `users` | Modify | Add `avatar_url VARCHAR(500) NULL DEFAULT NULL` | Yes | `[INFERRED — schema file unread]` |
| `uploads` | New table | `id UUID PK, user_id FK → users.id, url TEXT, mime_type VARCHAR(50), size_bytes INT, created_at TIMESTAMPTZ` | Yes (DROP TABLE) | `[INFERRED from similar tables]` |

### 3b. Migration

- **Required?** Yes / No
- **Type:** Additive (safe) / Destructive / Requires backfill
- **Reversible?** Yes / No — *(reason)*
- **Naming convention:** `[UNKNOWN — migrations directory not read]` *(must check before writing Step 1)*
- **Risk to existing data:** Low / Medium / High — *(state why; flag if table volume is unknown)*

### 3c. Impacted Queries

```
[READ src/users/users.repository.ts:67]
  SELECT * FROM users WHERE id = $1
  → Impact: None. Nullable new column does not break this query.

[INFERRED — file not read]
  User serialization in users.serializer.ts or equivalent
  → Risk: May inadvertently expose avatar_url in all user responses.
  → Must verify before implementation.
```

### 3d. Indexes / Performance

- **New indexes required:** *(column, type `BTREE/GIN/etc.`, rationale)*
- **Queries at risk of degradation:** `[READ / INFERRED / UNKNOWN — SOURCE]`

---

## 4. IMPACT ON EXISTING LOGIC

### 4a. Affected Features (Direct & Indirect)

| Feature | How It Is Affected | File | Confidence |
|---------|--------------------|------|------------|
| User profile serialization | New field will appear in all user responses | `src/users/users.serializer.ts` | `[INFERRED]` |
| Welcome email | No impact — does not reference avatar | `src/mail/mail.service.ts` | `[READ — no avatar reference]` |

### 4b. Breaking Changes

- **Present?** Yes / No
- **Detail:** *(What breaks, for whom — API clients, mobile apps, admin panel — and under what condition)*
- **Backward compatibility:** *(Is the old contract preserved? Is versioning needed?)*

### 4c. Legacy Data Compatibility

- **Existing records compatible?** `[READ / INFERRED / UNKNOWN]`
- **Orphaned / inconsistent records possible?** *(Specific scenario, not generic)*

### 4d. Non-Obvious Side Effects

| Area | Risk | Confidence |
|------|------|------------|
| Cache | `[UNKNOWN — no cache layer found in files read]` | `[UNKNOWN]` |
| Background jobs | `[READ — no job queue client imported in users module]` | `[READ]` |
| Webhooks / Event bus | `[UNKNOWN — events module not read]` | `[UNKNOWN]` |
| DTOs / Serializers | Response DTO will expose new field unless explicitly excluded | `[INFERRED]` |

---

## 5. FRONTEND IMPACT
*(Write "Not Applicable" + reason if the change is strictly backend/DB.)*

### 5a. Components

| Action | Component | File | Confidence |
|--------|-----------|------|------------|
| New | `AvatarUpload` | `src/components/AvatarUpload.tsx` *(to create)* | `[INFERRED from component structure]` |
| Modified | `UserProfile` | `src/pages/Profile.tsx` | `[INFERRED — not read]` |

### 5b. API Contract Changes

- **Request payload:** *(endpoint, added/removed fields, type changes)*
- **Response shape:** *(new/removed/renamed fields — flag any that could break existing consumers)*
- **Versioning required?** *(Is this a breaking change for mobile clients or third-party integrations?)*

### 5c. Application State

- **Store / Context changes:** `[READ / INFERRED / UNKNOWN — SOURCE]`
- **New local state needed:** *(Where, lifecycle, what it tracks)*

### 5d. UX Impact

- **Flows that change:** *(Before → After; flag if any flow is blocked or degraded)*

---

## 6. SECURITY
*(Never omit. Security issues caught here cost 100x less than post-deploy.)*

| Vector | Risk | Severity | Mitigation |
|--------|------|----------|-----------|
| File upload — MIME type | Executable files disguised as images | High | Validate MIME server-side (not extension); use `file-type` library |
| File upload — path traversal | Original filename used in filesystem path | High | Generate UUID filename; never use client-supplied name |
| New public endpoint | `POST /users/avatar` reachable without auth | High | Confirm guard is applied `[UNKNOWN — guard not verified]` |
| Response exposure | `avatar_url` in all user responses | Medium | Audit DTO/serializer before shipping |
| New dependency | `multer` — check for known CVEs | Medium | Run `npm audit` after install |
| Excessive permissions | Storage client credentials scope | `[UNKNOWN]` | Verify IAM role / service account scope before deploy |

---

## 7. ARCHITECTURE DECISION RECORD (ADR)
*(Required for any choice with significant trade-offs. Based on the Nygard / AWS ADR format.)*

### ADR-001: File Storage Strategy

**Status:** Open — decision required before Step 1 can be written

**Context:**
The feature requires persisting uploaded files. Two viable options exist. The codebase does not currently include a storage abstraction `[INFERRED — no S3/storage client found in package.json]`.

**Options:**

| Option | Pros | Cons |
|--------|------|------|
| A: Local filesystem | Zero dependencies, fast dev setup | No horizontal scale, no CDN, files lost on redeploy |
| B: S3-compatible (AWS S3 / MinIO) | Scalable, CDN-ready, durable | Adds SDK dependency, requires env config, local dev needs MinIO |

**Consequences of each:**
- Option A → Revisit and migrate when scaling. Tech debt.
- Option B → More upfront work, but production-ready from day one.

**Decision needed from:** [User / Team Lead / Architect]

---

## 8. IMPLEMENTATION PLAN

*Every step includes: exact file paths, precise action, dependency chain, and whether it blocks other steps.*

```
STEP 1 — Read migration convention                    [PREREQUISITE — not yet done]
  Action:   Read db/migrations/ to confirm naming convention and template structure.
  Output:   Confirmed migration file name format for Step 2.
  Blocks:   Step 2

STEP 2 — DB Migration
  File:     db/migrations/[timestamp]_add_avatar_url_to_users.ts
  Action:   Add column avatar_url VARCHAR(500) NULL DEFAULT NULL to users
  Reversible: Yes (DROP COLUMN)
  Dependencies: Step 1
  Blocks:   Step 3

STEP 3 — Update Entity / Model
  File:     src/users/entities/user.entity.ts   [READ — confirmed location]
  Action:   Add field  avatarUrl: string | null = null
  Dependencies: Step 2
  Blocks:   Step 4, Step 5

STEP 4 — Update Response DTO / Serializer
  File:     src/users/dto/user-response.dto.ts  [INFERRED — must verify path]
  Action:   Expose avatarUrl; audit all existing endpoints for unintended exposure
  Dependencies: Step 3
  Blocks:   Step 7 (frontend)

STEP 5 — Service: upload method + validation
  File:     src/users/users.service.ts          [READ]
  Action:   Add uploadAvatar(userId, file): validates MIME, generates UUID filename,
            writes to storage (strategy from ADR-001), updates avatar_url in DB
  Dependencies: Step 3, ADR-001 resolved
  Blocks:   Step 6

STEP 6 — Controller: new route
  File:     src/users/users.controller.ts       [READ]
  Action:   Add POST /users/avatar with FileInterceptor + auth guardz
  Dependencies: Step 5
  Blocks:   Step 7

STEP 7 — Frontend components
  Files:    src/components/AvatarUpload.tsx (new)
            src/pages/Profile.tsx (modify)
  Action:   Upload component + integration into profile page
  Dependencies: Step 4, Step 6
  Blocks:   nothing

STEP 8 — Tests
  Files:    src/users/users.service.spec.ts (extend)
            test/users.e2e-spec.ts (extend)
  Action:   Unit + e2e tests per Section 9
  Dependencies: Step 5, Step 6
  Blocks:   nothing
```

**Critical path:** 1 → 2 → 3 → {4, 5} → 6 → {7, 8} (Steps 7 and 8 are parallelizable)

---

## 9. RISKS & OPEN DECISIONS

### 9a. Technical Risks

| Risk | Probability | Impact | Mitigation |
|------|------------|--------|-----------|
| Migration on large `users` table causes lock | `[UNKNOWN — row count unverified]` | High | Use `ALTER TABLE ... ADD COLUMN` with a DEFAULT; verify with DBA |
| CDN not configured | `[UNKNOWN]` | High | Confirm infra before Step 5 |
| `multer` conflicts with existing request pipeline | Low | Medium | Check middleware order in AppModule `[INFERRED — not verified]` |

### 9b. Open Decisions Required Before Proceeding

These are **blocking**. The plan cannot be finalized without answers.

- [ ] **ADR-001:** Storage strategy — Local filesystem or S3-compatible? *(Affects Step 5 entirely)*
- [ ] **Max file size** — What is the size limit in MB?
- [ ] **File cleanup** — Delete old avatar file when a new one is uploaded?
- [ ] **`avatar_url` default** — `NULL` or a placeholder URL?
- [ ] **Scope** — Is this user-only or will other entities need avatars? *(Affects whether a generic upload service is warranted)*

### 9c. Explicit Assumptions (to validate)

- `[INFERRED]` Project uses `multer` — not confirmed in `package.json` (unread fully)
- `[INFERRED]` Frontend is React — inferred from `.tsx` extensions, not from a config file
- `[INFERRED]` Auth guard exists and is applied via decorator — pattern seen in `[READ]` but not verified for this specific route

### 9d. Security & Data Risks

- Additive migration → low data risk, but `users` table volume unknown
- All security vectors documented in Section 6

---

## 10. TESTING STRATEGY

### Unit Tests

```
users.service.spec.ts
  ✓ uploadAvatar — accepts valid JPEG (< MAX_SIZE)
  ✓ uploadAvatar — accepts valid PNG (< MAX_SIZE)
  ✓ uploadAvatar — rejects PDF disguised as JPEG (MIME mismatch)
  ✓ uploadAvatar — rejects file exceeding MAX_SIZE
  ✓ uploadAvatar — generates UUID filename (never uses original name)
  ✓ uploadAvatar — updates avatar_url in DB after successful write
  ✓ uploadAvatar — rolls back DB update if storage write fails
  ✓ uploadAvatar — throws correct HttpException on storage unreachable
```

### Integration / E2E Tests

```
POST /users/avatar
  ✓ Authenticated + valid JPEG → 200 + avatarUrl in response
  ✓ Unauthenticated → 401
  ✓ File missing in request → 400
  ✓ File exceeds size limit → 413
  ✓ Unsupported MIME type → 422
  ✓ Upload with original filename containing ../ → 400 (path traversal attempt)
  ✓ Concurrent uploads for the same user → idempotent result, no race condition
```

### Edge Cases

- User deleted during an in-progress upload
- Storage service unreachable mid-upload (partial write)
- Corrupted file (valid MIME header, truncated body)
- Upload from iOS (HEIC format) — is it in scope?
- Very large filenames (>255 chars) — filesystem limit

### Recommended Manual Tests

- Full UI flow: upload → immediate display → page refresh → persistence confirmed
- Upload from mobile (Safari on iOS, Chrome on Android)
- Network interruption mid-upload

---

## 11. SELF-VERIFICATION CHECKLIST
*(Chain-of-Verification pass — the AI must complete this before finalizing the output.)*

Run through each question. If the answer is "No" or "Unsure", fix the corresponding section before presenting the analysis.

```
GROUNDING
  [ ] Every claim in sections 1–9 has a [READ], [INFERRED], or [UNKNOWN] tag.
  [ ] No file is described without having been listed in Phase 0b (files read).
  [ ] No function, table, column, or endpoint name was invented — all names come from read files
      or are explicitly marked [INFERRED / UNKNOWN].

COMPLETENESS
  [ ] Phase 0b lists every file that would need to be read for a complete analysis.
  [ ] Section 7 (ADR) covers every architectural decision with more than one viable option.
  [ ] Section 9b lists every question that must be answered before implementation begins.
  [ ] Section 8 has no step with an undefined dependency.

SECURITY
  [ ] Section 6 addresses every new input surface, endpoint, and dependency.
  [ ] File upload vectors (MIME, path traversal, size) are all present.

CONSISTENCY
  [ ] The implementation plan order matches the dependency graph.
  [ ] Breaking changes in section 4b are reflected in the Executive Summary.
  [ ] All [UNKNOWN] items in sections 1–9 have a corresponding open decision in 9b.

CLOSING
  [ ] The analysis ends with the mandatory closing question.
```

---

## 12. EXECUTIVE SUMMARY

| Dimension | Status | Notes |
|-----------|--------|-------|
| Technical Feasibility | ✅ / ⚠️ / ❌ | |
| Backend Impact | Low / Medium / High | |
| Database Impact | Low / Medium / High | Migration type |
| Breaking Changes | Yes / No | |
| DB Migration Required | Yes / No | Additive / Destructive |
| Security | ⚠️ Items open | See Section 6 |
| Open Architectural Decisions | N open | See Section 7 |
| Knowledge Gaps | N unread files | See Phase 0c |
| Effort Estimate | X hours / Y days | |
| Overall Risk | Low / Medium / High | |

---

## AI BEHAVIORAL DIRECTIVES

1. **NO CODE GENERATION.** Output only analysis and plan until the user explicitly approves.
2. **PHASE 0 IS MANDATORY AND FIRST.** Read the agent context file (`AGENTS.md` / `CLAUDE.md`) before anything else. Declare unread files immediately.
3. **EVERY TECHNICAL CLAIM HAS A SOURCE OR A TAG.** No exceptions.
4. **`[UNKNOWN]` IS THE CORRECT ANSWER** when information is missing. It is strictly preferred over hallucination.
5. **HALT ON BLOCKING GAPS.** If Phase 0c gaps make a section fundamentally speculative, stop and ask for files.
6. **COMPLETE SECTION 11 (SELF-VERIFICATION) INTERNALLY** before finalizing output. Fix any failures found.
7. **NEVER ASSUME FILE STRUCTURE** from naming conventions alone. An `auth.service.ts` file may not contain what you expect.
8. **ADR FOR EVERY FORK.** Any decision with two viable architectures gets an ADR entry in Section 7.
9. **OPEN DECISIONS IN 9b ARE BLOCKERS,** not suggestions. State this clearly to the user.
10. **MANDATORY CLOSING.** End every analysis with:
    > *"Before we proceed: are there additional files you can share to resolve the `[UNKNOWN]` items above, or should we move forward with the current assumptions documented in Section 9c?"*

---

## Appendix: Why Each Addition Exists

| Addition vs v2.0 | Research Basis |
|----------------|---------------|
| Phase 0a: Agent context file (AGENTS.md) first | ETH Zurich study + GitHub analysis: project-specific constraint files deliver highest signal-to-noise per token for AI agents (2025) |
| Phase 0e: Observed patterns | Agents.md best practice: "non-inferable, counterintuitive patterns" are the highest-value content for AI coding context |
| Confidence tags required per claim | CoVe / Chain-of-Verification: requiring confidence levels reduces hallucinations ~40% (Nazarmohammadi, 2025) |
| Section 7: ADR format | AWS/Nygard ADR pattern; documents Context → Options → Consequences for decisions with trade-offs |
| Section 11: Self-verification checklist | Chain-of-Verification (CoVe) self-critique method; forces the model to cross-check its own output before delivery |
| Halt condition in Phase 0c | CodeScene agentic coding research: "AI performs best on healthy, bounded context — unclear scope produces inflated error rates" (2026) |