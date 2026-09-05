---
name: agritech-docs-and-hardware
description: >-
  Workflows for managing documentation in HumanitIA-Docs, syncing technical architecture specifications, updating the Obsidian vault, and documenting electronic hardware/sensors (ESP32-CAM, Heltec V4, RS485, GY33, voltage converters, logic analyzers). Use when updating project documentation, sprint tasks, component datasheets, or cross-referencing hardware schematics.
---

# Agritech Documentation & Hardware Knowledge Sync

This skill establishes the documentation standards and synchronization workflows between the codebase (`hub_agritech_core`) and the central documentation repository (`HumanitIA-Docs`).

## Repository Architecture Mapping

```
HumanitIA-Docs/
├── docs/
│   ├── Arquitectura/         # PDF diagrams: Hardware, Microservices, Edge AI
│   ├── Capas/                # Communication, Hardware, Frontend, Core API, Persistence
│   ├── Documentacion/        # Sprint logs, handoff_context.md
│   └── Plan/                 # Pitch, Executive summaries, Gantt charts
└── Obsidian/
    ├── 01_Planificacion_y_Tiempos/  # Tasks_Semanales.md, Cronograma
    ├── 02_Documentacion_General/    # System explanation, basic electronics
    ├── 03_Documentacion_Tecnica/    # Backend, Hardware, Infrastructure, Frontend
    ├── 04_Revisiones_y_Bitacora/    # Periodic audit logs
    ├── 05_Propuestas_y_Competencia/ # Market analysis
    ├── 06_Negocio_y_Economia/       # Financial models & strategy
    └── 07_Componentes/              # Hardware component sheets & pinouts
```

## Hardware Component Documentation Standard

When documenting new electronics, sensors, or modules in `Obsidian/07_Componentes/`, follow the project template:

### Structure:
1. **Header**: Name, Model, Category, Working Voltage (e.g., 3.3V / 5V / 12V), Operating Current.
2. **Key Technical Specifications**: Protocols (I2C, SPI, UART/RS485, LoRa 915MHz), Baudrates, Pins.
3. **Pinout Table**:
   | Pin | Function | Connects To |
   | :--- | :--- | :--- |
   | `VCC` | Power (3.3V/5V) | Power Rail |
   | `GND` | Ground | Common Ground |
   | `TX` / `DI` | Transmit Data | Microcontroller RX |
   | `RX` / `RO` | Receive Data | Microcontroller TX |
4. **Integration Code Snippet**: Minimal working driver or simulation example.
5. **Safety / Level Shifting Notes**: E.g. TXS0108E bidirectional level shifter requirement between 5V sensors and 3.3V ESP32 inputs.
