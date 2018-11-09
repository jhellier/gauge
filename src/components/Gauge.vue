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

<script>
/* eslint-disable */

import { EventBus } from '../main.js';
import * as d3 from 'd3';


export default {
  name: 'Gauge',

  props: {
    displayText: {
      type: String,
      default() {
        return 'Hello';
      }
    },
    gauge_id: String,
    gauge_range_max: String,
    gauge_range_default: String,
    gauge_icon: String,
    icon_color: String,
    gauge_text: String

  },

  data() {
    let innerRadius = 100;
    let radiusWidth = 5;
    let outerRadius = innerRadius + radiusWidth;
    let labelOffset = 20;
    let tickCount = 10;

    return {
      gaugeId: this.gauge_id,
      innerRadius,
      outerRadius,
      gaugeRange: this.gauge_range_max,
      gaugeRangeMax: this.gauge_range_max,
      gaugeRangeDefault: this.gauge_range_default,
      radianMultipler: 5,
      tickSpacing: this.gauge_range_max/tickCount,
      tickStart: outerRadius + radiusWidth,
      tickLength: -(2 * radiusWidth),
      labelRadius: outerRadius + labelOffset,
      radToDegree: 180 / Math.PI,
      gaugeG: {},
      gaugeText: this.gauge_text,
      gaugeCounterText: this.gauge_range_default,
      gaugeMarkerRing: {},      
      gaugeArc: {},
      changeEventName: this.gauge_id + 'ChangeEvent',
      toggleEventName: this.gauge_id + 'ToggleEvent',
      resetEventName: this.gauge_id + 'ResetEvent',
      viewShowing: false,
      gaugeIcon: this.gauge_icon,
      iconColor: this.icon_color
    };
  },

  created: function() {
    //console.log(this.displayText);
  },

  mounted: function() {
    this.setGaugeArc();  
    this.buildGauge();
    this.buildGaugeMarker(this.getGaugeMarkerPosInRadians());
  },

  methods: {

    fireChangeEvent: function(args) {

    },
    getRadianAngle: function(target) {
      let newXPos = d3.mouse(target)[0];
      let newYPos = -d3.mouse(target)[1];
      let newRadianAngle = Math.atan2(newXPos, newYPos);
      return newRadianAngle;
    },

    setGaugeValue: function(value) {
      //this.gaugeCounterText.text(value);
    },

    setGaugeArc: function() {
      this.gaugeArc = d3
        .arc()
        .innerRadius(100)
        .outerRadius(105);
    },

    arcTween: function(angle) {
        let that = this;
        return function(d) {
          let interpolate = d3.interpolate(d.endAngle, angle);
          return function(t) {
            d.endAngle = interpolate(t);
            return that.gaugeArc(d);
          };
        };
      },


      getGaugeMarkerPosInRadians: function() {
        if (this.gaugeRangeDefault == undefined || this.gaugeRangeDefault == 0)
          return -2.5;
        let gaugeRangeDivisor = this.gaugeRangeMax/this.radianMultipler;
        let gaugePortion = this.gaugeRangeDefault/gaugeRangeDivisor;
        return gaugePortion - 2.5;
      },


    buildGaugeMarker: function(endAngle) {
      let that = this;  
      that.gaugeMarkerRing = that.gaugeG
        .append('path')
        .datum({ startAngle: -2.5, endAngle: endAngle })
        .attr('id', 'marker')
        .style('fill', '#777')
        .attr('d', that.gaugeArc)
        .on('mouseover', function(event) {
          // console.log('This is the other one',event);
        })
        .on('mouseenter', function() {
          d3.select(this).style('cursor', 'pointer');
        })
        .on('mouseout', function() {
          d3.select(this).style('cursor', 'default');
        })
        .on('click', function() {
          let radians = that.getRadianAngle(this);
          that.gaugeMarkerRing
            .transition()
            .duration(1000)
            .attrTween('d', that.arcTween(radians));

          let text = Math.round(
            (that.gaugeRange / that.radianMultipler) * (radians + 2.5)
          );
          that.gaugeCounterText.text(text);

          EventBus.$emit(that.changeEventName,text);
          
          return;
        });

    },

    buildGauge: function() {
      let that = this;

      that.gaugeG = d3
        .selectAll('#' + that.gaugeId)
            .append('svg')
                .attr('viewBox', '0 0 300 275')
            .append('g')
                .attr('transform', 'translate(150,150)');

      that.gaugeG.append('svg:foreignObject')
              .attr('x',-35)
              .attr('y',-70)
              .style('font-size',80)
              .style('color',that.iconColor)
              .html('<i class="fab ' + that.gaugeIcon + '"></i>')
              .on('click', function() {
                that.viewShowing = !that.viewShowing;

                EventBus.$emit(that.toggleEventName, that.viewShowing);
              })
              .on('mouseenter', function() {
                d3.select(this).style('cursor', 'pointer');
              })
              .on('mouseout', function() {
                d3.select(this).style('cursor', 'default');
              })              


      let ticks = that.gaugeG
        .selectAll('.gauge-tick')
            .data(d3.range(-2.5, 3, 0.5))
            .enter()
            .append('line')
                .attr('class', 'gauge-tick')
                .attr('x1', 0)
                .attr('x2', 0)
                .attr('y1', that.tickStart)
                .attr('y2', that.tickStart + that.tickLength)
                .attr('transform', function(d) {
                  return 'rotate(' + (d * that.radToDegree + 180) + ')';
                });

      let tickLabelValues = d3.range(that.gaugeRange, -100, -that.tickSpacing);
      let tickLabels = that.gaugeG
        .selectAll('.gauge-tick-label')
            .data(d3.range(-2.5, 3.0, 0.5))
            .enter()
            .append('text')
                .attr('class', 'gauge-tick-label')
                .attr('text-anchor', 'middle')
                .attr('x', function(d) {
                  return Math.sin(d + Math.PI) * 130;
                })
                .attr('y', function(d) {
                  return Math.cos(d + Math.PI) * 125 + 5;
                })
                .text(function(d, i) {
                  return Number(tickLabelValues[i]).toFixed(0);
                });

      that.gaugeCounterText = that.gaugeG
        .append('text')
            .attr('dy', 70)
            .style('text-anchor', 'middle')
            .style('font-size', 24)
            .text(that.gaugeCounterText)
         .on('click', function() {
           that.gaugeCounterText.text(that.gaugeRangeDefault);
            that.gaugeMarkerRing
              .transition()
              .duration(1000)
              .attrTween('d', that.arcTween(that.getGaugeMarkerPosInRadians()));
            
            EventBus.$emit(that.resetEventName, that.gaugeRangeDefault);

         }) 
        .on('mouseenter', function() {
            d3.select(this).style('cursor', 'pointer');
         })
        .on('mouseout', function() {
            d3.select(this).style('cursor', 'default');
         })   

      that.gaugeG
        .append('text')
            .attr('dy', 40)
            .style('text-anchor', 'middle')
            .style('font-size', 18)
            .text(that.gaugeText);

      let tau = 2 * Math.PI;

      that.gaugeG
        .append('path')
        .datum({ startAngle: -2.515, endAngle: 2.515 })
            .style('fill', '#ddd')
            .attr('id', 'baseCircle')
            .attr('d', that.gaugeArc)
            .on('click', function() {
                let radians = that.getRadianAngle(this);
                that.gaugeMarkerRing
                    .transition()
                    .duration(1000)
                    .attrTween('d', that.arcTween(radians));

                let text = Math.round(
                    (that.gaugeRange / that.radianMultipler) * (radians + 2.5)
                );
                that.gaugeCounterText.text(text);

                EventBus.$emit(that.changeEventName,text);

                return;
            })
            .on('mouseover', function() {
                d3.select(this).style('cursor', 'pointer');
            })
            .on('mouseout', function() {
                d3.select(this).style('cursor', 'default');
            });
    }
  }
};
</script>


<style>
.gauge-div {
  height: 100%;
  width: 100%;
}

.gauge-tick {
  stroke-width: 3;
  stroke: #aaa;
}

.gauge-tick-label {
  font-size: 14px;
}

.spanTitle {
  font-size: 24px;
  text-align: center;
}

div.toolTip {
  position: absolute;
  text-align: center;
  width: 40px;
  height: 10px;
  padding: 2px;
  font: 12px sans-serif;
  background: rgba(218, 220, 223, 0);
  /* border: 1px;
  border-style: solid;
  border-radius: 8px;
  pointer-events: none; */
  z-index: 2000;
}
</style>
