---
tags:
  - bitacora
  - revision
  - diagramas
  - arquitectura
date: 2026-09-02
---
# Revisión y Bitácora de Diagramas Técnicos (2026-09-02)

## Objetivo
Realizar una revisión de la documentación técnica de Obsidian centrándose en los diagramas arquitectónicos y de flujo, identificando lo existente, las correcciones necesarias y las áreas de mejora.

## 1. Lo que ya se tiene (Estado Actual)
En la carpeta `03_Documentacion_Tecnica` y en bitácoras previas se han identificado los siguientes diagramas Mermaid:

- **`Infraestructura_y_Graficos.md`**:
  - **Diagrama de Orquestación Docker Compose**: Muestra la red local `edge_network` y las relaciones entre Mosquitto, API FastAPI, PostgreSQL, Worker y Frontend (Nginx).
  - **Diagrama de Ingesta (Gateway)**: Detalla la entrada de telemetría de campo (LoRa/USB) e imágenes (RTSP) hacia Mosquitto y el Worker.
  - **Diagrama de Base de Datos Multimodal**: Detalla la distribución interna de PostgreSQL con sus extensiones (Particionamiento Nativo, PostGIS y pgvector).
  - **Diagrama de Core API**: Muestra las interacciones de FastAPI con el Broker, la DB y módulos de AI.
  - **Diagrama de Presentación**: Ilustra el proxy reverso Nginx sirviendo la aplicación Vue 3 y estáticos.

- **`Hardware_y_Fisico.md`**:
  - **Diagrama de Secuencia (Sequence Diagram)**: Representa excelentemente el viaje de datos desde el sensor RS485 en campo, transmisión P2P LoRa, Gateway USB, hasta el backend determinista para accionar una electroválvula de riego.

- **`04_Revisiones_y_Bitacora/documentacion_2026-08-24.md`**:
  - **Diagrama Macro del Sistema**: Un diagrama general que resume las 4 capas del proyecto (Ingesta, Persistencia, API Core, Presentación).

## 2. Correcciones Realizadas durante la Revisión
- En **`Infraestructura_y_Graficos.md`**: El diagrama de la Capa de Presentación mencionaba erróneamente `React / Vue`. Se corrigió para que refleje exclusivamente **Vue 3**, cumpliendo con la regla estricta del proyecto de utilizar Vue 3 Composition API sin Tailwind.

## 3. Lo que falta (Áreas de Mejora y Próximos Pasos)

Para tener una documentación técnica exhaustiva y robusta, se recomienda incorporar los siguientes diagramas:

1. **Diagrama de Entidad-Relación (ERD) en `Arquitectura_Backend.md`**:
   - Falta un diagrama que muestre los esquemas de bases de datos. Al usar SQLAlchemy, PostGIS y pgvector, es vital visualizar cómo se relacionan los `Sensors`, `TelemetryHistory`, y las geometrías (Parcelas/Nodos).

2. **Diagrama de Componentes Vue 3 en `Integracion_Frontend.md`**:
   - Actualmente este documento no tiene gráficos. Es necesario un diagrama `flowchart` que explique la jerarquía de componentes frontend (`AppLayout` -> `Sidebar` y `DashboardView` -> `SummaryCard`), y cómo fluyen los datos y peticiones HTTP.

3. **Diagrama de Arquitectura de Hardware/Wiring en `Hardware_y_Fisico.md`**:
   - Aunque hay un diagrama de secuencia lógico, falta un diagrama esquemático/físico (por ejemplo un diagrama de bloques o topología) que ilustre las conexiones: pines del Heltec V4/ESP32 <-> módulo RS485 <-> Sonda NPK, así como la alimentación con Panel Solar y Batería LiFePO4.

4. **Diagrama de Tópicos MQTT (Ingesta)**:
   - Sería de gran utilidad documentar de forma gráfica el árbol de tópicos de Mosquitto (ej. `sensors/#`, `actuators/rele/1`) para facilitar el mapeo de paquetes enviados por la placa Heltec LoRa32.

## Conclusión
La documentación cuenta con una base sólida de diagramas de contenedores, despliegue macro y flujo lógico principal. El enfoque a futuro debe ser aterrizar el diseño a las implementaciones directas: el ERD de Postgres, la jerarquía de componentes Vue y un esquema físico de cableado de hardware.
