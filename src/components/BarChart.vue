<template>
  
  <div class="flex justify-center items-center min-h-screen bg-gray-100">
  <div class="w-4/5 max-w-xl p-4 bg-white rounded shadow mx-auto">
    <h1 class="text-center text-xl font-bold text-blue-500">Confirma o número de casos de covid em cada país</h1>
    <canvas ref="barChartCanvas" class="w-full h-100"></canvas>
    <div class="flex justify-between mt-4">
      <button @click="changePage(-1)" class="px-4 py-2 bg-blue-500 text-white rounded hover:bg-blue-600">Anterior</button>
      <button @click="changePage(1)" class="px-4 py-2 bg-blue-500 text-white rounded hover:bg-blue-600">Próximo</button>
    </div>
  </div>
</div>

</template>

<script>
import Chart from 'chart.js/auto';
import axios from 'axios';

export default {
  data() {
    return {
      chart: null,
      countriesData: [],
      currentPage: 1,
      itemsPerPage: 10, // Define quantos itens por página você quer
    };
  },
  mounted() {
    this.fetchDataAndRenderChart(this.currentPage);
  },
  methods: {
    async fetchDataAndRenderChart(page) {
      try {
        const response = await axios.get('https://covid19-brazil-api.vercel.app/api/report/v1/countries');
        this.countriesData = response.data.data;

        // Paginação dos dados
        const start = (page - 1) * this.itemsPerPage;
        const end = page * this.itemsPerPage;
        const paginatedData = this.countriesData.slice(start, end);

        // Extracting relevant data for the chart
        const labels = paginatedData.map(country => country.country);
        const confirmedCases = paginatedData.map(country => country.confirmed);

        this.renderBarChart(labels, confirmedCases);
      } catch (error) {
        console.error('Error fetching data:', error);
      }
    },
    renderBarChart(labels, confirmedCases) {
      const data = {
        labels: labels,
        datasets: [
          {
            label: 'Número de casos confirmados em cada país',
            backgroundColor: 'rgba(75, 192, 192, 0.2)',
            borderColor: 'rgba(75, 192, 192, 1)',
            borderWidth: 1,
            data: confirmedCases,
          },
        ],
      };

      const options = {
        scales: {
          y: {
            beginAtZero: true,
          },
        },
      };

      const ctx = this.$refs.barChartCanvas.getContext('2d');

      if (this.chart) {
        this.chart.destroy();
      }

      this.chart = new Chart(ctx, {
        type: 'bar',
        data: data,
        options: options,
      });
    },
    changePage(direction) {
      const nextPage = this.currentPage + direction;
      if (nextPage < 1 || nextPage > Math.ceil(this.countriesData.length / this.itemsPerPage)) {
        return;
      }
      this.currentPage = nextPage;
      this.fetchDataAndRenderChart(this.currentPage);
    },
  },
};
</script>

<style scoped>
/* Adicione estilos específicos do componente aqui, se necessário */
</style>
