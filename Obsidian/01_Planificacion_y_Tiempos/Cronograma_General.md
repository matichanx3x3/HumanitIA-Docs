# Cronograma General y Fases de Desarrollo

Este documento rastrea los tiempos de planificación a gran escala del proyecto Hub Agritech Core, basándose en la estrategia de 4 Fases de despliegue.

## Fases de Desarrollo Recomendadas

### Fase 1: Infraestructura Core e Ingesta (Completada)
- Configuración del Mini PC Linux, Podman, PostgreSQL+PostGIS y Mosquitto.
- Desarrollo del worker en Python para leer de MQTT y escribir series temporales.

### Fase 2: Motor Determinista y API Base (En Curso)
- Desarrollo de FastAPI para exponer telemetría. 
- Creación del sistema de roles (RBAC).
- Lógica para enviar comandos MQTT de vuelta a los actuadores (ej. relés de riego).

### Fase 3: Integración GIS y Multimedia (Próximamente)
- Almacenamiento de puntos geográficos (PostGIS). 
- Endpoint GeoJSON para mapeado territorial.
- Implementación de captura RTSP e interfaz básica local.

### Fase 4: Preparación para AI (El Hub listo)
- Creación de las interfaces "Function Calling" (Stubs).
- Activación de `pgvector`.
- Empaquetado del software dejándolo 100% comercializable y listo para el despliegue futuro de SLM (Small Language Models).

---

## Recursos Adjuntos Originales
Los documentos originales de planeación en PDF se encuentran en la estructura original (`D:\Proyectos\hub_agritech_core\docs\Plan\Plan_Desarrollo_Gantt_Hub_AgritecA.pdf`). Puedes embeber o vincular el Gantt detallado en esta sección para su rápido acceso.
