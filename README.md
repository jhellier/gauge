# gauge

Work In Progress. This is a control gauge supporting a variety of actions outlined below. It is a Vue.js supported gauge which will allow any range and any set of values. The gauge is a D3.js implementation.  It is responsive, enabling it to be used in any size area. The gauge icon is a standard font awesome icon where you pass the icon name as an attribute. Ex. gauge_icon="fa-facebook" for the facebook icon. 

* Actions Supported
  * Click on the icon toggles display of data
  * Click on the gauge ring sets the value of the gauge
  * Click on the gauge text display resets the gauge to the default value
  
Here is the current screenshot: (Updated as I progress)

![alt text](gauge.png)

This is a code block from a Vue file showing the gauge tag definition. 
```
<template>
    <div class="gauge-div" :gauge_id="gauge_id" 
          :gauge_range_max="gauge_range_max"
          :gauge_range_default="gauge_range_default"
          :gauge_text="gauge_text"
          :gauge_icon="gauge_icon"
          :color="icon_color">
        <div :id="gauge_id" >
        </div>
    </div>
</template>
```
The :attributeName="attributeName" seems clumsy but it works. Need to see if there is a cleaner way to do this.

This is the use of it on a Vue page
```
<div class="col-sm">   
   <Gauge  gauge_id="facebookFilter" gauge_range_max="25000" gauge_range_default="2000" gauge_icon="fa-facebook"    gauge_text="Checkins" icon_color="blue"/>
</div>

<script>
import Gauge from './Gauge.vue'

export default {
  name: 'somePageSection',
  components: {
    SomeComponent,
    Gauge
    },

  ...
  }
 </script> 

```



## Project setup
```
npm install
```

### Compiles and hot-reloads for development
```
npm run serve
```

### Compiles and minifies for production
```
npm run build
```

### Run your tests
```
npm run test
```

### Lints and fixes files
```
npm run lint
```
