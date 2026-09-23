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
- [ ] **2. Alimentación 5V y Cables del HW-726:** El cable **Negro** del conector JST va al pin **`5V`** del Heltec (Header J2, Pin 2) porque el chip transceptor MAX485 requiere 5V para el bus diferencial; el cable **Rojo** va a `GND`.
- [ ] **3. Ubicación de Pines GPIO 4 y 5:** Conectar el cable **Amarillo (RXD del módulo)** al `GPIO 4` (Header J3, Pin 15 / RX) y el cable **Azul (TXD del módulo)** al `GPIO 5` (Header J3, Pin 16 / TX). *(Evitar estrictamente los pines U0TXD y U0RXD del puerto USB)*.
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
     ✔ Conectado exitosamente a /dev/cu.usbserial-0001 (o COM8 / /dev/ttyUSB0)
     [2026-09-22 18:30:15] [#0001] [TELEMETRIA] Nodo: [nodo_campo_01]
        ├─ Reg 0: Temperatura:          22.5 °C
        ├─ Reg 1: Humedad de suelo:     45.0 %
        ├─ Reg 2: Conductividad (EC):   800.0 µS/cm
        ├─ Reg 3: Salinidad (Salt):     400.0 mg/L
        ├─ Reg 4..6: Nutrientes NPK:    N=35 mg/kg | P=18 mg/kg | K=52 mg/kg
        ├─ Reg 7: Acidez / Alcalinidad: pH 6.8
        └─ RAW: nodo_campo_01|Temp:22.5C, Hum:45.0%, EC:800, Sal:400, NPK:35-18-52, pH:6.8
     ```
   - **En la Pantalla OLED del Heltec:**
     Muestra en tiempo real `T:22.5C H:45.0%`, `EC:800 Sal:400` y `pH:6.8 NPK:35-18-52`.
   - **En el Archivo de Texto:**
     Se genera y actualiza en tiempo real `./sensor_debug_log.txt` con el historial de mediciones.

---

## 4. Nivel 2: Test de Enlace LoRa P2P (Emisor ↔ Gateway RX)

**Objetivo:** Validar que el paquete de telemetría viaja por radiofrecuencia (868.0 MHz) desde el nodo de campo hasta el gateway concentrador.

```
[Nodo Emisor + Sensor + 12V]  ── LoRa P2P (868 MHz) ──►  [Heltec V4 Gateway RX] ──(USB)──► [PC: lora_debug_logger.py]
```

### Procedimiento:
1. Mantén el nodo emisor encendido y transmitiendo cada 5 segundos.
2. Flashea en la **segunda placa Heltec V4** el firmware `firmware/receptor_gateway/receptor_gateway.ino`.
3. Conecta este Heltec Gateway por cable USB al ordenador.
4. Ejecuta el logger apuntando al puerto del receptor:
   ```bash
   python lora_debug_logger.py
   ```
5. **Resultados Esperados:**
   - La pantalla OLED del Gateway muestra: `HELTEC V4 GATEWAY`, contador de paquetes `Pkts: <N>`, valor RSSI (ej: `-45 dBm`), y los datos decodificados en vivo (`T`, `H`, `EC`, `Sal`, `NPK`, `pH`).
   - El script captura el evento de recepción con RSSI/SNR y la trama compacta entrante:
     ```text
     [2026-09-22 18:35:10] [#0001] [GATEWAY INFO] -> [LORA RX #1] RSSI: -45.0 dBm | SNR: 9.5 dB
     [2026-09-22 18:35:10] [#0002] [TELEMETRIA] Nodo: [nodo_campo_01]
        ├─ Reg 0: Temperatura:          22.5 °C
        ├─ Reg 1: Humedad de suelo:     45.0 %
        ├─ Reg 2: Conductividad (EC):   800.0 µS/cm
        ├─ Reg 3: Salinidad (Salt):     400.0 mg/L
        ├─ Reg 4..6: Nutrientes NPK:    N=35 mg/kg | P=18 mg/kg | K=52 mg/kg
        ├─ Reg 7: Acidez / Alcalinidad: pH 6.8
        └─ RAW: nodo_campo_01|Temp:22.5C, Hum:45.0%, EC:800, Sal:400, NPK:35-18-52, pH:6.8
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
   **Salida en consola:**
   ```text
   [2026-09-22 18:36:00] [LORA RECEPTOR GATEWAY] -> Paquete recibido de [nodo_campo_01]
      ├─ Reg 0: Temperatura:          22.5 °C
      ├─ Reg 1: Humedad de suelo:     45.0 %
      ├─ Reg 2: Conductividad (EC):   800.0 µS/cm
      ├─ Reg 3: Salinidad (Salt):     400.0 mg/L
      ├─ Reg 4..6: Nutrientes NPK:    N=35 mg/kg | P=18 mg/kg | K=52 mg/kg
      ├─ Reg 7: Acidez / Alcalinidad: pH 6.8
      └─> Publicado en MQTT -> Tópico: 'sensors/nodo_campo_01/telemetry'
   ```
3. **Verificación en Base de Datos:**
   - Accede a Adminer en tu navegador: `http://localhost:8080`.
   - Motor: **PostgreSQL** | Servidor: `db` | Usuario: `agritech_user` | Clave: `agritech_secret` | Base: `agritech_db`.
   - Consulta la tabla `sensor_data` para ver los registros insertados con timestamp y las 8 variables agronómicas persistidas (`temperature`, `humidity`, `soil_moisture`, `ec`, `salinity`, `nitrogen`, `phosphorus`, `potassium`, `ph`).
4. **Verificación en Dashboard:**
   - Abre `http://localhost:8081` para ver los gráficos en tiempo real actualizándose con las métricas de suelo.

---

## 6. Nivel 4: Test de Configuración Bidireccional (Downlink LoRa Clase A)

**Objetivo:** Verificar que el tiempo de muestreo y transmisión del emisor físico a batería puede ser reprogramado remotamente por aire desde el Dashboard web sin reiniciar el nodo.

```
[Dashboard Web: 8081] ──(HTTP POST)──► [FastAPI: 8000] ──(MQTT downlink)──► [lora_serial_listener.py: Encolado]
                                                                                   │
                                                  ┌────────────────────────────────┘ (Al recibir telemetría)
                                                  ▼
[Emisor de Campo (Ventana RX 3s)] ◄──(LoRa Air)── [Receptor Gateway] ◄──(Serial USB)─┘
       │
       ├─► Guarda nuevo intervalo en Flash NVS (Preferences.h)
       ├─► Muestra en pantalla OLED: "NUEVA CONF Int: Xs"
       └─► Aplica nuevo ritmo de transmisión autónomo
```

### Procedimiento:
1. Con los contenedores y `lora_serial_listener.py` activos, ingresa a `http://localhost:8081/devices`.
2. Ubica el panel de tu nodo (ej. `nodo_campo_01`) y haz clic en **`⚙️ Configurar Sensor`**.
3. Ajusta el deslizador a un nuevo intervalo (por ejemplo, `5 seg` o `60 seg`) y haz clic en **Aplicar Configuración**.
4. Observa la consola de `lora_serial_listener.py`:
   ```text
   ⏳ [DOWNLINK ENCOLADO] Configuración encolada para [nodo_campo_01]: INTERVAL|60
      El Gateway la transmitirá automáticamente en cuanto [nodo_campo_01] envíe su próximo reporte LoRa.
   ```
5. En el instante en que el emisor transmite su siguiente lectura:
   ```text
   🎯 [DOWNLINK SINCRONIZADO] Enviando comando a [nodo_campo_01] en su ventana de escucha: <nodo_campo_01|INTERVAL|60>
      ✔ Comando enviado al Gateway para transmisión aérea hacia [nodo_campo_01]
   ```
6. **Verificación en el Emisor:**
   - En la consola serial del Emisor:
     ```text
     ✔ [LORA RX OK] Paquete recibido por aire: <nodo_campo_01|INTERVAL|60>
     ★ [CONFIG OK] ¡Nuevo intervalo guardado en Flash NVS!: 60 segundos
     ```
   - En la pantalla OLED del Emisor aparecerá temporalmente: **`NUEVA CONF Int: 60s`**.
   - El nodo pasará a transmitir exactamente cada 60 segundos de forma continua y el valor persistirá tras apagar o reiniciar la placa.

---

## 7. Guía de Solución de Problemas (Troubleshooting)

| Síntoma | Causa Probable | Solución |
| :--- | :--- | :--- |
| `0 bytes recibidos` o `Sin respuesta sensor` | Falta de energía o voltaje < 9V en el sensor | Verificar que la fuente de 12V esté conectada y entregue voltaje medible con multímetro o voltímetro digital LM2596. |
| `0 bytes recibidos` persistente | Falta de masa común | Conectar firmemente el borne `(-)` de la fuente al `GND` del Heltec V4. |
| `0 bytes recibidos` | Cables RS485 A/B invertidos | Intercambiar los cables Amarillo y Verde en las borneras `A+` y `B-` del módulo HW-726. |
| `0 bytes recibidos` | Pines UART equivocados | Conectar cable Azul (`<--- TXD`) al GPIO 5 (TX Heltec) y Amarillo (`---> RXD`) al GPIO 4 (RX Heltec). |
| Destello de un solo LED en HW-726 | Baudrate o esclavo incompatible | El firmware usa 9600 bps y dirección esclava 0x02 según Manual V2.2 oficial. |
| Gateway no recibe paquetes LoRa | Parámetros RF o antena | Confirmar que ambas placas usen 868.0 MHz (SX1262) y tengan `VEXT_PIN` en LOW y amplificadores encendidos. |
| Paquetes en OLED pero Python muestra `(Paquetes: 0)` | `USB CDC On Boot` deshabilitado | En Arduino IDE ir a **Tools -> USB CDC On Boot:** y seleccionar **"Enabled"**. Re-flashear el Gateway. |
| Error `short-name did not resolve` en Podman WSL2 | Registries no definidos en Ubuntu | Ejecutar: `sudo bash -c 'cat <<EOF > /etc/containers/registries.conf\n[registries.search]\nregistries = ["docker.io", "quay.io"]\nEOF'` |
| `404 Not Found` en NGINX al recargar página web | Configuración por defecto de NGINX en SPA | Resuelto con `frontend/nginx.conf` (`try_files $uri $uri/ /index.html;`). |
| `Permission denied: '/dev/ttyS7'` en `lora_serial_listener.py` | Ejecución dentro de WSL2 sin puente USB | Ejecutar el listener nativamente en Windows PowerShell (`python lora_serial_listener.py --port COM4`). |
| El sensor no actualiza su intervalo de envío | El emisor duerme y pierde el comando | Resuelto mediante la **cola sincronizada de downlink** en `lora_serial_listener.py` y ventana RX de 3s en el emisor. |
