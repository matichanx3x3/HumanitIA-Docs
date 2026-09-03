---
name: agritech-lora-mesh
description: >-
  Handles LoRa mesh communication, Heltec V4 LoRa32 node integration, serial port interfaces (Meshtastic API on Linux /dev/ttyUSB* and Windows COM*), channel configuration, packet crafting, and hardware telemetry injection. Use when working on LoRa communication, Heltec nodes, serial gateway, or testing field packet transmissions.
---

# LoRa Mesh & Heltec V4 Integration Runbook

This skill covers the integration, testing, and operation of the LoRa Mesh network using **Heltec V4 LoRa32** nodes and the **Meshtastic** Python API.

## Hardware & Connection Configuration

- **Supported Boards**: Heltec WiFi LoRa 32 V3 / V4 (ESP32-S3 + SX1262).
- **Serial Ports**:
  - Linux / WSL: `/dev/ttyUSB0`, `/dev/ttyUSB1`, `/dev/ttyACM0`
  - Windows: `COM3`, `COM7` (adjust depending on Device Manager)
- **Meshtastic Channel Setup**:
  - Primary Channel (`0`): Administrative / Network Mesh
  - Secondary Channel (`1`): `"test"` or `"agritech"` - Used for sensor telemetry broadcasts to prevent polluting the default channel.

## Telemetry Payload Format

The Agritech LoRa network encodes sensor metrics into concise text strings or JSON payloads:

### Compact String Format:
```text
Temp:<float>C, Hum:<float>%, pH:<float>, EC:<float>, NPK:<N>-<P>-<K>
```
Example:
```text
Temp:24.5C, Hum:62.3%, pH:6.8, EC:1.45, NPK:45-22-68
```

## Running the LoRa Simulator / Injector

```bash
# Run simulator
python heltec_lora_sim.py
```
