<template>
    <div class="container">
        <input v-model="filter" placeholder="Enter something to filter results" class="filter-input"
               @input="filterItems">
        <table class="data-table">
            <tr>
                <th @click="sortItems('id')">
                    Id
                </th>
                <th @click="sortItems('name')">
                    Name
                </th>
                <th @click="sortItems('city')">
                    City
                </th>
                <th @click="sortItems('totalIncome')">
                    Total income
                </th>
                <th @click="sortItems('averageIncome')">
                    Average income
                </th>
                <th @click="sortItems('lastMonthIncome')">
                    Last month income
                </th>
            </tr>
            <tr v-for="company in paginatedCompanies" :key="company.id">
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
        <div v-if="!isBusy" class="pagination">
            <button :disabled="currentPageNumber === 1" @click="currentPageNumber--">Previous</button>
            <div>
                {{currentPageNumber + '/' + pageCount}}
            </div>
            <button :disabled="currentPageNumber === pageCount" @click="currentPageNumber++">Next</button>
        </div>
    </div>
</template>

<script>
  import axios from 'axios'

  export default {
    name: 'DataTable',
    data () {
      return {
        companies: [],
        displayedCompanies: [],
        filter: '',
        sortColumn: '',
        sortDescending: false,
        currentPageNumber: 1,
        perPage: 20,
        isBusy: true
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
      this.displayedCompanies = this.companies
      this.sortItems('id')
      this.isBusy = false
    },
    computed: {
      paginatedCompanies () {
        let start = (this.currentPageNumber - 1) * this.perPage
        let end = start + this.perPage
        return this.displayedCompanies.slice(start, end)
      },
      pageCount () {
        return this.displayedCompanies.length ? Math.ceil(this.displayedCompanies.length / this.perPage) : 1
      }
    },
    methods: {
      async calculateIncomes (company) {
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
        company.averageIncome = totalIncome / response.data.incomes.length
        company.lastMonthIncome = lastMonthIncome
      },
      filterItems () {
        this.displayedCompanies = this.companies.filter(company => this.stringifyCompany(company).toLocaleLowerCase().includes(this.filter.toLowerCase()))
        this.currentPageNumber = 1
      },
      sortItems (column) {
        if (column === this.sortColumn) {
          this.sortDescending = !this.sortDescending
        } else {
          this.sortDescending = false
        }
        this.sortColumn = column
        this.displayedCompanies = this.displayedCompanies.sort((a, b) => {
          if (this.sortDescending) {
            return a[column] < b[column] ? 1 : -1
          } else {
            return a[column] > b[column] ? 1 : -1
          }
        })
      },
      getLastDayOfPreviousMonth () {
        let date = new Date()
        date.setDate(0)
        return date
      },
      getFirstDayOfPreviousMonth () {
        let date = new Date()
        date.setDate(0)
        date.setDate(1)
        return date
      },
      stringifyCompany (company) {
        return `${company.id} ${company.name} ${company.city} ${company.totalIncome} ${company.averageIncome} ${company.lastMonthIncome}`
      }
    }
  }
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
    .container {
        display: flex;
        flex-direction: column;
        margin: auto;
        width: 70%;
    }

    .data-table {
        flex-basis: 100%;
    }

    .filter-input {
        flex-basis: 30%;
        align-self: flex-end;
    }

    .pagination {
        display: flex;
        flex-direction: row;
        align-items: center;
        justify-content: center;
    }
</style>
