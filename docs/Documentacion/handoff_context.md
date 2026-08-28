# Contexto y Estado del Proyecto: Hub Agritech Core

Este documento sirve como resumen completo del progreso, decisiones arquitectónicas y próximos pasos para transferir el contexto a otro agente o continuar el desarrollo en una nueva sesión.

## 1. Visión General del Proyecto
**Hub Agritech Core** es una plataforma de IoT e IA para el sector agrícola (Edge AI). Su objetivo es simular e ingestar datos de telemetría de sensores, procesarlos y mostrarlos en un panel de control moderno, preparando el terreno para la futura integración de Inteligencia Artificial (SLM/Geoespacial).

- **Stack Tecnológico Acordado:**
  - **Frontend**: Vue 3 (Composition API), Vite, Vanilla CSS moderno (sin Tailwind).
  - **Backend API**: FastAPI (Python), SQLAlchemy, GeoAlchemy2.
  - **Broker IoT**: Mosquitto (MQTT).
  - **Base de Datos**: PostgreSQL 15 + PostGIS + pgvector (Configurado vía un Dockerfile personalizado).
  - **Orquestación**: Docker Compose / Podman Desktop (Nota: El frontend corre en el puerto 8081 para evitar conflictos de *rootless ports* menores a 1024).

## 2. Trabajo Realizado (Fase 1 completada)

### Infraestructura (Base de Datos y Red)
- Se configuró la red tipo *bridge* `edge_network`.
- Se creó la imagen personalizada de BD en [`db/Dockerfile`](file:///d:/Proyectos/hub_agritech_core/db/Dockerfile) instalando la extensión `pgvector` sobre `postgis/postgis:15-3.3`.
- Se configuró el `docker-compose.yml` para levantar PostgreSQL, Adminer (puerto 8080), Mosquitto y el Frontend (puerto 8081).

### API Core
- Se inicializó la aplicación FastAPI en [`app/main.py`](file:///d:/Proyectos/hub_agritech_core/app/main.py) con CORS configurado.
- Se configuró el motor y la sesión de base de datos en [`app/db/database.py`](file:///d:/Proyectos/hub_agritech_core/app/db/database.py).
- Se creó el enrutador base `v1` con un endpoint de *healthcheck* en [`app/api/v1/endpoints.py`](file:///d:/Proyectos/hub_agritech_core/app/api/v1/endpoints.py).

### UI Base (Esqueleto Visual Frontend)
- Se desarrolló el esqueleto visual en Vue 3 basándose en referencias de UI/UX premium.
- **Sistema de diseño**: CSS puro basado en variables globales (`main.css`) preparado para ser sobreescrito o tematizado por un panel de administrador. Tipografía *Inter* de Google Fonts.
- **Componentes**: 
  - `AppLayout.vue` y `Sidebar.vue` para el menú lateral oscuro.
  - `SummaryCard.vue` componente clickeable para las tarjetas de KPI.
- **Dashboard**: `DashboardView.vue` configurado como ruta principal (`/`), que organiza las tarjetas de telemetría (Humedad, Temperatura, etc.) y una tarjeta enlazable para el Mapa del Terreno.

### Documentación y Tareas en Segundo Plano
- Existe un plan de documentación recurrente (programado para ejecutarse cada lunes mediante un proceso en segundo plano) que genera un archivo Markdown del estado del proyecto en la carpeta `docs/Documentacion/`.

## 3. Trabajo Pendiente (Siguiente Fase)

El próximo agente debe retomar el trabajo según el cronograma de Gantt, enfocándose en la **Semana 3: Scripts de Simulación MQTT y Workers**.

### Siguientes Tareas Inmediatas:
1. **Crear el Emulador MQTT (Simulador de Sensores)**
   - Desarrollar un script en Python (por ejemplo, en `app/simulators/`) que publique datos JSON con valores aleatorios o estructurados (humedad, temperatura, pH, nivel de suelo) en los tópicos de Mosquitto.
2. **Implementar el Worker de Ingesta**
   - El archivo [`app/workers/mqtt_ingest.py`](file:///d:/Proyectos/hub_agritech_core/app/workers/mqtt_ingest.py) es un esqueleto actualmente.
   - Debe actualizarse para suscribirse al broker MQTT, recibir los *payloads* del simulador y procesarlos para guardarlos en la base de datos a través de SQLAlchemy.

## 4. Notas Claves para el Próximo Agente
- **Restricción de Frontend:** No uses TailwindCSS. Mantén la filosofía de diseño premium usando CSS Variables nativas, flexbox y CSS Grid.
- **Podman Desktop:** Si se te pide levantar contenedores o encuentras problemas con puertos de red (Ejem: Puerto 80), recuerda que el usuario usa Podman *rootless*, por lo que los puertos menores a 1024 pueden fallar en enlazarse. El frontend se movió exitosamente al `8081` por este motivo.
- **Revisión de Arquitectura:** En la carpeta `docs/` el usuario ha adjuntado los PDFs con la arquitectura y el diagrama de capas del proyecto para futuras consultas.

---
*Este documento fue generado para facilitar el traspaso fluido del flujo de trabajo y resume todo el contexto necesario para continuar el desarrollo.*
