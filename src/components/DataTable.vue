<template>
    <div class="container">
        <table>
            <tr>
                <th>
                    Id
                </th>
                <th>
                    Name
                </th>
                <th>
                    City
                </th>
                <th>
                    Total income
                </th>
                <th>
                    Average income
                </th>
                <th>
                    Last month income
                </th>
            </tr>
            <tr v-for="company in companies" :key="company.id">
                <td>
                    {{company.id}}
                </td>
                <td>
                    {{company.name}}
                </td>
                <td>
                    {{company.city}}
                </td>
                <td>
                    {{company.totalIncome.toFixed(2)}}
                </td>
                <td>
                    {{company.averageIncome.toFixed(2)}}
                </td>
                <td>
                    {{company.lastMonthIncome.toFixed(2)}}
                </td>
            </tr>
        </table>
    </div>
</template>

<script>
  import axios from 'axios'

  export default {
    name: 'DataTable',
    data () {
      return {
        companies: [],
      }
    },
    async created () {
      const fetchedCompanies = await axios.get('https://recruitment.hal.skygate.io/companies')
      const companies = []
      for (let company of fetchedCompanies.data) {
        await this.calculateIncomes(company)
        companies.push(company)
      }
      this.companies = companies
    },
    methods: {
      async calculateIncomes(company) {
        const response = await axios.get('https://recruitment.hal.skygate.io/incomes/' + company.id)
        let totalIncome = 0
        let lastMonthIncome = 0
        let lastDayOfPreviousMonth = this.getLastDayOfPreviousMonth()
        let firstDayOfPreviousMonth = this.getFirstDayOfPreviousMonth()

        for (let income of response.data.incomes) {
          let incomeDate = new Date(income.date)
          if (incomeDate > firstDayOfPreviousMonth && incomeDate < lastDayOfPreviousMonth) {
            lastMonthIncome += parseFloat(income.value)
          }
          totalIncome += parseFloat(income.value)
        }
        company.totalIncome = totalIncome
        company.averageIncome = totalIncome/response.data.incomes.length
        company.lastMonthIncome = lastMonthIncome
      },
      getLastDayOfPreviousMonth() {
        let date = new Date()
        date.setDate(0)
        return date
      },
      getFirstDayOfPreviousMonth() {
        let date = new Date()
        date.setDate(0)
        date.setDate(1)
        return date
      }
    }
  }
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
    .container {
        display: flex;
        margin: auto;
        width: 70%;
    }
</style>
