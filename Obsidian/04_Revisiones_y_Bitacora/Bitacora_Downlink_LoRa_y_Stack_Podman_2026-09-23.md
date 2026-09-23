---
tags:
  - bitacora
  - podman
  - lora-mesh
  - downlink
  - frontend
  - backend
  - troubleshooting
fecha: 2026-09-23
autor: "Equipo Agritech Core & Antigravity"
estado: "completado_y_validado"
---
# Bitácora Técnica: Despliegue de Stack Podman, Downlink LoRa P2P Clase A y Actualización de WebApp

## 1. Resumen Ejecutivo
En esta jornada se completó la consolidación integral del ecosistema **Agritech HumanitIA**:
1. **Infraestructura Podman / WSL2:** Resolución de dependencias de registries en Ubuntu WSL2, gestión limpia de señales de apagado (`SIGTERM`) en workers MQTT y estandarización del puente serial multiplataforma.
2. **Derivación Agronómica de Salinidad:** Corrección del comportamiento de sondas físicas Modbus RS485 (cuyo Registro 3 inicializa en 0) mediante la derivación agronómica oficial desde la Conductividad Eléctrica ($\text{Salinidad} = \text{EC} \times 0.5$).
3. **Comunicación LoRa Bidireccional (Clase A):** Implementación de control remoto del intervalo de muestreo del nodo emisor de campo (1s a 12h) utilizando ventana RX de 3s post-TX (`LORA_DIO1`), almacenamiento no volátil en Flash (`Preferences.h`) y una cola sincronizada de downlinks en `lora_serial_listener.py`.
4. **Renovación de la WebApp:** Corrección de rutas SPA en NGINX (`try_files 404`), reorganización ergonómica de vistas (*Overview* resumido vs *Sensores y Dispositivos* con 8 parámetros completos), purga de datos simulados y reportes exportables en CSV y PDF.

---

## 2. Infraestructura y Orquestación Podman (WSL2)

### A. Error de Registries no Calificados (`short-name did not resolve`)
* **Problema:** Al ejecutar `podman-compose up -d`, la compilación fallaba al resolver nombres cortos de imágenes como `postgis/postgis:15-3.3` o `nginx:alpine`.
* **Causa:** En instalaciones limpias de Ubuntu en WSL2, `/etc/containers/registries.conf` no incluye registros de búsqueda predeterminados por políticas de seguridad de Podman.
* **Solución:** Se configuró Docker Hub y Quay como registros de búsqueda en la máquina WSL:
  ```bash
  sudo bash -c 'cat <<EOF > /etc/containers/registries.conf
  [registries.search]
  registries = ["docker.io", "quay.io"]
  EOF'
  ```

### B. Apagado Limpio y Señales de Sistema (`SIGTERM` en `hub_worker_ingesta`)
* **Problema:** Al hacer `podman-compose down`, el contenedor del worker tardaba 10 segundos y forzaba un `SIGKILL` (código 137).
* **Causa:** `paho-mqtt` bloqueaba el bucle `client.loop_forever()` sin capturar señales de interrupción del sistema operativo.
* **Solución:** Se añadieron manejadores explícitos con la librería `signal` en `app/workers/mqtt_ingest.py`:
  ```python
  def handle_exit_signal(signum, frame):
      client.disconnect()
      client.loop_stop()
      sys.exit(0)

  signal.signal(signal.SIGTERM, handle_exit_signal)
  signal.signal(signal.SIGINT, handle_exit_signal)
  ```

### C. Puente Serial USB y Topología WSL2 vs Windows
* **Problema:** Al intentar correr `python3 lora_serial_listener.py` dentro de WSL2, arrojaba error de permisos en `/dev/ttyS7`.
* **Causa:** WSL2 es una máquina virtual que no tiene acceso directo a los puertos COM de Windows a menos que se use `usbipd-win`.
* **Solución y Regla Operativa:** Ejecutar el listener nativamente en Windows PowerShell:
  ```powershell
  python lora_serial_listener.py --port COM4
  ```
  El script detecta la placa Heltec en Windows y se conecta sin fricción al broker Mosquitto en `localhost:1883` expuesto por Podman.

---

## 3. Telemetría Agronómica: Derivación de Salinidad

* **Hallazgo Físico:** En pruebas de laboratorio con soluciones salinas, las sondas 7-en-1 comerciales retornaban `0x0000` en el Registro Modbus 3 (Salinidad), mientras que el Registro 2 (EC) reaccionaba inmediatamente.
* **Explicación Agronómica:** La mayoría de sensores industriales no poseen electrodos electroquímicos dedicados a iones de sal; calculan la salinidad como un factor de conversión lineal de la conductividad eléctrica.
* **Solución:** Si `data.salt == 0` y `data.ec > 0`, se deriva mediante la fórmula oficial del manual V2.2:
  $$\text{Salinidad (mg/L o ppm)} = \text{EC} \times 0.5$$
