---
tags:
  - general
  - arquitectura
  - iot
  - edge-ai
aliases:
  - Documentación Principal
  - Resumen Hub Agritech
---
# Hub Agritech Core - Documentación General

## 1. Resumen Ejecutivo y Visión del Producto
**Hub Agritech Core** es una plataforma de IoT e IA para el sector agrícola (Edge AI). Su diseño responde a las exigencias del entorno agroindustrial bajo una premisa **Offline-First**, operando de manera autónoma sin dependencia de la nube para resolver carencias de conectividad en zonas rurales.

El sistema se basa en una arquitectura agnóstica de microservicios, altamente escalable para soportar Big Data y análisis territorial (GIS). Aunque de momento obvia la IA directa, establece los cimientos (Function Calling, pgvector) para interconexión futura con Small Language Models (ej. Ollama).

👉 **Ver también:** [[Arquitectura_Backend]], [[Hardware_y_Fisico]], [[Infraestructura_y_Graficos]]

## 2. Stack Tecnológico y Matriz de Licenciamiento Comercial
Para asegurar cero costes de licenciamiento y evitar licencias "virales" (como AGPL o GPL enlazada) que comprometan el código propietario, se ha seleccionado el siguiente stack de código abierto permisivo:

| Componente | Tecnología | Licencia | Justificación de Uso |
| :--- | :--- | :--- | :--- |
| **Orquestación** | Podman | Apache 2.0 | Alternativa a Docker Desktop, 100% libre para uso comercial/corporativo (daemonless). |
| **Broker IoT (MQTT)** | Eclipse Mosquitto | EPL 2.0 / EDL 1.0 | Business-friendly, ligero y estándar industrial. |
| **Base de Datos Principal**| PostgreSQL | PostgreSQL License | Permisiva (MIT-like). Soporta particionado nativo para series temporales masivas. |
| **Motor GIS** | PostGIS | GPLv2 | Su comunicación vía TCP/IP la aísla del código propietario. Estándar de facto (ARCGIS). |
| **Motor Vectorial (IA)** | pgvector | PostgreSQL License | Evita integrar otra BD. Preparado para RAG nativo. |
| **Archivos Estáticos/Media**| NGINX + Local FS | BSD 2-clause | Sustituye a MinIO (AGPLv3). Seguro para distribución comercial. |
| **Core API (Backend)** | Python + FastAPI | PSF / MIT | Asíncrono, rendimiento óptimo, autogeneración Swagger. |
| **Frontend PWA** | Vue.js (o React) | MIT | Altamente permisivas para creación de interfaces dinámicas. |
