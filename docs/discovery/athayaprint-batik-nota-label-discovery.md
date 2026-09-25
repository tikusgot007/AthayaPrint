# 🔭 Project Discovery Draft: Athaya Batik Nota & Label Printer

**Document Type:** Phase 0 Discovery Draft
**Target Repository:** `D:\AthayaPrint` — Greenfield Android App
**Status:** **Draft / Under Review**
**Architect:** Veteran Principal Fullstack Engineer

---

## 1. Problem Statement & Business Opportunity (Getting Acquainted)

*Pólya Heuristic: Familiarity and Conception (Pólya, 1945, p. 33).*

- **The Problem:** Toko Batik Athaya needs to print Nota (sales receipt) and Label Pengiriman (shipping label) without a computer. Current flow requires PC / manual handwriting, slow and error-prone for a small retail store.
- **Business Value & Why Now:** Direct mobile printing from single HP reduces operational friction, enables reprint from local history without retyping, supports existing asset (Epson L-300 Inkjet via USB OTG) while leaving upgrade path to Thermal 80mm. No backend / no internet dependency keeps cost near zero.
- **Target Persona & Users:** Store owner / cashier at Toko Batik Athaya, single device, non-technical user, offline daily operation.

---

## 2. The Unknown, Known Data, and Operational Bounds

*Pólya Heuristic: Isolate the principal parts — Unknown, Data, Condition.*

| Element | Description |
| :--- | :--- |
| **The Unknown** | Offline-first Android app that creates, stores, edits, and prints Nota and Label Pengiriman separately via dual printer path (Inkjet USB + Thermal ESC/POS option), with customizable header/logo/layout/footer. |
| **Known Data** | Greenfield repo (no app source yet). Existing printer: Epson L-300 Inkjet USB. Requirements locked: Nota = Nama, Tanggal, item list, SUBTOTAL/DISKON/TOTAL/TUNAI/KEMBALI, No INV, header ATHAYA BATIK; Label = nama, no telp, alamat, keterangan, TANPA barcode/QR, plus PENGIRIM block; Input manual, stored locally, editable history + reprint; Cetak terpisah; Kustomisasi template; Single HP only. |
| **The Conditions & Invariants** | Offline-first, zero network dependency; Single device, no multi-HP sync; USB OTG must work reliably; No data loss (Room persistence); Preview before print; No computer in loop; Kustomisasi must not break print layout. |

---

## 3. Codebase Exploration & Architectural Critique

*Pólya Heuristic: Decomposing and Recombining (p. 75) & Analogy (p. 37).*

### 3.1 Existing Topography Analysis

> **Greenfield Projects:** Repository currently contains only `.claude/`, `awesome-copilot-id/`, `AGENTS.md`, `docs/discovery/athayaprint-android-discovery.md` (obsolete printing-services scope). No Android source. This section proposes initial Clean Architecture scaffold.

- **Relevant Directories & Modules (proposed):**
  - `app/src/main/java/com/athaya/print/domain/model/`: `Nota`, `NotaItem`, `ShippingLabel`, `PrintTemplate` — pure Kotlin entities, no Android dependency.
  - `app/src/main/java/com/athaya/print/domain/repository/`: `NotaRepository`, `LabelRepository`, `TemplateRepository` interfaces (Ports).
  - `app/src/main/java/com/athaya/print/domain/usecase/`: `CreateNota`, `UpdateNota`, `ReprintNota`, `CreateLabel`, etc.
  - `app/src/main/java/com/athaya/print/data/local/`: Room `AppDatabase`, `NotaEntity`, `LabelEntity`, DAO, mappers Entity<->Domain.
  - `app/src/main/java/com/athaya/print/data/printer/`: `PrinterDriver` interface + `InkjetPdfDriver` (Android Print Framework / PDF rendering for Epson L-300 USB OTG) + `ThermalEscPosDriver` (ESC/POS USB/Bluetooth for 80mm option).
  - `app/src/main/java/com/athaya/print/presentation/`: Jetpack Compose screens `NotaEditor`, `LabelEditor`, `HistoryList`, `TemplateSettings`, `PrintPreview`; MVVM with StateFlow UDF.
  - `app/src/main/java/com/athaya/print/di/`: Hilt modules.
- **Architectural Debt & Seam Vulnerabilities:**
  - None yet (greenfield). Risk to avoid: leaking Android `Context` / printer SDK types into domain; hardcoding layout in Composables without template entity.

### 3.2 Analogous Solutions & Prior Art
- **Internal Analogies:** None in repo. Reusable pattern from Polya kit: Repository + UDF StateFlow established in prior Android topography notes.
- **External Analogies:** Standard Indonesian retail nota (header, INV number, item table, tunai/kembali, footer terima kasih); Ekspedisi label (PENGIRIM/PENERIMA/KETERANGAN blocks, no barcode per user decision); Android Print Framework for inkjet PDF printing; ESC/POS command set for thermal 80mm (e.g., RawBT, Printooth apps use same seam).

