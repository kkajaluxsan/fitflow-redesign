# FitFlow Redesign

Technology evaluation, architecture and project scaffold for the **FitFlow** fitness application
redesign, prepared for IT3060 Human Computer Interaction, Lab Exercise 05.

FitFlow is being rebuilt around five research-driven priorities: AI-powered adaptive workout
plans, camera-based nutrition logging, a short (max 5 question) onboarding, user-controlled
privacy, and rich progress visualisation, delivered as one seamless iOS / Android / Web
experience.

---

## Recommended Technology Stack

| Layer | Technology | Why |
|---|---|---|
| Mobile client | **Flutter 3.x (Dart)** | One codebase for iOS and Android. Impeller and Skia rendering hold the data-heavy Variant 3 UI at 60 to 120 fps |
| Web client | **Next.js 15 (React + TypeScript)** | SEO-indexable marketing and companion dashboard; avoids Flutter Web's large initial payload |
| Shared design system | **Figma tokens → Flutter + Tailwind** | Single source of truth for the selected Variant 3 visual language |
| Core API | **NestJS (Node.js + TypeScript)** | Modular DI architecture, first-class WebSocket/Socket.IO support, large hiring pool |
| AI microservice | **Python 3.12 + FastAPI** | Native access to PyTorch / Hugging Face for the workout planner and food-vision models |
| Primary database | **PostgreSQL 16 (AWS RDS Multi-AZ)** | Relational integrity for health records, window functions for progress trends, row-level security |
| Time-series metrics | **TimescaleDB extension** | Efficient storage/roll-up of steps, weight, heart-rate and hydration series |
| Cache + real-time bus | **Redis 7 (ElastiCache)** | Feed caching, leaderboard sorted-sets, Socket.IO pub/sub adapter |
| Object storage | **Amazon S3 + CloudFront** | Meal photos, exercise videos, avatars; presigned direct uploads |
| Authentication | **Auth0 (OIDC / JWT)** | MFA, social + passwordless login, RBAC, SOC 2 Type II, GDPR DPA, HIPAA-ready tier |
| Async messaging | **Amazon SQS + EventBridge** | Decouples workout/nutrition events from analytics and achievement processing |
| Containers / hosting | **Docker + AWS ECS Fargate** | Independent scaling of API and AI workloads without cluster maintenance |
| Observability | **OpenTelemetry + CloudWatch + Sentry** | Distributed tracing and crash reporting across all services |

Full scoring and justification: [`docs/comparison-matrix.md`](docs/comparison-matrix.md) and
[`docs/tech-stack.md`](docs/tech-stack.md).

---

## Repository Structure

```
fitflow-redesign/
├── .github/workflows/ci.yml     # Lint, test and build pipeline for all three services
├── frontend/                    # Flutter mobile app + Next.js web client
│   ├── lib/                     # Dart source (features, widgets, state)
│   ├── web/                     # Next.js web client
│   └── assets/                  # Fonts, icons, design tokens
├── backend/                     # NestJS core API
│   ├── src/                     # Feature modules (auth, workout, nutrition, social, progress)
│   └── test/                    # Unit and e2e tests
├── ai-service/                  # Python FastAPI AI microservice
│   ├── app/                     # Routers, inference services, schemas
│   └── models/                  # Model artefacts and training notebooks
├── infra/                       # Terraform / IaC for AWS resources
├── docs/                        # Lab 05 deliverables
│   ├── tech-stack.md            # Technology summary and justification
│   ├── comparison-matrix.md     # Weighted decision matrices (Activities 1 to 3)
│   ├── architecture.md          # High-level architecture and data flows (Activity 4)
│   ├── adr/                     # Architecture Decision Records
│   └── diagrams/                # Architecture and data-flow diagrams
├── .env.example
├── .gitignore
└── README.md
```

---

## Getting Started

```bash
git clone https://github.com/<your-username>/fitflow-redesign.git
cd fitflow-redesign
cp .env.example .env

# Core API
cd backend && npm install && npm run start:dev

# AI microservice
cd ../ai-service && pip install -r requirements.txt && uvicorn app.main:app --reload

# Mobile client
cd ../frontend && flutter pub get && flutter run
```

---

## Documentation

| Document | Contents |
|---|---|
| [docs/tech-stack.md](docs/tech-stack.md) | Chosen stack, per-layer rationale, rejected alternatives |
| [docs/comparison-matrix.md](docs/comparison-matrix.md) | Frontend, backend, database, auth and consolidated weighted matrices |
| [docs/architecture.md](docs/architecture.md) | Components, data flows, security, scalability, integrations |
| [docs/adr/ADR-001-technology-stack.md](docs/adr/ADR-001-technology-stack.md) | Architecture Decision Record for the stack selection |
| [docs/diagrams/](docs/diagrams/) | High-level architecture and nutrition data-flow diagrams |

---

## Repository Settings

| Setting | Configuration |
|---|---|
| Default branch | `main` |
| Branch protection | `main` requires a pull request, 1 approving review, and passing CI before merge; force-push and deletion blocked |
| Working branches | `develop`, `feature/*`, `fix/*` |
| CI/CD | GitHub Actions runs lint, test and build on every push and pull request |
| Secrets | Stored as GitHub Actions secrets; `.env` is git-ignored |

---

## Contributing

1. Branch from `develop`: `git checkout -b feature/<short-name>`
2. Commit using Conventional Commits (`feat:`, `fix:`, `docs:`, `chore:`)
3. Open a pull request into `develop`; CI must pass and one review is required
4. `develop` is merged into `main` for each release

---

## Licence

Academic coursework submission for SLIIT, BSc (Hons) in Information Technology, Year 3,
IT3060 Human Computer Interaction, Semester 2 2026.
