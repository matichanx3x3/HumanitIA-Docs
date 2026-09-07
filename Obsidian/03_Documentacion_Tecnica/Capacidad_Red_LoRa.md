---
tags:
  - lora
  - redes
  - escalabilidad
  - heltec
fecha: 2026-09-07
---
# Capacidad y Escalabilidad de la Red LoRa P2P

Este documento detalla el análisis de capacidad de tráfico (RF) para el **Gateway Heltec WiFi LoRa 32 V4** utilizado en el proyecto Agritech HumanitIA, calculando cuántos nodos físicos puede soportar antes de sufrir saturación.

## Parámetros de Configuración Actual (RadioLib SX1262)
- **Frecuencia:** 868.0 MHz
- **Ancho de banda (BW):** 125.0 kHz
- **Spreading Factor (SF):** 9
- **Coding Rate (CR):** 4/7
- **Sync Word:** 0x12

## Cálculo de Time on Air (ToA)
Para una trama de telemetría compacta (aprox. 40 bytes) enviada sin el protocolo Meshtastic (usando P2P nativo), el paquete tarda aproximadamente **250 milisegundos (0.25 segundos)** en viajar por el aire.

## Límites de Recepción y Colisiones
El Heltec V4 es un Gateway Single-Channel. Esto implica que solo puede escuchar un paquete a la vez.

1. **Capacidad Teórica Máxima:** Si los envíos estuvieran 100% coordinados (TDMA), el nodo podría recibir 4 paquetes por segundo (14.400 paquetes/hora).
2. **Capacidad Real (ALOHA no ranurado):** Dado que los nodos despiertan de Deep Sleep de forma aleatoria, se aplica el cálculo de colisiones ALOHA puro. El rendimiento máximo antes de que la pérdida de paquetes sea inaceptable es del **18.4%**.
3. **Paquetes Seguros por Hora:** 14.400 * 0.184 ≈ **2.649 paquetes por hora**.

## Densidad Máxima (Nodos por Gateway)
Si configuramos los nodos en el código de C++ (`emisor_nodo.ino`) para enviar una lectura **cada 15 minutos** (4 transmisiones por hora por cada nodo):
* Nodos máximos simultáneos = 2.649 / 4 = **~ 660 nodos físicos por Gateway**.

### Conclusión para el Campo
Para una parcela agrícola estándar con una densidad de **4 nodos por hectárea**, un único Gateway de 30€ tiene la capacidad de radiofrecuencia para cubrir **hasta 150 hectáreas** sin que el ESP32 ni la base de datos PostgreSQL/FastAPI sufran cuellos de botella por volumen de ingesta. La limitación del sistema radicará estrictamente en el alcance físico/distancia de la señal (Línea de Vista - LoS) y no en el procesamiento.
