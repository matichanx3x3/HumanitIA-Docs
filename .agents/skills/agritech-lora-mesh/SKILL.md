---
name: agritech-lora-mesh
description: >-
  Handles native LoRa P2P communication using RadioLib (SX1262) on Heltec WiFi LoRa 32 V4 boards, serial port interfaces (pyserial on Linux /dev/ttyUSB* and Windows COM*), compact telemetry frame format, and the serial-to-MQTT gateway bridge. Use when working on LoRa communication, Heltec node firmware, serial gateway listener, or testing field packet transmissions.
---

# LoRa P2P Native Communication & Heltec V4 Integration Runbook

This skill covers the integration, testing, and operation of the **native LoRa P2P** network using **Heltec WiFi LoRa 32 V4** nodes, the **RadioLib** library (SX1262), and the Python serial gateway bridge.

> [!IMPORTANT]
> **Meshtastic is NOT used in this project.** The architecture uses native LoRa P2P via RadioLib to keep payloads lightweight and enable Deep Sleep on field nodes. Do not introduce Meshtastic dependencies.

## Hardware & RF Configuration

### Supported Board: Heltec WiFi LoRa 32 V4 (ESP32-S3 + SX1262)

#### Critical Pin Map (Heltec V4)

| Pin | GPIO | Function | Notes |
| :--- | :--- | :--- | :--- |
| `LORA_NSS` | 8 | SPI Chip Select (SX1262) | |
| `LORA_DIO1` | 14 | Radio Interrupt | |
| `LORA_RST` | 12 | Radio Reset | |
| `LORA_BUSY` | 13 | Radio Busy Status | |
| `VEXT_PIN` | 36 | Power for OLED & RF stage | **ACTIVE LOW** — set LOW to power on |
| `VFEM_PWR` | 7 | Front-End Module (GC1109) power | Set HIGH to enable |
| `FEM_EN` | 2 | Front-End Module enable | Set HIGH to enable |
| `FEM_CPS` | 46 | TX/RX antenna switch | Set HIGH |
| `LED_PIN` | 35 | Onboard user LED | Active HIGH |
| `OLED_SDA` | 17 | I2C Data (SSD1306/SSD1315) | |
| `OLED_SCL` | 18 | I2C Clock | |
| `OLED_RST` | 21 | OLED Reset | |

#### Radio Parameters (RadioLib SX1262)

| Parameter | Value |
| :--- | :--- |
| Frequency | **868.0 MHz** |
| Bandwidth | 125.0 kHz |
| Spreading Factor | 9 |
| Coding Rate | 4/7 |
| Sync Word | 0x12 |
| TX Power | 14 dBm |

RadioLib initialization call:
```cpp
int state = radio.begin(868.0, 125.0, 9, 7, 0x12, 14);
```

### Serial Port Configuration & Multiplatform Support
- **Baud Rate**: 115200 bps
- **macOS (Darwin - Solo diagnóstico / firmware)**: `/dev/cu.usbmodem*` (Heltec V4 CDC), `/dev/cu.usbserial-*` (CP2102/CH340/FTDI).
  > [!NOTE]
  > macOS queda **descartado de los dispositivos requeridos para montar el stack Podman**. En macOS únicamente se realizan diagnósticos de hardware o pruebas locales. Siempre usar `/dev/cu.*` (Calling Unit) y no `/dev/tty.*`.
- **Linux (Ubuntu / Fedora / Arch - Plataforma Oficial)**: `/dev/ttyUSB0`, `/dev/ttyACM0`. Requiere grupo `dialout` (o `uucp` en Arch) y reglas udev. Entorno nativo para el stack Podman.
- **Windows (Host Nativo / WSL2)**: `COM3`, `COM7`, `COM8`. Acceso directo con Python en PowerShell/CMD sin privilegios adicionales. Stack Podman soportado bajo WSL2.
- **Configurador Multiplataforma**: Ejecutar `bash setup_iot_permissions.sh` para diagnosticar y configurar permisos automáticamente según el SO.

## Architecture Flow

```
[Nodo Campo (Emisor)]                    [Gateway (Receptor)]
  RadioLib SX1262 TX                       RadioLib SX1262 RX
  firmware/emisor_nodo/                    firmware/receptor_gateway/
         │                                          │
         │  LoRa P2P (868 MHz, air)                 │
         └──────────────────────────────────────────►│
                                                    │ Serial.println(payload)
                                                    ▼
                                          [USB Serial Cable]
                                                    │
                                                    ▼
                                      [lora_serial_listener.py]
                                        pyserial + paho-mqtt
                                                    │
                                                    ▼ MQTT Publish
                                        [Eclipse Mosquitto Broker]
                                                    │
                                                    ▼ Subscribe sensors/#
                                        [mqtt_ingest.py → PostgreSQL]
```

## Telemetry Payload Format (8 Registros Modbus Oficiales)

The LoRa network uses a compact text format with a `node_id|data` structure based on the official Manual V2.2:

### Frame Structure:
```text
<node_id>|Temp:<float>C, Hum:<float>%, EC:<float>, Sal:<float>, NPK:<N>-<P>-<K>, pH:<float>
```

### Example:
```text
nodo_campo_01|Temp:22.5C, Hum:45.0%, EC:800, Sal:400, NPK:35-18-52, pH:6.8
```

