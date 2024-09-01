<template>
  <div class="bg-white rounded-xl shadow-md overflow-hidden w-full">
    <div class="p-6">
      <h2 class="text-xl font-semibold text-gray-800 mb-4">資料視覺化</h2>
      <div class="grid grid-cols-1 md:grid-cols-2 gap-8">
        <div ref="yearlyTotalChart" class="w-full h-80"></div>
        <div ref="growthRateChart" class="w-full h-80"></div>
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref, onMounted, onBeforeUnmount, watch } from 'vue'
import * as echarts from 'echarts'

const props = defineProps({
  data: Object
})

const yearlyTotalChart = ref(null)
const growthRateChart = ref(null)
let yearlyTotalChartInstance = null
let growthRateChartInstance = null

const initCharts = () => {
  if (props.data && props.data.yearlyTotals && props.data.growthRates) {
    renderYearlyTotalChart()
    renderGrowthRateChart()
  }
}

const renderYearlyTotalChart = () => {
  if (!yearlyTotalChart.value) return
  yearlyTotalChartInstance = echarts.init(yearlyTotalChart.value)
  const option = {
    title: {
      text: '年度總額',
      left: 'center'
    },
    tooltip: {
      trigger: 'axis',
      formatter: '{b}: ${c}'
    },
    xAxis: {
      type: 'category',
      data: Object.keys(props.data.yearlyTotals)
    },
    yAxis: {
      type: 'value',
      name: '總額 (USD)',
      axisLabel: {
        formatter: (value) => {
          if (value >= 1e9) return value / 1e9 + 'B'
          if (value >= 1e6) return value / 1e6 + 'M'
          if (value >= 1e3) return value / 1e3 + 'K'
          return value
        }
      }
    },

    series: [
      {
        data: Object.values(props.data.yearlyTotals).map((value) => value.toFixed(2)),
        type: 'bar',
        showBackground: true,
        backgroundStyle: {
          color: 'rgba(180, 180, 180, 0.2)'
        },
        label: {
          show: true,
          position: 'top',
          formatter: (params) => `$${parseFloat(params.value).toFixed(2)}`
        }
      }
    ]
  }
  yearlyTotalChartInstance.setOption(option)
}

const renderGrowthRateChart = () => {
  if (!growthRateChart.value) return
  growthRateChartInstance = echarts.init(growthRateChart.value)
  const years = Object.keys(props.data.growthRates)
  const rates = Object.values(props.data.growthRates).map((rate) => {
    if (rate === 'N/A') return null
    return parseFloat(rate)
  })

  const option = {
    title: {
      text: '年度增長率',
      left: 'center'
    },
    tooltip: {
      trigger: 'axis',
      formatter: function (params) {
        const value = params[0].value
        return `${params[0].name}: ${value == null ? 'N/A' : value.toFixed(2) + '%'}`
      }
    },
    xAxis: {
      type: 'category',
      data: years
    },
    yAxis: {
      type: 'value',
      name: '增長率 (%)'
    },
    series: [
      {
        data: rates,
        type: 'line',
        smooth: true,
        label: {
          show: true,
          position: 'top',
          formatter: function (params) {
            return params.value === null ? 'N/A' : params.value.toFixed(2) + '%'
          }
        },
        connectNulls: true
      }
    ]
  }
  growthRateChartInstance.setOption(option)
}

const handleResize = () => {
  if (yearlyTotalChartInstance) yearlyTotalChartInstance.resize()
  if (growthRateChartInstance) growthRateChartInstance.resize()
}

onMounted(() => {
  initCharts()
  window.addEventListener('resize', handleResize)
})

onBeforeUnmount(() => {
  window.removeEventListener('resize', handleResize)
  if (yearlyTotalChartInstance) {
    yearlyTotalChartInstance.dispose()
    yearlyTotalChartInstance = null
  }
  if (growthRateChartInstance) {
    growthRateChartInstance.dispose()
    growthRateChartInstance = null
  }
})

watch(
  () => props.data,
  () => {
    initCharts()
  },
  { deep: true }
)
</script>
