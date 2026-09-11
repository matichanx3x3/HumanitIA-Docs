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

## Telemetry Ingest Worker Specification (`app/workers/mqtt_ingest.py`)

### Expected Payload JSON:
```json
{
  "node_id": "sensor_01",
  "temperature": 23.4,
  "humidity": 65.2,
  "ph": 6.7,
  "soil_moisture": 42.0
}
```

### Database Model (`SensorData` in `app/db/models.py`):
- `id` (Integer, Primary Key, auto-increment)
- `node_id` (String, indexed)
- `topic` (String, indexed)
- `temperature` (Float)
- `humidity` (Float)
- `ph` (Float)
- `soil_moisture` (Float)
- `timestamp` (DateTime with timezone, defaults to `func.now()`)

## Testing and Manual Ingestion Injection

You can test the ingestion worker locally without running the simulator container:

### 1. Publish a Test MQTT Message via CLI:
```bash
# Using Mosquitto CLI client inside the mosquitto container
podman exec -it hub_mosquitto mosquitto_pub -h localhost -t "sensors/test_node_99/telemetry" -m '{"node_id": "test_node_99", "temperature": 25.4, "humidity": 70.1, "ph": 6.8, "soil_moisture": 55.3}'
```

### 2. Verify Ingestion in PostgreSQL:
```bash
podman exec -it hub_postgres psql -U agritech_user -d agritech_db -c "SELECT * FROM sensor_data ORDER BY timestamp DESC LIMIT 5;"
```

## Troubleshooting Ingestion Failures

1. **Broker Connection Refused**:
   - Verify `hub_mosquitto` container is up.
   - Inside containers, broker host is `mosquitto` on port `1883`.
2. **JSON Decode Errors**:
   - When non-JSON payloads are received, `mqtt_ingest.py` logs a warning and does not crash.
   - If string parsing for compact LoRa format is needed, implement a regex or string splitter fallback before `json.loads`.
3. **Database Session Leaks**:
   - Always wrap database sessions in `try...finally: db.close()` to avoid exhausting PostgreSQL connection pools.
