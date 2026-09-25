---
title: "Athaya Batik Nota & Label Printing Android Specification"
version: "1.1.0"
date_created: "2026-09-25"
last_updated: "2026-09-26"
status: "Clarified (Post-Audit)"
tags: ["polya", "spec", "clean-architecture", "android", "printing"]
---
<!-- markdownlint-disable-->

# Technical Specification: Athaya Batik Nota & Label Printing Android Application

## 1. Pólya Phase 1: Understanding the Problem

### 1.1 The Unknown (Goal & Desired State)
- **Android offline mobile application** untuk toko batik Athaya yang bisa mencetak Nota Penjualan dan Label Pengiriman langsung dari HP tanpa komputer
- **Dual printer support**: Inkjet USB OTG (Epson L-300) via Android Print Framework PDF, dan Thermal 80mm ESC/POS (USB/Bluetooth) via raw byte commands
- **Offline-first data storage**: Data Nota dan Label disimpan lokal di Room database, bisa diedit, di-reprint, dengan template customizable (header, logo, layout, footer)
- **Single device only**: Satu HP saja, no sync ke cloud atau multi-device
- **Cetak terpisah**: Nota dan Label adalah dua fungsi terpisah dengan format masing-masing

### 1.2 The Data (Inputs, State & Environmental Context)
- **Nota Input**: Nama pelanggan, tanggal, list item (nama, qty, harga), SUBTOTAL, DISKON, TOTAL, TUNAI, KEMBALI, No INV, header custom "ATHAYA BATIK"
- **Label Input**: Nama penerima, no telp, alamat, keterangan (tanpa barcode/QR), block PENGIRIM (nama toko, alamat, telp)
- **Existing Printer**: Epson L-300 Inkjet via USB OTG, atau future thermal printer 80mm ESC/POS
- **Environment**: Android 8+ (API 26+), USB OTG permission, USB host mode
- **State Machine**: Nota/Label creation → Save to Room → Edit/Reprint → Print Preview → Printer selection → Physical print
- **Template Storage**: Customizable header, logo, tagline, layout, footer dalam Room

### 1.3 The Condition (Requirements, Constraints & Invariants)

*Every requirement and constraint MUST be assigned a unique ID for 100% bidirectional traceability with Implementation Plans (`docs/plan/`):*

- **REQ-001:** System harus bisa input Nota Penjualan dengan format standar: Nama, Tanggal, item (nama, qty, harga), SUBTOTAL, DISKON, TOTAL, TUNAI, KEMBALI, No INV, header "ATHAYA BATIK"
- **REQ-002:** System harus bisa input Label Pengiriman dengan format: PENERIMA (nama, no telp, alamat), KETERANGAN, PENGIRIM block (nama toko, alamat, telp), TANPA barcode/QR
- **REQ-003:** Data Nota dan Label harus tersimpan lokal di Room database dan bisa diedit ulang atau di-reprint nanti
- **REQ-004:** System harus support cetak via Inkjet USB OTG (Epson L-300) menggunakan Android Print Framework dengan PDF generation
- **REQ-005:** System harus support cetak via Thermal 80mm ESC/POS printer (USB atau Bluetooth) menggunakan raw ESC/POS byte commands
- **REQ-006:** Template Nota dan Label harus bisa dikustomisasi: header, logo, tagline, layout, footer
- **REQ-007:** System harus fully offline, single HP only, no sync ke cloud atau multi-device
- **REQ-008:** UI harus ada preview sebelum print, dan opsi pilih printer (Inkjet vs Thermal)
- **REQ-009:** System harus support history list Nota dan Label dengan filter berdasarkan tanggal
- **REQ-010:** System harus memberikan feedback print status (success, error, printer not connected)

- **CON-001 (Performance / SLA):** Save to Room < 200ms, Print PDF generation < 1s, ESC/POS command transmission < 500ms
- **CON-002 (Security & Invariants):** No data loss on app kill (Room persistence), handle USB disconnect gracefully, validate numeric input (qty > 0, harga > 0, TUNAI >= TOTAL)
- **CON-003 (Resource):** App size < 15MB, memory usage < 100MB
- **CON-004 (Compatibility):** Android 8+ (API 26+), USB OTG host mode support

