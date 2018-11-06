<template>
  <div id="app">
    <main1 />
  </div>
</template>

<script>

import main1 from './components/Main.vue'

export default {
  name: 'app',
  components: {
    main1
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
