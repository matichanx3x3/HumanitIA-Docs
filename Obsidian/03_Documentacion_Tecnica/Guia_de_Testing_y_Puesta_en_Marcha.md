---
tags:
  - testing
  - guia
  - puesta-en-marcha
  - hardware
  - backend
  - lora
aliases:
  - Guía de Pruebas
  - Runbook de Testing
---

# Guía de Testing y Puesta en Marcha (End-to-End)

Esta guía detalla los pasos exactos, el instrumental necesario y las etapas progresivas para verificar el funcionamiento de **Agritech HumanitIA**, desde la lectura física del sensor de suelo en el banco de pruebas hasta la persistencia y visualización en el Dashboard web.

---

## 1. Materiales e Instrumental Requeridos para el Test

### Hardware Esencial
| Componente | Función en el Test | Estado |
| :--- | :--- | :--- |
| **[[Heltec_V4_LoRa32\|Placa Heltec V4 (Emisor)]]** | Lectura Modbus UART y transmisión LoRa P2P | Adquirido |
| **[[Heltec_V4_LoRa32\|Placa Heltec V4 (Gateway RX)]]** | Recepción LoRa y puente Serial USB hacia el Hub | Adquirido |
| **[[Sensor_Suelo_7en1_RS485\|Sensor de Suelo 7 en 1]]** | Sonda física de Humedad, Temp, EC, pH, NPK | Adquirido |
| **[[Módulo_RS485_a_TTL_Auto_Flow\|Módulo Conversor HW-726]]** | Puente UART TTL ↔ RS485 diferencial con Auto-Flow | Adquirido |
| **Fuente de Alimentación 12V DC** | Alimentación aislada del sensor (mínimo 1A con bornera jack) | Requerido |
| **Cables Dupont M-H / H-H** | Interconexión entre placas y módulos | Adquirido |
| **Cable USB-C (Datos)** | Conexión del Heltec a la PC para flasheo y lectura Serial | Adquirido |
| **Antenas LoRa 868 MHz** | Antenas de resorte con pigtail U.FL (una para cada Heltec) | Adquirido |

### Software y Entorno
- **Python 3.10+** con dependencias: `pip install pyserial paho-mqtt`
- **Arduino IDE** o **PlatformIO** con soporte ESP32 instalado y librerías:
  - `RadioLib` (v6.0+)
  - `Adafruit SSD1306` y `Adafruit GFX`
- **Podman / Docker Compose** (para pruebas completas del Hub central)

---

## 2. Checklist Crítico Previo al Encendido

Antes de energizar el circuito, revisa estrictamente los siguientes 5 puntos de seguridad:

- [ ] **1. Antenas LoRa Conectadas:** Ambas placas Heltec V4 deben tener su antena conectada a la toma U.FL. *(Transmitir sin antena destruye el chip SX1262).*
- [ ] **2. Cables Invertidos del HW-726:** El cable **Negro** del conector JST va al pin `3V3` del Heltec; el cable **Rojo** va a `GND`. *(Guiarse siempre por la serigrafía trasera del módulo).*
- [ ] **3. Ubicación de Pines 4 y 5:** Los pines `GPIO 4` (RX) y `GPIO 5` (TX) en el Heltec V4 están situados en la parte **superior derecha** del header J3, al lado del conector de antena (no contar desde abajo).
- [ ] **4. Tierra Común (GND):** El borne negativo `(-)` de la fuente externa de 12V debe estar puenteado con el pin `GND` del Heltec V4.
- [ ] **5. Aislamiento de 12V:** La salida positiva `(+)` de la fuente de 12V va **exclusivamente** al cable Rojo del sensor de suelo. Jamás conectarla a pines del Heltec ni del HW-726.

---

## 3. Nivel 1: Test de Lectura Física y Debug en Texto Plano

**Objetivo:** Verificar que el sensor 7 en 1 responde a las tramas Modbus-RTU y entrega lecturas coherentes, sin requerir Docker, broker MQTT ni base de datos.

```
[Fuente 12V] ──(12V/GND)──► [Sensor Suelo] ──(RS485 A/B)──► [HW-726] ──(UART)──► [Heltec V4 TX] ──(USB)──► [PC: lora_debug_logger.py]
```

### Procedimiento:
1. Conecta los cables siguiendo el esquema de [[Sensor_Suelo_7en1_RS485]].
2. Flashea en el Heltec emisor el archivo `firmware/emisor_nodo/emisor_nodo.ino`.
3. Conecta la placa al puerto USB de tu equipo y enciende la fuente externa de 12V.
4. En la terminal de tu ordenador, ejecuta el script de depuración rápida:
   ```bash
   python lora_debug_logger.py
   ```
