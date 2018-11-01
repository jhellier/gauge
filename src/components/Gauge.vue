<template>
    <div>
        <span class="spanTitle">D3 Gauge</span>
        <div id="gauge">
        </div>
    </div>
</template>

<script>
/* eslint-disable */

import * as d3 from 'd3';

export default {
        name: 'Gauge',

        data() {
            return {
                count: '',
                gaugeRange: 1000,
                radianMultipler: 5,
                gaugeText: '',
                tickStart: 110,
                tickLength: -10,
                tickScale: d3.scaleLinear().range([0,200]).domain([0,200])
            }
        },

        created: function() {
            this.buildGauge();
        },

        mounted: function() {
            this.buildGauge();
        },

        methods: {
            getRadianAngle: function(target) {
                let newXPos = d3.mouse(target)[0];
                let newYPos = -d3.mouse(target)[1];
                let newRadianAngle = Math.atan2(newXPos,newYPos);
                console.log('Radians ', newRadianAngle);


                return newRadianAngle;
            },

            setGaugeValue: function(gaugeText, value) {
                gaugeText.text(value);
            },

            buildGauge: function() {

                let that = this;

                let gaugeG = d3.selectAll('#gauge')
                    .append('svg')
                        .attr('height',600)
                        .attr('width',600)
                    .append('g')
                        .attr('transform','translate(300,150)')

                    gaugeG.append('circle')
                        .attr('r', 30)
                        .attr('cx',0)
                        .attr('cy',0)
                        .attr('id','wCircle')
                        .style('fill','#ccc')
                        .on('click',function() {
                            console.log('Hello, event');
                            //d3.select(this).style('fill','green')
                        });
                        
                that.gaugeText = gaugeG.append('text')
                                    .attr('dy', 70)
                                    .style('text-anchor','middle')
                                    .style('font-size',24)
                                    .text('2000')        

                let tau = 2 * Math.PI;

                let arc = d3.arc()
                            .innerRadius(100)
                            .outerRadius(105)
                            .startAngle(-2.5)
                            //.endAngle(2.5)


                let gauge = gaugeG.append('path')
                                .datum({endAngle: 2.5})
                                .style('fill','#ddd')
                                .attr('id','baseCircle')
                                .attr('d',arc)
                                .on('click', function() {    
                                    let radians = that.getRadianAngle(this);                              
                                    gaugeMarkerRing.transition().duration(2000).attrTween('d',arcTween(radians));

                                    let text = Math.round(that.gaugeRange/that.radianMultipler * (radians + 2.5));
                                    that.gaugeText.text(text);

                                    return;
                                })
                                .on('mouseover', function() {
                                    d3.select(this).style('cursor','pointer')
                                    let radians = that.getRadianAngle(this);                              

                                    let text = Math.round(that.gaugeRange/that.radianMultipler * (radians + 2.5));

                                     d3.select('body')
                                        .append('div')
                                        .attr('class', 'toolTip')
                                        .attr('id','toolTip')
                                        .html(text)
                                        .style('left', d3.event.pageX + 'px')
                                        .style('top', d3.event.pageY - 18 + 'px');

                                })
                                .on('mouseout', function() {
                                    d3.select(this).style('cursor','default')
                                    d3.selectAll('#toolTip').remove();

                                })

                let ticks = gaugeG.selectAll('.gauge-tick')
                                .data(d3.range(38,338,30))
                                .enter()
                                .append('line')
                                .attr('class','gauge-tick')
                                .attr('x1',0)
                                .attr('x2',0)
                                .attr('y1', that.tickStart)
                                .attr('y2', that.tickStart + that.tickLength)
                                .attr('transform', function(d) {
                                    return 'rotate(' + that.tickScale(d) + ')';
                                });

                let gaugeMarkerRing = gaugeG.append('path')                                
                                .datum({endAngle: 1})
                                .attr('id','marker')
                                .style('fill','#aaa')
                                .attr('d',arc)
                                .on('mouseover', function(event) {
                                    console.log('This is the other one',event);
                                    
                                })
                                .on('mouseenter', function() {
                                    d3.select(this).style('cursor','pointer')
                                   let radians = that.getRadianAngle(this);                              

                                    let text = Math.round(that.gaugeRange/that.radianMultipler * (radians + 2.5));

                                     d3.select('body')
                                        .append('div')
                                        .attr('class', 'toolTip')
                                        .attr('id','toolTip')
                                        .html(text)
                                        .style('left', d3.event.pageX + 'px')
                                        .style('top', d3.event.pageY - 18 + 'px');                                    
                                })
                                .on('mouseout', function() {
                                    d3.select(this).style('cursor','default')
                                    d3.selectAll('#toolTip').remove();
                                })                                
                                .on('click', function() {
                                    let radians = that.getRadianAngle(this);                              
                                    gaugeMarkerRing.transition().duration(2000).attrTween('d',arcTween(radians));

                                    let text = Math.round(that.gaugeRange/that.radianMultipler * (radians + 2.5));
                                    that.gaugeText.text(text);
                                    return;
                                })                                


                function arcTween(angle) {
                    return function(d) {
                        let interpolate = d3.interpolate(d.endAngle, angle);
                        return function(t) {
                            d.endAngle = interpolate(t);
                            return arc(d);
                        }
                    }
                }                

                gaugeMarkerRing.transition().duration(2000).attrTween('d',arcTween(1.99));






                                            
            }
        }

}
</script>


<style>

.gauge-tick {
    stroke-width:3;
	stroke: #ddd;
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
  background: rgba(218, 220, 223,0);
  /* border: 1px;
  border-style: solid;
  border-radius: 8px;
  pointer-events: none; */
  z-index: 2000;
}
</style>
