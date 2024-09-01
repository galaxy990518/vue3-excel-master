<template>
  <div class="bg-white rounded-xl shadow-md overflow-hidden w-full mb-8">
    <div class="p-6">
      <h2 class="text-xl font-semibold text-gray-800 mb-4">上傳文件</h2>
      <input
        type="file"
        @change="handleFileUpload"
        accept=".csv,.xlsx"
        class="block w-full text-sm text-gray-500 file:mr-4 file:py-2 file:px-4 file:rounded-full file:border-0 file:text-sm file:font-semibold file:bg-indigo-50 file:text-indigo-700 hover:file:bg-indigo-100"
      />
      <p v-if="fileName" class="mt-2 text-sm text-gray-600">已選擇文件: {{ fileName }}</p>
      <button
        @click="processFile"
        :disabled="!fileName"
        class="mt-4 px-4 py-2 bg-indigo-600 text-white rounded-md hover:bg-indigo-700 focus:outline-none focus:ring-2 focus:ring-indigo-500 focus:ring-offset-2 disabled:opacity-50"
      >
        處理文件
      </button>
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
