# 🔭 Project Discovery Draft: AthayaPrint Android Application

**Document Type:** Phase 0 Discovery Draft
**Target Repository:** `AthayaPrint Android App`
**Status:** **Draft**
**Architect:** Veteran Principal Fullstack Engineer
**Date:** 2026-09-25

---

## 1. Problem Statement & Business Opportunity (Getting Acquainted)

*Pólya Heuristic: Familiarity and Conception (Pólya, 1945, p. 33). Understand the problem as a whole, its primary purpose, and its overarching value.*

- **The Problem:** Customers currently lack a dedicated mobile platform to conveniently manage printing orders, track real-time printing progress, and communicate directly with the printing service provider. This creates friction in the customer journey, limits business scalability, and forces reliance on manual communication channels (phone, WhatsApp, in-person visits).

- **Business Value & Why Now:**
  - **Revenue Growth:** Mobile-first ordering reduces friction, increases order frequency, and enables push notification re-engagement
  - **Operational Efficiency:** Automated status updates reduce customer support inquiries by an estimated 40-60%
  - **Competitive Advantage:** Digital presence differentiates from competitors still relying on manual processes
  - **Data-Driven Insights:** Order history and user behavior analytics enable personalized marketing and inventory optimization

- **Target Persona & Users:**
  - **Primary:** Individual customers (students, professionals, small business owners) who need document/photo printing services
  - **Secondary:** Corporate clients with recurring bulk printing needs
  - **Tertiary:** Printing service provider staff (admin dashboard for order management - future phase)

---

## 2. The Unknown, Known Data, and Operational Bounds

*Pólya Heuristic: Isolate the principal parts of the problem: What is the unknown? What are the data? What is the condition?*

| Element | Description |
| :--- | :--- |
| **The Unknown** | A production-ready Android application implementing:<br>• User authentication & profile management<br>• Print order creation & management (document upload, specifications, pricing)<br>• Real-time order status tracking (Received → Processing → Printing → Ready → Completed)<br>• In-app communication with provider (chat/messages)<br>• Payment integration (optional, depends on business model)<br>• Order history & reordering capability |
| **Known Data** | • Business domain: Printing services (documents, photos, banners, merchandise)<br>• Target platform: Android (API 24+ recommended for market coverage)<br>• Architecture constraint: Clean Architecture + SOLID (per AGENTS.md)<br>• No existing codebase (greenfield project) |
| **The Conditions & Invariants** | • **Platform:** Android (Kotlin preferred, Java optional)<br>• **Architecture:** Strict Clean Architecture layers (Domain → Use Cases → Interface Adapters → Frameworks)<br>• **Offline Capability:** Basic order caching for offline viewing (optional enhancement)<br>• **Security:** Secure document upload, authenticated API calls, no hardcoded secrets<br>• **Performance:** < 3 second app launch, < 2 second screen transitions<br>• **Scalability:** Support 10,000+ concurrent users without architecture refactor |

---

## 3. Codebase Exploration & Architectural Critique

*Pólya Heuristic: Decomposing and Recombining (p. 75) & Analogy (p. 37). Examine existing components, critique architectural debt, and look for analogous solutions.*

### 3.1 Existing Topography Analysis

> [!NOTE]
> **Greenfield Project:** This is a greenfield Android application. No existing codebase exists. The following section proposes the initial Clean Architecture folder topography and technology stack scaffold.

**Proposed Clean Architecture Folder Topography:**

