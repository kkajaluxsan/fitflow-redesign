# ADR-001: Technology Stack for the FitFlow Redesign

| Field | Value |
|---|---|
| **Status** | Accepted |
| **Date** | 2026-09-17 |
| **Deciders** | FitFlow product team: mobile lead, backend lead, ML lead, security lead |
| **Supersedes** | None |

---

## Context

The FitFlow redesign has to deliver one seamless experience across iOS, Android and the web for a
consumer fitness product. Five high-priority requirements came out of the Lab 03 user research, and
they shape this decision:

- **AI-powered adaptive workout plans** that respond to logged performance. P3 said "Weights have
  not increased in 2 weeks."
- **Camera-based nutrition logging** to replace manual search. P1 said "I spent 5 minutes searching
  for each item."
- **Onboarding of at most 5 questions.** P1 said "Onboarding was 10 minutes. Almost gave up."
- **User-controlled privacy** over anything shared socially. P5 said "Privacy concerns, many apps
  sell user data."
- **Rich progress visualisation** with charts, trends, milestones and celebrations. P2 said "Seeing
  stats is good, but I want celebrations."

The interface direction selected in Lab 03 was **Variant 3, Data-Focused and Detailed**, with a
weighted score of 8.20. That design is full of charts, so rendering performance on mid-range
Android hardware is a real constraint here and not a nice-to-have. The team is mid-sized, health
data falls under GDPR, and the product has to be ready for HIPAA in case it integrates with a
provider later on.

Three candidate stacks were evaluated with a weighted decision matrix. The full scoring sits in
`docs/comparison-matrix.md`.

| Stack | Composition | Weighted score |
|---|---|---|
| **A** | Flutter and Next.js, NestJS and FastAPI, PostgreSQL with Redis and S3, Auth0, AWS ECS | **8.64** |
| **B** | React Native, Firebase Cloud Functions, Firestore, Firebase Auth | 7.49 |
| **C** | Swift/SwiftUI and Kotlin native, Go on Lambda, DynamoDB, Cognito | 7.19 |

---

## Decision

FitFlow will be built on **Stack A**:

- **Flutter 3.x** for the iOS and Android clients, with **Next.js 15** for the web experience.
- **NestJS (Node.js and TypeScript)** for the core API, with a separate **Python 3.12 and FastAPI**
  microservice for all machine-learning inference.
- **PostgreSQL 16** with the **TimescaleDB** extension as the system of record, **Redis 7** for
  caching and real-time pub/sub, and **Amazon S3 with CloudFront** for media.
- **Auth0** for authentication and authorisation.
- **Docker on AWS ECS Fargate**, behind CloudFront, AWS WAF and API Gateway.

---

## Rationale

1. **Performance where it shows.** Flutter compiles to native ARM and renders through its own
   engine instead of handing work to platform widgets across a bridge. It holds 60 to 120 fps on
   the chart-heavy Variant 3 screens, which is exactly where React Native still pays a bridge cost.
2. **One mobile codebase, plus a web app that feels native.** Flutter gives roughly 90% code reuse
   across iOS and Android. Flutter Web was turned down for the public web surface because its
   canvas renderer hurts SEO and first-load time, so Next.js serves the web experience against the
   same API. This is a deliberate hybrid, not a compromise forced on the team.
3. **The right language for each workload.** TypeScript covers the API and the web client, so
   contracts are written once. Python owns inference, because PyTorch, Hugging Face and
   scikit-learn have no real equivalent in the Node ecosystem. Splitting them lets GPU-bound
   inference scale separately from latency-sensitive CRUD traffic.
4. **Relational data belongs in a relational database.** Users, plans, sessions, sets, meals and
   achievements are deeply interrelated, and progress trends are window-function queries.
   PostgreSQL scored 8.69 against MongoDB at 7.60, DynamoDB at 7.03 and Firestore at 6.62.
   Row-level security implements the privacy requirement inside the database instead of leaving it
   to application code alone.
5. **Onboarding friction is really an auth problem.** Auth0 social and passwordless sign-in takes
   account creation out of the five-question budget completely, and it brings MFA, RBAC, SOC 2
   Type II, a GDPR DPA and a HIPAA-eligible tier along with it.

---

## Alternatives considered

| Alternative | Why it was rejected |
|---|---|
| **Stack B, React Native with Firebase** | Fastest to a first release, with excellent real-time support, but Firestore cannot express the analytical queries behind progress trends, cost becomes unpredictable at scale, and the compliance position is weaker for health data |
| **Stack C, fully native with serverless** | The highest raw performance, but no cross-platform reuse, no web target, and roughly double the team cost. It scored lowest at 7.19 |
| **Kotlin Multiplatform** | Shares business logic well, but the UI has to be built twice and Kotlin/Wasm is not production-ready, so the seamless-web requirement fails |
| **Go for the core API** | The best runtime performance, but slower feature delivery and a thin ML ecosystem. Kept as a candidate for a future high-throughput ingestion service |
| **AWS Cognito instead of Auth0** | Cheaper at scale and natively integrated with AWS, but a weaker developer experience and a thinner feature set. It scored 7.67 against 7.93 |
| **Flutter Web for the web client** | It would have unified the whole frontend, but the SEO and first-load penalties conflict with user-acquisition goals |

---

## Consequences

### Positive

- One mobile codebase shortens feature delivery time and stops iOS and Android behaviour from
  drifting apart.
- Rendering performance matches native on the data-dense screens the design direction requires.
- ML models can be retrained and redeployed without touching the core API.
- SQL analytics power progress trends and AI feature extraction without a separate warehouse.
- Row-level security and a dedicated Privacy Service make the privacy requirement enforceable
  instead of merely stated.
- GDPR obligations and HIPAA readiness are satisfied by the platform choices from day one.

### Negative and risks

| Risk | How it is handled |
|---|---|
| Dart is unfamiliar to most of the team | A two-week ramp-up, a shared widget library, and pair programming on the first two features |
| Two frontend codebases, Flutter and Next.js | Shared design tokens and a shared OpenAPI client keep them consistent |
| A polyglot backend widens the operational surface | Uniform Docker packaging, one CI pipeline, and OpenTelemetry tracing across both runtimes |
| Auth0 cost grows with MAU | Auth0 holds identity only, so a migration to Cognito is contained to the token-issuing layer if paid MAU passes about 50,000 |
| An AWS-centric design implies some lock-in | Core workloads run in containers on standard PostgreSQL and Redis, both of which are portable |
| AI inference latency on food photos | Warm containers, model quantisation, and an optimistic UI with a pending-log state |

### Neutral

- Team structure moves toward feature-vertical squads instead of platform-based teams.
- A model registry and a retraining pipeline become a permanent part of the delivery process.

---

## Review trigger

This decision gets revisited if any of the following happens: paid monthly active users pass
50,000; FitFlow signs a contract with a health plan or provider that brings it under HIPAA as a
covered entity; or Flutter Web reaches parity on SEO and first-load performance, at which point
folding the web client back into Flutter is worth another look.
