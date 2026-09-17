# Technology Comparison Matrices

All options are scored from **0 to 10** against weighted criteria taken from the FitFlow user research
(Lab 03). Weighted score = Σ (weight × score) ÷ 100.

---

## 1. Frontend / Cross-Platform Framework

### 1.1 Qualitative comparison

| Criterion | Flutter | React Native | Kotlin Multiplatform | Swift / SwiftUI |
|---|---|---|---|---|
| Development speed | Hot reload, one UI layer for all targets | Hot reload, huge npm ecosystem | UI written separately per platform | Fast on iOS only |
| Code reusability | ~90 % across iOS/Android/Web | ~85 % mobile, ~70 % with RN Web | ~60 % (logic only, UI native) | 0 % outside Apple |
| Performance | Compiled to ARM, Impeller renderer, 60 to 120 fps | JS bridge / JSI overhead on heavy lists | Fully native performance | Best-in-class on iOS |
| Ecosystem | 40 k+ pub.dev packages, Google-backed | Largest community, Meta-backed | Youngest ecosystem | Mature but Apple-only |
| Learning curve | New language (Dart), consistent APIs | Easy for React/JS developers | Kotlin + two native UI toolkits | Swift + Apple frameworks |
| Web compatibility | Flutter Web (canvas; weak SEO, large payload) | React Native Web (real DOM) | Kotlin/Wasm still experimental | None |
| AI/ML integration | TFLite, ML Kit, platform channels | TFLite / ONNX via native modules | Direct native ML APIs | Core ML, Vision, Neural Engine |
| Real-time features | Streams, web_socket_channel, Firebase SDK | Socket.IO, Firebase SDK | Ktor WebSockets in shared code | URLSession WebSocket, Combine |
| Maintenance cost | One team, one codebase | One team, but native modules drift | Two UI codebases to maintain | Separate iOS and Android teams |
| Security | Compiled binary, harder to reverse | JS bundle readable unless obfuscated | Native + Kotlin obfuscation | Keychain, Secure Enclave, App Attest |

### 1.2 Weighted decision matrix

| Criterion | Weight | Flutter | React Native | Kotlin MP | Swift/SwiftUI |
|---|---|---|---|---|---|
| Development speed | 10 % | 9 | 9 | 6 | 7 |
| Code reusability | 15 % | 9 | 8 | 7 | 2 |
| Performance | 15 % | 9 | 7 | 9 | 10 |
| Ecosystem support | 10 % | 8 | 9 | 6 | 8 |
| Learning curve | 8 % | 7 | 9 | 5 | 6 |
| Web compatibility | 12 % | 6 | 7 | 5 | 1 |
| AI/ML integration | 8 % | 7 | 7 | 8 | 9 |
| Real-time features | 7 % | 8 | 8 | 8 | 8 |
| Maintenance cost | 10 % | 8 | 7 | 6 | 4 |
| Security | 5 % | 7 | 6 | 8 | 9 |
| **Weighted total** | **100 %** | **7.95** | **7.73** | **6.80** | **6.03** |

**Selected: Flutter** for the mobile client, paired with **Next.js** for the web experience
(a hybrid approach, described in `docs/tech-stack.md`).

---

## 2. Backend Framework

| Criterion | Weight | Node.js / NestJS | Python / FastAPI | Go (Gin/Fiber) | Java / Spring Boot |
|---|---|---|---|---|---|
| Runtime performance | 15 % | 7 | 7 | 10 | 8 |
| Development speed | 15 % | 9 | 9 | 6 | 5 |
| Real-time / WebSocket support | 12 % | 9 | 7 | 8 | 7 |
| AI/ML ecosystem | 12 % | 6 | 10 | 4 | 6 |
| Scalability | 13 % | 8 | 7 | 10 | 9 |
| Library ecosystem | 10 % | 9 | 8 | 7 | 9 |
| Built-in security features | 10 % | 8 | 7 | 8 | 9 |
| Maintainability / hiring pool | 13 % | 9 | 8 | 7 | 7 |
| **Weighted total** | **100 %** | **8.11** | **7.89** | **7.55** | **7.39** |

**Selected: NestJS for the core API + FastAPI for the AI microservice.** NestJS wins overall;
FastAPI's 10/10 AI ecosystem score is exactly why the machine-learning workload is split into
its own Python service instead of forcing one language onto two very different concerns.

---

## 3. Database

| Criterion | Weight | PostgreSQL | MongoDB | Firebase Firestore | DynamoDB |
|---|---|---|---|---|---|
| Query power / analytics | 18 % | 10 | 7 | 4 | 4 |
| Scalability | 15 % | 8 | 9 | 9 | 10 |
| Health / time-series data handling | 15 % | 9 | 7 | 5 | 6 |
| Real-time capability | 12 % | 7 | 7 | 10 | 8 |
| Security & compliance (HIPAA/GDPR) | 15 % | 9 | 8 | 7 | 9 |
| Cost predictability | 10 % | 8 | 7 | 5 | 7 |
| Ecosystem / maintainability | 15 % | 9 | 8 | 7 | 6 |
| **Weighted total** | **100 %** | **8.69** | **7.60** | **6.62** | **7.03** |

**Selected: PostgreSQL 16 + TimescaleDB extension**, with **Redis** as cache and real-time bus
and **S3** for meal photos.

---

## 4. Authentication & Authorization

| Criterion | Weight | Firebase Auth | AWS Cognito | Auth0 | Supabase Auth |
|---|---|---|---|---|---|
| Security & compliance (HIPAA/GDPR/SOC 2) | 25 % | 6 | 9 | 9 | 7 |
| Feature completeness (MFA, RBAC, social, passwordless) | 18 % | 7 | 8 | 10 | 8 |
| Integration & developer experience | 15 % | 9 | 6 | 9 | 9 |
| Cost at scale | 17 % | 8 | 9 | 4 | 8 |
| Portability / low vendor lock-in | 10 % | 4 | 5 | 5 | 7 |
| Maintainability | 15 % | 8 | 7 | 9 | 8 |
| **Weighted total** | **100 %** | **7.07** | **7.67** | **7.93** | **7.80** |

**Selected: Auth0.** Cost is its one weak score; the mitigation is to store only identity in
Auth0 (profile and health data stay in PostgreSQL) and to re-evaluate against Cognito if paid
MAU passes ~50,000.

---

## 5. Consolidated Stack Decision Matrix

| Candidate stack | Composition |
|---|---|
| **Stack A (recommended)** | Flutter + Next.js · NestJS + FastAPI · PostgreSQL + Redis + S3 · Auth0 · AWS ECS |
| **Stack B (rapid BaaS)** | React Native · Firebase Cloud Functions · Firestore · Firebase Auth |
| **Stack C (native + serverless)** | Swift/SwiftUI + Kotlin native · Go on Lambda · DynamoDB · Cognito |

| Criterion | Weight | Stack A | Stack B | Stack C |
|---|---|---|---|---|
| Performance | 15 % | 9 | 7 | 10 |
| Scalability | 12 % | 9 | 8 | 9 |
| Development speed | 12 % | 8 | 10 | 5 |
| Cross-platform reusability | 10 % | 9 | 8 | 3 |
| Security & compliance | 15 % | 9 | 6 | 9 |
| AI/ML support | 10 % | 9 | 6 | 8 |
| Real-time capability | 8 % | 8 | 10 | 7 |
| Cost | 8 % | 7 | 6 | 5 |
| Maintainability | 10 % | 9 | 7 | 6 |
| **Weighted total** | **100 %** | **8.64** | **7.49** | **7.19** |

**Recommended: Stack A, with a weighted score of 8.64 out of 10.**
