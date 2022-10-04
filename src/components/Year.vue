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
  Filler,
} from "chart.js";

ChartJS.register(
  Title,
  Tooltip,
  Legend,
  CategoryScale,
  LinearScale,
  PointElement,
  LineElement,
  Filler
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
      default: 1000,
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
  },
  data() {
    return {
      chartData: {
        labels: [],
        datasets: [
          {
            label: "Paid Amount",
            backgroundColor: (ctx) => {
              const canvas = ctx.chart.ctx;
              const gradient = canvas.createLinearGradient(0, 0, 0, 550);

              gradient.addColorStop(0, "rgba(75, 192, 192, .6)");
              gradient.addColorStop(0.5, "rgba(75, 192, 192, .45)");
              gradient.addColorStop(1, "rgba(255, 255, 255, 0.1)");

              return gradient;
            },
            borderColor: "rgba(75, 192, 192, 1)",
            data: this.paidAmount || [],
            tension: 0.25,
            fill: true,
          },
          {
            label: "Due Amount",
            // backgroundColor: (ctx) => {

            //   const canvas = ctx.chart.ctx;
            //   const gradient = canvas.createLinearGradient(0, 0, 0, 550);

            //   gradient.addColorStop(0, "rgba(255, 69, 122, 0.6)");
            //   gradient.addColorStop(0.5, "rgba(255, 69, 122, 0.25)");
            //   gradient.addColorStop(1, "rgba(255, 0, 0, 0)");

            //   return gradient;
            // },
            borderColor: "rgba(255, 99, 132, .5)",
            borderColor: "rgba(255, 99, 132, 1)",
            data: this.dueAmount || [],
            tension: 0.25,
          },
          {
            label: "Total Cost",
            backgroundColor: "rgba(29,104,137,.5)",
            borderColor: "#1d6889",
            data: this.totalCost || [],
            tension: 0.25,
          },
        ],
      },
      chartOptions: {
        responsive: true,
        maintainAspectRatio: false,
        scales: {
          y: {
            beginAtZero: true,
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
        plugins: {
          title: {
            display: true,
            text: "Revenue Chart By Yearly",
            font: {
              size: 20,
            },
          },
          legend: {
            labels: {
              font: {
                size: 13,
              },
            },
          },
        },
      },
      paidAmount: [],
      totalCost: [],
      dueAmount: [],
    };
  },
  methods: {
    async getAllRevenue() {
      const response = await fetch("./data.json");
      const results = await response.json();

      for (const result of results) {
        const dueCost = result.total_cost - result.paid_amount;

        this.addAmount(result, result.paid_amount, this.paidAmount);
        this.addAmount(result, dueCost, this.dueAmount);
        this.addAmount(result, result.total_cost, this.totalCost);
      }

      this.sortData(this.paidAmount);
      this.sortData(this.totalCost);
      this.sortData(this.dueAmount);
    },

    //calculate total sum in yearly on paid amount
    addAmount(result, amountName, arrayOfAmount) {
      const year = new Date(result.bill_created_at).getFullYear();
      const existData = arrayOfAmount.find((data) => data.x === year);
      if (existData) {
        existData.y += amountName;
        const yearIndex = arrayOfAmount.findIndex((data) => data.x === year);
        arrayOfAmount[yearIndex] = existData;
      } else {
        arrayOfAmount.push({
          x: year,
          y: amountName,
        });
      }
    },

    sortData(arrayOfAmount) {
      arrayOfAmount.sort((a, b) => a.x - b.x);
    },
  },
  watch: {
    paidAmount: {
      handler(newVal) {
        this.chartData.datasets[0].data = newVal;
      },
      deep: true,
    },

    dueAmount: {
      handler(newVal) {
        this.chartData.datasets[1].data = newVal;
      },
      deep: true,
    },

    totalCost: {
      handler(newVal) {
        this.chartData.datasets[2].data = newVal;
      },
      deep: true,
    },
  },
  mounted() {
    //reverse the year array
    this.chartData.labels = this.years.map((year) => year).reverse();
    this.getAllRevenue();
  },
};
</script>

<style></style>
