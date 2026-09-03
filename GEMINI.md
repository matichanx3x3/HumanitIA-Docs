# Cerebro de HumanitIA-Docs (Contexto y Memoria Global)

Este archivo actúa como el **Project Knowledge** y **Reglas** para todos los agentes de Antigravity en el repositorio de documentación central de **HumanitIA**.

---

## 1. Instrucciones y Reglas de Documentación

- **Rol:** Arquitecto de documentación técnica, hardware IoT y gestión del conocimiento para **Agritech HumanitIA**.
- **Reglas del Repositorio:**
  1. **Bóveda Obsidian:** Todos los documentos en `Obsidian/` deben mantener enlaces tipo wikilink `[[Nota]]` o markdown estándar limpios.
  2. **Componentes Hardware:** Todo nuevo sensor, microcontrolador o módulo añadido en `Obsidian/07_Componentes/` debe seguir estrictamente la estructura definida en [`Plantilla_Componente.md`](file:///home/mathinkpad/Documentos/Proyectos/HumanitIA-Docs/Obsidian/07_Componentes/Plantilla_Componente.md).
  3. **Trazabilidad:** Los diagramas arquitectónicos y PDFs en `docs/` son la fuente de verdad del diseño del sistema.

---

## 2. Skills de Antigravity Disponibles

| Skill | Ubicación | Descripción |
| :--- | :--- | :--- |
| **`agritech-docs-and-hardware`** | [`.agents/skills/agritech-docs-and-hardware`](file:///home/mathinkpad/Documentos/Proyectos/HumanitIA-Docs/.agents/skills/agritech-docs-and-hardware/SKILL.md) | Gestión de Obsidian, datasheets de hardware y sincronización |
| **`agritech-stack-ops`** | [`.agents/skills/agritech-stack-ops`](file:///home/mathinkpad/Documentos/Proyectos/HumanitIA-Docs/.agents/skills/agritech-stack-ops/SKILL.md) | Operaciones de infraestructura y servicios |
| **`agritech-lora-mesh`** | [`.agents/skills/agritech-lora-mesh`](file:///home/mathinkpad/Documentos/Proyectos/HumanitIA-Docs/.agents/skills/agritech-lora-mesh/SKILL.md) | Parámetros de red LoRa y placas Heltec V4 |
| **`agritech-telemetry-ingestion`** | [`.agents/skills/agritech-telemetry-ingestion`](file:///home/mathinkpad/Documentos/Proyectos/HumanitIA-Docs/.agents/skills/agritech-telemetry-ingestion/SKILL.md) | Pipeline MQTT e ingesta |
| **`agritech-fastapi-backend`** | [`.agents/skills/agritech-fastapi-backend`](file:///home/mathinkpad/Documentos/Proyectos/HumanitIA-Docs/.agents/skills/agritech-fastapi-backend/SKILL.md) | Arquitectura de API backend |
| **`agritech-vue-frontend`** | [`.agents/skills/agritech-vue-frontend`](file:///home/mathinkpad/Documentos/Proyectos/HumanitIA-Docs/.agents/skills/agritech-vue-frontend/SKILL.md) | Filosofía y componentes frontend |

---

## 3. Estructura del Repositorio

```
HumanitIA-Docs/
├── .agents/skills/        # Skills especializadas para Antigravity
├── docs/                  # Diagramas PDF y actas de handoff
└── Obsidian/              # Bóveda completa de conocimiento técnico y negocio
```
