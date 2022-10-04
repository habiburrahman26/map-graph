
<script>
  import { Pie } from "vue-chartjs";
  
  import {
    Chart as ChartJS,
    Title,
    Tooltip,
    Legend,
    ArcElement,
    CategoryScale,
  } from "chart.js";
  
  ChartJS.register(Title, Tooltip, Legend, CategoryScale, ArcElement);
  
  export default {
    name: "LineChart",
    components: {
      Pie,
    },
    props: {
      chartId: {
        type: String,
        default: "pie-chart",
      },
      datasetIdKey: {
        type: String,
        default: "label",
      },
      width: {
        type: Number,
        default: 400,
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
      paidAmountPercentage: {
        type: Number,
        default:0
      },
      dueAmountPercentage: {
        type: Number,
        default: 0,
      },
    },
  
    data() {
      return {
        selected: this.selectedYear,
        chartData: {
          labels: ["Paid Amount", "Due Amount"],
          datasets: [
            {
              data: [this.paidAmountPercentage,this.dueAmountPercentage],
              backgroundColor: ["#74c0fc", "#ffa8a8"],
              hoverOffset: 4,
            },
          ],
        },
        chartOptions: {
          responsive: true,
          plugins: {
            title: {
              display: true,
              text: "Paid amount vs Due amount",
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
  };
  </script>


<template>
  <div class="line-chart-container">
    <Pie
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
