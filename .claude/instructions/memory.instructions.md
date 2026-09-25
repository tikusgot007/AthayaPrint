# Project Memory Log

> Active Location: `.claude\instructions\memory.instructions.md`
> Last Recorded: 2026-09-26
> This file is managed by the `memory-manager` skill.
> It persists context across AI chat sessions to prevent knowledge loss.
> Do NOT manually edit this file unless necessary.

---

## 🧠 Knowledge Base

> This section accumulates cross-session knowledge that must survive compaction.
> Updated during Compaction Mode (Workflow 4). Do NOT delete entries here.

### Architecture & Patterns

- **Clean Architecture Seams:** Decoupled boundaries enforced (Entities ➔ Use Cases ➔ Adapters ➔ Frameworks).
- **Pólya Heuristic Pipeline:** Spec ➔ Clarify ➔ Plan ➔ Pause Rule ➔ Implement ➔ Review / Bug Fix.
- **Vertical Slicing Mandate:** All tasks organized into DB-to-UI tracer bullets; horizontal slicing prohibited.

### Dead-Ends (Do NOT Repeat)

| # | Attempted | Why It Failed | Correct Solution |
|---|-----------|---------------|------------------|
| 1 | Premature Coding | Bypassing Spec/Plan leads to 90% rework | Enforce Polya Phase 1 & The Pause Rule |
| 2 | Horizontal Slicing | Slicing by technical layer breaks incremental demoability | Formulate end-to-end vertical tracer bullets |
| 3 | Blind Shotgun Patching | Patching without First Principles & reproduction test leads to infinite error loops | Apply Bisection search & Incubation Circuit-Breaker (Hard-Stop after 2-3 fails) |
| 4 | Dogmatic Over-engineering | Forcing 4-layer directories onto single-file scripts / CLI tools creates useless overhead | Apply Scope Proportionality & Clean Code micro-principles (SRP, pure functions) |

### Key Metrics & Baselines

- **Target Test Coverage:** 100% test pass with zero linter suppressions (Floor-Guard).
- **Readiness Score Gate:** Minimum 80/100 to proceed between SDLC phases.
## 📋 Session Checkpoint: 2026-09-26 (Phase 1 → Phase 2 Handoff)

### Project Context
- **Domain:** Toko Batik Athaya — Android App Nota & Label Pengiriman, offline-first, single HP
- **Phase Completed:** Phase 1 (Specification & Clarification)
- **Readiness Score:** 94/100 (Passed audit threshold)

### Artifacts Finalized
- `docs/spec/athayaprint-batik-nota-label-spec.md` v1.1.0 (Clarified Post-Audit)
- `docs/discovery/athayaprint-batik-nota-label-discovery.md` (Canonical, Status: APPROVED)
- `docs/audit/clarification-report-athayaprint-batik-nota-label-2026-09-25.md` (94/100 PASSED)

### Key Decisions Locked (No Further Debate)
1. **Dual Print Path:** Inkjet (USB OTG PDF A6 105×148mm) + Thermal (ESC/POS 80mm roll)
2. **Device Scope:** Single HP, offline, Room local, no cloud/sync
3. **Separation:** Nota & Label terpisah, distinct CRUD + templates
4. **Data Model:** Nota (9 fields) + ShippingLabel (4 fields) + PrintTemplate (custom header/footer/logo)
5. **Currency/Locale:** IDR (cents), Indonesia-only, INV-YYYY-MM-NNNN format
6. **Printers Supported:** Epson L-300 USB OTG (default), generic Thermal 80mm
7. **Label Spec:** NO barcode/QR code (user locked)

### Resolved Assumptions
- ✅ ASSUMPTION-002: PrinterDriver abstraction (generic Inkjet + Thermal)
- ⏳ ASSUMPTION-001: Spike USB OTG L-300 PrintedPdfDocument (Scheduled 2026-09-26, Risk Accepted, Fallback custom PDF→ESC/POS)
- 🔒 ASSUMPTION-003: A6 105×148mm Nota only, 80mm Label roll (User locked)

### Requirements Snapshot
| # | Category | Count | Status |
|---|----------|-------|--------|
| REQ | Functional | 10 | LOCKED v1.1.0 |
| CON | Non-Functional | 4 | LOCKED v1.1.0 |
| AC | Acceptance Criteria | 8 | LOCKED v1.1.0 |

### Next Phase Entry Conditions (Phase 2: Planning)
- [ ] Approval: User confirms `/polya-plan` handoff (Pause Rule)
- [ ] Spike Completion: USB OTG L-300 test 2026-09-26 (parallel to planning, non-blocking fallback)
- [ ] New Session: Recommended context hygiene before Phase 2 kickoff

