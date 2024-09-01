<template>
  <div class="bg-white rounded-xl shadow-md overflow-hidden max-w-2xl w-full mb-8">
    <div class="p-6">
      <div class="flex items-center justify-between mb-4">
        <h2 class="text-xl font-semibold text-gray-800 flex items-center">
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
              d="M7 16a4 4 0 01-.88-7.903A5 5 0 1115.9 6L16 6a5 5 0 011 9.9M15 13l-3-3m0 0l-3 3m3-3v12"
            />
          </svg>
          上傳文件
        </h2>
        <a
          href="https://github.com/galaxy990518/vue3-excel-master/tree/master/exampleFiles"
          target="_blank"
          class="text-sm font-semibold text-indigo-600 hover:text-indigo-800 hover:underline focus:outline-none focus:ring-2 focus:ring-indigo-500 focus:ring-offset-2 flex items-center"
        >
          <svg
            xmlns="http://www.w3.org/2000/svg"
            class="h-4 w-4 mr-1"
            fill="none"
            viewBox="0 0 24 24"
            stroke="currentColor"
          >
            <path
              stroke-linecap="round"
              stroke-linejoin="round"
              stroke-width="2"
              d="M13 16h-1v-4h-1m1-4h.01M21 12a9 9 0 11-18 0 9 9 0 0118 0z"
            />
          </svg>
          查看 data 的範例
        </a>
      </div>
      <div class="mt-4">
        <label
          for="file-upload"
          class="flex items-center justify-center w-full px-4 py-2 border border-gray-300 rounded-md shadow-sm text-sm font-medium text-gray-700 bg-white hover:bg-gray-50 focus:outline-none focus:ring-2 focus:ring-offset-2 focus:ring-indigo-500 cursor-pointer"
        >
          <svg
            xmlns="http://www.w3.org/2000/svg"
            class="h-5 w-5 mr-2 text-indigo-600"
            fill="none"
            viewBox="0 0 24 24"
            stroke="currentColor"
          >
            <path
              stroke-linecap="round"
              stroke-linejoin="round"
              stroke-width="2"
              d="M15.172 7l-6.586 6.586a2 2 0 102.828 2.828l6.414-6.586a4 4 0 00-5.656-5.656l-6.415 6.585a6 6 0 108.486 8.486L20.5 13"
            />
          </svg>
          選擇文件
        </label>
        <input
          id="file-upload"
          type="file"
          @change="handleFileUpload"
          accept=".csv,.xlsx"
          class="sr-only"
        />
      </div>

      <p v-if="fileName" class="mt-2 text-sm text-gray-600 flex items-center">
        <svg
          xmlns="http://www.w3.org/2000/svg"
          class="h-4 w-4 mr-1 text-green-500"
          fill="none"
          viewBox="0 0 24 24"
          stroke="currentColor"
        >
          <path
            stroke-linecap="round"
            stroke-linejoin="round"
            stroke-width="2"
            d="M9 12l2 2 4-4m6 2a9 9 0 11-18 0 9 9 0 0118 0z"
          />
        </svg>
        已選擇文件: {{ fileName }}
      </p>
      <div class="flex items-center mt-4 space-x-4">
        <button
          @click="processFile"
          :disabled="!fileName"
          class="flex items-center px-4 py-2 bg-indigo-600 text-white rounded-md hover:bg-indigo-700 focus:outline-none focus:ring-2 focus:ring-indigo-500 focus:ring-offset-2 disabled:opacity-50 disabled:cursor-not-allowed"
        >
          <svg
            xmlns="http://www.w3.org/2000/svg"
            class="h-5 w-5 mr-2"
            fill="none"
            viewBox="0 0 24 24"
            stroke="currentColor"
          >
            <path
              stroke-linecap="round"
              stroke-linejoin="round"
              stroke-width="2"
              d="M5 3v4M3 5h4M6 17v4m-2-2h4m5-16l2.286 6.857L21 12l-5.714 2.143L13 21l-2.286-6.857L5 12l5.714-2.143L13 3z"
            />
          </svg>
          處理文件
        </button>
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref, reactive } from 'vue';
import * as XLSX from 'xlsx';

const fileName = ref('');
const fileData = ref(null);
const yearlyTotals = reactive({});
const growthRates = reactive({});

const emit = defineEmits(['dataProcessed']);

const handleFileUpload = (event) => {
  const file = event.target.files[0];
  fileName.value = file.name;

  const reader = new FileReader();
  reader.onload = (e) => {
    const data = e.target.result;
    if (file.name.endsWith('.csv')) {
      fileData.value = parseCSV(data);
    } else if (file.name.endsWith('.xlsx')) {
      fileData.value = parseXLSX(data);
    }
  };
  reader.readAsBinaryString(file);
};

const parseCSV = (data) => {
  const lines = data.split('\n');
  const headers = lines[0].split(',').map((header) => header.trim());
  return lines
    .slice(1)
    .map((line) => {
      const values = line.split(',');
      return headers.reduce((obj, header, index) => {
        obj[header] = cleanValue(values[index]);
        return obj;
      }, {});
    })
    .filter((row) => Object.values(row).some((value) => value !== ''));
};

const parseXLSX = (data) => {
  const workbook = XLSX.read(data, { type: 'binary', codepage: 950 });
  const sheetName = workbook.SheetNames[0];
  const worksheet = workbook.Sheets[sheetName];
  return XLSX.utils
    .sheet_to_json(worksheet, {
      raw: false,
      dateNF: 'm/d/yy',
      defval: ''
    })
    .map((row) => {
      const cleanedRow = {};
      for (const [key, value] of Object.entries(row)) {
        cleanedRow[key.trim()] = cleanValue(value);
      }
      return cleanedRow;
    });
};

const cleanValue = (value) => {
  if (typeof value === 'string') {
    value = value.trim();
    const num = Number(value.replace(/,/g, ''));
    return isNaN(num) ? value : num;
  }
  return value;
};

const processFile = () => {
  if (fileData.value) {
    processData(fileData.value);
    emit('dataProcessed', { yearlyTotals, growthRates });
  }
};

const processData = (data) => {
  const years = new Set();
  Object.keys(yearlyTotals).forEach((key) => delete yearlyTotals[key]);
  Object.keys(growthRates).forEach((key) => delete growthRates[key]);

  data.forEach((row) => {
    const date = new Date(row['Cut off Date']);
    const year = date.getFullYear();
    years.add(year);

    const qty = typeof row['QTY'] === 'number' ? row['QTY'] : 0;
    const price = typeof row['Price/USD'] === 'number' ? row['Price/USD'] : 0;

    if (!isNaN(qty) && !isNaN(price)) {
      const total = qty * price;
      yearlyTotals[year] = (yearlyTotals[year] || 0) + total;
    }
  });

  const sortedYears = Array.from(years).sort();
  sortedYears.forEach((year, index) => {
    if (index === 0) {
      growthRates[year] = 'N/A';
    } else {
      const previousYear = sortedYears[index - 1];
      const previousTotal = yearlyTotals[previousYear] || 0;
      const currentTotal = yearlyTotals[year] || 0;

      if (previousTotal === 0) {
        growthRates[year] = 'N/A';
      } else {
        const growthRate = ((currentTotal - previousTotal) / previousTotal) * 100;
        growthRates[year] = parseFloat(growthRate.toFixed(2));
      }
    }
  });
};
</script>
