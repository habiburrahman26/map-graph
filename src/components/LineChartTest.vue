<template>
  <div>
    <div class="canvas">
      <canvas id="myChart" width="600" height="400"></canvas>
    </div>
    <div class="select-container">
      <span> Selected Year:</span>
      <select v-model="selectedYear">
        <option disabled paid_amount="">Please select</option>
        <option paid_amount="2022" selected>2022</option>
        <option paid_amount="2021">2021</option>
        <option paid_amount="2020">2020</option>
      </select>
    </div>
  </div>
</template>

<script>
import Chart from "chart.js/auto";

const defaultData = [
  { x: "Jan", y: 0 },
  { x: "Feb", y: 0 },
  { x: "Mar", y: 0 },
  { x: "Apr", y: 0 },
  { x: "May", y: 0 },
  { x: "Jun", y: 0 },
  { x: "Jul", y: 0 },
  { x: "Aug", y: 0 },
  { x: "Sep", y: 0 },
  { x: "Oct", y: 0 },
  { x: "Nov", y: 0 },
  { x: "Dec", y: 0 },
];

export default {
  data() {
    return {
      selectedYear: "2022",
      bills: [],
      patients: {
        2021: [
          { x: "2021-1", y: 0 },
          { x: "2021-2", y: 0 },
          { x: "2021-3", y: 1 },
          { x: "2021-4", y: 2 },
          { x: "2021-5", y: 2 },
          { x: "2021-6", y: 3 },
          { x: "2021-7", y: 3 },
          { x: "2021-8", y: 2 },
          { x: "2021-9", y: 2 },
          { x: "2021-10", y: 3 },
          { x: "2021-11", y: 1 },
          { x: "2021-12", y: 1 },
        ],
        2022: [
          { x: "Jan", y: 3 },
          { x: "Feb", y: 2 },
          { x: "Mar", y: 4 },
          { x: "Apr", y: 2 },
          { x: "May", y: 3 },
          { x: "Jun", y: 6 },
          { x: "Jul", y: 7 },
          { x: "Aug", y: 4 },
          { x: "Sep", y: 8 },
          { x: "Oct", y: 2 },
          { x: "Nov", y: 1 },
          { x: "Dec", y: 9 },
        ],
      },
      paid_amount: [...defaultData],
      total_cost: [...defaultData],
      due_amount: [...defaultData],
      labels: [
        "Jan",
        "Feb",
        "Mar",
        "Apr",
        "May",
        "Jun",
        "Jul",
        "Aug",
        "Sep",
        "Oct",
        "Nov",
        "Dec",
      ],
      myChart: null,
    };
  },
  watch: {
    selectedYear(newVal) {
      this.paid_amount = [];
      this.getBillData();
    },
  },
  methods: {
    async getBillData() {
      const res = await fetch("./data.json");
      const result = await res.json();
      for (const data of result) {
        const dueCost = data.total_cost - data.paid_amount;
        const date = new Date(data.bill_created_at);
        const year = date.getFullYear();

        if (year === +this.selectedYear) {
          const id = this.labels[date.getMonth()];

          this.sumAmount(id, data.paid_amount, this.paid_amount);
          this.sumAmount(id, data.total_cost, this.total_cost);
          this.sumAmount(id, dueCost, this.due_amount);
        }
      }

      //sort data month wise
      this.paid_amount.sort(function (a, b) {
        return this?.labels.indexOf(a.x) - this?.labels.indexOf(b.x);
      });
      this.ChartJs(this.paid_amount,this.total_cost,this.due_amount);
    },

    sumAmount(id, value, arrayOfData) {
      const paidAmountObj = {
        x: id,
        y: value,
      };

      //check month is already exist
      const existMonth = arrayOfData.find((month) => month.x === id);

      if (existMonth) {
        const objIndex = arrayOfData.findIndex((month) => month.x === id);
        let sum = existMonth.y + value;
        arrayOfData[objIndex] = { ...existMonth, y: sum };
      } else {
        arrayOfData.push(paidAmountObj);
      }
    },

    ChartJs(paidAmount, totalCost, dueAmount) {
      const data = {
        labels: this.labels,
        datasets: [
          {
            label: "Paid Amount",
            data: paidAmount || this.paid_amount,
            backgroundColor: "rgba(54,73,93,.5)",
            borderColor: "#36495d",
            borderWidth: 3,
            tension: 0.2,
          },
          {
            label: "Due Amoun2t",
            data:dueAmount || this.due_amount,
            backgroundColor: "rgba(255, 99, 132, 0.2)",
            borderColor: "rgba(255, 99, 132, 1)",
            borderWidth: 3,
            tension: 0.2,
          },
          {
            label: "Total Cost",
            data: totalCost || this.total_cost,
            backgroundColor: "rgba(75, 192, 192, 0.2)",
            borderColor: "rgba(75, 192, 192, 1)",
            borderWidth: 3,
            tension: 0.2,
          },
        ],
      };
      const ctx = document.getElementById("myChart").getContext("2d");

      //if chart already exist then destroy it
      if (this.$data._chart) {
        this.$data._chart.destroy();
      } else {
        this.$data._chart = new Chart(ctx, {
          type: "line",
          data: data,
          options: {
            responsive: true,
          },
        });
      }
    },
  },
  mounted() {
    this.getBillData();
  },
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
}

/* .canvas{
  background-color: rgb(211, 126, 126);
} */
</style>
