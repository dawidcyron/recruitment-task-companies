<template>
    <div class="container">
        <input v-model="filter" placeholder="Enter something to filter results" class="filter-input" v-if="!isBusy"
               @input="filterItems">
        <div class="sort-wrapper" v-if="!isBusy">
            <div class="label">
                Select column used for sorting
            </div>
            <select v-model="sortColumn" class="sort-input filter-input" @change="sortItems(sortColumn, true)">
                <option value="id">Id</option>
                <option value="name">Name</option>
                <option value="city">City</option>
                <option value="totalIncome">Total income</option>
                <option value="averageIncome">Average income</option>
                <option value="lastMonthIncome">Last month income</option>
            </select>
            <div class="label">Select sorting order</div>
            <select v-model="sortDescending" class="sort-input filter-input" @change="sortItems(sortColumn, true, true)">
                <option value="true">Descending</option>
                <option value="false">Ascending</option>
            </select>

        </div>
        <table class="data-table">
            <tr>
                <th @click="sortItems('id')">
                    <font-awesome-icon icon="sort-up" v-if="sortColumn === 'id' && !sortDescending"></font-awesome-icon>
                    <font-awesome-icon icon="sort-down"
                                       v-if="sortColumn === 'id' && sortDescending"></font-awesome-icon>
                    <div>Id</div>
                </th>
                <th @click="sortItems('name')">
                    <font-awesome-icon icon="sort-up"
                                       v-if="sortColumn === 'name' && !sortDescending"></font-awesome-icon>
                    <font-awesome-icon icon="sort-down"
                                       v-if="sortColumn === 'name' && sortDescending"></font-awesome-icon>
                    <div>Name</div>
                </th>
                <th @click="sortItems('city')">
                    <font-awesome-icon icon="sort-up"
                                       v-if="sortColumn === 'city' && !sortDescending"></font-awesome-icon>
                    <font-awesome-icon icon="sort-down"
                                       v-if="sortColumn === 'city' && sortDescending"></font-awesome-icon>
                    <div>City</div>
                </th>
                <th @click="sortItems('totalIncome')">
                    <font-awesome-icon icon="sort-up"
                                       v-if="sortColumn === 'totalIncome' && !sortDescending"></font-awesome-icon>
                    <font-awesome-icon icon="sort-down"
                                       v-if="sortColumn === 'totalIncome' && sortDescending"></font-awesome-icon>
                    <div>Total income</div>
                </th>
                <th @click="sortItems('averageIncome')">
                    <font-awesome-icon icon="sort-up"
                                       v-if="sortColumn === 'averageIncome' && !sortDescending"></font-awesome-icon>
                    <font-awesome-icon icon="sort-down"
                                       v-if="sortColumn === 'averageIncome' && sortDescending"></font-awesome-icon>
                    <div>Average income</div>
                </th>
                <th @click="sortItems('lastMonthIncome')">
                    <font-awesome-icon icon="sort-up"
                                       v-if="sortColumn === 'lastMonthIncome' && !sortDescending"></font-awesome-icon>
                    <font-awesome-icon icon="sort-down"
                                       v-if="sortColumn === 'lastMonthIncome' && sortDescending"></font-awesome-icon>
                    <div>Last month income</div>
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
        <div class="loader" v-if="isBusy">
            Loading...
        </div>
        <div v-if="!isBusy" class="pagination">
            <button :disabled="currentPageNumber === 1" @click="currentPageNumber--">Previous</button>
            <div class="page-counter">
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
        perPage: 10,
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
      sortItems (column, passedFromInput, sortingFromInput) {

        if (passedFromInput) {
          if (sortingFromInput === undefined) {
            this.sortDescending = false
          } else {
            // Because select field is passed as string, it is converted to bool here
            this.sortDescending = (this.sortDescending === 'true')
          }
        } else if (column === this.sortColumn) {
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
<style scoped lang="scss">
    .container {
        display: flex;
        flex-direction: column;
        width: 70%;
        margin: auto;
        padding: 2rem 0;

        .sort-input {
            display: none;
        }

        .sort-wrapper {
            display: none;
        }

        @media screen and (max-width: 1023px) {
            width: 100%;
            .filter-input {
                margin-right: 1rem;
            }
        }

        @media screen and (max-width: 640px) {
            .data-table, thead, tbody, th, td, tr {
                display: block;
            }

            .sort-wrapper {
                display: block;

                .label {
                    font-weight: bolder;
                    color: #ffffff;
                }
            }

            .sort-input {
                display: block;
                width: 90%;
                margin: auto;
                background-color: #ffffff;
            }

            .data-table {
                border-radius: 0;
                text-align: left;

                tr:first-of-type {
                    position: absolute;
                    top: -9999px;
                    left: -9999px;
                }

                tr {
                    border: 1px solid #ccc;
                }

                tr:not(:last-of-type) {
                    border-bottom: 0;
                }

                td {
                    /* Behave  like a "row" */
                    border: none;
                    border-bottom: 1px solid #eee;
                    position: relative;
                    padding-left: 50%;
                }

                td:before {
                    position: absolute;
                    left: 6px;
                    width: 45%;
                    padding-right: 10px;
                    white-space: nowrap;
                    font-weight: bolder;
                }

                /*
                Label the data
                */
                td:nth-of-type(1):before {
                    content: "Id";
                }

                td:nth-of-type(2):before {
                    content: "Name";
                }

                td:nth-of-type(3):before {
                    content: "City";
                }

                td:nth-of-type(4):before {
                    content: "Total income";
                }

                td:nth-of-type(5):before {
                    content: "Average income";
                }

                td:nth-of-type(6):before {
                    content: "Last month income";
                }
            }
        }

        .filter-input {
            margin-bottom: 1rem;
            padding: 0.5rem;
            border-radius: 0.7rem;
            outline: none;
            flex-basis: 30%;
            align-self: flex-end;
        }
    }

    .data-table {
        flex-basis: 100%;
        border-spacing: 0;
        border-radius: 1rem;
        border-collapse: collapse;
        overflow: hidden;

        tr {
            background-color: #ffffff;

            td {
                padding: 0.7rem 1rem;
            }

            th {
                padding: 0.7rem 1rem;
            }
        }

        tr:not(:last-of-type) {
            border-bottom: 2px #f1f1f1 solid;
        }

        tr:first-of-type {
            border-bottom: 0;
            background-color: #edda70;
            font-weight: bolder;
        }
    }

    .pagination {
        display: flex;
        flex-direction: row;
        align-items: center;
        justify-content: center;
        justify-self: flex-end;
        margin-top: 2rem;

        * {
            margin: 0 1rem;
        }

        .page-counter {
            font-weight: bolder;
            font-size: 1.2rem;
            color: #ffffff;
        }

        button {
            background-color: white;
            border: none;
            border-radius: 0.5rem;
            padding: 1rem 2rem;
            font-weight: bolder;
        }
    }

    .loader {
        justify-self: center;
        font-size: 10px;
        margin: 50px auto;
        text-indent: -9999em;
        width: 11em;
        height: 11em;
        border-radius: 50%;
        background: #edda70;
        background: -moz-linear-gradient(left, #edda70 10%, rgba(109, 126, 210, 0) 42%);
        background: -webkit-linear-gradient(left, #edda70 10%, rgba(109, 126, 210, 0) 42%);
        background: -o-linear-gradient(left, #edda70 10%, rgba(109, 126, 210, 0) 42%);
        background: -ms-linear-gradient(left, #edda70 10%, rgba(109, 126, 210, 0) 42%);
        background: linear-gradient(to right, #edda70 10%, rgba(109, 126, 210, 0) 42%);
        position: relative;
        -webkit-animation: load3 1.4s infinite linear;
        animation: load3 1.4s infinite linear;
        -webkit-transform: translateZ(0);
        -ms-transform: translateZ(0);
        transform: translateZ(0);
    }

    .loader:before {
        width: 50%;
        height: 50%;
        background: #edda70;
        border-radius: 100% 0 0 0;
        position: absolute;
        top: 0;
        left: 0;
        content: '';
    }

    .loader:after {
        background: #9cbd5e;
        width: 75%;
        height: 75%;
        border-radius: 50%;
        content: '';
        margin: auto;
        position: absolute;
        top: 0;
        left: 0;
        bottom: 0;
        right: 0;
    }

    @-webkit-keyframes load3 {
        0% {
            -webkit-transform: rotate(0deg);
            transform: rotate(0deg);
        }
        100% {
            -webkit-transform: rotate(360deg);
            transform: rotate(360deg);
        }
    }

    @keyframes load3 {
        0% {
            -webkit-transform: rotate(0deg);
            transform: rotate(0deg);
        }
        100% {
            -webkit-transform: rotate(360deg);
            transform: rotate(360deg);
        }
    }
</style>
