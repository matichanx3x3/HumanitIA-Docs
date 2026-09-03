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

## Semana 4: Consolidación Serial y Tiempo Real (Próximamente)
- [ ] **Refactorización Serial-LoRa P2P:** Migrar `heltec_lora_sim.py` y el listener para utilizar `pyserial` puro leyendo tramas nativas (RadioLib), eliminando la dependencia de `meshtastic` para cumplir la regla de bajo consumo.
- [ ] **Detección Dinámica de Puertos:** Añadir soporte multiplataforma al script serial para autodetectar `/dev/ttyUSB*` (Linux) o `COM*` (Windows).
- [ ] **WebSockets en Dashboard:** Implementar WebSockets en FastAPI y Vue 3 para actualizar las métricas interactivas (`SummaryCard`, gráficos) en tiempo real sin polling.
- [ ] **Integración Geoespacial Base:** Habilitar el consumo de GeoJSON desde PostGIS para el mapa de parcelas en el frontend.