### 1.4 Condition Sanity Check & Completeness Audit (Pólya, p. 7, 33)
- **Is the condition sufficient to determine the unknown?** **Yes** — Semua input, output, constraints, dan printer paths sudah didefinisikan. Unknown bisa ditentukan.
- **Is it insufficient?** **No** — Semua data (Nota/Label fields, printer types, storage) sudah tercakup. User sudah confirm format final.
- **Is it redundant or contradictory?** **No contradiction** — Dual printer path complementary, bukan mutually exclusive.
- **Did you use ALL the data?** **Yes** — Semua fields Nota/Label, Epson L-300, USB OTG, Room storage, single HP constraint digunakan.
- **Did you use the WHOLE condition?** **Yes** — Semua 10 REQ dan 4 CON dipertimbangkan.

### 1.5 Indirect Proof & Negative Invariant Analysis (Pólya, p. 162–171)
*Assume the negation of the invariant (Reductio ad Absurdum) to verify defensive barriers:*

- **Negative Hypothesis 1:** What if user input TUNAI < TOTAL saat bayar?
- **Contradiction & Safe Rejection:** System harus validasi TUNAI >= TOTAL sebelum save, tampilkan error "Uang tunai kurang" di UI, prevent save.

- **Negative Hypothesis 2:** What if printer disconnect saat sedang kirim ESC/POS bytes?
- **Contradiction & Safe Rejection:** PrinterDriver harus detect USB disconnect via exception/timeout, rollback transaction state, tampilkan "Printer disconnected" error, allow retry.

- **Negative Hypothesis 3:** What if Room database corrupt atau disk full?
- **Contradiction & Safe Rejection:** Use try-catch dengan Room, tampilkan "Storage error", backup data ke file JSON fallback, prevent data loss.

- **Negative Hypothesis 4:** What if user mau print tanpa memilih printer terlebih dahulu?
- **Contradiction & Safe Rejection:** UI harus enforce printer selection sebelum tombol print, disable button jika belum pilih.

---

## 2. Ubiquitous Language & Domain Glossary (`CONTEXT.md`)

*Karena `CONTEXT.md` belum ada, saya define canonical terms di sini:*

- **Nota:** Dokumen penjualan resmi toko batik Athaya yang berisi detail transaksi (Nama, Tanggal, item, SUBTOTAL, DISKON, TOTAL, TUNAI, KEMBALI, No INV)  
  _Avoid_: Invoice, Bill, Kwitansi
- **Label Pengiriman:** Label untuk paket pengiriman yang berisi data penerima (nama, no telp, alamat) dan keterangan  
  _Avoid_: Shipping Label, Label Kirim, Stiker Pengiriman
- **TUNAI:** Uang tunai yang dibayarkan customer  
  _Avoid_: Cash, Bayar
- **KEMBALI:** Uang kembalian (TUNAI - TOTAL)  
  _Avoid_: Change, Kembali Uang
- **No INV:** Nomor invoice unik format "INV-YYYY-MM-NNNN" (4 digit auto-increment per bulan)  
  _Avoid_: Invoice Number, Nomor Nota
- **Template Cetak:** Layout, header, logo, footer yang bisa dikustomisasi untuk Nota atau Label  
  _Avoid_: Print Template, Layout Cetak
- **History:** Daftar Nota/Label yang pernah dibuat dan tersimpan di HP untuk reprint  
  _Avoid_: Riwayat, Log
- **Reprint:** Cetak ulang Nota/Label dari history tanpa input ulang data  
  _Avoid_: Print Again, Cetak Lagi
- **PrinterDriver:** Interface abstrak untuk komunikasi dengan printer (Inkjet atau Thermal)  
  _Avoid_: Printer Service, Driver Printer
- **USB OTG:** USB On-The-Go untuk koneksi HP ke printer via kabel USB  
  _Avoid_: USB Connection, USB Host
- **Kustomisasi:** Fitur edit header, logo, footer, layout template cetak  
  _Avoid_: Customization, Personalisasi

**Scope Exclusions & Auto-Resolved (2026-09-25 Clarification):**
- **Currency:** IDR Rupiah only (MoneyInCents unit). Multi-currency deferred to Phase 2.
- **Language:** Bahasa Indonesia only. Multi-language i18n deferred to Phase 2.
- **Invoice Format:** Locked to "INV-YYYY-MM-NNNN" pattern (4-digit monthly counter).
- **Logo Storage:** content:// URI or file:// path only. Cloud logo hosting deferred to Phase 2.
- **Print Retry Logic:** Automatic 3x retry with exponential backoff for USB/ESC/POS errors.
- **History:** Daftar Nota dan Label yang pernah dibuat dan disimpan  
  _Avoid_: Riwayat, Daftar Cetakan
- **Reprint:** Mencetak ulang Nota atau Label dari data yang sudah tersimpan  
  _Avoid_: Print Ulang, Cetak Kembali
