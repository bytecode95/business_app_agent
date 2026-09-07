# CRM Intelligence RAG — Complete Production Folder Structure

```
multi-agent-rag/
│
├── app/                                      # Main application package
│   ├── main.py                               # FastAPI entrypoint — /ask, /history, /health, /sync/trigger
│   │
│   ├── agents/                               # Individual pipeline agents
│   │   ├── router_agent.py                   # Classifies intent → module(s) + strategy (vector/api/both)
│   │   ├── query_rewriter.py                 # NEW — NL query → structured CRM filter params
│   │   ├── crm_retriever_agent.py            # NEW — Chroma query with permission where-filter
│   │   ├── retriever_agent.py                # Base retriever (generic vector search)
│   │   ├── web_search_agent.py               # Fallback DuckDuckGo (non-CRM queries only)
│   │   ├── synthesizer_agent.py              # Drafts answer from retrieved context
│   │   └── critic_agent.py                   # Grounding check + retry (max 2)
│   │
│   ├── orchestrator/
│   │   └── pipeline.py                       # Wires all agents end-to-end
│   │
│   ├── retrieval/                            # RAG plumbing
│   │   ├── loaders.py                        # Chunk Documents from any source
│   │   ├── embeddings.py                     # HuggingFace embedding wrapper
│   │   └── vectorstore.py                    # Chroma: build, upsert, retrieve, delete
│   │
│   ├── crm_fetcher/                          # NEW — Read-only NestJS CRM API client
│   │   ├── __init__.py
│   │   ├── crm_client.py                     # Async httpx client, JSON-RPC 2.0 helper
│   │   ├── auth.py                           # Service-account JWT: sign-in + refresh loop
│   │   └── module_fetchers/
│   │       ├── __init__.py
│   │       ├── leads.py                      # listCrmLeads, listCrmLeadStages
│   │       ├── tickets.py                    # listHelpDeskTickets, getTicketById
│   │       ├── visits.py                     # listVisitPlans, listOffTakeObservations
│   │       ├── expenses.py                   # listExpenseClaims, findExpenseMeasurements
│   │       ├── territories.py                # listSalesTerritories, getTerritory
│   │       ├── sales.py                      # computePerformance, listSalesTargets
│   │       ├── contacts.py                   # listPartnerContacts
│   │       ├── campaigns.py                  # listCrmCampaigns
│   │       └── products.py                   # listProductTemplates
│   │
│   ├── crm_sync/                             # NEW — Periodic sync: CRM → Chroma
│   │   ├── __init__.py
│   │   ├── sync_scheduler.py                 # APScheduler: module sync jobs per tenant
│   │   ├── sync_runner.py                    # Fetch → chunk → embed → upsert per module
│   │   └── delta_tracker.py                  # Tracks last-synced timestamp per module+tenant
│   │
│   ├── models/
│   │   ├── llm_client.py                     # Gemini API wrapper
│   │   └── schemas.py                        # Pydantic: AskRequest, AskResponse, SyncStatus
│   │
│   ├── db/                                   # Postgres persistence (SQLAlchemy)
│   │   ├── database.py                       # Engine, session, Base
│   │   ├── models.py                         # QueryLog, QuerySource, EvalResult, SyncLog (NEW)
│   │   └── crud.py                           # Query/insert helpers
│   │
│   ├── utils/                                # NEW — Shared utilities
│   │   ├── __init__.py
│   │   ├── prompts.py                        # ROUTER_PROMPT, SYNTHESIZER_PROMPT, CRITIC_PROMPT, REWRITER_PROMPT
│   │   ├── logger.py                         # Structured logging (JSON format for prod)
│   │   ├── retry.py                          # NEW — Exponential backoff decorator
│   │   ├── token_counter.py                  # NEW — Count tokens before LLM calls
│   │   └── exceptions.py                     # NEW — Custom exception classes
│   │
│   └── config.py                             # Settings via pydantic-settings (reads .env)
│
├── scripts/                                  # One-off operational scripts
│   ├── init_db.py                            # Creates all Postgres tables
│   ├── ingest_documents.py                   # Legacy: loads data/raw/* into vector store
│   ├── trigger_sync.py                       # NEW — Manually trigger CRM sync for a tenant
│   └── evaluate_rag.py                       # Runs eval_dataset.json through pipeline
│
├── tests/
│   ├── unit/
│   │   ├── test_router_agent.py
│   │   ├── test_query_rewriter.py            # NEW
│   │   ├── test_permission_filter.py         # NEW
│   │   └── test_crm_client.py                # NEW — mock httpx calls
│   ├── integration/
│   │   ├── test_pipeline.py                  # End-to-end pipeline test
│   │   └── test_crm_sync.py                  # NEW — sync runner against mock CRM
│   └── eval_dataset.json                     # Labeled CRM Q&A pairs
│
├── data/
│   ├── raw/                                  # Static seed files (if any)
│   └── processed/                            # Reserved for cleaned intermediate data
│
├── deployment/                               # NEW — All deployment config
│   ├── docker/
│   │   ├── Dockerfile                        # Multi-stage: builder + slim runtime
│   │   └── .dockerignore
│   ├── docker-compose.yml                    # Local dev: FastAPI + Postgres + Chroma
│   ├── docker-compose.prod.yml               # Prod overrides: no volumes, restart policy
│   │
│   ├── k8s/                                  # Kubernetes manifests (if needed)
│   │   ├── namespace.yaml
│   │   ├── deployment.yaml                   # RAG service deployment
│   │   ├── service.yaml                      # ClusterIP / LoadBalancer
│   │   ├── configmap.yaml                    # Non-secret env vars
│   │   ├── secret.yaml                       # Sealed secrets (do not commit plaintext)
│   │   ├── hpa.yaml                          # Horizontal pod autoscaler
│   │   └── ingress.yaml                      # NGINX ingress + TLS
│   │
│   └── nginx/
│       └── nginx.conf                        # Reverse proxy config for local docker-compose
│
├── .github/
│   └── workflows/
│       ├── ci.yml                            # NEW — Lint + test on every PR
│       └── deploy.yml                        # NEW — Build → push image → deploy on merge to main
│
├── notebooks/                                # Exploratory analysis
│
├── docs/
│   ├── folder_structure.md                   # This file
│   ├── api_reference.md                      # FastAPI endpoint docs
│   ├── crm_module_mapping.md                 # Which CRM APIs map to which RAG module
│   └── architecture.md                       # Diagram + decisions
│
├── .env.example                              # All required env vars with descriptions
├── .env.test                                 # Test-only overrides (no secrets)
├── .gitignore
├── pyproject.toml                            # NEW — replaces requirements.txt (uv/poetry)
├── requirements.txt                          # Pinned deps (kept for compatibility)
└── README.md
```

