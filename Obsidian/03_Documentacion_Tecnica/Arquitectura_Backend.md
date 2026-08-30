---
tags:
  - backend
  - podman
  - fastapi
  - microservicios
aliases:
  - Backend
  - API Core
---
# Arquitectura de Backend (Core Lógico)

El software del Hub Central se orquesta utilizando **Podman** (licencia Apache 2.0), una tecnología daemonless que aísla cada proceso, mejora la seguridad y asegura la distribución comercial libre de regalías. Se estratifica en capas conectadas por redes virtuales.

## Los 6 Microservicios Contenedorizados

### Capa 1: Comunicación e Ingesta
1. **Contenedor 1 - Eclipse Mosquitto (Broker MQTT):** Recibe las tramas JSON del Gateway LoRa vía USB.
2. **Contenedor 2 - Worker Ingesta (Python):** Demonio de fondo (Background Worker). Limpia el ruido eléctrico de las lecturas, normaliza datos y realiza escrituras por lotes (batch) hacia la BD.
3. **Contenedor 3 - Worker Multimedia (Bash/Python):** Capturador RTSP que extrae periódicamente fotogramas clave (keyframes) de las cámaras IP (ESP32-CAM) y los guarda en el FS local.

### Capa 2: Persistencia Multimodal (Data & GIS)
4. **Contenedor 4 - PostgreSQL (Motor Principal):** Con *Write-Ahead Logging (WAL)* obligatorio para evitar corrupción por apagones. 
   - *Big Data (Series Temporales):* Particionamiento declarativo por mes.
   - *PostGIS:* Aislado vía TCP/IP, guarda polígonos y coordenadas.
   - *pgvector:* Almacén de embeddings para indexar manuales agrícolas.

### Capa 3: Lógica de Negocio y Core API
5. **Contenedor 5 - FastAPI Central (Core):** Expone información y orquesta reglas.
   - *Motor Determinista:* Reglas If-Then-Else para control de riego ("Activar relé si humedad < 30%").
   - *RBAC:* Seguridad JWT diferenciando Operario vs Agrónomo.
   - *Geo-REST API:* Traduce telemetría a GeoJSON.
   - *AI Stubs (Function Calling):* Funciones envolventes de actuadores para futuras IA.

### Capa 4: Presentación
6. **Contenedor 6 - NGINX (Servidor Web):** Sirve archivos estáticos de la PWA offline (Vue) e imágenes fotográficas guardadas por el Worker Multimedia.

👉 **Visualiza estas conexiones en:** [[Infraestructura_y_Graficos]]

---

## Estructura de Proyecto Sugerida (Clean Architecture)
El código de FastAPI se estructurará siguiendo principios de *Clean Architecture*:

```text
hub_agritech_core/
├── app/
│   ├── api/          # Controladores y rutas (v1/telemetry.py, spatial.py)
│   ├── core/         # Configuración, seguridad (JWT), settings
│   ├── db/           # Modelos SQLAlchemy, PostGIS y migraciones (Alembic)
│   ├── services/     # Lógica de negocio pesada, algoritmos
│   ├── workers/      # Procesos en segundo plano (Ingesta MQTT, RTSP)
│   └── main.py       # Punto de entrada de FastAPI
├── scripts/          # Utilidades (backups, restauraciones)
├── Dockerfile        
├── docker-compose.yml 
└── requirements.txt  
```