```
app/
├── build.gradle.kts (app-level)
├── src/main/java/com/athayaprint/
│   ├── AthayaPrintApp.kt (Application class, DI setup)
│   │
│   ├── domain/                          # DOMAIN LAYER (Pure Kotlin, no Android deps)
│   │   ├── model/                       # Business entities
│   │   │   ├── User.kt
│   │   │   ├── Order.kt
│   │   │   ├── OrderStatus.kt
│   │   │   ├── PrintSpecification.kt
│   │   │   ├── Document.kt
│   │   │   └── Message.kt
│   │   ├── repository/                  # Repository interfaces (ports)
│   │   │   ├── UserRepository.kt
│   │   │   ├── OrderRepository.kt
│   │   │   └── MessageRepository.kt
│   │   └── usecase/                     # Business logic (Use Cases)
│   │       ├── user/
│   │       │   ├── LoginUseCase.kt
│   │       │   ├── RegisterUseCase.kt
│   │       │   └── GetProfileUseCase.kt
│   │       ├── order/
│   │       │   ├── CreateOrderUseCase.kt
│   │       │   ├── GetOrdersUseCase.kt
│   │       │   ├── UpdateOrderUseCase.kt
│   │       │   ├── CancelOrderUseCase.kt
│   │       │   └── TrackOrderStatusUseCase.kt
│   │       └── message/
│   │           ├── SendMessageUseCase.kt
│   │           └── GetMessagesUseCase.kt
│   │
│   ├── data/                            # DATA LAYER (Repository implementations)
│   │   ├── remote/                      # Network layer
│   │   │   ├── api/
│   │   │   │   ├── AthayaPrintApi.kt
│   │   │   │   └── ApiEndpoints.kt
│   │   │   ├── dto/                     # Data Transfer Objects
│   │   │   │   ├── UserDto.kt
│   │   │   │   ├── OrderDto.kt
│   │   │   │   └── MessageDto.kt
│   │   │   └── interceptor/
│   │   │       └── AuthInterceptor.kt
│   │   ├── local/                       # Local storage
│   │   │   ├── database/
│   │   │   │   ├── AppDatabase.kt
│   │   │   │   └── dao/
│   │   │   │       ├── OrderDao.kt
│   │   │   │       └── UserDao.kt
│   │   │   └── preferences/
│   │   │       └── UserPreferences.kt
│   │   ├── mapper/                      # DTO ↔ Domain Entity mappers
│   │   │   ├── UserMapper.kt
│   │   │   ├── OrderMapper.kt
│   │   │   └── MessageMapper.kt
│   │   └── repository/                  # Repository implementations
│   │       ├── UserRepositoryImpl.kt
│   │       ├── OrderRepositoryImpl.kt
│   │       └── MessageRepositoryImpl.kt
│   │
│   ├── presentation/                    # PRESENTATION LAYER (Android UI)
│   │   ├── common/                      # Shared UI components
│   │   │   ├── components/
│   │   │   │   ├── LoadingState.kt
│   │   │   │   ├── ErrorState.kt
│   │   │   │   └── EmptyState.kt
│   │   │   └── theme/
│   │   │       ├── Color.kt
│   │   │       ├── Theme.kt
│   │   │       └── Type.kt
│   │   ├── navigation/
│   │   │   └── NavGraph.kt
│   │   ├── auth/                        # Auth feature module
│   │   │   ├── LoginScreen.kt
│   │   │   ├── RegisterScreen.kt
│   │   │   └── AuthViewModel.kt
│   │   ├── home/                        # Home/Dashboard
│   │   │   ├── HomeScreen.kt
│   │   │   └── HomeViewModel.kt
│   │   ├── order/                       # Order management
│   │   │   ├── CreateOrderScreen.kt
│   │   │   ├── OrderListScreen.kt
│   │   │   ├── OrderDetailScreen.kt
│   │   │   ├── TrackOrderScreen.kt
│   │   │   └── OrderViewModel.kt
│   │   ├── profile/                     # User profile
│   │   │   ├── ProfileScreen.kt
│   │   │   └── ProfileViewModel.kt
│   │   └── chat/                        # Communication
│   │       ├── ChatScreen.kt
│   │       ├── ChatListScreen.kt
│   │       └── ChatViewModel.kt
│   │
│   └── di/                              # Dependency Injection
│       ├── AppModule.kt
│       ├── NetworkModule.kt
│       ├── DatabaseModule.kt
│       └── RepositoryModule.kt
│
build.gradle.kts (project-level)
settings.gradle.kts
gradle/
└── libs.versions.toml (version catalog)
```

### 3.2 Analogous Solutions & Prior Art

