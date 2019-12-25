<template>
  <div id="app">
    <board-table />
  </div>
</template>

<script>
import BoardTable from './components/BoardTable.vue'

export default {
  name: 'app',
  components: {
    BoardTable
  }
}
</script>

<style>
#app {
  font-family: 'Avenir', Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  margin-top: 60px;
}

body {
  background-color: #282c34;
}
</style>
