---
tags:
  - hardware
  - componente
  - rs485
  - conversor
  - uart
fabricante: "Genérico (HW-726 / Chip MAX485 con Auto-flow)"
estado: "adquirido_y_validado"
---
# Módulo Conversor TTL a RS485 (Auto Flow Control - HW-726)

## 1. Descripción General y Uso
Este módulo (frecuentemente marcado como HW-726 o XY-017) permite a un microcontrolador con puertos UART TTL (como el ESP32-S3 en el Heltec V4) comunicarse con redes y sensores industriales que utilizan el protocolo RS485 (como los sensores de humedad, temperatura y NPK de suelo).

La mayor ventaja de esta versión específica frente a los conversores MAX485 tradicionales es que incluye **Auto Flow Control (Control automático de flujo de dirección)** mediante una etapa de transistores. En los módulos básicos se requieren pines GPIO adicionales (`DE` y `/RE`) para decirle al chip cuándo transmitir y cuándo recibir. Este módulo conmuta automáticamente la dirección según detecta actividad en la línea de transmisión, ahorrando pines en el microcontrolador.

![RS485 a TTL Frontal](assets/rs485_ttl_front.png)

## 2. Datos Técnicos y Requisitos Eléctricos
- **Chip Principal:** Transceptor diferencial **MAX485ESA**.
- **Voltaje de Alimentación (VCC):** 
  - Aunque la serigrafía trasera indica `3.3~33V VCC` gracias a un regulador LDO integrado, **el chip MAX485 requiere 5V nominales (mínimo 4.75V) para excitar con la potencia adecuada el bus diferencial RS485.**
  - **Recomendación probada:** Alimentar directamente desde el pin **`5V` del Heltec V4** (Header J2, Pin 2, alimentado por USB-C).
- **Nivel Lógico (TTL):** Compatible con señales lógicas de 3.3V del ESP32.
- **Protocolo Industrial:** RS-485 (Half-duplex, diferencial $A+/B-$).
- **Protección de Bus:** Incorpora fusibles rearmables PPTC (marcados "010 K") y diodos TVS de supresión de transitorios contra picos de tensión.
- **Control de Dirección:** Automático por hardware (Auto-Flow).

## 3. Guía de Conexión y Pinout Validado

El módulo incluye un conector JST de 4 pines pre-cableado con código de colores invertido respecto a la norma convencional.

![RS485 a TTL Trasera](assets/rs485_ttl_back.png)

### Lado Microcontrolador (Cable JST de 4 hilos)

> [!CAUTION] ¡CÓDIGO DE COLORES Y FLECHAS DE LA SERIGRAFÍA TRASERA!
> 1. **Colores invertidos:** El cable Rojo físico está en el pin `GND` y el cable Negro está en `VCC`.
> 2. **Sentido de las flechas (`接线图`):** El fabricante rotuló las patillas indicando la función del microcontrolador externo:
>    * `TXD <--- TXD` (Cable Azul): Es una **entrada** al módulo; debe conectarse al pin **TX** del microcontrolador.
>    * `RXD ---> RXD` (Cable Amarillo): Es una **salida** del módulo; debe conectarse al pin **RX** del microcontrolador.

| Pin Serigrafía Trasera | Color Cable Físico | Conexión en Heltec V4 | Función Eléctrica |
| :--- | :--- | :--- | :--- |
| **VCC** (Pin Superior) | **Negro** | **Pin `5V`** (Header J2, Pin 2) | Alimentación 5V requerida por el MAX485. |
| **TXD** (2do Pin) | **Azul** | **GPIO 5** (Header J3, Pin 16 / TX) | Entrada de datos que el ESP32 envía al bus. |
| **RXD** (3er Pin) | **Amarillo** | **GPIO 4** (Header J3, Pin 15 / RX) | Salida de datos que el módulo entrega al ESP32. |
| **GND** (Pin Inferior) | **Rojo** | **Pin `GND`** (Cualquiera del Heltec) | Masa de referencia lógica. |

### Lado Industrial (Borneras de Tornillo RS485)
- **Bornera `A+`**: Conectar al cable **Amarillo** del sensor RS485 (Señal D+).
- **Bornera `B-`**: Conectar al cable **Verde** del sensor RS485 (Señal D-).
- **Bornera `接大地` (Earth)**: Terminal opcional para drenar la malla de apantallamiento de cables de campo largos a tierra física.

## 4. Diagnóstico Visual con LEDs Integrados
El módulo incorpora dos indicadores LED de actividad:
* **LED `TXD`:** Parpadea cuando el microcontrolador transmite una petición hacia el sensor (indica que el pin TX del ESP32 está excitando el módulo).
* **LED `RXD`:** Parpadea cuando el sensor de campo devuelve una respuesta hacia el microcontrolador.
*(En reposo normal, ambos LEDs permanecen apagados).*