- **PrinterDriver:** Interface abstraksi untuk mengontrol printer (Inkjet vs Thermal)  
  _Avoid_: PrinterController, PrintHandler
- **USB OTG:** USB On-The-Go untuk connect printer langsung ke HP Android  
  _Avoid_: USB Host Mode, USB Direct
- **Kustomisasi:** Kemampuan user mengubah tampilan Nota/Label (header, logo, dll)  
  _Avoid_: Customization, Personalisasi

---

## 3. Assumptions & Open Clarifications

*All provisional guesses or unverified assumptions made during drafting:*

> [!WARNING] [ASSUMPTION-001]: Epson L-300 bisa dicetak via Android Print Framework dengan PrintedPdfDocument tanpa driver khusus
> *Status (2026-09-25 Clarification):* ⏳ **Pending Spike 2026-09-26, Risk Accepted for Parallel Execution.** Physical device test scheduled besok. Fallback jika incompatible: custom PDF→ESC/POS converter atau thermal-only path.
> *Risk / Trade-off:* Jika L-300 tidak support standard Android printing, butuh custom PDF to ESC/POS converter untuk inkjet path
> *Proposed Verification:* Test di device fisik dengan Epson L-300 connected via USB OTG

> [!WARNING] [ASSUMPTION-002]: Thermal 80mm ESC/POS printer compatible dengan command set ESC/POS standar  
> *Status (2026-09-25 Clarification):* ✅ **Resolved: Generic PrinterDriver abstraction**. Implementasi PrinterDriver interface memungkinkan swap thermal vendor tanpa perubah core logic.
> *Risk / Trade-off:* Jika printer pakai command set proprietary, vendor-specific driver ditambah di Phase 2  
> *Proposed Verification:* Integration test dengan mock thermal ESC/POS emulator

> [!WARNING] [ASSUMPTION-003]: Paper size Nota = A6 (105x148mm), Label = 80mm width roll thermal paper  
> *Resolution (2026-09-25 Clarification):* **Locked scope for MVP: Nota = A6 only, Label = 80mm only**. Parameterized flexibility deferred to Phase 2 enhancement.  
> *Risk / Trade-off:* Jika user mau print Nota di ukuran lain (misal 80mm struk thermal), perlu update template system di Phase 2

- **[CLARIFICATION-001]:** Apakah perlu support barcode/QR di Label untuk future enhancement? **User sudah konfirmasi: TIDAK PERLU**.
- **[CLARIFICATION-002]:** Apakah perlu sync ke cloud atau backup ke Google Drive? **User sudah konfirmasi: SINGLE HP ONLY, NO SYNC**.
- **[CLARIFICATION-003]:** Apakah perlu multi-language (English/Indonesia)? **Auto-resolved: Bahasa Indonesia only**. Multi-language i18n deferred to Phase 2.
- **[CLARIFICATION-004]:** Currency handling: IDR Rupiah only (sen/cents unit). Multi-currency deferred to Phase 2.

---

## 4. Setting Up Equations & Expressive Notation (Pólya, p. 134–141)

*Splitting natural language requirements clause-by-clause into formal structures and making invalid states unrepresentable via Type-Driven Design:*

- **Type-Driven Domain Invariants:**
  - `PositiveInteger` value object untuk qty dan harga (must be > 0)
  - `MoneyInCents` value object untuk currency (integer cents, bukan float)
  - `InvoiceNumber` value object dengan pattern "INV-YYYY-MM-NNN"
  - `PhoneNumber` value object dengan validation Indonesia format
  - `PrintTemplateId` sealed class: `NotaTemplate` | `LabelTemplate`

