---
goal: "AthayaPrint Android — Offline Nota & Shipping Label Printing (Dual Printer Path)"
version: "1.0.0"
date_created: "2026-09-26"
last_updated: "2026-09-26"
status: "Planned"
spec_ref: "docs/spec/athayaprint-batik-nota-label-spec.md"
tags: ["polya", "plan", "tracer-bullets", "vertical-slicing", "clean-architecture", "android"]
---
<!-- markdownlint-disable -->

# Implementation Plan: AthayaPrint Batik Nota & Label Printing (Android)

> [!IMPORTANT]
> **THE PAUSE RULE & VERTICAL SLICING MANDATE:**
> All tasks in this plan are organized into **Vertical Feature Slices (Tracer Bullets)** spanning from Domain to UI. Horizontal layer-by-layer slicing is strictly prohibited.
> After each phase is verified, execution MUST halt for explicit user approval before proceeding to the next phase. Functional code MUST NOT be written until this plan is approved.

---

## 1. Overview & High-Level Mental Model

AthayaPrint is a **greenfield offline-first Android application** (zero source files exist today) that lets a single shop phone print:

1. **Nota Penjualan** on A6 (105x148mm) paper via an Epson L-300 inkjet over USB OTG.
2. **Label Pengiriman** on 80mm thermal roll via ESC/POS raw byte commands.

The architectural spine is a **single dependency-inversion seam**: `PrinterDriver` is declared in the Domain/Use-Case layer, and two adapters (`InkjetPdfDriver`, `ThermalEscPosDriver`) implement it in the data layer. This is the load-bearing decision ([ADR-0001](docs/adr/0001-dual-printer-abstraction.md)) — it is what prevents printer-specific branching from leaking into ViewModels.

**Execution strategy:** *Land and Expand*. Phase 0 builds a walking skeleton that breathes (builds, installs, runs on device). Phase 1 lands the first complete vertical slice (Nota creation, end to end, with zero mocks). Phases 2–4 expand outward into Label, History, Templates, the Print Engine, and finally hardening.

### Topological Data Flow Diagram

```text
[COMPOSE UI]
  NotaEditorScreen / LabelEditorScreen / HistoryScreen / PrintPreviewScreen
        |
        v
[VIEWMODEL]  (Interface Adapter — UI state holder, no printer knowledge)
  NotaViewModel / LabelViewModel / HistoryViewModel / PrintViewModel
        |
        v
[USE CASE]   (Application Layer — orchestration, one reason to change)
  CreateNotaUseCase / UpdateNotaUseCase / PrintDocumentUseCase / ManageTemplateUseCase
        |
        +----------------------------+-----------------------------+
        v                            v                             v
[DOMAIN PORT]                [DOMAIN PORT]                 [DOMAIN PORT]
NotaRepository               LabelRepository               PrinterDriver   <-- ADR-0001 seam
        |                            |                             |
        v                            v                             v
[ADAPTER: Room]              [ADAPTER: Room]              [ADAPTER: Printer]
NotaRepositoryImpl           LabelRepositoryImpl          InkjetPdfDriver  (PrintedPdfDocument -> PDF)
                                                          ThermalEscPosDriver (raw ESC/POS bytes)
        |                            |                             |
        v                            v                             v
[DATA SOURCE]                [DATA SOURCE]                 [DEVICE]
Room DB  notas /             Room DB  shipping_labels /     Android Print Framework
         print_templates              print_templates      USB Host API (OTG)
```

**Dependency Rule compliance:** every arrow points inward toward policy. The `PrinterDriver` interface lives with the Use Cases; the USB byte-poking lives in `data/printer/`. Room annotations never appear in `domain/`.

---

## 2. Task Sizing Matrix

Every task must fit into one focused session. AI agents work most reliably on XS to M tasks.

| Size   | Files Impacted | Scope                                           | Example                                                 |
| :----- | :------------: | :---------------------------------------------- | :------------------------------------------------------ |
| **XS** |       1        | Single pure function or config change           | Domain invariant rule / DTO definition                  |
| **S**  |      1-2        | Small focused vertical interaction              | Endpoint mapping + DTO validation                       |
| **M**  |      3-5        | Standard Vertical Feature Slice (Tracer Bullet) | Core user feature: Schema + Domain + UseCase + API + UI |
| **L**  |      5-8        | Multi-component vertical flow                   | Full checkout pipeline with payment gateway             |
| **XL** |       8+       | **Strictly Prohibited (Too Large)**             | Must be decomposed into smaller vertical slices         |

