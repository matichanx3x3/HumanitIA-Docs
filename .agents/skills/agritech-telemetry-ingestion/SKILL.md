---
name: agritech-telemetry-ingestion
description: >-
  Develops, tests, and debugs the MQTT ingestion pipeline and background workers (mqtt_ingest.py, paho-mqtt, Mosquitto topics, payload schema validation, SQLAlchemy persistence). Use when creating new MQTT topics, ingesting sensor payloads, modifying telemetry schema, or handling ingestion failures.
---

# Agritech Telemetry Ingestion Pipeline

This skill guides the implementation, extension, and maintenance of the telemetry ingestion worker that subscribes to Mosquitto MQTT and writes data to PostgreSQL.

## Ingestion Architecture

```
[Sensors / ESP32 / Gateway]
            │
            ▼ (MQTT Publish)
   [Eclipse Mosquitto Broker]
            │
            ▼ (MQTT Subscribe `sensors/#`)
 [app.workers.mqtt_ingest (Python / paho-mqtt)]
            │
            ▼ (SQLAlchemy ORM)
 [PostgreSQL 15 (sensor_data table)]
```

## MQTT Topic Conventions

- Standard telemetry topic: `sensors/{node_id}/telemetry`
- System & diagnostic topic: `sensors/{node_id}/status`
- LoRa Gateway ingestion topic: `gateway/lora/rx`

Wildcard subscription: `sensors/#`

## Manual Test Injection
```bash
podman exec -it hub_mosquitto mosquitto_pub -h localhost -t "sensors/test_node_99/telemetry" -m '{"node_id": "test_node_99", "temperature": 25.4, "humidity": 70.1, "ph": 6.8, "soil_moisture": 55.3}'
```