### 4.1 Data Transfer Objects (DTOs) & Interfaces
```kotlin
// Domain Entities
data class NotaItem(
    val id: String,
    val productName: String,
    val quantity: PositiveInteger,
    val unitPrice: MoneyInCents,
    val totalPrice: MoneyInCents // calculated: quantity * unitPrice
)

data class Nota(
    val id: String,
    val invoiceNumber: InvoiceNumber,
    val customerName: String,
    val date: LocalDateTime,
    val items: List<NotaItem>,
    val subtotal: MoneyInCents,
    val discount: MoneyInCents,
    val total: MoneyInCents,
    val cashPaid: MoneyInCents,
    val change: MoneyInCents, // calculated: cashPaid - total
    val templateId: PrintTemplateId,
    val createdAt: LocalDateTime
)

data class ShippingLabel(
    val id: String,
    val recipientName: String,
    val recipientPhone: PhoneNumber,
    val recipientAddress: String,
    val notes: String, // KETERANGAN
    val senderName: String = "Athaya Batik Store",
    val senderAddress: String = "Jl. Contoh No.123",
    val senderPhone: PhoneNumber,
    val templateId: PrintTemplateId,
    val createdAt: LocalDateTime
)

data class PrintTemplate(
    val id: PrintTemplateId,
    val name: String,
    val type: TemplateType, // NOTA or LABEL
    val headerText: String?,
    val logoUri: String?, // URI to local image (content:// URI)
    val footerText: String?,
    val paperSize: PaperSize, // A6_105x148MM (Nota) or ROLL_80MM (Label)
    val layoutConfig: LayoutConfig // JSON or data class
)

// Request/Response DTOs
data class CreateNotaRequest(
    val customerName: String,
    val items: List<NotaItemDto>,
    val discount: MoneyInCents,
    val cashPaid: MoneyInCents
)

data class PrintRequest(
    val documentId: String, // Nota ID or Label ID
    val printerType: PrinterType, // INKJET_PDF or THERMAL_ESC_POS
    val copies: PositiveInteger = PositiveInteger(1)
)

data class PrintResponse(
    val success: Boolean,
    val printerType: PrinterType,
    val errorMessage: String? = null,
    val timestamp: LocalDateTime
)
```

### 4.2 Database Schema / Persistence Models
```sql
-- Room Entities
@Entity(tableName = "notas")
data class NotaEntity(
    @PrimaryKey val id: String,
    val invoiceNumber: String,
    val customerName: String,
    val date: Long, // timestamp
    val itemsJson: String, // JSON serialized List<NotaItem>
    val subtotalCents: Long,
    val discountCents: Long,
    val totalCents: Long,
    val cashPaidCents: Long,
    val changeCents: Long,
    val templateId: String,
    val createdAt: Long
)

@Entity(tableName = "shipping_labels")
data class ShippingLabelEntity(
    @PrimaryKey val id: String,
    val recipientName: String,
    val recipientPhone: String,
    val recipientAddress: String,
    val notes: String,
    val senderName: String,
    val senderAddress: String,
    val senderPhone: String,
    val templateId: String,
    val createdAt: Long
)

@Entity(tableName = "print_templates")
data class PrintTemplateEntity(
    @PrimaryKey val id: String,
    val name: String,
    val type: String, // "NOTA" or "LABEL"
    val headerText: String?,
    val logoUri: String?,
    val footerText: String?,
    val paperSize: String, // "A4", "LETTER", "ROLL_80MM"
    val layoutConfigJson: String
)
```

---

## 5. Clean Architecture Seams & Component Boundaries

> [!NOTE]
> **Architectural Pragmatism Check:** Ini **Enterprise Core Domain** (aplikasi Android production dengan dual printer, offline persistence, complex UI). Jadi enforce full 4-layer Clean Architecture.

```text
Entities (Domain Layer)
   └── Nota.kt, NotaItem.kt, ShippingLabel.kt, PrintTemplate.kt, value objects
   └── Repository interfaces (NotaRepository, LabelRepository, TemplateRepository)

Use Cases (Application Layer)
   └── CreateNotaUseCase.kt, UpdateNotaUseCase.kt, ReprintNotaUseCase.kt
   └── CreateLabelUseCase.kt, PrintDocumentUseCase.kt, ManageTemplateUseCase.kt
   └── PrinterDriver interface (abstraction layer)

Interface Adapters (Controllers, Gateways, Presenters)
   └── ViewModel (NotaViewModel, LabelViewModel, HistoryViewModel, PrintViewModel)
   └── Room implementations (NotaRepositoryImpl, LabelRepositoryImpl)
   └── Printer adapters (InkjetPdfDriver.kt, ThermalEscPosDriver.kt)
   └── UI Composables (NotaEditorScreen, LabelEditorScreen, PrintPreviewScreen)

Frameworks & Drivers (DB, Web Server, UI Runtimes)
   └── Room Database, Android Print Framework, USB Host API
   └── Jetpack Compose UI, Hilt DI, Coroutines/Flow
```

- **Domain Layer:** Pure business logic dan entities di `domain/` module
- **Use Case Layer:** Transaction orchestration di `usecases/` module
- **Adapter Layer:** Android-specific implementations di `data/` dan `presentation/`
- **Dependency Inversion Seams:** 
  - `PrinterDriver` interface di domain → implementations di data/printer/
  - Repository interfaces di domain → Room implementations di data/local/
  - UseCase classes depend on interfaces, bukan concrete implementations