---

## Key file contents guide

### app/config.py — centralised settings
```python
from pydantic_settings import BaseSettings

class Settings(BaseSettings):
    # NestJS CRM
    crm_base_url: str                        # e.g. https://api.yourcrm.com
    crm_service_login: str                   # service-account username
    crm_service_password: str                # service-account password
    crm_token_refresh_interval: int = 50     # minutes

    # Vector store
    chroma_host: str = "localhost"
    chroma_port: int = 8000
    chroma_persist_dir: str = "./chroma_db"

    # LLM
    gemini_api_key: str
    gemini_model: str = "gemini-1.5-pro"

    # Postgres
    database_url: str

    # Sync
    sync_interval_minutes: int = 30
    sync_modules: list[str] = ["leads","tickets","visits","expenses","territories","sales"]

    # RAG tuning
    chunk_size: int = 512
    chunk_overlap: int = 64
    retrieval_top_k: int = 8
    critic_max_retries: int = 2

    class Config:
        env_file = ".env"

settings = Settings()
```

### .env.example — complete variable list
```
# NestJS CRM backend
CRM_BASE_URL=https://api.yourcrm.com
CRM_SERVICE_LOGIN=rag-service-account
CRM_SERVICE_PASSWORD=

# Chroma vector store
CHROMA_HOST=localhost
CHROMA_PORT=8000
CHROMA_PERSIST_DIR=./chroma_db

# LLM
GEMINI_API_KEY=

# PostgreSQL
DATABASE_URL=postgresql+asyncpg://user:pass@localhost:5432/crm_rag

# Sync schedule
SYNC_INTERVAL_MINUTES=30
SYNC_MODULES=leads,tickets,visits,expenses,territories,sales

# RAG tuning
CHUNK_SIZE=512
CHUNK_OVERLAP=64
RETRIEVAL_TOP_K=8
CRITIC_MAX_RETRIES=2
```

### deployment/docker/Dockerfile — multi-stage
```dockerfile
# Stage 1: builder
FROM python:3.12-slim AS builder
WORKDIR /build
COPY requirements.txt .
RUN pip install --no-cache-dir --prefix=/install -r requirements.txt

# Stage 2: runtime
FROM python:3.12-slim
WORKDIR /app
COPY --from=builder /install /usr/local
COPY app/ ./app/
COPY scripts/ ./scripts/
COPY .env.example .env.example
ENV PYTHONPATH=/app
EXPOSE 8000
CMD ["uvicorn", "app.main:app", "--host", "0.0.0.0", "--port", "8000", "--workers", "2"]
```

### deployment/docker-compose.yml — local dev
```yaml
version: "3.9"
services:
  rag:
    build:
      context: .
      dockerfile: deployment/docker/Dockerfile
    env_file: .env
    ports: ["8000:8000"]
    depends_on: [postgres, chroma]
    volumes:
      - ./app:/app/app          # hot-reload in dev
      - chroma_data:/app/chroma_db

  postgres:
    image: postgres:16-alpine
    environment:
      POSTGRES_USER: raguser
      POSTGRES_PASSWORD: ragpass
      POSTGRES_DB: crm_rag
    ports: ["5432:5432"]
    volumes: [pg_data:/var/lib/postgresql/data]

  chroma:
    image: chromadb/chroma:latest
    ports: ["8001:8000"]
    volumes: [chroma_data:/chroma/chroma]

volumes:
  pg_data:
  chroma_data:
```

### .github/workflows/ci.yml
```yaml
name: CI
on: [pull_request]
jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-python@v5
        with: { python-version: "3.12" }
      - run: pip install -r requirements.txt
      - run: pip install ruff pytest pytest-asyncio
      - run: ruff check app/ tests/
      - run: pytest tests/unit/ -v
```
