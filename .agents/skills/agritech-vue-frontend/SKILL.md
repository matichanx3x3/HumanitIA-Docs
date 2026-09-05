---
name: agritech-vue-frontend
description: >-
  Guides development of the Agritech Vue 3 frontend dashboard using Composition API, Vite, Chart.js / vue-chartjs for sensor telemetries, and native modern CSS variables (strict NO-Tailwind rule). Use when building UI components, charts, KPI summary cards, or updating dashboard views.
---

# Agritech Vue 3 Frontend Runbook

This skill establishes the frontend architecture, styling standards, and charting patterns for the **Agritech HumanitIA Dashboard**.

## Frontend Philosophy & Constraints

- **Framework**: Vue 3 (Composition API `<script setup lang="ts">`) + Vite.
- **Styling Architecture**: **Pure Vanilla CSS with Native CSS Custom Properties** (`src/assets/main.css`).
- > [!WARNING]
  > **STRICT RULE: NO TAILWIND CSS.** Do not introduce Tailwind classes or dependencies. All UI styling must adhere to the design token variables in `main.css`, CSS Grid, and Flexbox for clean and modular UI.
- **Visual Style**: Dark modern Theme, frosted glass cards (backdrop-filter), accent greens (`#10b981`), high contrast typography (*Inter* font).

## Local Development & Build

```bash
# In frontend directory
cd frontend
npm install
npm run dev

# Production build test
npm run build
```
Port binding in Docker / Podman is mapped to **`8081`** on the host.
