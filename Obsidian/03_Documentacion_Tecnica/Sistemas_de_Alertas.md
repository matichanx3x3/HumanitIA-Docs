---
tags:
  - backend
  - alertas
  - notificaciones
  - fastapi
fecha: 2026-09-07
---
# Sistemas de Alertas y Notificaciones (Reglas de Negocio)

Para dar valor real a la información almacenada en PostgreSQL ("*qué hacer con la información*"), el backend en FastAPI debe contar con un Motor de Reglas (Rule Engine) que evalúe umbrales (ej. Humedad < 15%) y detone notificaciones hacia los usuarios.

A continuación, se documentan las principales vías de integración investigadas para el envío de alertas directas al móvil del agricultor.

## 1. Telegram Bot API (Recomendación para Desarrollo)
- **Costo:** 0 € (Totalmente gratuito e ilimitado).
- **Ventajas:**
  - Creación de bot en 1 minuto vía *BotFather*.
  - Integración nativa en Python rapidísima (solo requiere un POST a la API REST de Telegram).
  - Permite crear grupos por parcela donde se envíen las alarmas de todos los nodos.
- **Uso:** Ideal para el desarrollo actual, pilotos internos y primeras validaciones de las "macetas".

## 2. CallMeBot (API no oficial para WhatsApp)
- **Costo:** 0 €.
- **Ventajas:**
  - Permite enviar mensajes directamente a una cuenta de WhatsApp personal mediante una petición HTTP GET/POST muy sencilla.
- **Desventajas:**
  - No es una solución corporativa. Depende de un servicio de terceros sin SLA garantizado.
- **Uso:** Estrictamente para pruebas (PoC) donde se quiere validar la UX de recibir la alerta en WhatsApp sin pasar por la burocracia de Meta.

## 3. Meta WhatsApp Cloud API (Oficial)
- **Costo:** Modelo de pago por conversación, pero Meta ofrece las primeras **1.000 conversaciones de servicio gratuitas al mes** por cada cuenta empresarial.
- **Ventajas:**
  - Solución profesional, escalable y oficial B2B.
  - El cliente recibe el mensaje desde una cuenta oficial de "Agritech HumanitIA" (WhatsApp Business), otorgando gran estatus al producto.
- **Desventajas:**
  - Requiere registro de la empresa en Facebook Business Manager.
  - Los mensajes iniciados por el bot ("Plantillas" o Templates de advertencia) deben ser pre-aprobados por Meta.
- **Uso:** Es la plataforma definitiva para la **Masificación** y despliegue en clientes reales.

## 4. Twilio (SMS y WhatsApp)
- **Costo:** ~$0.005 USD por mensaje de WhatsApp. Los SMS varían por país.
- **Ventajas:**
  - Una sola API que puede enviar tanto WhatsApp como SMS (Fallback si el cliente no tiene datos móviles en el campo).
- **Desventajas:**
  - Agrega un costo recurrente operativo por cada notificación enviada.

## Siguiente Paso Lógico
Para la fase de pruebas actual (piloto de 2 plantas), la arquitectura de software debe integrar un script simple en los workers de FastAPI (`app/workers/`) que utilice **Telegram Bot API** o **CallMeBot**. Posteriormente, el módulo de envío de mensajes se abstraerá para poder inyectar la API de **Meta** cuando el producto pase a producción masiva.
