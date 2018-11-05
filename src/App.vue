<template>
  <div id="app">
    <Gauge id="facebookFilter" gaugeRangeMax="3000" />
  </div>
</template>

<script>
import Gauge from './components/Gauge.vue'

export default {
  name: 'app',
  components: {
    Gauge
  }
}
</script>

<style>
#app {
  font-family: 'Avenir', Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;
}
</style>
