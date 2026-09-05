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

![[Componentes.base]]

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
- [[Sensor_Suelo_7en1_RS485|Sensor de Suelo 7 en 1 (NPK, pH, EC, Temp, Humedad)]]
- [[Batería LiFePO4 y Panel Solar]]
- [[Módulo_ESP32_CAM_OV3660|Kit ESP32-CAM (OV3660 + Placa USB-C)]]
- [[Módulo_Sensor_Color_GY33|Módulo Sensor de Color GY-33 (TCS34725)]]

### Accesorios y Herramientas
- [[Cables_Dupont_Jumpers|Cables Jumper Dupont (M-M, M-H, H-H)]]
- [[Analizador_Lógico_24MHz|Analizador Lógico USB 24MHz (8 Canales)]]

---

## 2. Análisis de Brechas (Estado del Hardware)
Evaluando los diagramas generados en la sección técnica (🗜️ [[Hardware_y_Fisico]]) frente a los módulos base, se identificaron los siguientes componentes secundarios críticos. **Actualización:** Gran parte de estas brechas ya han sido cubiertas.

- ~~**Convertidores UART/RS485 a TTL:**~~ **✅ RESUELTO:** Se adquirió el [[Módulo_RS485_a_TTL_Auto_Flow|Módulo HW-726 Auto-Flow]], que permite la comunicación directa entre el ESP32 y el Sensor de Suelo 7 en 1.
- ~~**Adaptadores de Niveles Lógicos:**~~ **✅ RESUELTO:** Se adquirió el [[Módulo_TXS0108E_Conversor_Nivel|TXS0108E]] para asegurar la transición segura entre lógica de 3.3V y 5V sin dañar el ESP32.
- ~~**Reguladores de Voltaje (Buck/Boost):**~~ **✅ RESUELTO:** Se dispone del reductor [[Módulo_LM2596_Step_Down_Voltimetro|LM2596]] (hasta 3A) para bajar voltajes solares/baterías grandes a 5V, y el elevador [[Módulo_MT3608_Step_Up|MT3608]] para subir de 3.7V/5V a 12V para actuadores.
- **Electroválvulas Solenoide (12V/24V DC):** ❌ **FALTANTE:** El relé por sí solo es un interruptor. Se requiere comprar las llaves de paso físicas que cortarán el paso del agua.
- **Cajas Estancas IP67 o IP68:** ❌ **FALTANTE:** Enclosures plásticos con sellos de goma y pasacables impermeables (glándulas estopas) para evitar que la electrónica en el campo se dañe con la lluvia, el riego o la humedad matutina.