> [!NOTE]
> **Derivación Agronómica de Salinidad**: Si una sonda física 7-en-1 retorna `0` en el Registro 3 (Sal) con `EC > 0`, tanto el firmware como el listener derivan automáticamente la salinidad con la fórmula oficial: $\text{Sal} = \text{EC} \times 0.5$.

### Parsed JSON (by `lora_serial_listener.py` before publishing to MQTT):
```json
{
  "temperature": 22.5,
  "humidity": 45.0,
  "soil_moisture": 45.0,
  "ec": 800.0,
  "salinity": 400.0,
  "nitrogen": 35,
  "phosphorus": 18,
  "potassium": 52,
  "ph": 6.8
}
```
MQTT Topic: `sensors/<node_id>/telemetry`

## Bidirectional LoRa P2P (Class A Downlink)

Para modificar remotamente el intervalo de muestreo del nodo emisor sin sacrificar batería:
1. **Frontend / API:** Envía `INTERVAL|<segundos>` hacia MQTT `sensors/<node_id>/downlink`.
2. **Listener Gateway:** Encola el comando en `pending_downlinks[node_id]`.
3. **Sincronización:** Cuando el nodo emisor transmite su telemetría y abre su **ventana de recepción de 3 segundos** (`LORA_DIO1`), el Gateway transmite inmediatamente el comando `<node_id|INTERVAL|<segundos>>`.
4. **Persistencia Flash (NVS):** El emisor recibe el comando, lo guarda en memoria permanente (`Preferences.h`) y ajusta su ciclo de lectura.

## Firmware Reference

### Emisor (Field Node TX/RX): `firmware/emisor_nodo/emisor_nodo.ino`
- Lee en vivo los 8 registros del sensor de suelo RS485 Modbus a 9600 bps (Dirección esclava `0x02`).
- Transmite telemetría vía `radio.transmit(payload)` a 868.0 MHz.
- Inmediatamente abre una ventana de escucha RX de 3 segundos esperando downlinks antes de dormir.
- Persistencia en Flash NVS (`Preferences.h`) para conservar el intervalo entre reinicios y Deep Sleep. Arreglo por defecto en 15 segundos para pruebas.

### Receptor (Gateway RX/TX): `firmware/receptor_gateway/receptor_gateway.ino`
- Escucha permanentemente en `radio.startReceive()` con interrupciones (`setPacketReceivedAction`).
- Entrega las tramas recibidas por USB Serial al PC y muestra métricas completas y RSSI en OLED.
- Monitorea `Serial.available()` para transmitir inmediatamente comandos downlink hacia los nodos.


## Python Scripts (Multiplataforma)
 
### LoRa Serial Simulator: `heltec_lora_sim.py`
Conecta a la placa Heltec emisora e inyecta tramas de telemetría simulada (útil para pruebas en laboratorio sin un segundo nodo):
 
```bash
# Auto-detecta el puerto según el SO (macOS /dev/cu.*, Linux /dev/ttyUSB*, Windows COM*)
python heltec_lora_sim.py

# O especificando puerto y baudrate manualmente
python heltec_lora_sim.py --port /dev/cu.usbserial-0001 --interval 15
```
 
### Serial-to-MQTT Gateway: `lora_serial_listener.py`
Lee tramas de la placa Heltec RX (Gateway) por puerto serie, las parsea con regex y las publica como JSON estructurado al broker MQTT:
 
```bash
# Auto-detecta el puerto serie del receptor
python lora_serial_listener.py

# O indicando parámetros
python lora_serial_listener.py --port /dev/cu.usbmodem1101 --broker localhost --mqtt-port 1883
```
 
Publica hacia el tópico MQTT: `sensors/<node_id>/telemetry`.
 
## Troubleshooting LoRa & Serial Issues
 
1. **Permisos de Puerto Denegados (Linux)**:
   ```bash
   bash setup_iot_permissions.sh
   # O manualmente: sudo usermod -aG dialout $USER && sudo chmod 666 /dev/ttyUSB0
   ```
2. **Puerto bloqueado / 'Device or resource busy'**:
   - En macOS: Asegurarse de usar `/dev/cu.*` y no `/dev/tty.*`.
   - Cerrar cualquier monitor serial abierto (Arduino IDE, PlatformIO Serial Monitor, Cura, screen).
3. **Hardware en Windows / WSL**:
   - Para máxima estabilidad, ejecutar los scripts Python en el PowerShell nativo de Windows.
   - Si se usa WSL2, recordar vincular el dispositivo con `usbipd attach --wsl --busid <BUSID>`.
3. **No Packets Received on Gateway**:
   - Verify both boards use identical RF parameters (868.0 MHz, BW 125, SF 9, CR 4/7, SyncWord 0x12).
   - Ensure `VEXT_PIN` is set to LOW and `VFEM_PWR`, `FEM_EN`, `FEM_CPS` are HIGH on both boards.
   - Check antenna is connected (transmitting without antenna can damage the SX1262).
4. **OLED Not Displaying**:
   - Confirm I2C address `0x3C` via `Wire.beginTransmission(0x3C)`.
   - Ensure `OLED_RST` (GPIO 21) is toggled LOW→HIGH during init.
5. **Production Deep Sleep**:
   - Replace `delay(10000)` with `ESP.deepSleep(microseconds)` in the emisor firmware for battery-powered field deployments.
