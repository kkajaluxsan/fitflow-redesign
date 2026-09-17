# FitFlow Technology Stack Summary

## 1. Requirements that drove the selection

The stack was chosen to serve the prioritised requirements captured in the Lab 03 user research.

| Requirement (priority) | What it means technically |
|---|---|
| AI-powered adaptive workout plans (High) | A Python ML runtime, a feature store, and an event-driven retraining loop |
| Camera-based nutrition logging (High) | On-device capture, fast image upload, and computer-vision inference in under 3 seconds |
| Short onboarding, max 5 questions (High) | Social and passwordless sign-in, progressive profiling, and a cold-start recommendation model |
| Privacy controls (High) | A per-field consent model, row-level security, and GDPR export and erase endpoints |
| Progress trends (High) | Time-series storage, window-function aggregation, and smooth chart rendering |
| Support groups, celebrations, notifications (Medium) | A real-time feed, WebSockets, push notifications, and event-driven achievements |
| Seamless iOS, Android and web experience | A shared codebase plus a web client that search engines can index |

---

## 2. Selected stack

### Frontend: Flutter on mobile, Next.js on web

Flutter scored **7.95 out of 10**, ahead of React Native at 7.73, Kotlin Multiplatform at 6.80 and
Swift/SwiftUI at 6.03. It compiles to native ARM code and draws through its own Impeller rendering
engine, so the chart-heavy "Variant 3, Data-Focused" interface selected in Lab 03 scrolls and
animates at 60 to 120 fps on mid-range Android devices. A JavaScript-bridged framework would drop
frames on those same screens. One Dart codebase covers both mobile platforms, which shortens
delivery time and removes the iOS and Android behaviour drift that two native teams tend to
produce over time.

Flutter Web is not used for the public web experience. Its canvas-based renderer sends a large
payload on first load and search engines cannot index it reliably, which works against FitFlow's
user-acquisition goals. A **Next.js 15** client serves the marketing site and the web dashboard
instead, consuming the same REST and WebSocket API and the same Figma design tokens. This split
keeps native-class performance where users spend their time and web-class reach where FitFlow
needs to be found.

### Backend: a NestJS core API with a FastAPI AI microservice

NestJS scored **8.11 out of 10**. Its module and provider architecture maps cleanly onto the
FitFlow bounded contexts: identity, workout, nutrition, social, progress, notification and privacy.
Socket.IO integration is built in for the community feed, and TypeScript is shared with the Next.js
client, so DTOs and validation schemas only get written once.

The AI workload sits in its own **Python 3.12 and FastAPI** microservice. It scored 7.89 overall
but 10 out of 10 for AI ecosystem, and that single score explains the split. Adaptive workout
planning and food-image recognition need PyTorch, Hugging Face and scikit-learn. Running them
in-process with the API would tie GPU-bound inference to latency-sensitive CRUD traffic. As
separate services they scale on their own terms, and models can be redeployed without touching the
core API.

### Database: PostgreSQL 16 with TimescaleDB, Redis and S3

PostgreSQL scored **8.69 out of 10**, well ahead of MongoDB at 7.60, DynamoDB at 7.03 and Firestore
at 6.62. Health data is highly relational. Users, plans, sessions, sets, foods, meals and
achievements all reference one another, and referential integrity plus ACID transactions matter
when a workout log feeds both a progress chart and an AI training set. Window functions and
`GROUP BY` rollups compute weight trends and streaks inside the database. Row-level security
enforces the privacy controls users asked for, and pgcrypto gives field-level encryption for
sensitive metrics.

The **TimescaleDB** extension adds hypertables and continuous aggregates for high-volume
time-series data such as steps, heart rate, hydration and weight, which avoids introducing a second
database engine. **Redis 7** caches feeds and daily nutrition totals, backs leaderboards with
sorted sets, and carries the Socket.IO pub/sub adapter. **Amazon S3 with CloudFront** stores meal
photos and exercise media, uploaded straight from the client through presigned URLs so image bytes
never pass through the API.

### Authentication: Auth0

Auth0 scored **7.93 out of 10**, narrowly ahead of Supabase Auth at 7.80 and AWS Cognito at 7.67.
It offers the shortest path to the five-question onboarding limit. Apple, Google and passwordless
email sign-in remove account creation from the question budget entirely, and Actions allow profile
details to be collected gradually after first use. It also provides MFA, RBAC, anomaly detection,
brute-force protection, SOC 2 Type II certification, a GDPR Data Processing Addendum, EU data
residency and a HIPAA-eligible tier.

Cost is its weak point, scoring 4 out of 10. The architecture works around this. Auth0 holds
identity and credentials only, while all profile and health data lives in PostgreSQL keyed by the
Auth0 `sub` claim. If paid monthly active users pass roughly 50,000, moving to AWS Cognito would be
a contained change affecting only the token-issuing layer.

### Platform services

| Concern | Technology |
|---|---|
| Containers and orchestration | Docker with AWS ECS Fargate |
| Edge and API entry | CloudFront, AWS WAF, API Gateway / ALB |
| Async events | Amazon SQS with EventBridge |
| Push notifications | Firebase Cloud Messaging on Android, APNs on iOS |
| CI/CD | GitHub Actions into ECR and ECS, with Codemagic or Fastlane for store builds |
| Observability | OpenTelemetry, CloudWatch, Sentry |
| Secrets | AWS Secrets Manager with KMS |

---

## 3. Alternatives considered and rejected

| Option | Why it was rejected |
|---|---|
| React Native for the mobile client | A strong web story and a large hiring pool, but JS-bridge overhead shows on chart-heavy screens, and the heavier reliance on third-party native modules raises long-term maintenance risk |
| Kotlin Multiplatform | Excellent native performance and clean logic sharing, but the UI has to be written twice and Kotlin/Wasm web support is not production-ready, so the seamless-web requirement fails |
| Swift/SwiftUI native, plus Kotlin for Android | The best raw iOS performance, but no code reuse, no web target, and roughly double the team cost. It scored lowest in the frontend matrix at 6.03 |
| A Firebase and Firestore backend (Stack B) | Fastest to build with excellent real-time support, but limited query and aggregation power for progress analytics, unpredictable cost at scale, and a weaker compliance position for health data. Consolidated score 7.49 |
| DynamoDB | Outstanding scalability, but modelling access patterns first makes the ad-hoc analytical queries behind progress trends and AI feature extraction expensive and rigid |
| Go for the core API | The best runtime performance at 10 out of 10, but slower feature delivery and a thin ML ecosystem. Kept as an option for a future high-throughput ingestion service |
| MongoDB | A flexible schema suits feed documents, but it gives up the relational integrity and SQL analytics that health and progress reporting depend on |

---

## 4. Compliance position

FitFlow is a consumer wellness product, so it is generally not a HIPAA covered entity. HIPAA only
applies if FitFlow integrates with a health plan or a provider. The architecture is still built to
HIPAA-eligible standards so that such an integration would not force a re-platform: BAA-eligible
AWS services, encryption in transit through TLS 1.3 and at rest through AES-256 with KMS, audit
logging, and least-privilege IAM.

GDPR applies no matter what. It is handled through EU data residency, explicit consent captured
during onboarding, per-field privacy controls, documented retention periods, and API endpoints for
data export under Article 20 and erasure under Article 17. Erasure cascades into S3 media and
cached feed entries as well as the database.
