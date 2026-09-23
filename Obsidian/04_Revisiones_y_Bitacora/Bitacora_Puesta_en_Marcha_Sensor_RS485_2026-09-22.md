---
tags:
  - bitacora
  - hardware
  - rs485
  - modbus
  - heltec-v4
  - troubleshooting
fecha: 2026-09-22
autor: "Equipo Agritech Core & Antigravity"
estado: "resuelto_y_validado"
---
# Bitácora Técnica: Depuración y Puesta en Marcha Exitosa del Sensor de Suelo RS485 Modbus con Heltec V4

## 1. Resumen Ejecutivo
En esta sesión se abordó el problema crítico de integración del **Sensor Multiparamétrico de Suelo RS485 (7-en-1 / 8 parámetros)** conectado a la placa de desarrollo **Heltec WiFi LoRa 32 V4 (ESP32-S3)** a través del módulo conversor **HW-726 (MAX485 Auto-Flow)**. 

Inicialmente, el sistema presentaba un silencio absoluto (**estrictamente 0 bytes recibidos**). Mediante un proceso de aislamiento sistemático y gracias a la obtención del **Manual Técnico Oficial V2.2 del fabricante**, se identificaron y subsanaron 5 discrepancias críticas (eléctricas, de pinout y de protocolo), logrando la **lectura en vivo 100% exitosa** de todas las variables agronómicas y su transmisión LoRa.

---

## 2. Cronología de Problemas Detectados y Soluciones Aplicadas

### Problema 1: Tierra Flotante (Ausencia de Masa Común)
* **Síntoma:** El sensor estaba alimentado por una fuente externa de 12V pero sin unión de referencia con el microcontrolador.
* **Causa Física:** En RS485, aunque la señal es diferencial ($V_A - V_B$), el límite de modo común del silicio es de -7V a +12V respecto al GND local. Al no compartir masa, las tierras flotan libremente por estática o fugas capacitivas de la fuente conmutada, bloqueando los comparadores internos.
* **Solución:** Se implementó un puente eléctrico permanente entre el borne **`(-)`** de la fuente externa de 12V y el pin **`GND`** del Heltec V4.

### Problema 2: Incompatibilidad de Voltaje del Transceptor MAX485
* **Síntoma:** El módulo HW-726 estaba alimentado al pin `3V3` del Heltec.
* **Causa Física:** El chip montado es un **MAX485ESA**, cuyo datasheet exige un voltaje nominal de **4.75V a 5.25V**. Con 3.3V, el chip no puede polarizar adecuadamente sus transistores de salida para excitar los 2 metros de cable industrial del sensor.
* **Solución:** Se trasladó el cable de alimentación `VCC` del módulo al pin **`5V` del Heltec V4** (Header J2, Pin 2, alimentado por USB-C).

### Problema 3: Confusión de Pines UART en Heltec V4 (`U0TXD/U0RXD` vs `GPIO 4/5`)
* **Síntoma:** Los cables estaban conectados físicamente a los pines `U0TXD` y `U0RXD` del Header J2.
* **Causa Física:** Esos pines corresponden a la **UART0 (GPIO 43 y 44)** del ESP32-S3, reservada para la consola USB de la PC. El firmware estaba enviando las tramas Modbus por la **UART1 (GPIO 4 y 5)** en el Header J3. Por tanto, el código emitía hacia pines vacíos y lo que excitaba el módulo era el tráfico de bootloader del ESP32.
* **Solución:** Se reubicaron los cables en los pines reales:
  * **GPIO 4 (RX):** Header J3, Pin físico 15 (4º pin desde arriba).
  * **GPIO 5 (TX):** Header J3, Pin físico 16 (3º pin desde arriba).

### Problema 4: Interpretación de Flechas de la Serigrafía Trasera del HW-726
* **Síntoma:** Cableado TX/RX invertido respecto al transceptor.
* **Causa Física:** El fabricante del módulo HW-726 dibujó en su dorso un diagrama de cableado (`接线图`) con flechas directas:
  * `TXD <--- TXD`: Es una entrada al módulo que recibe la señal del TX del microcontrolador.
  * `RXD ---> RXD`: Es una salida del módulo que entrega los datos al RX del microcontrolador.
* **Solución:** Se conectó el pin `TXD` (cable Azul) al GPIO 5 (TX del Heltec) y el pin `RXD` (cable Amarillo) al GPIO 4 (RX del Heltec). Con esto, el LED de transmisión del módulo comenzó a oscilar inmediatamente al ritmo del código.

### Problema 5: Discrepancias de Protocolo Modbus Resueltas con el Manual V2.2
Una vez corregida la electrónica, el sensor seguía mudo debido a que los parámetros de fábrica diferían radicalmente de las asunciones estándar:

| Parámetro | Asunción Estándar Previa | Especificación Real (Manual V2.2) | Impacto |
| :--- | :--- | :--- | :--- |
| **Dirección Esclavo** | `0x01` | **`0x02`** | El sensor ignoraba las tramas dirigidas a ID 1. |
| **Baudrate** | `4800 bps` | **`9600 bps` (8N1)** | Desincronización de reloj UART. |
| **Dirección Broadcast**| `0xFF` | **`0x00`** | En este sensor, broadcast es `0x00`; `0xFF` es descartado. |
| **Cantidad de Registros**| 7 registros (`0x0007`) | **8 registros (`0x0008`)** | Rechazaba peticiones con longitud incompleta de bloque. |
| **Variables Medidas** | 7 variables | **8 variables (Incluye Salinidad / Salt)** | Se añade el parámetro de salinidad en el registro 3. |
| **Orden de Registros** | Humedad (0), Temp (1) | **Temp (0), Humedad (1), EC (2), Sal (3), NPK (4..6), pH (7)** | El mapa de memoria estaba invertido. |

---

## 3. Tramas Definitivas Validadas

* **Trama de Consulta Maestra Oficial (Master $\rightarrow$ Sensor):**
  ```text
  0x02 0x03 0x00 0x00 0x00 0x08 0x44 0x3F
  ```
* **Trama de Consulta Broadcast Universal de Respaldo:**
  ```text
  0x00 0x03 0x00 0x00 0x00 0x08 0x45 0xDD
  ```
* **Respuesta del Sensor:** Ráfaga de **21 bytes** decodificada correctamente:
  * Byte 0: `0x02` (Addr)
  * Byte 1: `0x03` (Func)
  * Byte 2: `0x10` (16 bytes de datos)
  * Bytes 3–4: Temperatura (°C / 10.0)
  * Bytes 5–6: Humedad (% / 10.0)
  * Bytes 7–8: Conductividad Eléctrica (EC µS/cm)
  * Bytes 9–10: Salinidad (Salt)
  * Bytes 11–16: Nitrógeno, Fósforo y Potasio (NPK mg/kg)
  * Bytes 17–18: pH (pH / 10.0)
  * Bytes 19–20: CRC16

---

## 4. Estado y Próximos Pasos
* **Hardware:** Circuito cerrado, estable y validado en banco con fuente de 12.5V.
* **Firmware:** `firmware/emisor_nodo/emisor_nodo.ino` consolidado y funcionando.
* **Transmisión:** Los datos leídos del sensor físico se inyectan a la radio LoRa SX1262 (868.0 MHz) y se transmiten hacia el Gateway.
* **Siguiente Sprint:** Integración de WebSockets en el backend FastAPI y actualización de las tarjetas KPI en el Dashboard Vue 3.
