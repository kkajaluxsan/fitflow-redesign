# FitFlow Core API

NestJS (Node.js + TypeScript) REST and WebSocket API.

## Modules
| Module | Responsibility |
|---|---|
| `identity` | Auth0 token validation, profile, onboarding answers, consent |
| `workout` | Plans, sessions, sets/reps, AI plan orchestration |
| `nutrition` | Meal logs, macros, food lookup, photo log reconciliation |
| `social` | Groups, feed, posts, likes, comments, challenges |
| `progress` | Trends, streaks, achievements, milestone evaluation |
| `notification` | Push via FCM/APNs, in app messages |
| `privacy` | Per field sharing settings, GDPR export and erasure |

## Commands
```bash
npm install
npm run start:dev
npm run test
npm run test:e2e
npm run build
```

## Configuration
Copy `../.env.example` to `../.env` and fill in database, Redis, Auth0 and AWS values.
