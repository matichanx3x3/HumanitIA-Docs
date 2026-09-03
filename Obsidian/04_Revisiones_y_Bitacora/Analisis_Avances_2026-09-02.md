---
tags:
  - bitacora
  - revision
  - desarrollo
  - lora
date: 2026-09-02
---
# Análisis de Avances y Próximos Pasos (2026-09-02)

## 1. Estado Actual vs Cronograma
Tras revisar el código fuente desarrollado en `hub_agritech_core/` y compararlo con el `Cronograma_General.md` y `Tasks_Semanales.md` de Obsidian, se ha constatado lo siguiente:

- **Fase 1 (Ingesta e Infraestructura):** Se confirma como **COMPLETADA**. El entorno multi-contenedor (PostgreSQL + pgvector + PostGIS, Mosquitto, FastAPI, Nginx) está operativo bajo Podman/Docker Compose.
- **Worker y Emuladores (Semana 3):** Los objetivos de la Semana 3 están completados. El `mqtt_ingest.py` y el emulador de sensores funcionan end-to-end, llevando la telemetría hasta el Dashboard Vue 3.

## 2. Hallazgos en el Desarrollo (Prueba de Comunicación LoRa)
Al revisar el script `heltec_lora_sim.py` que se usó para validar la comunicación Serial-LoRa, se detectó una **divergencia arquitectónica importante**:
- El script actual utiliza la librería de Python `meshtastic` y se comunica mediante su API serial hacia canales privados (`INDICE_CANAL`).
- **Problema:** Esto contradice la **Regla Crítica #4** definida en el cerebro del proyecto, la cual estipula explícitamente el uso de **LoRa Nativo Punto a Punto (P2P)** mediante `RadioLib` y evitar protocolos pesados como Meshtastic para poder mantener el *Deep Sleep* en los nodos de campo.

## 3. Próximos Pasos (Definición de Tareas - Semana 4)
Considerando el estado del roadmap y las discrepancias encontradas, el enfoque inmediato para avanzar en la **Fase 2 (Motor Determinista y API Base)** debe ser:

1. **Refactorización del Gateway Serial (Deuda Técnica):**
   - Migrar o refactorizar el script `heltec_lora_sim.py` para que utilice `pyserial` nativo en lugar del API de `meshtastic`. Debe leer tramas crudas (ej. JSON por serial) emitidas por un Gateway Heltec V4 programado puramente con RadioLib.
2. **Soporte Multiplataforma para Puerto Serial:**
   - Implementar detección dinámica de puertos en el script de ingesta serial para que detecte automáticamente `/dev/ttyUSB*` en Linux (Mini PC) o `COM*` en Windows, facilitando el despliegue automático.
3. **Tiempo Real en el Dashboard (WebSockets):**
   - Consolidar las vistas gráficas del Frontend (Vue 3) migrando de peticiones HTTP tradicionales (polling) a **WebSockets**, conectando directamente con FastAPI o Mosquitto para actualizaciones de métricas en vivo.
4. **Preparación Capa Espacial (Inicio de Fase 3):**
   - Integrar los endpoints geoespaciales (GeoJSON) en el mapa de parcelas del dashboard.

*Nota: Se actualizará el archivo `Tasks_Semanales.md` en la bóveda para reflejar estas actividades en la Semana 4.*
