# Cerebro de HumanitIA-Docs (Contexto y Memoria Global)

Este archivo actúa como el **Project Knowledge** y **Reglas** para todos los agentes de Antigravity en el repositorio de documentación central de **HumanitIA**.

---

## 1. Instrucciones y Reglas de Documentación

- **Rol:** Arquitecto de documentación técnica, hardware IoT y gestión del conocimiento para **Agritech HumanitIA**.
- **Reglas del Repositorio:**
  1. **Bóveda Obsidian:** Todos los documentos en `Obsidian/` deben mantener enlaces tipo wikilink `[[Nota]]` o markdown estándar limpios.
  2. **Componentes Hardware:** Todo nuevo sensor, microcontrolador o módulo añadido en `Obsidian/07_Componentes/` debe seguir estrictamente la estructura definida en `Obsidian/07_Componentes/Plantilla_Componente.md`.
  3. **Trazabilidad:** Los diagramas arquitectónicos y PDFs en `docs/` son la fuente de verdad del diseño del sistema.

---

## 2. Skills de Antigravity Disponibles

El proyecto cuenta con runbooks y skills especializados en `.agents/skills/`:

| Skill | Ubicación | Descripción |
| :--- | :--- | :--- |
| **`agritech-docs-and-hardware`** | `.agents/skills/agritech-docs-and-hardware/SKILL.md` | Gestión de Obsidian, datasheets de hardware y sincronización con hub_agritech_core |
| **`agritech-stack-ops`** | `.agents/skills/agritech-stack-ops/SKILL.md` | Orquestación Podman/Docker Compose, logs, puertos y volúmenes |
| **`agritech-lora-mesh`** | `.agents/skills/agritech-lora-mesh/SKILL.md` | Comunicación LoRa P2P nativa (RadioLib SX1262 868 MHz), Heltec V4, tramas compactas y serial gateway |
| **`agritech-telemetry-ingestion`** | `.agents/skills/agritech-telemetry-ingestion/SKILL.md` | Pipeline MQTT (`mqtt_ingest.py`), suscripciones `sensors/#` y persistencia PostgreSQL |
| **`agritech-fastapi-backend`** | `.agents/skills/agritech-fastapi-backend/SKILL.md` | Endpoints REST `/api/v1/...`, agregaciones, GeoAlchemy2 y CORS |
| **`agritech-vue-frontend`** | `.agents/skills/agritech-vue-frontend/SKILL.md` | Vue 3 Composition API, Chart.js, KPI Cards y Vanilla CSS (sin Tailwind) |

---

## 3. Estructura del Repositorio

```
HumanitIA-Docs/
├── .agents/skills/        # Skills especializadas para Antigravity
├── docs/
│   ├── Arquitectura/      # PDF: diagramas Hardware, Microservicios, Edge AI
│   ├── Capas/             # PDF: Comunicación, Hardware, Frontend, Core API, Persistencia
│   ├── Documentacion/     # Sprint logs, handoff_context.md
│   └── Plan/              # Pitch, resúmenes ejecutivos, diagramas Gantt
└── Obsidian/              # Bóveda completa de conocimiento técnico y negocio
    ├── 01_Planificacion_y_Tiempos/   # Tasks_Semanales.md, Cronograma
    ├── 02_Documentacion_General/     # Explicación del sistema, electrónica básica
    ├── 03_Documentacion_Tecnica/     # Backend, Hardware, Infraestructura, Frontend
    ├── 04_Revisiones_y_Bitacora/     # Auditorías periódicas
    ├── 05_Propuestas_y_Competencia/  # Análisis de mercado
    ├── 06_Negocio_y_Economia/        # Modelos financieros y estrategia
    ├── 07_Componentes/               # Fichas técnicas de hardware (13 componentes)
    └── Dashboard.base                # Vista base de Obsidian
```

---

## 4. Inventario de Componentes Documentados (`07_Componentes/`)

| Componente | Archivo | Categoría |
| :--- | :--- | :--- |
| Heltec WiFi LoRa 32 V4 | `Heltec_V4_LoRa32.md` | Nodo LoRa / Gateway |
| Sensor de Suelo 7-en-1 RS485 | `Sensor_Suelo_7en1_RS485.md` | Sensor de campo |
| ESP32-CAM (OV3660) | `Módulo_ESP32_CAM_OV3660.md` | Visión artificial |
| Sensor de Color GY-33 (TCS34725) | `Módulo_Sensor_Color_GY33.md` | Sensor óptico |
| Conversor Nivel TXS0108E | `Módulo_TXS0108E_Conversor_Nivel.md` | Adaptador lógico 3.3V↔5V |
| Conversor RS485 a TTL (HW-726) | `Módulo_RS485_a_TTL_Auto_Flow.md` | Adaptador comunicación |
| Step-Down LM2596 | `Módulo_LM2596_Step_Down_Voltimetro.md` | Regulador de voltaje |
| Step-Up MT3608 | `Módulo_MT3608_Step_Up.md` | Regulador de voltaje |
| CP2102 USB a TTL | `Módulo_CP2102_USB_a_TTL.md` | Adaptador serial |
| Cables Dupont / Jumpers | `Cables_Dupont_Jumpers.md` | Accesorios |
| Analizador Lógico 24MHz | `Analizador_Lógico_24MHz.md` | Herramienta de debug |

Plantilla para nuevos componentes: `Plantilla_Componente.md`
Índice completo con análisis de brechas: `Indice_de_Componentes.md`

---

## 5. Sincronización con hub_agritech_core

Este repositorio de documentación se mantiene sincronizado con el repositorio de código [`hub_agritech_core`](https://github.com/matichanx3x3/hub_agritech_core):

- **Skills idénticos:** Las 6 skills en `.agents/skills/` deben ser copias exactas de las del Core. La fuente de verdad es siempre `hub_agritech_core/.agents/skills/`.
- **Flujo de datos completo del sistema:**
  ```
  Firmware RadioLib (emisor_nodo.ino)
       → LoRa P2P (868 MHz, SX1262)
       → Firmware RadioLib (receptor_gateway.ino)
       → USB Serial
       → lora_serial_listener.py (pyserial → MQTT)
       → Eclipse Mosquitto Broker
       → mqtt_ingest.py (paho-mqtt → SQLAlchemy)
       → PostgreSQL 15 (PostGIS + pgvector)
       → FastAPI REST API (/api/v1/...)
       → Vue 3 Dashboard (Chart.js, Vanilla CSS)
  ```
- **Documentos de referencia cruzada:**
  - `docs/Documentacion/handoff_context.md` — Estado y contexto de traspaso del proyecto.
  - `Obsidian/01_Planificacion_y_Tiempos/Tasks_Semanales.md` — Tareas de sprint activas.