**Module Structure Proposal:**
```
app/
├── domain/           # Pure Kotlin, no Android dependencies
├── usecases/         # Kotlin + Coroutines
├── data/
│   ├── local/       # Room DAOs, entities, mappers
│   └── printer/     # InkjetPdfDriver, ThermalEscPosDriver
└── presentation/    # Compose UI, ViewModels, navigation
```

---

## 6. Acceptance Criteria (Given-When-Then)

*Every Acceptance Criterion must have a unique ID that pairs with corresponding REQ IDs for verification in `docs/plan/`:*

- **AC-001 (Happy Path for REQ-001):**  
  *Given* user di Nota Editor screen dengan form kosong,  
  *When* user input Nama "Budi", tambah item "Batik Tulis" qty=2 harga=150000, isi DISKON=5000, TUNAI=300000,  
  *Then* system hitung SUBTOTAL=300000, TOTAL=295000, KEMBALI=5000, generate No INV "INV-2026-09-001", dan simpan ke Room.

- **AC-002 (Happy Path for REQ-002):**  
  *Given* user di Label Editor screen,  
  *When* user input PENERIMA "Sari", telp "08123456789", alamat "Jl. Merdeka No.5", KETERANGAN "2 Batik Cap",  
  *Then* system generate Label dengan PENGIRIM block default "Athaya Batik Store / Jl. Contoh No.123 / Telp", dan simpan ke Room.

- **AC-003 (Happy Path for REQ-004):**  
  *Given* Epson L-300 connected via USB OTG, user pilih Nota dan tekan "Print Inkjet",  
  *When* system generate PDF via PrintedPdfDocument dengan template Nota,  
  *Then* PDF dikirim ke Android Print Framework dan print sukses di kertas A4.

- **AC-004 (Happy Path for REQ-005):**  
  *Given* Thermal printer 80mm connected via USB/Bluetooth, user pilih Label dan tekan "Print Thermal",  
  *When* system kirim ESC/POS bytes (initialize, text, cut),  
  *Then* Label tercetak di thermal paper roll 80mm.

- **AC-005 (Boundary for REQ-003):**  
  *Given* Nota sudah disimpan di Room,  
  *When* user edit customerName dari "Budi" ke "Budi Santoso" dan save,  
  *Then* Room update record tersebut, dan perubahan persist.

- **AC-006 (Edge Case for CON-002):**  
  *Given* user input TUNAI=200000 sedangkan TOTAL=250000,  
  *When* user tekan "Save Nota",  
  *Then* system validasi dan tampilkan error "Uang tunai kurang Rp 50.000", prevent save.

- **AC-007 (Negative for REQ-010):**  
  *Given* printer tidak connected,  
  *When* user tekan tombol print,  
  *Then* system tampilkan error "Printer not connected" dan disable print button.

- **AC-008 (Template for REQ-006):**  
  *Given* user di Template Settings screen,  
  *When* user upload logo dari gallery dan ubah header text ke "ATHAYA BATIK PREMIUM",  
  *Then* system save template ke Room, dan Nota berikutnya pakai template baru.

---

## 7. Testing Strategy & Dimensional Bounds

- **Testing Seam:** Test di UseCase boundary dengan mock Repository dan mock PrinterDriver
- **Dimension Checks:** 
  - Currency dalam sen (cents) bukan rupiah float
  - Paper size: A4 = 210x297mm, 80mm roll = 80mm width unlimited length
  - USB timeout: 5000ms untuk ESC/POS command
- **Extreme Limiting Cases (Specialization):**
  - Nota dengan 100+ items (scroll/performance test)
  - Label dengan alamat sangat panjang (word wrap test)
  - Thermal printer paper out scenario (error handling)
  - Room database migration dari v1 ke v2
- **Integration Test:** USB OTG connection test di physical device dengan actual printer

---

## 8. Architectural Decision Records (ADRs)

- **[ADR-0001: Dual Printer Abstraction Layer](docs/adr/0001-dual-printer-abstraction.md)** — Implement PrinterDriver interface untuk decouple Inkjet-PDF dan Thermal-ESC/POS, memudahkan future printer types.
- **[ADR-0002: Offline-First Single Device](docs/adr/0002-offline-first-single-device.md)** — Pilih Room lokal tanpa sync ke cloud, trade-off data loss risk accepted untuk simplicity dan offline guarantee.
- **[ADR-0003: Type-Driven Domain Modeling](docs/adr/0003-type-driven-domain.md)** — Gunakan Value Objects (PositiveInteger, MoneyInCents, InvoiceNumber) untuk compile-time safety daripada runtime validation.
