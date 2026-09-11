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

## Project Structure (`frontend/`)

```
frontend/
├── src/
│   ├── assets/
│   │   └── main.css            # Global CSS variables & layout utilities
│   ├── components/
│   │   ├── dashboard/          # Metric cards, status badges, charts
│   │   └── layout/             # AppLayout, Sidebar, Navbar
│   ├── router/
│   │   └── index.ts            # Vue Router routes (Dashboard, Devices, DeviceDetail)
│   ├── views/
│   │   ├── DashboardView.vue   # Main telemetry overview
│   │   ├── DevicesView.vue     # Node inventory list
│   │   └── DeviceDetailView.vue# Historical charts & single-sensor analytics
│   ├── App.vue
│   └── main.ts
├── Dockerfile                  # Multi-stage build (npm build -> nginx:alpine)
├── package.json
└── vite.config.ts
```

## Chart.js & vue-chartjs Integration

Historical telemetry metrics are visualized using `vue-chartjs` with `Chart.js`:

```vue
<template>
  <div class="chart-container">
    <Line :data="chartData" :options="chartOptions" />
  </div>
</template>

<script setup lang="ts">
import { computed } from 'vue'
import { Line } from 'vue-chartjs'
import {
  Chart as ChartJS,
  Title,
  Tooltip,
  Legend,
  LineElement,
  LinearScale,
  PointElement,
  CategoryScale,
} from 'chart.js'

ChartJS.register(Title, Tooltip, Legend, LineElement, LinearScale, PointElement, CategoryScale)

const props = defineProps<{
  timestamps: string[]
  values: number[]
  metricLabel: string
  lineColor?: string
}>()

const chartData = computed(() => ({
  labels: props.timestamps,
  datasets: [
    {
      label: props.metricLabel,
      data: props.values,
      borderColor: props.lineColor || '#10b981',
      backgroundColor: 'rgba(16, 185, 129, 0.1)',
      fill: true,
      tension: 0.3,
    },
  ],
}))

const chartOptions = {
  responsive: true,
  maintainAspectRatio: false,
  plugins: {
    legend: { labels: { color: '#94a3b8' } },
  },
  scales: {
    x: { grid: { color: 'rgba(255,255,255,0.05)' }, ticks: { color: '#94a3b8' } },
    y: { grid: { color: 'rgba(255,255,255,0.05)' }, ticks: { color: '#94a3b8' } },
  },
}
</script>
```

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