**Internal Analogies:**
- N/A (greenfield project, no existing codebase to reference)

**External Analogies & Industry Patterns:**

| Pattern | Application in AthayaPrint |
| :--- | :--- |
| **MVVM + Clean Architecture** | Google's recommended architecture for Android. Separates UI (View), business logic (ViewModel + Use Cases), and data (Repository). Enables unit testing at each layer. |
| **Repository Pattern** | Abstracts data sources (remote API, local DB) behind interfaces. Domain layer depends on abstractions, not implementations (DIP). |
| **Unidirectional Data Flow (UDF)** | ViewModel exposes `StateFlow<UiState>` and receives `Intent`/`Action` events. Predictable state management, easier debugging. |
| **Offline-First** | Room database caches orders locally. Users can view cached data offline; syncs when connection restored. |
| **Single Activity, Multiple Fragments** | Jetpack Navigation Component manages fragment transactions. Reduces activity overhead, smoother transitions. |

**Reference Implementations:**
- Google's [Now in Android](https://github.com/android/nowinandroid) — Modern Android app showcasing best practices (Kotlin, Jetpack Compose, Hilt, Coroutines/Flow)
- Google's [Architecture Samples](https://github.com/android/architecture-samples) — Classic Clean Architecture implementation
- [Android Developer Guide: Guide to App Architecture](https://developer.android.com/topic/architecture)

---

## 4. Candidate Architectures & Trade-Off Matrix

*Pólya Heuristic: The Inventor's Paradox (p. 121) — Evaluate whether a more general architectural abstraction provides a cleaner, more maintainable solution than narrow ad-hoc modifications.*

### 4.1 Architecture Options Comparison

| Dimension | Option A: Minimal (MVP) | Option B: Target (Production-Ready) | Option C: Comprehensive (Enterprise) |
| :--- | :--- | :--- | :--- |
| **Summary** | Single-module app, basic Retrofit + Room, imperative UI (XML or basic Compose) | Multi-module Clean Architecture, Jetpack Compose, Hilt DI, offline-first, feature-based modularization | Multi-module + Dynamic Feature Modules, microservices-ready backend, CI/CD pipeline, comprehensive testing (unit, integration, E2E) |
| **Clean Architecture Seams** | Loosely applied (may leak Android deps into business logic) | Strict layer separation (Domain layer has zero Android dependencies) | Full bounded contexts, isolated feature modules, shared kernel |
| **Complexity & Risk** | Low initial cost, high technical debt accumulation, difficult to scale | Moderate initial effort, high long-term maintainability, easy to extend | High initial overhead, requires DevOps expertise, operational complexity |
| **Reversibility (Two-Way Door?)** | Hard to untangle later (requires full refactor to scale) | Easy to refactor behind interfaces, modular by design | High architectural commitment, but flexible feature delivery |
| **Development Speed** | Fastest initial delivery (4-6 weeks) | Balanced delivery (8-12 weeks for core features) | Slowest initial delivery (16-24 weeks for full suite) |
| **Testing Strategy** | Basic UI tests only | Unit tests (Domain + Use Cases), UI tests (Compose testing) | Full test pyramid (unit, integration, E2E, performance, security) |
| **Scalability** | Limited to ~1,000 users before performance issues | Supports 10,000+ users, horizontal scaling | Supports 100,000+ users, enterprise-grade reliability |
| **Recommendation** | **Alternative** (for prototype only) | **[RECOMMENDED]** | Future Phase (after market validation) |

### 4.2 Recommended Architecture: Option B (Target)

**Rationale:**
1. **Clean Architecture Adherence:** Strict layer separation ensures business logic remains independent of framework changes, enabling easy testing and future platform expansion (iOS, Web).
2. **Jetpack Compose:** Declarative UI toolkit reduces boilerplate, improves state management, and aligns with modern Android development (recommended by Google since 2021).
3. **Hilt Dependency Injection:** Simplifies DI setup, compile-time safety, and integrates seamlessly with Jetpack libraries.
4. **Offline-First:** Local caching via Room ensures users can view orders offline, critical for regions with unreliable connectivity.
5. **Scalability:** Architecture supports growth from 1,000 to 100,000+ users without fundamental restructuring.

---

## 5. Technical Feasibility Spikes & Open Risks

*Pólya Heuristic: Auxiliary Problem (p. 50). Can we introduce an easier, temporary stepping stone or prototype to resolve high-risk unknowns?*

### 5.1 Technical Unknowns & Risk Assessment

| Risk ID | Technical Unknown | Impact | Likelihood | Mitigation Strategy |
| :--- | :--- | :--- | :--- | :--- |
| RISK-001 | Backend API availability and contract | High | High | **Spike Required:** Clarify if backend exists. If not, design API contract first (OpenAPI spec) or use mock server (WireMock/MockWebServer) for initial development |
| RISK-002 | Real-time order tracking implementation | Medium | Medium | Evaluate WebSocket vs. Server-Sent Events vs. Polling. Start with polling (simplest), migrate to WebSocket in Phase 2 |
| RISK-003 | Payment gateway integration | High | Medium | Defer to Phase 2. Use "Pay at Store" placeholder for MVP. Research Indonesian payment gateways (Midtrans, Xendit, DOKU) |
| RISK-004 | Document upload & file handling | Medium | Low | Android File Provider + Multipart upload via Retrofit. Test with 10MB+ files for performance baseline |
| RISK-005 | Push notification for order status updates | Medium | Low | Firebase Cloud Messaging (FCM) — industry standard, well-documented |
| RISK-006 | Authentication strategy | High | Low | JWT tokens with refresh mechanism. Consider biometric authentication for UX enhancement |

### 5.2 Critical Decision: Backend API Availability

> [!WARNING] [ASSUMPTION-001]: **Backend API status is unknown.** This Discovery Draft assumes a RESTful backend API will be provided or developed in parallel. If no backend exists, this architecture requires a backend-for-frontend (BFF) or Firebase integration strategy.

**Spike Recommendation:**
- **Question:** Does AthayaPrint have an existing backend API? If yes, request API documentation (OpenAPI/Swagger spec).
- **If No Backend Exists:** Consider:
  - **Option A:** Firebase (Firestore + Auth + Cloud Storage + Cloud Functions) — Fastest path to backend, managed service, scales automatically
  - **Option B:** Develop backend in parallel (Spring Boot, NestJS, or Django) — More control, requires backend developer
  - **Option C:** Mock API for initial Android development, backend developed later — Decouples frontend/backend timelines

---

## 6. Strategic Recommendation & Handoff to Specification

### 6.1 Selected Architectural Path

**Option B: Target (Production-Ready Clean Architecture)** is recommended for the following reasons:

1. **Long-Term Maintainability:** Strict Clean Architecture ensures business logic remains decoupled from Android framework changes, enabling easier testing, refactoring, and future platform expansion.
2. **Scalability:** Architecture supports growth from MVP to enterprise-scale without fundamental restructuring.
3. **Modern Best Practices:** Jetpack Compose, Hilt, Coroutines/Flow, and Room represent the current industry standard for Android development (as of 2024-2026).
4. **Testability:** Domain layer (pure Kotlin) can be unit tested without Android emulator, enabling fast CI/CD feedback loops.

### 6.2 High-Impact Decisions (ADR Candidates)

The following decisions warrant Architecture Decision Records (ADRs) under Triple-Gate rules:

| Decision | Rationale for ADR |
| :--- | :--- |
| **ADR-001: Kotlin over Java** | Kotlin is Google's recommended language for Android development, offers null safety, coroutines, and concise syntax. Hard to reverse (requires full rewrite). |
| **ADR-002: Jetpack Compose over XML Views** | Declarative UI paradigm shift, future-proof, reduces boilerplate. Surprising without context (legacy developers may expect XML). |
| **ADR-003: Clean Architecture + MVVM** | Strict layer separation, testability, scalability. Real trade-off: increased initial complexity vs. long-term maintainability. |
| **ADR-004: Backend Strategy (Firebase vs. Custom Backend)** | Critical infrastructure decision affecting authentication, data storage, and real-time capabilities. High lock-in if Firebase chosen. |

### 6.3 Domain Vocabulary Candidates (`CONTEXT.md`)

The following domain terms should be registered in `CONTEXT.md` to establish a ubiquitous language:

| Term | Definition | Notes |
| :--- | :--- | :--- |
| **Order** | A customer's request for printing services, including documents, specifications, and delivery preferences. | _Avoid_: Job, Task, Request |
| **Print Specification** | Detailed parameters for a print job: paper size, paper type, color mode, quantity, binding options. | _Avoid_: Print Settings, Print Config |
| **Order Status** | Lifecycle state of an order: Received → Processing → Printing → Ready → Completed (or Cancelled). | _Avoid_: Order State, Job Status |
| **Document** | A file (PDF, image, or design file) uploaded by the customer for printing. | _Avoid_: File, Attachment |
| **Provider** | The printing service provider (AthayaPrint business entity). | _Avoid_: Vendor, Shop, Printer |
| **Tracking** | Real-time status updates on order progress, visible to the customer. | _Avoid_: Monitoring, Status Update |

---

## 7. Specification Readiness Checklist (for `/polya-spec`)

- [x] **The Unknown:** Primary target outcome (Android app for printing services) and system completion states are clearly defined.
- [ ] **The Data:** Input payloads, query parameters, DB models, and third-party integrations need detailed specification (requires backend API contract or mock server definition).
- [ ] **The Condition:** High-level invariants, SLAs, and performance bounds are documented. Security requirements (auth, secure upload) identified.
- [x] **Clean Architecture Seams:** Layer responsibilities (Domain, Use Case, Data, Presentation) are outlined with proposed folder structure.
- [x] **Risks & Spikes:** Backend API availability is the critical unknown requiring immediate resolution before specification phase.

### 7.1 Pre-Specification Blockers

> [!IMPORTANT]
> **BLOCKER:** Backend API availability must be confirmed before proceeding to `/polya-spec`. The technical specification cannot be completed without API contracts.

**Required Clarification:**
1. Does AthayaPrint have an existing backend API?
   - **If YES:** Provide API documentation (OpenAPI spec, Postman collection, or endpoint list).
   - **If NO:** Decide backend strategy:
     - Firebase (recommended for speed and managed infrastructure)
     - Custom backend (Spring Boot, NestJS, Django)
     - Mock API for initial development

2. What is the authentication strategy?
   - Email/Password
   - Phone Number + OTP
   - Social Login (Google, Facebook)
   - Biometric (Fingerprint/Face)

3. What payment methods should be supported in Phase 1?
   - Pay at Store (MVP)
   - Bank Transfer
   - E-Wallet (GoPay, OVO, Dana)
   - Credit Card

---

## 8. Direct Handoff Action to `/polya-spec`

Once the backend API blocker is resolved, copy and run this command in your chat session to initiate technical specification:

```text
/polya-spec @docs/discovery/athayaprint-android-discovery.md Formulate formal technical specification, DTO contracts, Clean Architecture seams, and Android module structure based on this approved Discovery Draft.
```

**Alternative Path:** If backend does not exist and Firebase is chosen, modify the handoff command:

```text
/polya-spec @docs/discovery/athayaprint-android-discovery.md Design Firebase-based backend architecture (Firestore, Auth, Storage) and Android Clean Architecture implementation.
```

---

## 9. Phase 0 Completion Summary

✅ **Completed:**
- Problem statement and business value articulated
- Problem triad (Unknown, Data, Condition) isolated
- Greenfield Clean Architecture folder structure proposed
- Candidate architectures evaluated (Minimal vs. Target vs. Comprehensive)
- Technical risks and feasibility spikes identified
- Backend API availability flagged as critical blocker

⚠️ **Pending Resolution:**
- Backend API contract confirmation
- Authentication strategy decision
- Payment integration scope decision

📍 **Next Phase:** `/polya-spec` (blocked until backend strategy is confirmed)