**Plan inventory:** 17 functional tracer bullets + 5 phase gates (each gate = 1 VERIFY + 1 APPROVAL task). Zero tasks exceed size M.

---

## 3. Phased Implementation Sequence (Vertical Tracer Bullets)

> **EXECUTION DIRECTIVE FOR AI AGENTS:**
> Execute phase by phase. Run the specific testing/verification task at the end of each phase. After verification passes, **YOU MUST STOP AND WAIT** for user approval before moving to the next phase.

### Implementation Phase 0: Alignment — Walking Skeleton

*Goal: Establish the documentation baseline that the Spec already presumes, and a skeleton that compiles, installs, and runs on the physical device.*

| Task | Description | Ref ID | AC Ref | Dep | Files | Completed | Date |
| :--- | :--- | :--- | :--- | :--- | :---: | :---: | :---: |
| TASK-001 | Author canonical `CONTEXT.md` (de-duplicated from Spec Section 2) plus `docs/adr/0001-dual-printer-abstraction.md`, `0002-offline-first-single-device.md`, `0003-type-driven-domain.md` | - | - | - | 1 (XS) | [ ] | |
| TASK-002 | Walking Skeleton: Gradle project, Hilt, Room v1, Compose shell, module folders (`domain/`, `usecases/`, `data/local/`, `data/printer/`, `presentation/`), one placeholder screen that launches on device. (`git init` + `.gitignore` already landed 2026-09-26 — do not repeat.) | CON-004 | - | TASK-001 | 5-6 (L) | [ ] | |
| TASK-003 | **VERIFY:** `./gradlew assembleDebug testDebugUnitTest` passes with zero failures; APK installs and launches on Android 8+ physical device | - | - | TASK-002 | - | [ ] | |
| TASK-004 | **APPROVAL:** Stop and wait for explicit user confirmation to proceed to Phase 1 | - | - | - | - | [ ] | |

### Implementation Phase 1: Land — Nota Vertical Slice (End-to-End)

*Goal: The first complete vertical slice. A user can open the app, enter a Nota, see it calculated correctly, and have it persist across an app kill. Zero mocks in this slice.*

| Task | Description | Ref ID | AC Ref | Dep | Files | Completed | Date |
| :--- | :--- | :--- | :--- | :--- | :---: | :---: | :---: |
| TASK-005 | Nota Domain Core Slice: `MoneyInCents`, `PositiveInteger`, `InvoiceNumber` (pattern `INV-YYYY-MM-NNNN`), `PhoneNumber` value objects; `Nota` + `NotaItem` entities; `GenerateInvoiceNumberUseCase` (monthly counter, 4-digit). Unit tests for every invariant. | REQ-001 | AC-001 | TASK-002 | 5 (M) | [ ] | |
| TASK-006 | Nota Persistence Slice: `NotaEntity`, `NotaDao`, `NotaMapper` (bidirectional), `NotaRepositoryImpl`, `NotaRepository` port in domain, `CreateNotaUseCase` orchestrating invoice-number generation + calculation + insert. Instrumented test proves Room round-trip. | REQ-003 | AC-005 | TASK-005 | 5 (M) | [ ] | |
| TASK-007 | Nota Editor UI Slice: `NotaEditorScreen` (customer name, date, item rows with add/remove), live SUBTOTAL/DISKON/TOTAL/TUNAI/KEMBALI display, Save action, `NotaViewModel` exposing `StateFlow`. | REQ-001 | AC-001 | TASK-006 | 4 (M) | [ ] | |
| TASK-008 | **VERIFY:** End-to-end test of AC-001 — input Budi, item "Batik Tulis" qty=2 @150000, DISKON=5000, TUNAI=300000, assert SUBTOTAL=300000 / TOTAL=295000 / KEMBALI=5000 / No INV matches `INV-2026-09-0001`. Kill app, relaunch, assert record persists. | - | AC-001 | TASK-007 | - | [ ] | |
| TASK-009 | **APPROVAL:** Stop and wait for explicit user confirmation to proceed to Phase 2 | - | - | - | - | [ ] | |

### Implementation Phase 2: Expand — Label, History & Templates

*Goal: Reach feature parity for the second document type, then make stored data reusable (history, edit, reprint) and printable output customizable.*

