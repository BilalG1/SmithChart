<template>
  <smith-chart-page />
</template>

<script>
import SmithChartPage from './components/SmithChartPage.vue'

export default {
  name: 'App',
  components: {
    SmithChartPage
  }
}
</script>

<style>
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #000;
}
html, body{
  margin: 0;
}
</style>
