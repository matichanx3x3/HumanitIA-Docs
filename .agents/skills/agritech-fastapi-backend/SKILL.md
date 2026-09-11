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

## Standard Patterns for New Endpoints

### 1. Database Dependency Injection
Always inject database sessions using FastAPI `Depends(get_db)`:

```python
from fastapi import APIRouter, Depends, HTTPException, Query
from sqlalchemy.orm import Session
from app.db.database import get_db
from app.db.models import SensorData

router = APIRouter()

@router.get("/sensors/{node_id}/stats")
def get_sensor_stats(node_id: str, db: Session = Depends(get_db)):
    # Perform query
    pass
```

### 2. Time-Series & Aggregation Queries
When building summary or analytics endpoints, compute metrics efficiently in SQL:

```python
from sqlalchemy import func

@router.get("/sensors/{node_id}/averages")
def get_sensor_averages(node_id: str, db: Session = Depends(get_db)):
    avg_data = db.query(
        func.avg(SensorData.temperature).label("avg_temp"),
        func.avg(SensorData.humidity).label("avg_hum"),
        func.avg(SensorData.soil_moisture).label("avg_moisture"),
        func.avg(SensorData.ph).label("avg_ph")
    ).filter(SensorData.node_id == node_id).first()
    
    return {
        "node_id": node_id,
        "avg_temperature": round(avg_data.avg_temp, 2) if avg_data.avg_temp else None,
        "avg_humidity": round(avg_data.avg_hum, 2) if avg_data.avg_hum else None,
        "avg_soil_moisture": round(avg_data.avg_moisture, 2) if avg_data.avg_moisture else None,
        "avg_ph": round(avg_data.avg_ph, 2) if avg_data.avg_ph else None
    }
```

### 3. PostGIS & pgvector Support
The PostgreSQL database is preconfigured with PostGIS and pgvector:
- For spatial queries, import `Geometry` from `geoalchemy2`.
- For vector embeddings (Edge AI recommendations), use `pgvector.sqlalchemy.Vector`.

## Local Development & Debugging

```bash
# Run FastAPI with live reload (from project root)
uvicorn app.main:app --host 0.0.0.0 --port 8000 --reload

# Interactive Swagger Documentation
# Open browser at: http://localhost:8000/docs
```
