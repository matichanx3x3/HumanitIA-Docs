# Contexto y Estado del Proyecto: Hub Agritech Core

Este documento sirve como resumen completo del progreso, decisiones arquitectónicas y próximos pasos para transferir el contexto a otro agente o continuar el desarrollo en una nueva sesión.

---

## 1. Visión General del Proyecto
**Hub Agritech Core** es una plataforma de IoT e IA para el sector agrícola (Edge AI). Su objetivo es simular e ingestar datos de telemetría de sensores, transmitirlos de forma eficiente en el campo mediante comunicación LoRa P2P nativa, procesarlos y mostrarlos en un panel de control moderno, preparando el terreno para la futura integración de Inteligencia Artificial (SLM/Geoespacial).

- **Stack Tecnológico:**
  - **Frontend**: Vue 3 (Composition API), Vite, Vanilla CSS moderno basado en variables (estricta prohibición de Tailwind CSS).
  - **Backend API**: FastAPI (Python), SQLAlchemy, GeoAlchemy2.
  - **Broker IoT**: Eclipse Mosquitto (MQTT 1883 / 9001).
  - **Base de Datos**: PostgreSQL 15 + PostGIS + pgvector (imagen Docker personalizada en `db/Dockerfile`).
  - **Orquestación**: Docker Compose / Podman Desktop (Frontend mapeado en `8081:80` y PostgreSQL en `5435:5432` para compatibilidad rootless).
  - **Hardware y Radio LoRa**: Heltec WiFi LoRa 32 V4 (ESP32-S3 + Semtech SX1262 a **868.0 MHz**, RadioLib P2P).

---

## 2. Trabajo Realizado (Semanas 1 a 4)

### Infraestructura y Base de Datos (Completado)
- Configuración de la red bridge `edge_network`.
- Imagen personalizada PostgreSQL 15 con extensiones `PostGIS` y `pgvector`.
- Orquestación con `docker-compose.yml` (Mosquitto, DB, Adminer en puerto 8080, Frontend en puerto 8081).
- Modelos de datos creados en `app/db/models.py` (`TelemetryRecord`, campos NPK, pH, EC, temperatura, humedad).

### Pipeline de Ingesta y Simulación MQTT (Completado)
- **Simulador de sensores:** `app/simulators/sensor_sim.py` genera periódicamente tramas JSON realistas hacia el broker MQTT.
- **Worker de Ingesta:** `app/workers/mqtt_ingest.py` implementado con `paho-mqtt` y SQLAlchemy, suscribiéndose a `sensors/#` y persistiendo lecturas liberando sesiones en bloques `try...finally: db.close()`.

### Comunicación LoRa P2P y Gateway Serial (Completado)
- **Migración arquitectónica:** Se eliminó por completo la dependencia de Meshtastic por su alto consumo y sobrecarga de protocolo, adoptando **LoRa Nativo P2P** con la librería `RadioLib`.
- **Firmware Emisor (Nodo de Campo):** `firmware/emisor_nodo/emisor_nodo.ino` para Heltec V4, configurado a 868.0 MHz (SX1262), con soporte para pantalla OLED, gestión de pines de RF (VEXT, VFEM, FEM_EN, FEM_CPS) y preparado para Deep Sleep en campo.
- **Firmware Receptor (Gateway LoRa):** `firmware/receptor_gateway/receptor_gateway.ino` para Heltec V4, con recepción no bloqueante por interrupciones (`setPacketReceivedAction`), reporte de RSSI/SNR e impresión serial estructurada.
- **Bridge Serial-a-MQTT:** `lora_serial_listener.py` escucha el puerto USB serial con `pyserial`, valida las tramas crudas recibidas por el Heltec receptor y las publica al broker Mosquitto.
- **Simulador Serial:** `heltec_lora_sim.py` para pruebas del gateway serial sin necesidad de conectar el microcontrolador físico.

### Integración y Validación de Hardware de Campo (Completado)
- **Sensor Suelo Multiparamétrico RS485 Modbus-RTU:** Integrado y validado en vivo con el nodo Heltec V4 (`firmware/emisor_nodo/emisor_nodo.ino`) y módulo transceptor HW-726 (MAX485).
- **Calibración con Manual Oficial V2.2 y Salinidad Agronómica:** Protocolo ajustado a 9600 bps (8N1), dirección de esclavo de fábrica `0x02` (broadcast `0x00`), trama de consulta de 8 registros (`02 03 00 00 00 08 44 3F`) con lectura en vivo de Temperatura, Humedad, Conductividad, Salinidad, NPK y pH. En sondas 7-en-1 físicas (cuyo Reg 3 viene en `0x0000`), el firmware y pipeline derivan automáticamente la salinidad desde la EC (factor oficial $0.5$, $\text{Sal} = \text{EC} \times 0.5$).
- **Topología Eléctrica Homologada:** Masa común (puente negativo fuente 12V hacia GND Heltec), 5V para excitador MAX485 y pines asignados en Header J3 (GPIO 4 / Pin 15 RX, GPIO 5 / Pin 16 TX).
- **USB CDC On Boot y Resiliencia Serial:** Resuelto el streaming serial en ESP32-S3 activando `USB CDC On Boot: Enabled`, configurando `DTR=True`/`RTS=False` y forzando codificación UTF-8 en Windows para evitar excepciones `cp1252`.
- **Pipeline de Telemetría Completo de 8 Parámetros:**
  - Firmware Emisor (`emisor_nodo.ino`): Log detallado de los 8 registros Modbus, cálculo agronómico de salinidad y transmisión LoRa P2P.
  - Firmware Gateway (`receptor_gateway.ino`): Pantalla OLED con los 8 parámetros agronómicos y emisión serial con RSSI/SNR (`Serial.flush()`).
  - Scripts Python (`lora_serial_listener.py`, `lora_debug_logger.py`): Despliegue estructurado en árbol de los 8 registros, soporte de espacios y derivación defensiva de salinidad.
  - Base de Datos y API: Modelo `SensorData`, worker `mqtt_ingest.py` y endpoints `/api/v1/sensors/summary` actualizados para persistir e ingerir `ec`, `salinity`, `nitrogen`, `phosphorus`, `potassium`, `ph`, `temperature`, `humidity`.
