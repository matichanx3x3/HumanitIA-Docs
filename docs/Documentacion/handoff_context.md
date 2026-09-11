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

### Backend API y Frontend (Completado)
- API FastAPI en `app/main.py` con CORS habilitado y endpoints REST `/api/v1/` (`summary`, `history`, `nodes`, `health`).
- Frontend Vue 3 en `frontend/` con componentes `AppLayout`, `Sidebar`, `SummaryCard`, gráficos históricos Chart.js y sistema de diseño en CSS puro (`src/assets/main.css`).

---

## 3. Trabajo Pendiente (Próximos Pasos)

1. **Soporte Multiplataforma para el Listener Serial:**
   - Implementar autodetección dinámica del puerto serie (`/dev/ttyUSB*` o `/dev/ttyACM*` en Linux/macOS y `COM*` en Windows).
2. **WebSockets o Polling en Dashboard:**
   - Vincular el flujo en tiempo real de la API al frontend para actualizar las métricas de `SummaryCard` y gráficos sin recargar la página.
3. **Capas Geoespaciales PostGIS:**
   - Generar endpoints GeoJSON y visualización de polígonos/parcelas en el mapa del Dashboard.
4. **Validación con Sensores Reales de Campo:**
   - Integrar la lectura Modbus-RTU RS485 del Sensor 7-en-1 en `emisor_nodo.ino` a través del módulo HW-726.

---

## 4. Reglas Críticas del Proyecto
- **Frontend:** Estrictamente **prohibido** utilizar Tailwind CSS. Utilizar variables nativas en `main.css`, CSS Grid y Flexbox.
- **Podman Rootless y Plataformas:** El frontend debe exponerse en el puerto `8081` y PostgreSQL en `5435` para evitar errores de puertos restringidos (< 1024). El stack se ejecuta exclusivamente en **Linux (Mini PC)** o **Windows (WSL2)**; **macOS queda descartado** de los dispositivos requeridos para montar Podman.
- **LoRa P2P:** Siempre utilizar RadioLib P2P (868.0 MHz, BW 125, SF 9, CR 4/7, Sync 0x12). Nunca introducir librerías tipo malla pesadas en los nodos emisores.
- **Sesiones DB:** Toda consulta o inserción con SQLAlchemy debe cerrar explícitamente la sesión en un bloque `try...finally: db.close()`.