5. **Resultados Esperados:**
   - **En la Terminal:**
     ```text
     ✔ Conectado exitosamente a /dev/cu.usbserial-0001 (o COM3 / /dev/ttyUSB0)
     [2026-09-15 10:00:15] [#0001] [SENSOR 7-EN-1 OK] Hum: 48.2% | Temp: 22.1C | EC: 1120 uS/cm | pH: 6.7 | NPK: 38-19-55
     ```
   - **En la Pantalla OLED del Heltec:**
     Muestra en tiempo real `T: 22.1C  H: 48.2%` y `pH: 6.7  EC: 1120`.
   - **En el Archivo de Texto:**
     Se genera y actualiza en tiempo real `./sensor_debug_log.txt` con el historial de mediciones.

---

## 4. Nivel 2: Test de Enlace LoRa P2P (Emisor ↔ Gateway RX)

**Objetivo:** Validar que el paquete de telemetría viaja por radiofrecuencia (868.0 MHz) desde el nodo de campo hasta el gateway concentrador.

```
[Nodo Emisor + Sensor + 12V]  ── LoRa P2P (868 MHz) ──►  [Heltec V4 Gateway RX] ──(USB)──► [PC: lora_debug_logger.py]
```

### Procedimiento:
1. Mantén el nodo emisor encendido y transmitiendo cada 10 segundos.
2. Flashea en la **segunda placa Heltec V4** el firmware `firmware/receptor_gateway/receptor_gateway.ino`.
3. Conecta este Heltec Gateway por cable USB al ordenador.
4. Ejecuta el logger apuntando al puerto del receptor:
   ```bash
   python lora_debug_logger.py
   ```
5. **Resultados Esperados:**
   - La pantalla OLED del Gateway muestra: `HELTEC V4 GATEWAY`, contador de paquetes `Pkts: <N>` y valor RSSI (ej: `-45 dBm`).
   - El script captura la trama compacta entrante:
     ```text
     [2026-09-15 10:05:22] [#0001] nodo_campo_01|Temp:22.1C, Hum:48.2%, pH:6.7, EC:1120, NPK:38-19-55
     ```

---

## 5. Nivel 3: Test del Stack Completo (Edge Hub, MQTT y Base de Datos)

**Objetivo:** Verificar la cadena completa de custodia del dato: persistencia en PostgreSQL con PostGIS y visualización gráfica en el Dashboard Vue 3.

```
[LoRa Air] ──► [Heltec RX] ──(Serial)──► [lora_serial_listener.py] ──(MQTT)──► [Mosquitto] ──► [mqtt_ingest.py] ──► [PostgreSQL] ──► [FastAPI] ──► [Vue Dashboard]
```

### Procedimiento:
1. Iniciar los contenedores del Hub central (en Linux o Windows WSL2):
   ```bash
   docker compose up -d
   ```
2. Ejecutar el puente Serial-a-MQTT:
   ```bash
   python lora_serial_listener.py
   ```
3. **Verificación en Base de Datos:**
   - Accede a Adminer en tu navegador: `http://localhost:8080`.
   - Motor: **PostgreSQL** | Servidor: `db` | Usuario: `agritech_user` | Clave: `agritech_secret` | Base: `agritech_db`.
   - Consulta la tabla `sensor_data` para ver los registros insertados con timestamp.
4. **Verificación en Dashboard:**
   - Abre `http://localhost:8081` para ver los gráficos en tiempo real actualizándose con las métricas de suelo.

---

## 6. Guía de Solución de Problemas (Troubleshooting)

| Síntoma | Causa Probable | Solución |
| :--- | :--- | :--- |
| `0 bytes recibidos` o `Sin respuesta sensor` | Falta de energía o voltaje < 9V en el sensor | Verificar que la fuente de 12V esté conectada y entregue voltaje medible con multímetro. |
| `0 bytes recibidos` persistente | Falta de masa común | Conectar firmemente el borne `(-)` de la fuente al `GND` del Heltec. |
| `0 bytes recibidos` | Cables RS485 A/B invertidos | Intercambiar los cables Amarillo y Verde en las borneras `A+` y `B-` del módulo HW-726. |
| `0 bytes recibidos` | Pines UART equivocados | Verificar que el cable Azul del HW-726 vaya al pin marcado `4` y el Amarillo al `5` (arriba a la derecha). |
| Destello de un solo LED en HW-726 | Baudrate incompatible | El firmware incluye auto-scan (4800/9600 bps). Espera dos ciclos para que alterne la velocidad. |
| Gateway no recibe paquetes LoRa | Parámetros RF o antena | Confirmar que ambas placas usen 868.0 MHz y tengan `VEXT_PIN` en LOW y amplificadores en HIGH. |
