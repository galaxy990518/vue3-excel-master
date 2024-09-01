<template>
  <div class="min-h-screen bg-gradient-to-r from-cyan-500 to-blue-500 py-6 px-4 sm:px-6 lg:px-8">
    <div class="max-w-7xl mx-auto">
      <h1 class="text-3xl font-bold text-white text-center mb-8">Vue3 Excel Master</h1>
      <div class="bg-white rounded-xl shadow-lg p-6">
        <FileUpload @dataProcessed="handleDataProcessed" />
        <DataVisualization v-if="processedData" :data="processedData" />
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref } from 'vue'
import FileUpload from '@/components/FileUpload.vue'
import DataVisualization from '@/components/DataVisualization.vue'

const processedData = ref(null)

const handleDataProcessed = (data) => {
  processedData.value = data
}
</script>