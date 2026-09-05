---
name: agritech-fastapi-backend
description: >-
  Develops and extends the FastAPI backend microservice, REST API endpoints (/api/v1/sensors/*), SQLAlchemy / GeoAlchemy2 ORM queries, PostGIS geospatial queries, pgvector embeddings, and CORS configurations. Use when adding endpoints, optimizing database queries, or handling API models.
---

# Agritech FastAPI Backend Runbook

This skill provides guidelines and patterns for building and maintaining the FastAPI REST API microservice (`app/main.py`, `app/api/v1/endpoints.py`).

## API Structure

```
app/
├── main.py                # App entrypoint, CORS configuration, router mounting
├── api/
│   └── v1/
│       └── endpoints.py   # REST API routes under /api/v1
├── db/
│   ├── database.py        # SQLAlchemy engine, SessionLocal, get_db dependency
│   └── models.py          # ORM models (SensorData, Base)
├── simulators/            # Sensor simulation scripts
└── workers/               # Ingestion background workers
```

## Existing Endpoints (`/api/v1`)

| Method | Path | Description |
| :--- | :--- | :--- |
| `GET` | `/` | Root health status |
| `GET` | `/api/v1/status` | Microservice & DB connection check |
| `GET` | `/api/v1/sensors` | List of distinct active sensor `node_id`s |
| `GET` | `/api/v1/sensors/summary` | Latest reading (temperature, humidity, pH, soil moisture) per node |
| `GET` | `/api/v1/sensors/{node_id}/history` | Historical readings in chronological order (query param `limit`) |
