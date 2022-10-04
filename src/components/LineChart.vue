<template>
  <div>
    <Line
      :chart-data="chartData"
      :chart-id="chartId"
      :dataset-id-key="datasetIdKey"
      :plugins="plugins"
      :css-classes="cssClasses"
      :styles="styles"
      :width="width"
      :height="height"
      :chart-options="chartOptions"
    />
    <div class="select-container">
      <span>Selected Year:</span>
      <select v-model="selected" @change="changeYear">
        <option disabled paid_amount="">Please select</option>
        <option v-for="year in years" :key="year" :value="year">
          {{ year }}
        </option>
      </select>
    </div>
  </div>
</template>

<script>
import { Line } from "vue-chartjs";

import {
  Chart as ChartJS,
  Title,
  Tooltip,
  Legend,
  LineElement,
  LinearScale,
  PointElement,
  CategoryScale,
} from "chart.js";

ChartJS.register(
  Title,
  Tooltip,
  Legend,
  CategoryScale,
  LinearScale,
  PointElement,
  LineElement
);

export default {
  name: "LineChart",
  components: {
    Line,
  },
  props: {
    chartId: {
      type: String,
      default: "line-chart",
    },
    datasetIdKey: {
      type: String,
      default: "label",
    },
    width: {
      type: Number,
      default: 600,
    },
    height: {
      type: Number,
      default: 400,
    },
    cssClasses: {
      default: "",
      type: String,
    },
    styles: {
      type: Object,
      default: () => {},
    },
    plugins: {
      type: Object,
      default: () => {},
    },
    years: {
      type: Array,
      default: () => [],
    },
    paidAmount: {
      type: Array,
      default: () => [],
    },
    totalCost: {
      type: Array,
      default: () => [],
    },
    dueAmount: {
      type: Array,
      default: () => [],
    },
    selectedYear: {
      type: Number,
    },
    labels: {
      type: Array,
      default: () => [],
    },
  },
  emits: ["selectYear"],

  data() {
    return {
      selected: this.selectedYear,
      chartData: {
        labels: this.labels,
        datasets: [
          {
            label: "Paid Amount",
            data: this.paidAmount,
            backgroundColor: "rgba(75, 192, 192, 0.5)",
            borderColor: "rgba(75, 192, 192, 1)",
            borderWidth: 3,
            tension: 0.25,
          },
          {
            label: "Due Amount",
            data: this.dueAmount,
            backgroundColor: "rgba(255, 99, 132, 0.5)",
            borderColor: "rgba(255, 99, 132, 1)",
            borderWidth: 3,
            tension: 0.25,
          },
          {
            label: "Total Cost",
            data: this.totalCost,
            backgroundColor: "rgba(29,104,137,.5)",
            borderColor: "#1d6889",
            borderWidth: 3,
            tension: 0.25,
          },
        ],
      },
      chartOptions: {
        responsive: true,
        plugins: {
          title: {
            display: true,
            text: "Revenue Chart By Monthly",
            font: {
              size: 20,
            },
          },
          legend: {
            labels: {
              // This more specific font property overrides the global property
              font: {
                size: 13,
              },
            },
          },
        },
        scales: {
          y: {
            begainAtZero: true,
            grid: {
              color: "white",
            },
          },
          x: {
            grid: {
              color: "white",
            },
          },
        },
      },
    };
  },
  methods: {
    changeYear() {
      this.$emit("selectYear", this.selected);
    },
  },
  watch: {
    paidAmount(newVal) {
      this.chartData.datasets[0].data = newVal;
    },
    dueAmount(newVal) {
      this.chartData.datasets[1].data = newVal;
    },
    totalCost(newVal) {
      this.chartData.datasets[2].data = newVal;
    },
  },
  mounted() {},
};
</script>

<style>
.select-container {
  padding-top: 20px;
}
select {
  padding: 2px 8px;
  background-image: none;
  cursor: pointer;
  border-radius: 5px;
  font-size: 16px;
}

span {
  font-size: 15px;
  font-weight: bold;
  margin-right: 4px;
}

</style>
