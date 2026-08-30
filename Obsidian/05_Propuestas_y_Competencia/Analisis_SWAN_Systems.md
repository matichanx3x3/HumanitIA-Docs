---
tags:
  - competencia
  - agtech
  - saas
  - analisis-profundo
aliases:
  - Análisis Profundo SWAN Systems
---
# Análisis de Competencia Profundo: SWAN Systems

## 1. Visión General del Producto y Enfoque de Mercado
**SWAN Systems** es una suite de software de gestión y planificación de riego de nivel empresarial. Su filosofía se basa en ser un "todo en uno" en la nube, alejándose completamente del desarrollo de hardware. Funcionan bajo un modelo de suscripción (SaaS) y se integran mediante APIs con hardware de terceros (ej. controladores TalGil, estaciones meteorológicas, sensores de humedad).

### El Contexto Local (Mercado LATAM / Perú)
Según el análisis de su propuesta de valor para clientes en Perú, SWAN ataca dolores específicos como:
- Optimización de costos de mano de obra.
- Privacidad y seguridad de la información agrícola.
- **Fertirrigación por pulsos** para ahorrar agua y recursos.
- Integración con múltiples softwares (ej. SCADA de Plantas de Ósmosis Inversa).

---

## 2. Desglose Técnico de Funcionalidades Clave

### A. Inteligencia Geoespacial e Índices (KcVI)
SWAN no solo usa **NDVI**, sino 7 índices vegetativos (GNDVI, MSAVI, EVI, SIPI, ARVI, NDMI) obtenidos por satélites (Sentinel-2 gratuito, o PlanetVision de pago hasta 3x3 metros).
- **Innovación principal:** Usan estos satélites para calcular dinámicamente el **Coeficiente de Cultivo (Kc)**, ajustando la demanda de agua (KcVI) semana a semana sin depender de tablas teóricas fijas.
- **Detección de anomalías:** Identifican estrés hídrico temprano y roturas de tuberías (*burst pipes*) observando parches inusualmente húmedos desde el espacio.

### B. Gestión Hídrica y de Nutrientes (Módulos Avanzados)
- **Water Budgeting & Supply:** Permite mapear de dónde viene el agua (pozos, ríos, represas), asignar presupuestos por sector y monitorear el cumplimiento legal/licencias (ej. normativas gubernamentales de uso de agua).
- **Nutrient Management:** Consideran la **calidad del agua de riego** (contaminantes o nutrientes ya presentes, medidos por sensores de conductividad) para calcular la mezcla exacta del tanque (*tank mixes*) y trazar la fertirrigación aplicada vs. planificada.

### C. DataHub y Analítica (Impact Module)
- **DataHub Integrado:** Ingestan datos complejos más allá del suelo: dendrómetros (crecimiento del tallo), medidores de flujo de savia, temperatura del dosel (canopy).
- **Métricas de Impacto:** Miden el *Water Application Efficiency* (Eficiencia de Aplicación) y el *Water Use Efficiency* (Rendimiento del cultivo por m³ de agua).

---

## 3. Matriz Competitiva: SWAN vs. Hub Agritech Core

| Dimensión Técnica             | SWAN Systems                                                                        | Hub Agritech Core                                                    | Nuestra Ventaja Estratégica (US)                                                                   |
| :---------------------------- | :---------------------------------------------------------------------------------- | :------------------------------------------------------------------- | :------------------------------------------------------------------------------------------------- |
| **Infraestructura Core**      | **Nube (AWS/Azure).** Requiere internet para operar la finca.                       | **Offline-First (Mini PC Edge).** Opera en la intranet de la finca.  | **Garantía absoluta de operatividad en campo** ante caídas de red y latencia cero para actuadores. |
| **Hardware & SCADA**          | API Cloud. Para leer Plantas de Ósmosis Inversa (Voens) requiere puentes a la nube. | Protocolos industriales nativos (**RS485/Modbus**) directos al Edge. | Hub Agritech puede leer la planta de Ósmosis (pH, EC) en milisegundos sin pasar por internet.      |
| **Privacidad de Datos**       | Promete "Almacenamiento privado" pero está en servidores de SWAN.                   | El motor PostgreSQL+pgvector **físicamente reside en la finca**.     | Cumplimiento estricto de propiedad intelectual; la granja es dueña de su servidor.                 |
| **Inteligencia (Cálculo Kc)** | Satélites (sujeto a nubosidad y altos costos por alta resolución).                  | Cámaras IP locales (ESP32-CAM) procesando visión artificial local.   | Mediciones bajo las nubes, en tiempo real, 24/7 y sin costo de suscripción a terceros.             |

---

## 4. Roadmap Estratégico para Hub Agritech (Inspirado en SWAN)

Para asegurar que **Hub Agritech Core** compita funcionalmente con SWAN sin perder su filosofía Edge, se deben derivar las siguientes historias de usuario hacia nuestro backlog:

1. **Módulo de Fertirrigación por Pulsos y Calidad de Agua:**
   - *Acción:* Extender el `Worker Ingesta` para que lea los sensores de pH y Conductividad Eléctrica (EC) de la matriz de agua principal (o de la Planta de Ósmosis).
   - *Lógica:* Antes de activar las electroválvulas, el motor debe validar la EC para ajustar las dosis de los tanques de fertilizante.
2. **Dashboard de Eficiencia (Water Use Efficiency):**
   - *Acción:* Agregar vistas en Vue.js que no solo muestren "cuánta agua se regó", sino que comparen el volumen regado contra la "Cuota Legal" o el "Presupuesto Hídrico Mensual" de la finca.
3. **El Camino al Kc Dinámico Local:**
   - *Acción:* En lugar de contratar APIs satelitales caras, preparar la base de datos `pgvector` para recibir embeddings de imágenes de nuestras Cámaras IP locales, entrenando un modelo pequeño que estime el estrés hídrico visualmente.