- **Soporte Multiplataforma de Hardware y Puertos Serie (Completado):** Autodetección dinámica de puertos seriales por VID/PID (Espressif, CP210x, CH340, FTDI) en Linux (`/dev/ttyUSB*`), macOS (`/dev/cu.*`) y Windows (`COM*`).
- **Downlink LoRa P2P Bidireccional Sincronizado (Completado):**
  - Implementación de arquitectura estilo *LoRaWAN Clase A*: el nodo emisor (`emisor_nodo.ino`) abre una ventana de escucha RX de 3 segundos justo después de transmitir (`LORA_DIO1`).
  - Persistencia de configuración en memoria Flash NVS del ESP32 (`Preferences.h`), garantizando que el intervalo de muestreo sobreviva a reinicios y al Deep Sleep (arranque por defecto en 15 segundos).
  - Cola de comandos pendientes (`pending_downlinks`) en `lora_serial_listener.py`, que retiene las instrucciones de la web y las transmite por aire vía Gateway justo cuando el nodo abre su ventana de recepción.

### Backend API y Frontend (Completado)
- **API FastAPI Extendida:** Endpoints `/api/v1/sensors/summary` y `/api/v1/sensors` con `last_seen`, `/api/v1/sensors/{node_id}/config` para downlink por MQTT, `/api/v1/sensors/export` para telemetría cruda en CSV, y `/api/v1/sensors/purge/dummy` para limpieza de registros de prueba.
- **Frontend Vue 3 Renovado:**
  - **Overview (`DashboardView.vue`):** Resumen ejecutivo con tarjetas compactas de sensores activos, estado en línea/inactivo y signos vitales mínimos.
  - **Sensores y Dispositivos (`DevicesView.vue`):** Paneles detallados con los 8 parámetros oficiales completos, modal de configuración remota con slider continuo (1s a 12h) y advertencia de batería, exportación a CSV/PDF y purga de nodos dummy.
  - **Detalle e Históricos (`DeviceDetailView.vue`):** Selector de 9 métricas agronómicas en gráficos de líneas, barras y tabla histórica.
  - **Solución NGINX SPA:** Incorporación de `frontend/nginx.conf` (`try_files $uri $uri/ /index.html;`) resolviendo de forma permanente el error 404 al recargar páginas.
- **Infraestructura Podman / WSL2 Consolidada:** Desactivación del simulador en `docker-compose.yml`, configuración de registries en Ubuntu WSL2 (`/etc/containers/registries.conf`), y manejo de `SIGTERM` limpio en `mqtt_ingest.py`.

---

## 3. Trabajo Pendiente (Próximos Pasos)

1. **Capas Geoespaciales PostGIS:**
   - Generar endpoints GeoJSON y visualización de polígonos/parcelas en el mapa del Dashboard.
2. **Alertas Agronómicas Inteligentes:**
   - Reglas umbral automáticas (ej. alerta por pH ácido < 5.5 o EC elevada > 2000 µS/cm) integradas con notificaciones en el frontend.

---

## 4. Reglas Críticas del Proyecto
- **Frontend:** Estrictamente **prohibido** utilizar Tailwind CSS. Utilizar variables nativas en `main.css`, CSS Grid y Flexbox.
- **Podman Rootless y Plataformas:** El frontend debe exponerse en el puerto `8081` y PostgreSQL en `5435` para evitar errores de puertos restringidos (< 1024). El stack se ejecuta exclusivamente en **Linux (Mini PC)** o **Windows (WSL2)**; **macOS queda descartado** de los dispositivos requeridos para montar Podman.
- **LoRa P2P:** Siempre utilizar RadioLib P2P (868.0 MHz, BW 125, SF 9, CR 4/7, Sync 0x12). Nunca introducir librerías tipo malla pesadas en los nodos emisores.
- **Sesiones DB:** Toda consulta o inserción con SQLAlchemy debe cerrar explícitamente la sesión en un bloque `try...finally: db.close()`.
