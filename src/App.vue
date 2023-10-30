<template>
  <main>
  <div class="select is-success" style="margin-bottom: 5px;">
    <select v-model="selected">
      <option v-for="company in companies">{{ company }}</option>
    </select>
  </div>
  <div>
    <input style="width: 225px;" class="input" placeholder="add nasdaq company" v-model="newCompany">
    <button @click="addCompany" class="button">Добавить</button>
  </div>
  <div style="margin-top:40px;">
    <LineChart :chartData="lineData" :options="options" />
  </div>
</main>
</template>

<script setup>
import axios from 'axios'
import { ref, reactive, computed, watch } from 'vue'
import Chart from 'chart.js/auto';
import { LineChart } from "vue-chart-3"

const dataLabels = ref([])
const dataValues = ref([])
const options = reactive({
  responsive: true,
  plugins: {
    legend: {
      position: 'top',
    },
    title: {
      display: true,
      text: 'Stock Changes in last 24 hours',
    },
  },
});
const companies = ref(['AAPL', 'AMZN', 'NFLX', 'MSFT', 'TSLA'])
const selected = ref(companies.value[0])
const newCompany = ref('')
function addCompany() {
  companies.value.push(newCompany.value)
}
watch(selected, (value) => {
  getStockData(value)
}, { immediate: true })
const lineData = computed(() => ({
  labels: dataLabels.value,
  datasets: [
    {
      data: dataValues.value,
      label: selected.value,

      backgroundColor:
        "#77CEFF",
    },
  ],
}));
function getStockData(value) {
  axios.get(`https://api.twelvedata.com/time_series?symbol=${value}&interval=1h&timezone=Europe/Moscow&apikey=cfa13d605ca34db19518a9dca57c1a8b`).then((res) => {
    dataLabels.value = [res.data.values[6].datetime, res.data.values[5].datetime, res.data.values[4].datetime, res.data.values[3].datetime, res.data.values[2].datetime, res.data.values[1].datetime, res.data.values[0].datetime]
    dataValues.value = [res.data.values[6].close, res.data.values[5].close, res.data.values[4].close, res.data.values[3].close, res.data.values[2].close, res.data.values[1].close, res.data.values[0].close]
  })
}
</script>