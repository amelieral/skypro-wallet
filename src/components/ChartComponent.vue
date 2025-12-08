<template>
  <div class="chart-container">
    <div class="chart-title">
      <h2>{{ formattedTotal }} ₽</h2>
      <p>Расходы за {{ periodLabel }}</p>
    </div>
    <div class="chart">
      <canvas ref="chartCanvas"></canvas>
    </div>
  </div>
</template>

<script setup>
import { ref, onMounted, watch, computed } from 'vue'
import ChartDataLabels from 'chartjs-plugin-datalabels'
import Chart from 'chart.js/auto'

const props = defineProps({
  categories: {
    type: Array,
    default: () => ['Еда', 'Транспорт', 'Жилье', 'Развлечения', 'Образование', 'Другое'],
  },
  expenses: {
    type: Array,
    default: () => [],
  },
  startDate: String,
  endDate: String,
})

const normalizedCategories = {
  food: 'Еда',
  transport: 'Транспорт',
  housing: 'Жилье',
  entertainment: 'Развлечения',
  education: 'Образование',
  other: 'Другое',
}

const chartCanvas = ref(null)
let chartInstance = null

const chartData = computed(() => {
  const data = new Array(props.categories.length).fill(0)

  props.expenses.forEach(({ category, amount }) => {
    const russianCategory = normalizedCategories[category] || 'Другое'
    const index = props.categories.indexOf(russianCategory)
    if (index !== -1) data[index] += Number(amount) || 0
  })

  return data
})

const formattedTotal = computed(() => {
  const total = chartData.value.reduce((acc, val) => acc + val, 0)
  return new Intl.NumberFormat('ru-RU').format(total)
})

const periodLabel = computed(() => {
  if (props.startDate && props.endDate) return `${props.startDate} — ${props.endDate}`
  return 'период не выбран'
})

Chart.register(ChartDataLabels)

const initChart = () => {
  if (chartInstance) chartInstance.destroy()

  if (!chartCanvas.value) return

  const ctx = chartCanvas.value.getContext('2d')

  chartInstance = new Chart(ctx, {
    type: 'bar',
    data: {
      labels: props.categories,
      datasets: [
        {
          label: 'Расходы по категориям',
          data: chartData.value,
          backgroundColor: ['#D9B6FF', '#FFB53D', '#6EE4FE', '#B0AEFF', '#FFB9B8', '#e17055'],
          borderColor: '#ffffff',
          borderWidth: 2,
          borderRadius: 8,
          barPercentage: 1,
        },
      ],
    },
    options: {
      responsive: true,
      maintainAspectRatio: false,
      plugins: {
        legend: { display: false },
        tooltip: { enabled: false },
        datalabels: {
          color: '#000000', // ← ВОТ ЭТО и меняет цвет!
          anchor: 'end',
          align: 'top',
          formatter: (value) =>
            value > 0 ? new Intl.NumberFormat('ru-RU').format(value) + ' Р' : '',
          font: {
            weight: '600',
            size: 16,
          },
        },
      },
      scales: {
        y: {
          display: false,
          grid: { display: false },
          ticks: { display: false },
        },
        x: {
          grid: { display: false },
          ticks: {
            color: '#000000',
            font: { weight: '500' },
          },
        },
      },
      layout: {
        padding: {
          top: 30,
        },
      },
    },
    plugins: [ChartDataLabels],
  })
}

onMounted(initChart)

watch(
  [chartData, periodLabel],
  () => {
    if (!chartInstance) return

    chartInstance.data.datasets[0].data = chartData.value
    chartInstance.options.plugins.tooltip.enabled = props.expenses.length > 0
    chartInstance.update()
  },
  { deep: true },
)
</script>

<style scoped>
.chart-container {
  flex: 1;
  background: white;
  border-radius: 12px;
  border: 1px solid #e2e8f0;
  box-shadow: 0 1px 3px rgba(0, 0, 0, 0.1);
  padding: 20px;
  width: 100%;
  height: 623.67px;
}

::v-deep(.chartjs-datalabel) {
  transform: translateY(-8px);
  text-shadow: 0 1px 2px rgba(0, 0, 0, 0.1);
}

.chart-title {
  margin-bottom: 20px;
}

.chart-title h2 {
  font-size: 24px;
  font-weight: 700;
  color: #000000;
  margin-bottom: 4px;
}

.chart-title p {
  font-size: 14px;
  color: #999999;
}

.chart {
  height: 480px;
  position: relative;
}

.chart canvas {
  width: 100%;
  height: 100%;
}
</style>
