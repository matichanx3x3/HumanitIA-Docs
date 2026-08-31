---
tags:
  - hardware
  - componentes
  - inventario
aliases:
  - Inventario de Hardware
---
# Índice de Componentes (Hardware)

Este espacio está destinado a documentar exhaustivamente todo el hardware físico requerido para la ejecución del proyecto **Hub Agritech Core**. Cada componente listado aquí tendrá su propia nota individual con toda su información técnica, guías y recursos.

## 1. Inventario y Lista de Deseo (Bill of Materials)
*(Haz clic en cada enlace para crear la nota del componente usando la [[Plantilla_Componente]] cuando lo vayas investigando)*

### Cerebro y Red Central
- [[Mini PC Industrial]] (Ej. Intel N100 / AMD Ryzen)
- [[Router 3G-4G IoT]]

### Nodos de Comunicación y Procesamiento
- [[Microcontrolador ESP32]] (Cerebro de los nodos de campo y Gateway)
- [[Módulos LoRa 915MHz]] (Ej. Transceptores SX1276 o RFM95W)

### Alimentación y Regulación
- [[Módulo_LM2596_Step_Down_Voltimetro|Módulo LM2596 DC-DC Step Down con Voltímetro LED]]
- [[Módulo_MT3608_Step_Up|Módulo MT3608 DC-DC Step-Up (Boost) con USB Type-C]]

### Convertidores y Adaptadores
- [[Módulo_TXS0108E_Conversor_Nivel|Convertidor de Nivel Lógico Bi-direccional TXS0108E (8 Canales)]]
- [[Módulo_RS485_a_TTL_Auto_Flow|Módulo Conversor TTL a RS485 (HW-726 Auto Flow)]]

### Periféricos de Campo
- [[Módulos de Relé de Estado Sólido]] (Para accionar válvulas)
- [[Sensores de Suelo NPK y Humedad]] (Protocolo industrial RS485 / Modbus-RTU)
- [[Batería LiFePO4 y Panel Solar]]
- [[Módulo_ESP32_CAM_OV3660|Kit ESP32-CAM (OV3660 + Placa USB-C)]]
- [[Módulo_Sensor_Color_GY33|Módulo Sensor de Color GY-33 (TCS34725)]]

### Accesorios y Herramientas
- [[Cables_Dupont_Jumpers|Cables Jumper Dupont (M-M, M-H, H-H)]]
- [[Analizador_Lógico_24MHz|Analizador Lógico USB 24MHz (8 Canales)]]

---

## 2. Análisis de Brechas (Hardware Faltante)
Evaluando los diagramas generados en la sección técnica (👉 [[Hardware_y_Fisico]]) frente a los módulos base, se identifica que **harán falta los siguientes componentes secundarios** para asegurar que el hardware funcione físicamente en el campo:

- **Convertidores UART/RS485 a TTL:** Dado que los sensores industriales de suelo usan RS485 (cables A y B), y el ESP32 trabaja con TTL (RX/TX), es estrictamente necesario un módulo conversor (como el MAX485) para que el ESP32 pueda "hablar" con el sensor de suelo.
- **Módulos Reductores de Voltaje (Buck Converters):** Si el panel solar o la batería operan a 12V o 24V (lo cual es estándar para mover electroválvulas), necesitaremos conversores Step-Down (ej. LM2596 o MP1584) para bajar el voltaje a 5V o 3.3V de forma segura para alimentar los ESP32 sin quemarlos.
- **Electroválvulas Solenoide (12V/24V DC):** El relé por sí solo es un interruptor. Se requiere comprar las llaves de paso físicas que cortarán el paso del agua.
- **Cajas Estancas IP67 o IP68:** Enclosures plásticos con sellos de goma y pasacables impermeables (glándulas estopas) para evitar que la electrónica en el campo se dañe con la lluvia, el riego o la humedad matutina.
