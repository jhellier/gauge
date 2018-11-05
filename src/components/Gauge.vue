<template>
    <div id="gauge-div">
        <span class="spanTitle">D3 Gauge</span>
        <div id="gauge">
        </div>
    </div>
</template>

<script>
/* eslint-disable */

import * as d3 from "d3";

export default {
  name: "Gauge",

  props: {
    displayText: {
      type: String,
      default() {
        return "Hello";
      }
    }
  },

  data() {
    let gaugeRangeMax = 1000;
    let innerRadius = 100;
    let radiusWidth = 5;
    let outerRadius = innerRadius + radiusWidth;
    let labelOffset = 20;
    let gaugeRangeExtra = 10;

    return {
      innerRadius,
      outerRadius,
      gaugeRange: gaugeRangeMax + gaugeRangeExtra,
      radianMultipler: 5,
      tickStart: outerRadius + radiusWidth,
      tickLength: -(2 * radiusWidth),
      labelRadius: outerRadius + labelOffset,
      radToDegree: 180 / Math.PI,
      gaugeG: {},
      gaugeText: "",
      gaugeMarkerRing: {},
      gaugeArc: {}
    };
  },

  created: function() {
    //console.log(this.displayText);
  },

  mounted: function() {
    this.setGaugeArc();  
    this.buildGauge();
    this.buildGaugeMarker();
  },

  methods: {
    getRadianAngle: function(target) {
      let newXPos = d3.mouse(target)[0];
      let newYPos = -d3.mouse(target)[1];
      let newRadianAngle = Math.atan2(newXPos, newYPos);
      return newRadianAngle;
    },

    setGaugeValue: function(gaugeText, value) {
      this.gaugeText.text(value);
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


    buildGaugeMarker: function() {
      let that = this;  
      that.gaugeMarkerRing = that.gaugeG
        .append("path")
        .datum({ startAngle: -2.5, endAngle: -2.5 })
        .attr("id", "marker")
        .style("fill", "#aaa")
        .attr("d", that.gaugeArc)
        .on("mouseover", function(event) {
          // console.log('This is the other one',event);
        })
        .on("mouseenter", function() {
          d3.select(this).style("cursor", "pointer");
        })
        .on("mouseout", function() {
          d3.select(this).style("cursor", "default");
        })
        .on("click", function() {
          let radians = that.getRadianAngle(this);
          that.gaugeMarkerRing
            .transition()
            .duration(2000)
            .attrTween("d", that.arcTween(radians));

          let text = Math.round(
            (that.gaugeRange / that.radianMultipler) * (radians + 2.5)
          );
          that.gaugeText.text(text);
          return;
        });

    },

    buildGauge: function() {
      let that = this;

      let gaugeX = document.body.clientWidth/2;
      that.gaugeG = d3
        .selectAll("#gauge")
            .append("svg")
                .attr("viewBox", "0 0 800 800")
            .append("g")
                .attr("transform", "translate(400,150)");

      that.gaugeG
        .append("circle")
            .attr("r", 30)
            .attr("cx", 0)
            .attr("cy", 0)
            .attr("id", "wCircle")
            .style("fill", "#ccc")
            .on("click", function() {
            console.log("Hello, event");
            });

      let ticks = that.gaugeG
        .selectAll(".gauge-tick")
            .data(d3.range(-2.5, 3, 0.5))
            .enter()
            .append("line")
                .attr("class", "gauge-tick")
                .attr("x1", 0)
                .attr("x2", 0)
                .attr("y1", that.tickStart)
                .attr("y2", that.tickStart + that.tickLength)
                .attr("transform", function(d) {
                return "rotate(" + (d * that.radToDegree + 180) + ")";
                });

      let tickLabelValues = d3.range(1000, -100, -100);
      let tickLabels = that.gaugeG
        .selectAll(".gauge-tick-label")
            .data(d3.range(-2.5, 3.0, 0.5))
            .enter()
            .append("text")
                .attr("class", "gauge-tick-label")
                .attr("text-anchor", "middle")
                .attr("x", function(d) {
                return Math.sin(d + Math.PI) * 130;
                })
                .attr("y", function(d) {
                return Math.cos(d + Math.PI) * 125 + 5;
                })
                .text(function(d, i) {
                return tickLabelValues[i];
                });

      that.gaugeText = that.gaugeG
        .append("text")
            .attr("dy", 70)
            .style("text-anchor", "middle")
            .style("font-size", 24)
            .text("");

      let tau = 2 * Math.PI;

      that.gaugeG
        .append("path")
        .datum({ startAngle: -2.515, endAngle: 2.515 })
            .style("fill", "#ddd")
            .attr("id", "baseCircle")
            .attr("d", that.gaugeArc)
            .on("click", function() {
                let radians = that.getRadianAngle(this);
                that.gaugeMarkerRing
                    .transition()
                    .duration(2000)
                    .attrTween("d", that.arcTween(radians));

                let text = Math.round(
                    (that.gaugeRange / that.radianMultipler) * (radians + 2.5)
                );
                that.gaugeText.text(text);

                return;
            })
            .on("mouseover", function() {
                d3.select(this).style("cursor", "pointer");
            })
            .on("mouseout", function() {
                d3.select(this).style("cursor", "default");
            });
    }
  }
};
</script>


<style>
#gauge-div {
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
