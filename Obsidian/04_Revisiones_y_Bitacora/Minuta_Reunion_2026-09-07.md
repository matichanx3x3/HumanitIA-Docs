---
tags:
  - reunion
  - hardware
  - negocio
  - estrategia
fecha: 2026-09-07
---
# Minuta de Reunión y Plan de Acción (2026-09-07)

## 📌 Puntos Relevantes Organizados

### 1. Hardware y Sensores
- **Componentes BBB:** Buscar sensores Buenos, Bonitos y Baratos que sean funcionales para el proyecto.
- **Diversidad y Compatibilidad:** Revisar diferentes tipos de sensores (tamaños, formas, usos y compatibilidad general).
- **Optimización:** Analizar el balance entre tener el "sensor ideal" vs. sacar el máximo provecho a nivel de software del sensor actual que se tiene.
- **Confiabilidad Física:** Las conexiones soldadas suelen fallar (se sulfatan). Buscar asegurar la calidad de los empates para que no sean asulfatados.
- **Sistemas Integrados:** Buscar soluciones en China que sean un "todo en uno" a nivel de hardware.

### 2. Arquitectura, Escalabilidad y Rendimiento
- **Densidad de Nodos:** Encontrar cuántos dispositivos caben en una unidad de "X hectáreas". Calcular cuál es el máximo de sensores que puede tener conectado una unidad CPU (Gateway) sin saturarse en su procesamiento.
- **Volumen de Datos:** Comparativa de escalabilidad de lectura: 2 grupos de datos vs 10 grupos de datos en el campo.
- **Control de Tiempos:** Es importante que los tiempos de respuesta de los nodos puedan estar programados de forma manual.

### 3. Negocio, Costes y Masificación
- **Visión Comercial:** Encontrar la forma de que este sistema sea masivo, orientado fuertemente a la venta del servicio y producto.
- **Presupuesto Objetivo:** Buscar que el equipamiento se mantenga en un rango de **500 - 600 €** con respecto al total. Validar si esto es posible.
- **Ventaja Competitiva:** Destacar el pricing frente a la competencia (ej. "no es lo mismo que inviertas 10k en 10 estaciones que 10k por 2 estaciones", ya que la competencia es muy cara).

### 4. Inteligencia de Datos y Alertas
- **Valor del Dato:** Responder a la gran pregunta: *¡Qué haces con la información que se recibe a la base de datos!*
- **Toma de Decisiones:** El objetivo es tomar decisiones con más información de sensores (mientras más, mejor).
- **Sistema de Alertas:** Si las lecturas pasan de un umbral de peligro o advertencia, enviar automáticamente un mensaje a un agente de WhatsApp o mensaje de texto (SMS).

### 5. Pruebas y QA (Quality Assurance)
- **Piloto Inmediato:** Montar las macetas y registrar la data (se han adquirido 2 plantas para hacer pruebas en entorno controlado).
- **Validación:** Buscar probar todo el sistema de extremo a extremo (referente a las tareas pendientes actuales del proyecto).

---

## 🚀 Plan de Búsqueda de Información y Tareas

Para dar respuesta a las inquietudes de la reunión, se propone el siguiente plan de búsqueda y ejecución:

### Fase 1: Sourcing y Hardware Low-Cost (China)
- [ ] **Búsqueda All-in-One:** Rastrear en proveedores como AliExpress, Alibaba, DFRobot o SeeedStudio plataformas integradas para agricultura (ej. placas que ya integren LoRa + sensores + panel solar en un solo enclosure).
- [ ] **Análisis de Mercado BBB:** Investigar alternativas de sensores RS485 o I2C que ofrezcan mejor relación calidad-precio que el sensor actual 7-en-1, o confirmar que el actual es la mejor opción ajustada por software.
- [ ] **Protección de Hardware:** Investigar conectores estancos (IP68), resinas epoxi o tubos termorretráctiles con adhesivo para evitar la sulfatación de los empates de cables.

### Fase 2: Estudio de Escalabilidad (Gateway CPU)
- [ ] **Cálculo de Saturación:** Investigar y calcular el *Airtime* (tiempo en el aire de paquetes LoRa) y la capacidad de procesamiento de interrupciones del ESP32 (Heltec V4).
- [ ] **Fórmula de Densidad:** Definir cuántos nodos pueden transmitir por hora hacia un mismo gateway sin colisiones severas (Ej. con tiempos de respuesta manuales/programables por nodo de 15 o 30 minutos).

### Fase 3: Estudio Financiero y Masificación
- [ ] **Creación del BOM (Bill of Materials):** Realizar un presupuesto exacto del costo de los componentes por "Estación" (Nodo + Sensores) para validar si se logra el objetivo de 500€ - 600€.
- [ ] **Análisis de Competencia:** Documentar los precios de los competidores (los de "10k por 2 estaciones") para armar un argumento de venta sólido centrado en la masificación.

### Fase 4: Arquitectura de Alertas (Software)
- [ ] **Diseño del Motor de Reglas (Rule Engine):** Diseñar cómo el backend (FastAPI) leerá la base de datos PostgreSQL para detectar umbrales cruzados.
- [ ] **Integración de Mensajería:** Investigar APIs de bajo costo o gratuitas para enviar alertas: Twilio (SMS/WhatsApp), Meta WhatsApp Business API, Telegram Bots o CallMeBot.

### Fase 5: Ejecución del Piloto (Laboratorio)
- [ ] **Setup Macetas:** Terminar el conexionado del nodo físico con los sensores disponibles.
- [ ] **Ingesta Real:** Validar que la telemetría de las 2 plantas llegue a PostgreSQL y se vea en el dashboard.