| Task | Description | Ref ID | AC Ref | Dep | Files | Completed | Date |
| :--- | :--- | :--- | :--- | :--- | :---: | :---: | :---: |
| TASK-010 | Label Vertical Slice: `ShippingLabel` entity + `ShippingLabelEntity`/DAO/mapper/`LabelRepositoryImpl` + `CreateLabelUseCase` + `LabelEditorScreen` with PENERIMA and PENGIRIM blocks. PENGIRIM prefilled from defaults. End-to-end, no barcode/QR. | REQ-002 | AC-002 | TASK-006 | 5 (M) | [ ] | |
| TASK-011 | History Vertical Slice: shared `HistoryScreen` listing Nota and Label with date-range filter, backed by `GetNotaHistoryUseCase` / `GetLabelHistoryUseCase` and `Flow`-based DAO queries. Tapping a row opens the detail. | REQ-009 | - | TASK-010 | 4 (M) | [ ] | |
| TASK-012 | Reprint & Edit Vertical Slice: `UpdateNotaUseCase` / `UpdateLabelUseCase` plus a reprint entry point that rehydrates a stored document into the print pipeline without re-entering data. Includes edit-persistence assertion. | REQ-003 | AC-005 | TASK-011 | 2 (S) | [ ] | |
| TASK-013 | Template Customization Vertical Slice: `PrintTemplate` domain model + Room persistence + `ManageTemplateUseCase` + `TemplateSettingsScreen` allowing header text, logo picker (SAF `content://` URI), footer text, and `PaperSize` selection constrained to `A6_105X148MM` (Nota) / `ROLL_80MM` (Label). | REQ-006 | AC-008 | TASK-012 | 5 (M) | [ ] | |
| TASK-014 | **VERIFY:** Run full suite; manually exercise AC-002 and AC-008 on device; assert date filter returns correct subset for a fixed fixture set spanning two months | - | AC-002, AC-008 | TASK-013 | - | [ ] | |
| TASK-015 | **APPROVAL:** Stop and wait for explicit user confirmation to proceed to Phase 3 | - | - | - | - | [ ] | |

### Implementation Phase 3: Print Engine

