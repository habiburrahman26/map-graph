<script>
import LineChart from "./components/LineChart.vue";
import Doughnut from "./components/Doughnut.vue";
import Year from "./components/Year.vue";
import PieChart from "./components/PieChart.vue";

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
  components: {
    LineChart,
    Doughnut,
    Year,
    PieChart
},
  data() {
    return {
      selectedYear: 2022,
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
      loaded: false,
      years: [],
      reverseYear: [],
      selectedComponent: "monthly",
    };
  },
  methods: {
    async getBillData() {
      const res = await fetch("./data.json");
      const result = await res.json();

      for (const data of result) {
        const date = new Date(data.bill_created_at);
        const year = date.getFullYear();
        const dueCost = data.total_cost - data.paid_amount;

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

    selectedYearHandler(year) {
      this.selectedYear = year;
    },

    async getYears() {
      const res = await fetch("./data.json");
      const result = await res.json();

      for (const data of result) {
        const date = new Date(data.bill_created_at);
        const year = date.getFullYear();

        if (!this.years.includes(year)) {
          this.years.push(year);
        }
      }

      //sort years
      this.years
        .sort(function (a, b) {
          return a - b;
        })
        .reverse();
    },

    getTotalSumAmount(amountArray) {
      let sumAmount = 0;
      for (const val of amountArray) {
        sumAmount += val.y;
      }
      return sumAmount;
    },
  },
  watch: {
    selectedYear(newVal) {
      this.paid_amount = [...defaultData];
      this.total_cost = [...defaultData];
      this.due_amount = [...defaultData];
      this.getBillData();
    },
  },
  mounted() {
    this.getBillData();
    this.getYears();
  },
  computed: {
    getTotalPaidAmountPercentage() {
      let totalPaidAmount = this.getTotalSumAmount(this.paid_amount);
      let totalCostAmount = this.getTotalSumAmount(this.total_cost);

      let paidAmountPercentage = (totalPaidAmount / totalCostAmount) * 100;
      if (paidAmountPercentage) return Math.round(paidAmountPercentage);
    },

    getTotalDueAmountPercentage() {
      let totalDueAmount = this.getTotalSumAmount(this.due_amount);
      let totalCostAmount = this.getTotalSumAmount(this.total_cost);

      let dueAmountPercentage = (totalDueAmount / totalCostAmount) * 100;
      if (dueAmountPercentage) return Math.round(dueAmountPercentage);
    },
  },
};
</script>

<template>
  <!-- <PatientChart/> -->
  <div class="containet">
    <div class="select-container">
      <span>Show by:</span>
      <select v-model="selectedComponent">
        <option disabled paid_amount="">Please select</option>
        <option value="monthly">Monthly</option>
        <option value="yearly">Yearly</option>
      </select>
    </div>

    <div v-if="selectedComponent === 'monthly'">
    <div class="container">
      <LineChart
        :paidAmount="paid_amount"
        :totalCost="total_cost"
        :dueAmount="due_amount"
        :selectedYear="selectedYear"
        :labels="labels"
        :years="years"
        @selectYear="selectedYearHandler"
      />
      <Doughnut
        v-if="
          getTotalDueAmountPercentage > 0 && getTotalDueAmountPercentage > 0
        "
        :paidAmountPercentage="getTotalPaidAmountPercentage"
        :dueAmountPercentage="getTotalDueAmountPercentage"
      />
      <PieChart
        v-if="
          getTotalDueAmountPercentage > 0 && getTotalDueAmountPercentage > 0
        "
        :paidAmountPercentage="getTotalPaidAmountPercentage"
        :dueAmountPercentage="getTotalDueAmountPercentage"
      />
    </div>
  </div>

    <Year v-if="selectedComponent === 'yearly'" :years="years"> </Year>

    <div>
      <!-- <LineChart /> -->
    </div>
  </div>
</template>

<style scoped>
.container {
  display: flex;
  gap: 40px;
  justify-content: center;
}
.select-container {
  margin-bottom: 10px;
  text-align: right;
}
</style>
