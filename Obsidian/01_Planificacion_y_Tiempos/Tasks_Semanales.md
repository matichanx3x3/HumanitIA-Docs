# Tasks Semanales de Planificación

## Semana 1 y 2 (Completado)
- [x] Configuración de la red tipo *bridge* `edge_network`.
- [x] Creación de imagen personalizada PostgreSQL con `pgvector` y `PostGIS`.
- [x] Configuración de `docker-compose.yml` (Mosquitto, DB, Adminer, Frontend).
- [x] Inicialización FastAPI con rutas, base de datos (SQLAlchemy) y healthcheck.
- [x] Sistema de diseño CSS puro nativo sin Tailwind.
- [x] Componentes Vue 3 (`AppLayout`, `Sidebar`, `SummaryCard`).
- [x] Rutas Base.

## Semana 3: Scripts de Simulación MQTT y Workers (Actual)
- [x] **Crear el Emulador MQTT (Simulador de Sensores)**
  Desarrollar un script en Python (ej. en `app/simulators/`) que publique datos JSON aleatorios/estructurados (humedad, temperatura, pH, nivel de suelo) en tópicos Mosquitto.
- [x] **Implementar el Worker de Ingesta**
  Actualizar `app/workers/mqtt_ingest.py` para suscribirse al broker MQTT, recibir *payloads* del simulador y procesarlos para persistencia en BD vía SQLAlchemy.

## Semana 4 (Próximamente)
- [ ] (A definir basados en avance)
