---
tags:
  - hardware
  - iot
  - lora
  - esp32
aliases:
  - Capa Física
  - Sensores
---
# Arquitectura de Hardware Físico

El diseño de hardware responde a las exigencias duras del entorno agroindustrial. En lugar de centralizar todo mediante cables largos propensos a caídas de tensión y ruido electromagnético, se implementa una **topología distribuida y descentralizada**.

## 1. Cerebro Hub (Caseta Central)
Se ubica en un lugar protegido con suministro eléctrico estable.
- **Servidor Central (Mini PC Linux):** Equipo industrial (Intel N100 o AMD Ryzen SFF). Aloja los contenedores, base de datos local y futuros módulos IA. Consumo ultra bajo (6W - 15W).
- **Nodo Concentrador (Gateway LoRa):** Un ESP32 configurado como receptor maestro, conectado directamente por cable USB al Mini PC. Escucha el aire, captura paquetes LoRa y los entrega por puerto serial (tty).
- **Conectividad Externa:** Router 3G/4G básico que dota al Hub de una salida mínima a internet (sólo notificaciones Push/WhatsApp).

👉 **Conoce el software de este servidor en:** [[Arquitectura_Backend]]

## 2. Entorno de Campo (Nodos Distribuidos IoT)
Equipos hiper-económicos y de muy bajo consumo distribuidos por las hectáreas.
- **Microcontrolador Base:** Familia ESP32 / Arduino. Funcionan con energía solar (panel + batería LiFePO4).
- **Sensórica Industrial:** Sondas NPK, humedad de suelo, y conductividad eléctrica (CE) cableadas al ESP32 usando protocolos robustos como **RS485 (Modbus-RTU)** o **SDI-12**.
- **Actuadores:** Módulos de relés de estado sólido optoacoplados para abrir/cerrar electroválvulas de agua.
- **Nodos de Visión:** Cámaras IP independientes que transmiten vía Wi-Fi direccional al hub central.

## 3. Capa de Comunicación Inalámbrica (Sin Internet)
- **Red LoRa (Larga Distancia - Nativo P2P):** Módulos RF a **868.0 MHz** (frecuencia operativa del hardware Heltec V4 adquirido / compatible con 915 MHz para despliegues en LATAM). Envían datos a kilómetros de distancia con un consumo ultra bajo. 
  - *Decisión Arquitectónica:* Se utiliza **LoRa Nativo Punto a Punto (P2P)** con la librería `RadioLib` en C++. **No se utiliza Meshtastic** debido a su alta sobrecarga de protocolo y dificultad para integrar lecturas directas de sensores industriales RS485 Modbus. El enfoque P2P permite a los nodos de campo entrar en Deep Sleep profundo y enviar telemetría directamente al Gateway (topología estrella).
- **Bluetooth (Emergencia):** Integrado en nodos ESP32 para lectura directa del operario mediante smartphone en el campo.

---

## Diagrama de Flujo Lógico y Físico

```mermaid
sequenceDiagram
    participant Campo as Sensores (RS485)
    participant Nodo as Nodo ESP32 (Campo)
    participant Gateway as Gateway LoRa (USB)
    participant Hub as Mini PC Linux (Hub)
    participant Rele as Electroválvula (Riego)

    Note over Campo, Nodo: Cable Eléctrico
    Campo->>Nodo: Lectura Analógica/Digital (Humedad, EC)
    Note over Nodo, Gateway: Ondas LoRa (868 MHz)
    Nodo->>Gateway: Transmisión de paquete encriptado
    Note over Gateway, Hub: Cable USB / Serial
    Gateway->>Hub: Ingesta a Broker MQTT y Postgres
    
    Note over Hub: Motor Determinista decide regar
    
    Hub->>Gateway: Comando de riego al broker
    Gateway->>Nodo: Paquete LoRa de actuación
    Nodo->>Rele: Excita PIN GPIO (Cierra relé)
```

---

## Diagrama de Topología y Cableado Físico (Nodo de Campo)

El siguiente diagrama detalla la arquitectura de alimentación y cableado interno que se utiliza para un nodo de campo (Heltec V4 LoRa32) interactuando con instrumentación industrial (RS485).

