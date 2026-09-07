---
tags:
  - negocio
  - presupuesto
  - hardware
  - BOM
fecha: 2026-09-07
---
# Presupuesto y Lista de Materiales (BOM) - Agritech HumanitIA

Este documento detalla la Lista de Materiales (BOM - *Bill of Materials*) para fabricar la infraestructura hardware del proyecto. El objetivo es validar si es posible mantener el costo total del equipamiento en un rango de **500 € a 600 €**, y proyectar los márgenes para la estrategia de "masificación".

## 1. Costo Unitario por Nodo de Campo (Sensor LoRa P2P)
El nodo de campo es la unidad que se coloca en la tierra, lee los sensores y transmite por LoRa en modo Deep Sleep.

| Componente | Descripción | Costo Estimado (Unitario) |
| :--- | :--- | :--- |
| **Heltec WiFi LoRa 32 V4** | Microcontrolador principal + Transceptor SX1262 LoRa | ~ 25.00 € |
| **Sensor de Suelo 7-en-1** | Sonda RS485 (NPK, Humedad, Temperatura, EC, pH) | ~ 22.00 € |
| **Adaptadores (RS485 y Voltaje)** | Módulo HW-726 (RS485 a TTL) + MT3608 (Step-Up a 12v) | ~ 3.00 € |
| **Energía Autónoma** | Panel Solar (5V 1W/2W) + 2x Baterías 18650 + BMS | ~ 15.00 € |
| **Protección IP68** | Caja estanca, prensaestopas, resina epóxica para aislar empates | ~ 11.00 € |
| **Total por Nodo Físico** | | **~ 76.00 €** |

## 2. Costo Unitario por Gateway LoRa
El Gateway es la unidad receptora conectada permanentemente a la energía y al router WiFi/Internet para inyectar datos en MQTT.

| Componente                          | Descripción                                                    | Costo Estimado (Unitario) |
| :---------------------------------- | :------------------------------------------------------------- | :------------------------ |
| **Heltec WiFi LoRa 32 V4**          | Receptor LoRa (Escucha continua sin Deep Sleep)                | ~ 25.00 €                 |
| **Antena Externa de Alta Ganancia** | Antena de fibra de vidrio 5dBi para 868MHz (Mejora el alcance) | ~ 15.00 €                 |
| **Fuente de Alimentación**          | Adaptador de corriente 5V estable y robusto                    | ~ 10.00 €                 |
| **Protección IP68**                 | Caja estanca                                                   | ~ 10.00 €                 |
| **Total por Gateway**               |                                                                | **~ 60.00 €**             |

---

## 3. Costo de la CPU Edge (Servidor de Inteligencia Artificial)
Para procesar localmente toda la analítica, la base de datos PostgreSQL/PostGIS, el motor de alertas y los modelos de Machine Learning (Edge AI) sin depender de la nube, se requiere un equipo industrial o robusto con al menos 8GB de RAM. Tras una revisión exhaustiva del mercado actual, los precios de estos equipos han experimentado alzas significativas.

| Componente | Descripción | Costo Estimado (Unitario) |
| :--- | :--- | :--- |
| **Mini PC / Edge AI Box (Barebone)** | Equipo base industrial o semi-industrial (sin RAM ni SSD). | ~ 200.00 € |
| **Memoria y Almacenamiento** | 8GB de RAM + 128GB SSD. | ~ 150.00 € |
| **Total por CPU Edge (8GB RAM + 128GB SSD)** | Equipo completo listo para alojar Docker, PostgreSQL, y modelos ML. | **~ 350.00 €** |

---

## 4. Escenarios de Inversión y el Umbral de los 600 €

Considerando los precios actualizados del mercado, el **Paquete Inicial Mínimo Viable** está compuesto por: 1 Nodo Emisor, 1 Nodo Receptor (Gateway) y 1 CPU Edge.

### Escenario A: Paquete Inicial Básico (1 Emisor + 1 Receptor + CPU Edge)
- 1x Nodo Emisor: 76 €
- 1x Nodo Receptor (Gateway): 60 €
- 1x CPU Edge AI (8GB RAM, 128GB SSD): 350 €
- **Costo Total Paquete Inicial: 486 €** (Cumple perfectamente con la meta de estar por debajo de los 600 €).

### Escenario B: Paquete Óptimo dentro de los 600 € (2 Emisores + 1 Receptor + CPU Edge)
Si aprovechamos al máximo el presupuesto de 500 - 600 €:
- 2x Nodos Emisores (2 * 76 €): 152 €
- 1x Nodo Receptor (Gateway): 60 €
- 1x CPU Edge AI (8GB RAM, 128GB SSD): 350 €
- **Costo Total: 562 €** (Justo dentro del margen de 600 €, entregando 2 puntos de medición en campo).

### Escenario C: Modelo Comercial de Alta Rentabilidad (10 Estaciones por 10k)
Si vendemos el ecosistema masivo de **10 nodos de campo** operados localmente por IA:
- 10x Nodos Emisores: 760 €
- 1x Nodo Receptor (Gateway): 60 €
- 1x CPU Edge AI (8GB RAM, 128GB SSD): 350 €
- **Costo de Fabricación (BOM Total): 1.170 €**

> [!tip] Ventaja Comercial y Masificación
> La competencia cobra alrededor de **10.000 € por 2 estaciones**.
> Nuestro costo de fabricación por un ecosistema autónomo con **10 estaciones + IA Local** es de **1.170 €**. 
> Si Agritech HumanitIA vende este sistema masivo a **10.000 €**, entrega un valor tecnológico abismalmente superior (Edge AI propio, sin pagos mensuales en la nube, y 5 veces más cobertura territorial) manteniendo un **margen de ganancia bruto cercano al 88% (8.830 € de beneficio por instalación)**.

## 5. Conclusión para la Viabilidad de Masificación
A pesar del incremento en los costos de los servidores Edge, la respuesta sigue siendo **positiva**. 
Con una inversión de **486 €**, fabricas el "Paquete Inicial" (Emisor + Receptor + Servidor Edge AI). Si llegas al límite de los **600 €**, el cliente puede llevarse el servidor y **2 nodos emisores** (562 €). La estructura de costos sigue permitiendo destrozar los precios de la competencia manteniendo altísima rentabilidad.