*Goal: Build the highest-blast-radius subsystem. The order is deliberate — the abstract port and its in-memory mock are proven first (Pólya's auxiliary problem), so that domain orchestration is verified before any USB byte is written.*

| Task | Description | Ref ID | AC Ref | Dep | Files | Completed | Date |
| :--- | :--- | :--- | :--- | :--- | :---: | :---: | :---: |
| TASK-016 | Printer Port & Mock Slice: `PrinterDriver` interface in the use-case layer (`suspend fun print(request: PrintRequest): PrintResponse`, `suspend fun isAvailable(): Boolean`), `MockPrinterDriver` capturing emitted payloads for assertion, `PrintDocumentUseCase` orchestrating document lookup then driver dispatch. | REQ-008 | - | TASK-013 | 3 (S) | [ ] | |
| TASK-017 | Thermal ESC/POS Driver Slice: `ThermalEscPosDriver` emitting initialize / text / line-feed / cut byte sequences for an 80mm roll, with `USB_TIMEOUT_MS = 5000`. Driver-level tests assert exact byte output against a golden fixture. | REQ-005 | AC-004 | TASK-016 | 4 (M) | [ ] | |
| TASK-018 | Inkjet PDF Driver Slice: `InkjetPdfDriver` rendering the Nota into an A6 (105x148mm) `PrintedPdfDocument` and handing it to the Android Print Framework. **High Risk — gated by ASSUMPTION-001 spike outcome.** | REQ-004 | AC-003 | TASK-016 | 4 (M) | [ ] | |
| TASK-019 | Print Preview & Selection UI Slice: `PrintPreviewScreen` rendering an on-screen facsimile of the A6 Nota / 80mm Label, plus a printer-type selector (INKJET_PDF vs THERMAL_ESC_POS) that gates the Print button. | REQ-008 | - | TASK-018 | 4 (M) | [ ] | |
| TASK-020 | Print Status Feedback Slice: surfaced success / error / `Printer not connected` states from `PrintResponse`, with the Print button disabled when no driver reports availability. | REQ-010 | AC-007 | TASK-019 | 2 (S) | [ ] | |
| TASK-021 | **VERIFY:** Execute AC-004 (thermal golden-bytes test) and AC-007 (disconnected-printer negative test). Physical-device print on both paths where hardware permits; otherwise record the hardware gap explicitly. | - | AC-004, AC-007 | TASK-020 | - | [ ] | |
| TASK-022 | **APPROVAL:** Stop and wait for explicit user confirmation to proceed to Phase 4 | - | - | - | - | [ ] | |

### Implementation Phase 4: Hardening — Invariants, Resilience & Budgets

*Goal: Close the negative-hypothesis gaps from Spec Section 1.5 and prove the non-functional budgets are actually met rather than assumed.*

| Task | Description | Ref ID | AC Ref | Dep | Files | Completed | Date |
| :--- | :--- | :--- | :--- | :--- | :---: | :---: | :---: |
| TASK-023 | Negative Invariant Slice: enforce `TUNAI >= TOTAL`, `qty > 0`, `harga > 0` at the domain boundary with typed errors; UI displays `Uang tunai kurang Rp X` and blocks save. Property-based tests over empty/negative/overflow inputs. | CON-002 | AC-006 | TASK-007 | 2 (S) | [ ] | |
| TASK-024 | Resilience Slice: USB disconnect mid-transmission detected via exception/timeout with transaction-state rollback and retry affordance; Room write failure surfaces `Storage error` and writes a JSON fallback snapshot; automatic 3x retry with exponential backoff for USB/ESC/POS errors. | CON-002 | - | TASK-020 | 4 (M) | [ ] | |
| TASK-025 | Non-Functional Budget Verification Slice: measure and record save-to-Room < 200ms, PDF generation < 1s, ESC/POS transmission < 500ms, app size < 15MB, memory < 100MB. Any budget exceeded becomes a documented finding, not a silent pass. | CON-001, CON-003 | - | TASK-024 | 1 (S) | [ ] | |
| TASK-026 | **VERIFY:** Full regression suite green with zero suppressions; all CON budgets measured and recorded; `docs/ARCHITECTURE.md` regenerated via `/polya-map` to reflect as-built topography | - | - | TASK-025 | - | [ ] | |
| TASK-027 | **APPROVAL:** Stop and wait for explicit user confirmation to declare the plan complete | - | - | - | - | [ ] | |

---

## 4. Contingency Plan B (Have Two Strings to Your Bow)

*Pólya Heuristic: Prepare for potential failure of the primary strategy.*

### Contingency A — Inkjet Path (ASSUMPTION-001)

- **Risk / Failure Point:** The Epson L-300 does not expose itself as a printable target through the Android Print Framework, making `PrintedPdfDocument` dispatch impossible.
- **Primary Strategy (Plan A):** Generate an A6 PDF via `PrintedPdfDocument` and hand it to `PrintManager` with the L-300 selected as the destination.
- **Contingency Strategy (Plan B):** Render the Nota template to a raster bitmap, wrap it in an ESC/POS `GS v 0` raster command block, and transmit it over the same USB bulk-transfer channel used by `ThermalEscPosDriver`.
- **Trigger Condition:** Pivot when the 2026-09-26 device spike shows the L-300 absent from the Android print destination list, or when a print job submitted through `PrintManager` returns a terminal failure state.
- **Blast Radius:** Contained to `InkjetPdfDriver` plus one new raster-encoder helper. `PrintDocumentUseCase`, ViewModels, and UI remain untouched because they depend on the `PrinterDriver` port, not the adapter.

### Contingency B — Thermal Vendor Variance (ASSUMPTION-002)

- **Risk / Failure Point:** The actual thermal unit uses a proprietary command dialect that diverges from standard ESC/POS.
- **Primary Strategy (Plan A):** Standard ESC/POS initialize / text / cut sequences.
- **Contingency Strategy (Plan B):** Introduce a vendor-specific `ThermalEscPosDriver` variant behind the same port and select it via configuration at composition root.
- **Trigger Condition:** Golden-fixture byte tests pass but physical output is garbled or inert.

### Contingency C — Room Corruption or Storage Exhaustion

- **Risk / Failure Point:** Room write fails (negative hypothesis 3), risking loss of an in-progress Nota.
- **Primary Strategy (Plan A):** Transactional Room insert with typed error surfaced to UI.
- **Contingency Strategy (Plan B):** Serialize the in-flight document to a JSON snapshot in app-private storage, surface `Storage error`, and offer recovery on next launch.
- **Trigger Condition:** Room write throws on three consecutive attempts within a single session.

---

## 5. Symmetry, Invariants & Problem Variation (Pólya, p. 199–214)

### Symmetric Operations & Round-Trip Invertibility ($f^{-1}(f(x)) = x$)

| Forward | Inverse | Invariant to Assert |
| :--- | :--- | :--- |
| `Nota.toEntity()` | `NotaEntity.toDomain()` | Round-trip equality for every field, including the `itemsJson` collection. |
| `ShippingLabel.toEntity()` | `ShippingLabelEntity.toDomain()` | Round-trip equality including nullable `notes`. |
| `PrintTemplate.serialize()` | `PrintTemplate.parse()` | `layoutConfigJson` survives an empty-object and a fully-populated case. |
| `acquireUsbInterface()` | `releaseUsbInterface()` | Interface is released on every exit path: success, exception, and cancellation. |
| `openPrintJob()` | `closePrintJob()` | No dangling job remains after a failed transmission. |
| `GenerateInvoiceNumberUseCase` | counter reset at month boundary | Sequence `0001..9999` resets when `YYYY-MM` changes; uniqueness holds within a month. |

### Variation of the Problem (Property-Based & Fuzz Exploration)

- **Scale Variation:** Nota with 1 item, 50 items, and 100+ items (Spec Section 7 extreme case) — assert layout and calculation stay correct as the collection grows.
- **Empty States:** Nota with zero items; Label with empty `notes`; History with zero records and with records spanning exactly one month boundary.
- **Negative Bounds:** `qty = 0`, `qty = -1`, `harga = 0`, `TUNAI < TOTAL`, `DISKON > SUBTOTAL`.
- **Overflow Bounds:** `MoneyInCents` at `Long.MAX_VALUE / 2` for a single item multiplied by a large quantity — confirm arithmetic does not wrap silently.
- **Unicode & Length:** Recipient names and addresses using accented characters and multi-line addresses exceeding typical roll width, exercising word wrap.
- **Random Permutations:** Shuffled item insertion order must not change `SUBTOTAL`, `TOTAL`, or `KEMBALI`.

### SOLID Checks at Blueprint Stage

- **Single Responsibility (SRP):** `PrintDocumentUseCase` orchestrates and nothing else — it does not know what a USB bulk transfer is. `ThermalEscPosDriver` encodes bytes and nothing else — it does not know what a Nota is.
- **Open/Closed (OCP):** Adding a future Bluetooth thermal printer means adding one adapter, not editing any use case.
- **Dependency Inversion (DIP):** `PrintDocumentUseCase` depends on the `PrinterDriver` abstraction. No use case imports an Android USB class.

---

## 6. Risks & Extracted Assumptions

*Extracted from the upstream Spec's `[ASSUMPTION]` tags.*

| ID | Risk / Assumption | Status | Affected Tasks | Mitigation |
| :--- | :--- | :--- | :--- | :--- |
| ASSUMPTION-001 | Epson L-300 can print via the Android Print Framework using `PrintedPdfDocument` without a custom driver. | Pending spike (2026-09-26) | TASK-018, TASK-019, TASK-021 | Run the physical device spike before TASK-018 begins. If it fails, execute Contingency A without touching the port. TASK-018 is flagged **High Risk**. |
| ASSUMPTION-002 | Thermal 80mm printer conforms to standard ESC/POS. | Resolved — abstracted behind `PrinterDriver` | TASK-017 | Golden byte-fixture tests. Vendor divergence handled by Contingency B. |
| ASSUMPTION-003 | Nota paper is A6 (105x148mm); Label is 80mm roll. | Locked by user (2026-09-26) | TASK-005, TASK-013, TASK-018, TASK-019 | `PaperSize` enum exposes exactly `A6_105X148MM` and `ROLL_80MM`. No A4/LETTER path is built. |
| RISK-001 | Spec v1.1.0 contains internal contradictions: AC-003 and Section 7 say "A4" while Sections 1.1 and 4.2 imply A6 but omit it from the enum; Section 4 uses a 3-digit `INV-YYYY-MM-NNN` while Section 2 and AC-001 use `INV-2026-09-001`. | Resolved by user decision (2026-09-26): **A6** and **4-digit `NNNN`** | TASK-005, TASK-018 | Spec must be amended to v1.2.0 so it stops contradicting this plan. Until amended, this plan is the source of truth for paper size and invoice format. |
| RISK-002 | Spec Section 8 links `docs/adr/0001`–`0003`, but none exist on disk. `CONTEXT.md` also does not exist, and Spec Section 2 defines the glossary twice with conflicting `_Avoid_` lists. | Open | TASK-001 | TASK-001 authors all four documents before any code is written. |
| RISK-003 | USB OTG host-mode behaviour varies across Android OEMs and API levels; a path proven on one device may fail on another. | Open | TASK-002, TASK-016, TASK-018, TASK-024 | Target the actual shop device as the sole acceptance device (single-HP constraint makes this tractable). Record the exact device model and API level in the verification notes. |
| RISK-004 | Project root `D:\AthayaPrint` was **not** a git repository, so atomic commits and `git bisect` were unavailable. | ✅ Resolved 2026-09-26 | TASK-002 | Repository initialized on branch `main` with baseline commit `897b4bb`. `.gitignore` isolates the `awesome-copilot-id/` toolkit checkout (a separate git repository) plus Gradle output, signing material, IDE cruft, and secrets. TASK-002 no longer performs `git init`. |
| RISK-005 | Bluetooth thermal printing is mentioned in REQ-005 but no transport-specific acceptance criterion exists for it. | Open | TASK-017 | Scope TASK-017 to USB transport first. Bluetooth transport is a follow-on slice behind the same port, scheduled only if the shop hardware requires it. |
| RISK-006 | `App size < 15MB` (CON-003) is aggressive for a Compose + Hilt + Room application. | Open | TASK-025 | Measure at TASK-025. If exceeded, record it as a finding with the R8/ProGuard and resource-shrinking options rather than silently accepting it. |

---

## 7. Rollback & Recovery Strategy

Step-by-step instructions to revert to a stable state if execution encounters unrecoverable issues:

1. **Prerequisite:** ✅ Already met. The repository was initialized on 2026-09-26 (branch `main`, baseline commit `897b4bb`). Every tracer bullet lands as its own commit, so any slice can be reverted independently.
2. **Revert a single tracer bullet:** `git revert <commit-sha>` for the offending slice. Because each slice is atomic and leaves the suite green, the revert restores a known-good build.
3. **Revert an entire phase:** `git revert --no-commit <first-sha>^..<last-sha>` then commit as a single phase rollback.
4. **Roll back the Room schema:** Room migrations are additive and versioned. Reverting a schema change means reverting the migration class together with its `@Database(version = N)` bump in the same commit. If a destructive migration was applied during development, uninstall and reinstall the debug build to reset the local database — acceptable only on the single development device, never on shop data.
5. **Recover from a dead-end implementation:** If 2–3 consecutive fix attempts fail, invoke the Incubation & Circuit-Breaker Rule — halt mutation, author a Contradiction Report, and return to Phase 1 rather than continuing to patch.
6. **Restore documentation state:** `docs/plan/athayaprint-batik-nota-label-plan.md` is the plan of record. Any deviation discovered mid-execution must be reflected back into this file or the Spec before the code is merged.

---

## 8. Pre-Flight Self-Correction Checklist (Anti-Patterns)

Before presenting this plan to the user, verify it is free from these anti-patterns:

- [x] **No Horizontal Slicing:** No task is described as "create all tables" or "build all APIs". Every functional task cuts Domain to UI (e.g., TASK-007 delivers entity, DAO, use case, ViewModel, and screen together).
- [x] **No Bloated Tasks (XL):** The largest tasks are size L (TASK-002, 5–6 files). None reaches the 8-file XL threshold.
- [x] **Strict Traceability:** Every functional task carries a valid `Ref ID` (`REQ-xxx` / `CON-xxx`) matching the Spec. Tasks marked `-` are process gates, not functional work.
- [x] **Deterministic Dependencies:** Every `Dep` cell points to a previously scheduled task, ordered bottom-up from domain foundation to UI.
- [x] **Mandatory Verification & Approval:** All five phases end with an explicit `VERIFY` task and an `APPROVAL` halt gate.
- [x] **No Requirement Drift:** No task introduces a capability absent from the Spec. Bluetooth transport (RISK-005) is explicitly deferred rather than silently added.
- [x] **Assumptions Extracted:** All three `[ASSUMPTION]` tags plus the Spec's internal contradictions are mapped to concrete mitigations in Section 6.
