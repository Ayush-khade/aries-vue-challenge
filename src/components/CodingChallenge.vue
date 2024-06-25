<template>
  <div class="coding-challenge">
    <h1>Options Profit Calculator</h1>
    <canvas ref="lineChart"></canvas>
    <div v-if="chartData" class="results">
      <p>Max Profit: {{ calculateMaxProfit }}</p>
      <p>Max Loss: {{ calculateMaxLoss }}</p>
      <p>Break Even Points: {{ calculateBreakEvenPoints.join(', ') }}</p>
    </div>
  </div>
</template>

<script>
import Chart from 'chart.js/auto';

export default {
  name: 'CodingChallenge',
  props: {
    optionsData: Array
  },
  data() {
    return {
      chartData:null,
      chartOptions: {
        responsive: true,
        plugins: {
          legend: {
            position: 'top',
          },
          title: {
            display: true,
            text: 'Options Profit/Loss Chart'
          }
        }
      },
      // maxProfit: this.calculateMaxProfit(),
      // maxLoss: this.calculateMaxLoss(),
      // breakEvenPoints: this.calculateBreakEvenPoints(),
      lineChart: null
    };
  },
  mounted() {
    this.calculateChartData();
  },
  methods: {
    calculateChartData() {
      if (this.optionsData.length === 0) return;
      const labels = this.optionsData.map(option => option.strike_price);
      const profitLossData = this.optionsData.map(option => option.bid - option.ask);
      this.chartData = {
        labels: labels,
        datasets: [{
          label: 'Profit/Loss',
          data: profitLossData,
          borderColor: 'rgb(75, 192, 192)',
          fill: false
        }]
      };
      this.maxProfit = Math.max(...profitLossData);
      this.maxLoss = Math.min(...profitLossData);
      this.breakEvenPoints = this.optionsData
        .filter(option => option.bid === option.ask)
        .map(option => option.strike_price);
      this.renderLineChart();
    },
    renderLineChart() {
      this.$nextTick(() => {
        const ctx = this.$refs.lineChart.getContext('2d');
        if (this.lineChart) {
          this.lineChart.destroy(); // Destroy existing chart if it exists
        }
        this.lineChart = new Chart(ctx, {
          type: 'line',
          data: this.chartData,
          options: this.chartOptions
        });
      });
    }
  },
  computed: {
    calculateBreakEvenPoints() {
      if (!this.optionsData) return [];
      
      return this.optionsData.map(option => {
        if (option.type === 'Call') {
          return option.strike_price + option.bid;
        } else if (option.type === 'Put') {
          return option.strike_price - option.ask;
        }
        return null;
      }).filter(point => point !== null);
    },
    calculateMaxProfit() {
      return this.optionsData.reduce((total, option) => {
        if (option.type === 'Call') {
          return total + (option.ask - option.bid);
        } else if (option.type === 'Put') {
          return total + (option.bid - option.ask);
        }
        return total;
      }, 0);
    },
    calculateMaxLoss() {
      return this.optionsData.reduce((total, option) => {
        if (option.type === 'Call') {
          return total - option.bid;
        } else if (option.type === 'Put') {
          return total + option.ask;
        }
        return total;
      }, 0);
    }
  },
  watch: {
    optionsData: {
      handler() {
        this.calculateChartData();
      },
      deep: true
    }
  },
  beforeDestroy() {
    if (this.lineChart) {
      this.lineChart.destroy();
    }
  }
};
</script>

<style scoped>
h1 {
  font-size: 1.5rem;
  margin-bottom: 1rem;
}
.coding-challenge {
  max-width: 800px;
  margin: 0 auto;
  padding: 20px;
  background-color: #f0f0f0;
  border-radius: 8px;
  box-shadow: 0 2px 4px rgba(0, 0, 0, 0.1);
}

.results {
  margin-top: 20px;
  padding: 10px;
  background-color: #fff;
  border: 1px solid #ccc;
  border-radius: 4px;
}

.results p {
  margin: 5px 0;
}

.line-chart {
  margin-top: 20px;
}
</style>
