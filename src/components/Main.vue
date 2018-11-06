<template>
  <div id="main-div">
    <Gauge gauge_id="facebookFilter"  
            gaugeRangeMax="5000" />
  </div>
</template>

<script>

import Gauge from './Gauge.vue'

export default {
    name: 'main-div',

    components: {
        Gauge
    }

}
</script>

<style>

</style>
