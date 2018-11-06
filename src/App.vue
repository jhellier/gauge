<template>
  <div id="app">
          <Gauge  gauge_id="facebookFilter" gauge_range_max="5000" gauge_icon="fa-facebook" icon_color="blue"/>
  </div>
</template>

<script>
/* eslint-disable */
import Vue from 'vue'
import Gauge from './components/Gauge.vue'

export const EventBus = new Vue();


export default {
  name: 'app',
  components: {
    Gauge
  },
  created: function() {
    this.initEventHandlers();
  },
  methods: {
    initEventHandlers: function() {
      console.log('Hello there');
      EventBus.$on('facebookFilterChangeEvent', msg => {
        console.log(msg);
      })

    }
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
