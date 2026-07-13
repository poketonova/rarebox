<template>
  <div class="distribution-charts">
    <div class="chart-container">
      <div class="chart-header">
        <span class="chart-title">Value by Rarity</span>
      </div>
      <div v-if="raritySeries.length > 0" class="chart-wrapper">
        <apexchart
          type="donut"
          height="220"
          :options="rarityOptions"
          :series="raritySeries"
        />
      </div>
      <div v-else class="chart-empty text-muted">
        No rarity data available
      </div>
    </div>

    <div class="chart-container">
      <div class="chart-header">
        <span class="chart-title">Value by Set</span>
      </div>
      <div v-if="setSeries.length > 0" class="chart-wrapper">
        <apexchart
          type="donut"
          height="220"
          :options="setOptions"
          :series="setSeries"
        />
      </div>
      <div v-else class="chart-empty text-muted">
        No set data available
      </div>
    </div>
  </div>
</template>

<script setup>
import { computed } from 'vue'

const props = defineProps({
  items: { type: Array, required: true }
})

const processData = (groupByFn) => {
  const groups = {}
  props.items.forEach(item => {
    const key = groupByFn(item) || 'Other'
    const qty = item.quantity || 1
    const val = item.type === 'card'
      ? (item.currentMarketPrice || item.purchasePrice || 0)
      : (item.currentValue || item.purchasePrice || 0)
    groups[key] = (groups[key] || 0) + (val * qty)
  })

  const sorted = Object.entries(groups)
    .filter(([_, val]) => val > 0)
    .sort((a, b) => b[1] - a[1])

  const top = sorted.slice(0, 7)
  const other = sorted.slice(7).reduce((sum, [_, val]) => sum + val, 0)

  if (other > 0) {
    top.push(['Others', other])
  }

  return {
    labels: top.map(t => t[0]),
    series: top.map(t => Math.round(t[1] * 100) / 100)
  }
}

const rarityData = computed(() => processData(item => item.cardData?.rarity))
const setData = computed(() => processData(item => item.cardData?.set?.name || item.setName))

const raritySeries = computed(() => rarityData.value.series)
const rarityOptions = computed(() => getOptions(rarityData.value.labels))

const setSeries = computed(() => setData.value.series)
const setOptions = computed(() => getOptions(setData.value.labels))

function getOptions(labels) {
  return {
    chart: { type: 'donut', background: 'transparent' },
    labels: labels,
    theme: { mode: 'dark' },
    stroke: { show: false },
    dataLabels: { enabled: false },
    legend: {
      position: 'bottom',
      fontSize: '11px',
      labels: { colors: '#8b949e' },
      markers: { width: 8, height: 8 }
    },
    tooltip: {
      theme: 'dark',
      y: { formatter: (v) => `$${v.toFixed(2)}` }
    },
    plotOptions: {
      pie: {
        donut: {
          size: '65%',
          labels: {
            show: true,
            name: { show: true, fontSize: '12px', color: '#8b949e', offsetY: -5 },
            value: {
              show: true,
              fontSize: '14px',
              fontWeight: 700,
              color: '#f5a623',
              offsetY: 5,
              formatter: (v) => `$${parseFloat(v).toFixed(0)}`
            },
            total: {
              show: true,
              label: 'Total',
              color: '#8b949e',
              formatter: (w) => {
                const total = w.globals.seriesTotals.reduce((a, b) => a + b, 0)
                return `$${total.toFixed(0)}`
              }
            }
          }
        }
      }
    },
    colors: ['#f5a623', '#58a6ff', '#3fb950', '#f85149', '#ab7df8', '#ff7b72', '#d29922', '#30363d']
  }
}
</script>

<style scoped>
.distribution-charts {
  display: grid;
  grid-template-columns: 1fr 1fr;
  gap: 16px;
}
.chart-container {
  background: var(--bg-secondary);
  border: 1px solid var(--border);
  border-radius: var(--radius-lg);
  padding: 16px;
  display: flex;
  flex-direction: column;
}
.chart-header {
  margin-bottom: 12px;
  text-align: center;
}
.chart-title {
  font-size: 13px;
  font-weight: 700;
  color: var(--text-secondary);
  text-transform: uppercase;
  letter-spacing: 0.5px;
}
.chart-wrapper {
  flex: 1;
  min-height: 220px;
}
.chart-empty {
  height: 220px;
  display: flex;
  align-items: center;
  justify-content: center;
  font-size: 13px;
}
@media (max-width: 640px) {
  .distribution-charts { grid-template-columns: 1fr; }
}
</style>