```mermaid
flowchart TD
    %% Alimentación y Energía
    subgraph Energia [Sistema de Alimentación Autónomo]
        Solar[Panel Solar 5V/6V] -->|Carga| TP4056[Módulo Carga TP4056]
        TP4056 -->|BATT+ / GND| Bateria[Batería LiFePO4 / 18650]
        Bateria -->|VCC 3.3V| LDO[Regulador LDO 3.3V]
    end
    
    %% Cerebro de Procesamiento
    subgraph Nodo [Caja Estanca IP67]
        LDO -->|Alimentación 3.3V| MCU[Placa Heltec V4 LoRa32 / ESP32]
        RS485[Módulo RS485 a TTL]
        MCU <-->|UART (TX/RX)| RS485
    end
    
    %% Instrumentos de Campo
    subgraph Terreno [Instrumentación Industrial]
        RS485 <-->|Cable Modbus A/B| NPK[Sonda NPK / CE / pH de Suelo]
        MCU -.->|Pin GPIO| Rele[Módulo Relé Estado Sólido]
    end
    
    %% Salida LoRa
    MCU -.->|Antena 868MHz| Lora(Radio LoRa P2P hacia Hub Central)

    %% Estilos
    classDef power fill:#fbd38d,stroke:#dd6b20,color:#000
    classDef mcu fill:#9ae6b4,stroke:#38a169,color:#000
    classDef sensor fill:#90cdf4,stroke:#3182ce,color:#000
    
    class Solar,TP4056,Bateria,LDO power
    class MCU mcu
    class RS485,NPK,Rele sensor
```

---

### Esquema Pin a Pin: Heltec V4 LoRa32 + Módulo HW-726 + Sensor Suelo 7-en-1

#### 1. Conexión Heltec V4 <--> Módulo HW-726 (Auto Flow Control)
> [!CAUTION] ¡ALERTA DE CABLE JST CON COLORES INVERTIDOS!
> En el conector de 4 hilos JST provisto con el HW-726 de fábrica, **el cable rojo corresponde a GND y el negro a VCC**. Guíate siempre de forma irrestricta por la **serigrafía impresa en el reverso de la placa**:

| Pin HW-726 (Serigrafía Trasera) | Color Físico del Cable JST | Pin en Heltec V4 (Header J3) | Función / Nivel Lógico |
| :--- | :--- | :--- | :--- |
| **VCC** (Pin Superior) | **Negro** *(Cuidado)* | **`3V3`** (Pin 2 o 3 de J3) | Alimentación 3.3V (Garantiza lógica TTL segura para ESP32) |
| **TXD** (2do Pin) | **Azul** | **`GPIO 4`** (Pin 15 de J3) | RX del microcontrolador (recibe datos del sensor) |
| **RXD** (3er Pin) | **Amarillo** | **`GPIO 5`** (Pin 16 de J3) | TX del microcontrolador (envía consulta Modbus) |
| **GND** (Pin Inferior) | **Rojo** *(Cuidado)* | **`GND`** (Pin 1 de J3) | Tierra común de referencia |

#### 2. Conexión Fuente 12V <--> Sensor Suelo 7-en-1 <--> Módulo HW-726

> [!CAUTION] ¡ALIMENTACIÓN AISLADA 12V Y TIERRA COMÚN (GND)!
> El sensor requiere una **fuente externa de 12V DC** conectada mediante bornera jack. El polo **Negativo (-)** de la fuente de 12V debe unirse obligatoriamente con el pin **GND del Heltec V4** (Masa Común). Los 12V van **únicamente al cable Rojo del sensor**.

| Origen | Terminal / Pin | Destino | Terminal / Pin | Función Eléctrica |
| :--- | :--- | :--- | :--- | :--- |
| **Fuente 12V** | Bornera Jack **`(+)`** | **Sensor Suelo** | **Cable Rojo** | 12V DC para encender la sonda y excitación de agujas |
| **Fuente 12V** | Bornera Jack **`(-)`** | **Sensor Suelo** | **Cable Negro** | Retorno de corriente 12V del sensor |
| **Fuente 12V** | Bornera Jack **`(-)`** | **Heltec V4** | **Pin `GND`** | **Tierra Común de Referencia (VITAL)** |
| **Sensor Suelo** | **Cable Amarillo** | **Módulo HW-726** | **Bornera `A+`** | Señal diferencial RS485 A (D+) |
| **Sensor Suelo** | **Cable Verde** | **Módulo HW-726** | **Bornera `B-`** | Señal diferencial RS485 B (D-) |

Para consultar la guía detallada de puesta en marcha paso a paso, ver [[Guia_de_Testing_y_Puesta_en_Marcha|Guía de Testing y Puesta en Marcha]].


