---
name: agritech-stack-ops
description: >-
  Manages, builds, and orchestrates the multi-container edge infrastructure for Agritech Core using Podman or Docker Compose (Mosquitto, PostgreSQL+PostGIS+pgvector, Ingestion Worker, FastAPI, Vue Dashboard Nginx, Adminer). Use when starting, stopping, rebuilding, inspecting, or debugging containers, network ports, and environment variables.
---

# Agritech Stack Operations & Container Orchestration

This skill provides step-by-step instructions and runbooks for orchestrating the multi-service architecture of **Agritech HumanitIA**.

## Service Architecture & Port Mappings

| Service | Container Name | Port Mapping | Description |
| :--- | :--- | :--- | :--- |
| **Mosquitto** | `hub_mosquitto` | `1883:1883` (MQTT), `9001:9001` (WS) | MQTT broker for ESP32 and edge gateway |
| **PostgreSQL** | `hub_postgres` | `5435:5432` | PostGIS + pgvector database (`agritech_db`) |
| **Adminer** | `adminer` | `8080:8080` | Web UI for database inspection |
| **Worker Ingesta** | `hub_worker_ingesta` | *internal only* | Python paho-mqtt subscriber storing to PostgreSQL |
| **Sensor Simulator**| `hub_simulator` | *internal only* | Python test telemetry generator |
| **FastAPI Core** | `hub_fastapi` | `8000:8000` | REST API (`/api/v1/...`) |
| **Frontend Nginx** | `hub_frontend_nginx` | `8081:80` | Vue 3 production build served via Nginx |

> [!IMPORTANT]
> **Podman Rootless Port Rule**: Ports under 1024 (e.g., standard port 80) cannot be bound in rootless Podman/WSL environments without root privileges. The frontend is exposed on port **8081** and Postgres on host port **5435** to prevent binding conflicts.

## Supported Host Platforms & macOS Exclusion

> [!WARNING]
> **macOS DESCARTADO / EXCLUIDO PARA EL STACK PODMAN**:
> El entorno multi-contenedor de Podman **NO** está soportado en macOS. Podman Machine sobre macOS (QEMU / Apple Virtualization framework) genera fallos sistemáticos al montar volúmenes (`permission denied`, bloqueos de sockets gvproxy, fallos de sincronización virtiofs/9p y mapeos de red en Nginx y PostgreSQL).
> 
> **Plataformas Oficiales Requeridas y Soportadas:**
> 1. **Linux Nativo (Plataforma Oficial - Mini PC Edge en finca):** Ubuntu 22.04+, Fedora 38+, Debian 12, Arch. Podman rootless opera con soporte nativo de namespaces del kernel.
> 2. **Windows 10/11:** Utilizando WSL2 con distribución Linux (ej. Ubuntu).

## Common Operations

### 1. Starting the Entire Stack
```bash
# Using podman-compose
podman-compose up -d

# Or using docker compose
docker compose up -d
```

### 2. Rebuilding Specific Services After Code Changes
```bash
# Rebuild Backend
podman-compose build api_core && podman-compose up -d api_core

# Rebuild Frontend
podman-compose build frontend_vue && podman-compose up -d frontend_vue

# Rebuild Worker
podman-compose build worker_ingesta && podman-compose up -d worker_ingesta
```

### 3. Inspecting Logs in Real Time
```bash
# Ingestion worker logs
podman logs -f hub_worker_ingesta

# FastAPI logs
podman logs -f hub_fastapi

# Mosquitto MQTT logs
podman logs -f hub_mosquitto
```

### 4. Database Access & Verification
- **Adminer URL**: `http://localhost:8080`
  - System: `PostgreSQL`
  - Server: `postgres_db` (inside network) or `localhost:5435` (external)
  - Username: `agritech_user`
  - Password: `SuperSecretPass`
  - Database: `agritech_db`
- **CLI Inspection**:
  ```bash
  podman exec -it hub_postgres psql -U agritech_user -d agritech_db -c "SELECT COUNT(*) FROM sensor_data;"
  ```

### 5. Troubleshooting & Healthcheck
1. Check running containers:
   ```bash
   podman ps -a
   ```
2. Check internal network connectivity:
   ```bash
   podman network inspect hub_edge_network
   ```
3. Restart stuck services:
   ```bash
   podman restart hub_worker_ingesta hub_fastapi
   ```