---

## 4. Candidate Architectures & Trade-Off Matrix

*Pólya Heuristic: The Inventor's Paradox (p. 121).*

| Dimension | Option A: Minimal / Direct | Option B: Target / Architectural | Option C: Comprehensive / Scalable |
| :--- | :--- | :--- | :--- |
| **Summary** | Single-module app, direct Room + direct print calls from UI, one PDF layout for both inkjet/thermal | Clean Architecture (Domain/Data/Presentation), `PrinterDriver` port with 2 adapters (InkjetPdf + ThermalEscPos), template entity for kustomisasi | Option B + cloud backup/sync, multi-HP, remote config, analytics |
| **Clean Architecture Seams** | Coupled UI->DB->printer, hard to add thermal later | Decoupled via Ports & Adapters, DTOs across boundaries, testable use cases | Isolated bounded contexts but overkill for single HP |
| **Complexity & Risk** | Low initial cost, layout breaks when thermal added, untestable | Moderate effort, stable long-term, fits dual-printer requirement | High overhead, needs backend, violates single-HP offline constraint |
| **Reversibility (Two-Way Door?)** | Hard to untangle print logic later | Easy to add new printer behind interface | High commitment, irreversible infra |
| **Recommendation** | [Alternative / Spike only] | **[RECOMMENDED]** | [Future Phase, out of scope] |

---

## 5. Technical Feasibility Spikes & Open Risks

*Pólya Heuristic: Auxiliary Problem (p. 50).*

- **Technical Unknown 1:** Can Android phone print to Epson L-300 Inkjet via USB OTG reliably without PC?
  - **Spike / Feasibility Check:** Minimal spike: generate PDF via `PrintedPdfDocument`, send through Android Print Framework + USB OTG (via Epson Print Enabler / Mopria / direct USB host). Tag `// SPIKE: prototype code`.
  - **Findings:** To be verified on device. Fallback: export/share PDF if driver missing.
- **Technical Unknown 2:** ESC/POS thermal 80mm path — USB vs Bluetooth command compatibility?
  - **Spike / Feasibility Check:** Send minimal ESC/POS byte sequence (init, text, cut) to 80mm emulator / borrowed device.
  - **Findings:** To be verified. Abstraction ensures inkjet path still ships even if thermal delayed.
- **Technical Unknown 3:** Nota/Label layout fitting both A4/Struk (inkjet) and 80mm (thermal)?
  - **Spike / Feasibility Check:** Paper-size parameterized template composable + PDF pagination test.
  - **Findings:** Template entity must store paper size, font scale, logo URI, footer text.
- **Open Risks:** USB OTG cable/permission UX; Epson L-300 ink cost vs thermal; logo image storage size in Room; no backup (single HP loss = data loss — document as accepted constraint).

---

## 6. Strategic Recommendation & Handoff to Specification

- **Selected Architectural Path:** Option B — Clean Architecture single-module Android (Kotlin, Compose, Hilt, Room, Coroutines/Flow, MVVM), offline-first, dual `PrinterDriver` abstraction, template-driven layout.
- **High-Impact Decisions (ADR Candidate):** (1) Dual printer abstraction Inkjet-PDF vs Thermal-ESC/POS — hard to reverse once layouts diverge; (2) Local-only persistence, no sync — surprising without context, real trade-off against data-loss risk; (3) No barcode/QR on label per explicit user decision — simplifies scope.
- **Domain Vocabulary Candidates (`CONTEXT.md`):** Nota, NotaItem, Label Pengiriman, Keterangan, TUNAI/KEMBALI, No INV, Template Cetak, History/Reprint, PrinterDriver (Inkjet/Thermal), USB OTG, Kustomisasi.

### 6.1 Specification Readiness Checklist (for `/polya-spec`)
- [x] **The Unknown:** Nota & Label printing via dual path, terpisah, customizable — defined.
- [x] **The Data:** Nota fields, Label 4 fields, Room entities, template fields — identified.
- [x] **The Condition:** Offline-first, single HP, USB OTG, no barcode — documented.
- [x] **Clean Architecture Seams:** Domain / UseCase / Data.local+printer / Presentation / DI — outlined.
- [ ] **Risks & Spikes:** USB OTG + ESC/POS spikes pending device test — tagged as [ASSUMPTION] for spec.

### 6.2 Direct Handoff Action to `/polya-spec`
Copy and run this command in your chat session to initiate technical specification:
```text
/polya-spec @docs/discovery/athayaprint-batik-nota-label-discovery.md Formulate formal technical specification, DTO contracts, and Clean Architecture seams based on this approved Discovery Draft.
```
