---
tags:
  - hardware
  - china
  - sourcing
  - opciones
fecha: 2026-09-07
---
# Alternativas de Hardware Agrícola: Soluciones "All-in-One"

Como parte de la estrategia de mitigación contra fallas físicas (sulfatación de soldaduras en campo) y opciones de masificación rápida, se han investigado soluciones de hardware integrado o "All-in-One" de proveedores chinos.

Estas alternativas vienen preensambladas y en muchos casos listas para la intemperie, simplificando el despliegue frente a la arquitectura custom (Heltec + RS485 manual).

## 1. Dragino LSE01 (LoRaWAN Soil Moisture & EC Sensor)
- **Rango de Precio:** ~$55 - $70 USD.
- **Características:** Integra sensor de humedad de suelo, temperatura y conductividad eléctrica (EC).
- **Ventajas para el Proyecto:** 
  - Carcasa industrial certificada **IP66**.
  - Incluye batería no recargable de cloruro de tionilo de litio (8500mAh Li-SOCI2) que dura años sin necesidad de paneles solares.
  - Elimina completamente la necesidad de soldaduras manuales.
- **Consideraciones:** Funciona típicamente sobre LoRaWAN. Habría que validar su uso en topología P2P transparente si se mantiene el Gateway Heltec, o migrar el Gateway a un concentrador LoRaWAN estándar.

## 2. Seeed SenseCAP S2104 (Soil Temperature and Moisture Sensor)
- **Rango de Precio:** ~$85 - $100 USD.
- **Ventajas para el Proyecto:**
  - Acabados muy premium y durabilidad extrema (IP66).
  - Incluye Bluetooth (BLE) para configurar rápidamente los intervalos de envío mediante una app móvil (ideal para clientes finales/instaladores).
- **Desventaja:** Su costo impactaría los márgenes en el modelo comercial agresivo de bajo costo.

## 3. Makerfabs ESP32 LoRa Soil Moisture
- **Rango de Precio:** ~$15 - $20 USD.
- **Características:** Es una placa electrónica con forma de estaca (el sensor capacitivo está impreso en el propio PCB).
- **Ventajas:** Extremadamente barato.
- **Desventajas:** No mide NPK, carece de recubrimiento industrial (hay que impermeabilizarlo manualmente rociando barniz o resina) y su sensor capacitivo impreso se degrada más rápido que las sondas de acero inoxidable del sensor 7-en-1 actual.

## 4. Estrategia con Arquitectura Actual (Heltec + 7-en-1 RS485)
Si se decide continuar fabricando el nodo de forma manual para mantener el costo bajo los 76€ y seguir midiendo NPK (que las opciones all-in-one baratas no suelen incluir), la solución para las fallas de soldadura es:
- **Conectores de Aviación (GX12 o M8) IP68:** En lugar de soldar cables directamente, usar estos conectores con rosca.
- **Resina Epóxica (Potting):** Encapsular los empates y componentes internos sensibles dentro de cajas de derivación rellenadas con resina. Es un método industrial definitivo contra la humedad y sulfatación.
