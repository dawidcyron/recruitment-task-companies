<template>
    <div id="app">
        <data-table></data-table>
    </div>
</template>

<script>
  import DataTable from './components/DataTable.vue'

  export default {
    name: 'App',
    components: {
      DataTable
    }
  }
</script>

<style>
    html {
        background-color: #9cbd5e;
        min-width: 100%;
        min-height: 100%;
    }

    body {
        min-width: 100%;
        min-height: 100%;
        display: flex;
        flex-direction: column;
        align-content: center;
        margin: 0;
    }

    #app {
        font-family: Avenir, Helvetica, Arial, sans-serif;
        -webkit-font-smoothing: antialiased;
        -moz-osx-font-smoothing: grayscale;
        text-align: center;
        color: #2c3e50;
    }
</style>
