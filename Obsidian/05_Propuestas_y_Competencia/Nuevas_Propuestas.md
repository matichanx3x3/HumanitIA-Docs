---
tags:
  - propuestas
  - edge-ai
  - roadmap
  - innovacion
aliases:
  - Nuevas Propuestas y Evolución
---
# Nuevas Propuestas y Evolución Edge AI

Este documento centraliza el *roadmap* de innovación de **Hub Agritech Core**. Basado en un análisis profundo de soluciones empresariales en la nube (como SWAN Systems), adaptamos las mejores prácticas de la industria a nuestra filosofía técnica **Offline-First y Edge AI**.

---

## 1. Integración de Conceptos Competitivos al Ecosistema Edge

Las grandes plataformas resuelven problemas complejos enviando datos a la nube. Nuestro reto técnico es resolverlos **localmente en el Mini PC** utilizando inteligencia artificial de borde (Edge AI).

### A. Coeficiente de Cultivo Dinámico (Kc) sin Satélites
- **El enfoque Cloud (Competencia):** Comprar imágenes satelitales (Planet/Sentinel), enviarlas a la nube, calcular NDVI y ajustar el requerimiento de agua (KcVI). **Problemas:** Costoso, alta latencia y ciego en días nublados.
- **Nuestra Propuesta Edge AI:** Utilizar nuestro actual `Contenedor 3: RTSP Worker`. Capturar imágenes de las cámaras IP o ESP32-CAM instaladas bajo el follaje.
  - *Técnica:* Ejecutar un modelo de visión computacional ligero (ej. TensorFlow Lite / YOLO) dentro del Mini PC. El modelo evalúa el color de la hoja y el estrés hídrico visual, actualizando el coeficiente (Kc) en nuestra base de datos PostgreSQL de forma autónoma y gratuita.

### B. Módulo de Presupuesto Hídrico (Water Budgeting)
- **El enfoque Cloud:** Dashboards que miden la eficiencia de riego cruzando datos de la API.
- **Nuestra Propuesta:** Desarrollar en nuestro frontend Vue.js vistas de *Water Use Efficiency*.
  - *Técnica:* Aprovechar el particionamiento de **Series Temporales de PostgreSQL** para realizar consultas SQL analíticas de alta velocidad (Agregaciones por mes/semana) comparando "Milímetros Regados vs. Cuota Hídrica Legal", sin que el Mini PC sufra penalizaciones de rendimiento.

---

## 2. Mejoras Operativas a Implementar (Roadmap Técnico)

### Mejora 1: Fertirrigación Autónoma e Integración SCADA
Los clientes avanzados (ej. Perú) usan Plantas de Ósmosis Inversa complejas.
- **Integración Técnica:** El `Worker Ingesta` (Python) debe suscribirse a los tópicos MQTT que vienen de los sensores de la matriz de agua (pH y Conductividad Eléctrica - CE). 
- **Lógica:** Antes de que FastAPI envíe el comando de abrir la válvula del fertilizante, el *Motor Determinista* evalúa el pH actual del agua purificada. Si el agua ya viene muy cargada de sales, reduce la dosis de fertilizante en milisegundos.

### Mejora 2: Riego por Pulsos Controlado por el Edge
Para evitar el encharcamiento (lixiviación) y ahorrar recursos.
- **Integración Técnica:** En lugar de que la API REST envíe un comando binario (`ON` por 1 hora), modificaremos los *AI Stubs (Function Calling)*. FastAPI enviará un payload JSON estructurado vía MQTT: `{"modo": "pulsos", "tiempo_on_min": 10, "tiempo_off_min": 15, "ciclos": 4}`. 
- El ESP32 en campo recibe el paquete y se encarga del ciclo localmente, liberando carga de procesamiento del Hub Central.

---

## 3. El Siguiente Nivel: Integrando Modelos de Lenguaje (SLM) en el Edge

La arquitectura actual ya cuenta con `pgvector` en la capa de persistencia. Esto no es casualidad; es la base para integrar Inteligencia Artificial conversacional sin depender de OpenAI o ChatGPT (lo cual requeriría internet).

**Flujo Propuesto (RAG Local):**
1. **Conocimiento:** Ingresamos manuales de agronomía, normativas locales de riego y el historial de decisiones de nuestra API en `pgvector` como *embeddings*.
2. **El Modelo:** Desplegamos un Small Language Model (SLM) de código abierto (ej. Llama 3 8B o Mistral) en el Mini PC usando herramientas como Ollama.
3. **El Resultado:** El agrónomo abre el Dashboard (Vue.js) en su tablet sin internet y le pregunta al sistema: *"¿Por qué regaste el sector 4 anoche si la humedad era buena?"*. El SLM consulta la base de datos vectorial local, lee el registro de que "se pronosticaba una ola de calor" y le responde al agricultor en lenguaje natural.

> **Conclusión Técnica:** La combinación de **MQTT local + PostgreSQL vectorizado + Nodos ESP32** convierte al Hub Agritech en un sistema que puede ofrecer el 100% de las analíticas de un gigante en la nube, pero con un costo operativo cercano a cero, privacidad total y ejecución en milisegundos.
