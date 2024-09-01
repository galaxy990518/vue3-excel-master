<template>
  <div class="bg-white rounded-xl shadow-md overflow-hidden w-full">
    <div class="p-6">
      <h2 class="text-xl font-semibold text-gray-800 mb-6 flex items-center">
        <svg
          xmlns="http://www.w3.org/2000/svg"
          class="h-6 w-6 mr-2 text-indigo-600"
          fill="none"
          viewBox="0 0 24 24"
          stroke="currentColor"
        >
          <path
            stroke-linecap="round"
            stroke-linejoin="round"
            stroke-width="2"
            d="M9 19v-6a2 2 0 00-2-2H5a2 2 0 00-2 2v6a2 2 0 002 2h2a2 2 0 002-2zm0 0V9a2 2 0 012-2h2a2 2 0 012 2v10m-6 0a2 2 0 002 2h2a2 2 0 002-2m0 0V5a2 2 0 012-2h2a2 2 0 012 2v14a2 2 0 01-2 2h-2a2 2 0 01-2-2z"
          />
        </svg>
        資料視覺化
      </h2>
      <div class="grid grid-cols-1 md:grid-cols-2 gap-8">
        <div class="bg-gray-50 rounded-lg p-4">
          <h3 class="text-lg font-medium text-gray-700 mb-3 flex items-center">
            <svg
              xmlns="http://www.w3.org/2000/svg"
              class="h-5 w-5 mr-2 text-indigo-500"
              fill="none"
              viewBox="0 0 24 24"
              stroke="currentColor"
            >
              <path
                stroke-linecap="round"
                stroke-linejoin="round"
                stroke-width="2"
                d="M7 12l3-3 3 3 4-4M8 21l4-4 4 4M3 4h18M4 4h16v12a1 1 0 01-1 1H5a1 1 0 01-1-1V4z"
              />
            </svg>
            年度總額
          </h3>
          <div ref="yearlyTotalChart" class="w-full h-72"></div>
        </div>
        <div class="bg-gray-50 rounded-lg p-4">
          <h3 class="text-lg font-medium text-gray-700 mb-3 flex items-center">
            <svg
              xmlns="http://www.w3.org/2000/svg"
              class="h-5 w-5 mr-2 text-indigo-500"
              fill="none"
              viewBox="0 0 24 24"
              stroke="currentColor"
            >
              <path
                stroke-linecap="round"
                stroke-linejoin="round"
                stroke-width="2"
                d="M13 7h8m0 0v8m0-8l-8 8-4-4-6 6"
              />
            </svg>
            年度成長率
          </h3>
          <div ref="growthRateChart" class="w-full h-72"></div>
        </div>
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref, onMounted, onBeforeUnmount, watch } from 'vue';
import * as echarts from 'echarts';

const props = defineProps({
  data: Object
});

const yearlyTotalChart = ref(null);
const growthRateChart = ref(null);
let yearlyTotalChartInstance = null;
let growthRateChartInstance = null;

const initCharts = () => {
  if (props.data && props.data.yearlyTotals && props.data.growthRates) {
    renderYearlyTotalChart();
    renderGrowthRateChart();
  }
};

const renderYearlyTotalChart = () => {
  if (!yearlyTotalChart.value) return;
  yearlyTotalChartInstance = echarts.init(yearlyTotalChart.value);
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
          if (value >= 1e9) return value / 1e9 + 'B';
          if (value >= 1e6) return value / 1e6 + 'M';
          if (value >= 1e3) return value / 1e3 + 'K';
          return value;
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
  };
  yearlyTotalChartInstance.setOption(option);
};

const renderGrowthRateChart = () => {
  if (!growthRateChart.value) return;
  growthRateChartInstance = echarts.init(growthRateChart.value);
  const years = Object.keys(props.data.growthRates);
  const rates = Object.values(props.data.growthRates).map((rate) => {
    if (rate === 'N/A') return null;
    return parseFloat(rate);
  });

  const option = {
    title: {
      text: '年度增長率',
      left: 'center'
    },
    tooltip: {
      trigger: 'axis',
      formatter: function (params) {
        const value = params[0].value;
        return `${params[0].name}: ${value == null ? 'N/A' : value.toFixed(2) + '%'}`;
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
            return params.value === null ? 'N/A' : params.value.toFixed(2) + '%';
          }
        },
        connectNulls: true
      }
    ]
  };
  growthRateChartInstance.setOption(option);
};

const handleResize = () => {
  if (yearlyTotalChartInstance) yearlyTotalChartInstance.resize();
  if (growthRateChartInstance) growthRateChartInstance.resize();
};

onMounted(() => {
  initCharts();
  window.addEventListener('resize', handleResize);
});

onBeforeUnmount(() => {
  window.removeEventListener('resize', handleResize);
  if (yearlyTotalChartInstance) {
    yearlyTotalChartInstance.dispose();
    yearlyTotalChartInstance = null;
  }
  if (growthRateChartInstance) {
    growthRateChartInstance.dispose();
    growthRateChartInstance = null;
  }
});

watch(
  () => props.data,
  () => {
    initCharts();
  },
  { deep: true }
);
</script>
