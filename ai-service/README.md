# FitFlow AI Service

Python 3.12 + FastAPI microservice for all machine-learning inference.

## Endpoints
| Endpoint | Purpose |
|---|---|
| `POST /infer/plan` | Adaptive workout plan from session history, goals and injury constraints |
| `POST /infer/food` | Food recognition and portion estimation from a meal photo |
| `POST /feedback` | Stores user corrections as labelled training data |
| `GET /health` | Liveness and model-version probe |

## Structure
```
ai-service/
├── app/
│   ├── main.py       # FastAPI application
│   ├── routers/      # Endpoint definitions
│   ├── services/     # Plan engine, food vision, feature store client
│   └── schemas/      # Pydantic request/response models
└── models/           # Model artefacts (git-ignored) and training notebooks
```

## Commands
```bash
python -m venv .venv && source .venv/bin/activate
pip install -r requirements.txt
uvicorn app.main:app --reload --port 8000
pytest -q
```