* **Implementación Defensiva:** Se incorporó idéntica lógica en `emisor_nodo.ino`, `lora_serial_listener.py` y `lora_debug_logger.py`.

---

## 4. Arquitectura de Downlink LoRa P2P Sincronizado (Clase A)

### El Desafío
El usuario solicitó cambiar remotamente desde la web el tiempo de lectura y transmisión del **nodo emisor de campo**. Sin embargo, para maximizar la vida útil de la batería, el emisor pasa el 99% del tiempo en sueño profundo (`Deep Sleep`). Si el usuario cambia el tiempo desde la web en un instante aleatorio, el emisor está sordo y el comando se pierde.

### La Solución Implementada

```
[WebApp Dashboard] (Slider 1s - 12h)
       ↓ POST /api/v1/sensors/{node_id}/config
[FastAPI] publica en MQTT: sensors/{node_id}/downlink -> "INTERVAL|60"
       ↓
[lora_serial_listener.py] Encola el comando en 'pending_downlinks[node_id]'
       ↓ (Espera a que el nodo emisor despierte y transmita)
[Emisor Heltec V4] Transmite lectura Modbus por LoRa P2P
       ↓
[Receptor Gateway] Recibe trama y la entrega por Serial al PC
       ↓
[lora_serial_listener.py] Detecta paquete de {node_id}, espera 200ms
       ↓ Escribe al Serial: <nodo_campo_01|INTERVAL|60>
[Receptor Gateway] Emite por LoRa P2P por el aire
       ↓
[Emisor Heltec V4] (En su ventana RX de 3s con LORA_DIO1)
       ↓ Recibe trama, extrae 60s, guarda en Flash NVS (Preferences)
       ↓ Muestra en OLED: "NUEVA CONF Int: 60s"
[Emisor Heltec V4] Ajusta su nuevo ciclo de sueño dinámicamente.
```

* **Persistencia NVS:** Uso de `#include <Preferences.h>` en ESP32 para que el intervalo sobreviva a reinicios y al Deep Sleep.
* **Intervalo inicial de fábrica:** Configurado a **15 segundos** para pruebas ágiles en banco de trabajo.

---

## 5. Actualización Integral de la WebApp (Frontend y Backend)

### A. Solución al Error 404 de NGINX en SPA
* **Problema:** Al recargar la página (F5) en `/devices` o `/devices/:id`, NGINX devolvía `404 Not Found`.
* **Solución:** Se creó `frontend/nginx.conf` con `try_files $uri $uri/ /index.html;` y se actualizó `frontend/Dockerfile`.

### B. Separación Ergonómica de Vistas
* **Overview (`DashboardView.vue`):** Muestra tarjetas compactas y limpias exclusivamente de los sensores activos, indicando estado (*En línea* o *Inactivo*), hora de última lectura y signos vitales mínimos (Temp, Hum, Suelo).
* **Sensores y Dispositivos (`DevicesView.vue`):** Concentra los paneles detallados con los **8 parámetros oficiales completos** (Temp, Hum, Suelo, pH, EC, Salinidad, N, P, K).
* **Modal de Configuración:** Slider continuo de 1 segundo a 12 horas con atajos (5 seg, 5 min, 1 hora) y advertencia visual sobre autonomía de batería.
* **Exportación de Reportes:** Generación de archivos tabulares `.csv` desde la API y reportes visuales con diseño de impresión `.pdf` nativo.
* **Mantenimiento y Purga:** Botón `🧹 Limpiar Nodos Dummy` y botón papelera `🗑️` para eliminar lecturas de sensores de prueba antiguos (`esp32_*`).
* **Análisis de Sensor (`DeviceDetailView.vue`):** Desplegable actualizado con las 9 métricas, gráficas de líneas, barras y tabla histórica.

---

## 6. Estado y Validación
* **Prueba de Campo:** Emisor Heltec V4 transmitiendo exitosamente telemetría real del sensor de suelo 7-en-1 cada 15 segundos.
* **Comprobación Downlink:** Comandos de intervalo enviados desde la web recibidos e interpretados correctamente por el emisor, reflejando el cambio de ritmo en tiempo real.
* **Resultados:** Stack de producción 100% operativo, libre de nodos simulados y sincronizado de extremo a extremo.