---

## 📝 Session Checkpoint: 2026-09-26 (Phase 2 → Phase 3 Handoff)

- **Active Memory Path:** `.claude/instructions/memory.instructions.md`
- **Current SDLC Phase:** Phase 2 (Planning) — ✅ Completed. Next: Phase 3 (Implementation).
- **Active Artifacts:**
  - `docs/discovery/athayaprint-batik-nota-label-discovery.md` — Status: ✅ Approved (Canonical)
  - `docs/spec/athayaprint-batik-nota-label-spec.md` — Status: ✅ v1.1.0 Clarified (94/100). **Pending amendment to v1.2.0 (RISK-001)**
  - `docs/audit/clarification-report-athayaprint-batik-nota-label-2026-09-25.md` — Status: ✅ Finalized (94/100)
  - `docs/plan/athayaprint-batik-nota-label-plan.md` — Status: ✅ Finalized (Readiness Score: 87/100)
  - `docs/reviews/{slug}-review.md` — Status: ⏳ Pending
- **Achieved Milestones:**
  - Authored the Phase 2 Implementation Plan: **17 functional tracer bullets + 5 phase gates** (VERIFY + APPROVAL per phase) across Phase 0 (Alignment) through Phase 4 (Hardening). Zero task exceeds size M.
  - Pre-Flight audit uncovered and resolved **2 internal Spec contradictions** before any code was written.
  - Established that the project root was **not** a git repository (RISK-004). **Resolved during this session:** the repository was initialized on branch `main` (baseline commit `897b4bb`, 38 files tracked) and published to `https://github.com/tikusgot007/AthayaPrint` as a **public** repository with `origin` tracking `main`. TASK-002 therefore no longer needs to perform `git init`.
- **Dead-Ends (Do NOT Repeat):** No new dead-ends this session. See Knowledge Base rows #1–#4 (Premature Coding, Horizontal Slicing, Blind Shotgun Patching, Dogmatic Over-engineering) — all remain valid.
- **Updated Files:**
  - `docs/plan/athayaprint-batik-nota-label-plan.md` — Created (v1.0.0, status: Planned)
  - `.claude/instructions/memory.instructions.md` — Checkpoint appended
  - `AGENTS.md` — Stale `.agents/` path references corrected to `.claude/`
  - `.gitignore` — Created; isolates the `awesome-copilot-id/` toolkit checkout (a separate git repository) plus Android/Gradle build output, signing material, IDE cruft, and secrets
  - `.gitattributes` — Created; normalizes line endings (LF in the repository, CRLF for Windows scripts, binary assets excluded from diffing)
- **Decisions Made:**
  - **Nota paper size = A6 (105x148mm).** Rejects the "A4" wording found in AC-003 and Spec §7. `PaperSize` enum exposes exactly `A6_105X148MM` and `ROLL_80MM`.
  - **No INV format = 4-digit `INV-YYYY-MM-NNNN`.** Rejects the 3-digit `INV-YYYY-MM-NNN` pattern found in Spec §4.
  - **`git init` was executed immediately (2026-09-26)** rather than deferred to TASK-002, because a rollback strategy is fictional without a VCS baseline. Branch `main`, baseline commit `897b4bb`.
  - **The Plan is the source of truth for paper size and invoice format until the Spec is amended to v1.2.0.**
  - Print Engine execution order is deliberate: `PrinterDriver` port + `MockPrinterDriver` (TASK-016) land **before** both real drivers, so use-case orchestration is provable without hardware.
- **Next Action / Pending:**
  - Start Phase 3 in a **new chat session**: `/polya-code @docs/plan/athayaprint-batik-nota-label-plan.md`, beginning at TASK-001.
  - **ASSUMPTION-001 spike (Epson L-300 + USB OTG + `PrintedPdfDocument`) must complete before TASK-018 begins.** Contingency A (raster bitmap → ESC/POS) is pre-planned with an explicit trigger.
  - **RISK-001:** amend Spec to v1.2.0 (A6 + 4-digit INV) so two sources of truth do not coexist.
  - **RISK-002:** TASK-001 must author `CONTEXT.md` (de-duplicated from Spec §2) plus `docs/adr/0001`–`0003`, all of which the Spec cites but which do not yet exist on disk.
  - **RISK-005:** Bluetooth thermal transport has no acceptance criterion; intentionally deferred rather than silently added.

<!-- checkpoint-tail: Phase 2 plan complete at 87/100 (17 tracer bullets + 5 gates); next session runs /polya-code from TASK-001; ASSUMPTION-001 spike gates TASK-018; Spec needs amendment to v1.2.0 for A6 + 4-digit INV. -->

---
