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

## Semana 4: Consolidación Serial, Hardware y Puesta en Marcha (Actual)
- [x] **Refactorización Serial-LoRa P2P:** Migrar `heltec_lora_sim.py` y el listener para utilizar `pyserial` puro leyendo tramas nativas (RadioLib), eliminando la dependencia de `meshtastic` para cumplir la regla de bajo consumo.
- [x] **Detección Dinámica de Puertos:** Añadir soporte multiplataforma al script serial para autodetectar `/dev/ttyUSB*` / `/dev/ttyACM*` (Linux), `/dev/cu.*` (macOS) o `COM*` (Windows).
- [x] **Integración Sensor Suelo RS485 Modbus-RTU Validada:** Conexión física con masa común (GND) y alimentación 5V para transceptor MAX485, pines GPIO 4/5 en Heltec V4, tramas oficiales calibradas según Manual V2.2 (Dirección 02, 9600 bps, 8 registros con salinidad). Lectura en vivo y transmisión LoRa 100% operativa. [[Bitacora_Puesta_en_Marcha_Sensor_RS485_2026-09-22|Ver Bitácora de Depuración]].
- [x] **Downlink LoRa P2P Clase A Sincronizado:** Configuración remota del intervalo de transmisión del emisor desde la web (1s a 12h), ventana RX de 3s con interrupción `LORA_DIO1`, persistencia en memoria Flash NVS (`Preferences.h`) y encolamiento inteligente en `lora_serial_listener.py`. [[Bitacora_Downlink_LoRa_y_Stack_Podman_2026-09-23|Ver Bitácora Técnica]].
- [x] **Despliegue y Estabilidad Podman (WSL2):** Resolución de registries en `/etc/containers/registries.conf`, manejo ordenado de `SIGTERM` en worker MQTT, y corrección definitiva del error 404 en NGINX para SPA con `try_files`.
- [x] **Renovación Ergonómica de la WebApp:** Separación de vista Overview limpia vs vista detallada de 8 parámetros en Sensores y Dispositivos, modales de exportación (CSV / PDF) y configuración remota, y función de purga de nodos dummy.
- [ ] **Capas Geoespaciales PostGIS:** Generar endpoints GeoJSON y visualización de polígonos/parcelas en el mapa del Dashboard.
- [ ] **Alertas Agronómicas Inteligentes:** Reglas umbral automáticas (acidez, salinidad, sequía) en FastAPI y notificaciones en UI.
