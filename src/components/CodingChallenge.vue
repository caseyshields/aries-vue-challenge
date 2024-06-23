<template>
  <div>
    <h1>Options Profit Calculator</h1>

    <!-- Chart -->
    <svg xmlns="http://www.w3.org/2000/svg"
        :width="width" :height="height" 
        v-on:mousemove="trace">
      <!-- labels -->
      <g class="background">
        <text :x="this.margin/2" :y="this.y(this.max/2)" font-size="12" text-anchor="middle">
            Profit
        </text>
        <text :x="this.margin/2" :y="this.y(-this.max/2)" font-size="12" text-anchor="middle">
            Loss
        </text>
        <text :x="this.y(this.max/2)" :y="this.margin/2" font-size="12" text-anchor="middle">
            Below Strike
        </text>
        <text :x="this.y(-this.max/2)" :y="this.margin/2" font-size="12" text-anchor="middle">
            Above Strike
        </text>
        <rect class='losing' fill='lightgrey'
            :x="this.margin" :y="this.y(0)" 
            :width="this.width-(this.margin*2)"
            :height="this.y(-this.max)-this.y(0)">
        </rect>
        <rect class='winning' fill='white'
            :x="this.margin" :y="this.margin" 
            :width="this.width-(this.margin*2)"
            :height="this.y(-this.max)-this.y(0)">
        </rect>
      </g>

      <g class="contracts">
        <!-- Contract plots added here -->
      </g>

      <g class="price-axis">
        <!-- Vertical rules and labels added here -->
      </g>
      
      <g class="profit-axis">
        <!-- Horizontal rules and labels added here -->
      </g>
    </svg>

    <!-- Legend -->
    <div class="legend">
      <!-- TODO would have more flexibility if I broke the legend out into it's own component -->
      <div v-for="(option, index) in optionsData" 
          :style="{color:legend(index)}"
          :key="index" >
        {{ index+1 }})
        {{ option.long_short }} 
        {{ option.type }}
        strike:{{ option.strike_price }}
        bid:{{ option.bid }}
        ask:{{ option.ask }}
        expires: {{ option.expiration_date }}
      </div>
    </div>
    
    <div class="coordinate">
      <!-- mouse coordinates displayed here -->
    </div>
  </div>

</template>

<script>
import * as d3 from "d3";

export default {
  name: 'CodingChallenge',
  props: {
    optionsData: Array,
    width: { type:Number, default:512 },
    height: { type:Number, default:512 },
    margin: { type:Number, default:48 }
  },

  // compute chart features from object attributes, and update when they change
  computed: {

    // find the maximum premium, which lets us size the chart's axis
    max: function() {
      let v = 0;
      for (let option of this.optionsData) {
        if (option.bid > v) v = option.bid;
        if (option.ask > v) v = option.ask;
      }
      return v * 1.1;
    },

    // The x scale maps the price change to the width of the screen, minus margins
    x: function() {
      return d3.scaleLinear()
        .domain([-this.max,this.max])
        .range([this.margin, this.width-this.margin]);
    },

    // the y scale maps the profit to the height of the screen, excepting margins
    y: function() {
      return d3.scaleLinear()
        .domain([-this.max,this.max])
        .range([this.height-this.margin, this.margin]);
    },

    // the legend is used to assign a color to each contract plot. Max of 10
    legend: function() {
      return d3.scaleOrdinal(d3.schemeCategory10);
    },

    // number formatter for money
    format: function() {
      return d3.format("$.2f");
    },

    // generates lines on the screen from price coordinates
    line: function() {
      return d3.line()
          .x( (d)=>this.x(d[0]) )
          .y( (d)=>this.y(d[1]) );
    },

    // generator for a price axis
    xAxis: function() {
      let w = this.width-(2*this.margin)
      return d3.axisBottom(this.x)
          .tickSizeInner(w)
          .tickSizeOuter(w)
          .ticks(5, this.format);
    },

    // generator for a profit axis
    yAxis: function() {
      let h = this.width-(2*this.margin);
      return d3.axisRight(this.y)
          .tickSizeInner(h)
          .tickSizeOuter(h)
          .ticks(5, this.format);
    },

    // computes plots and values from the raw contract information
    contracts: function() {
      let contracts = [];
      for (let i=0; i<this.optionsData.length; i++) {
        let option = this.optionsData[i];
        let contract = {option, i};
        
        // you can express each contract as a horizontal or vertical reflection of one graph
        let v = (option.long_short==='long') ? 1 : -1;
        let h = (option.type==='Call') ? 1 : -1;
        contract.v = v;
        contract.h = h;
        
        // by convention, I start with the plot of a long call option
        contract.plot = [
            [-this.max*h, -option.ask*v],[0,-option.ask*v],[this.max*h, (this.max-option.ask)*v], 
            [this.max*h, (this.max-option.bid)*v],[0,-option.bid*v],[-this.max*h, -option.bid*v]];
        
        // precompute locations and intervals
        contract.max = [h*option.ask, h*option.bid];
        contract.anchor = [-h*this.max/2, -v*(option.ask + option.bid)/2];

        contracts.push(contract);
      }
      return contracts;
    }
    
  },

  mounted: function () {
    // select some features of the svg
    this.svg = d3.select(this.$el).select('svg');
    this.coordinate = d3.select(this.$el).select('div.coordinate');
    // when the component mounts, render the chart
    this.render();
  },

  updated: function() {
    this.render();
  },

  methods: {
    // Use D3 to render the chart
    render: function() {
      // TODO might want to override the Vue render function with something like this, but I am unsure of the implications...

      // draw axes in appropriate group
      this.svg.select('g.price-axis')
        .attr('transform', `translate(0, ${this.margin})`)
        .call(this.xAxis);
      this.svg.select('g.profit-axis')
        .attr('transform', `translate(${this.margin}, 0)`)
        .call(this.yAxis);

      // draw/update a plot for each contract
      this.svg.select('g.contracts')
        .selectAll('g')
        .data(this.contracts)
        .join(
          // append plots for new contracts
          enter => {
            enter.append('text')
                .attr('x', d=>this.x(d.anchor[0]))
                .attr('y', d=>this.y(d.anchor[1])+3)
                .attr('text-anchor', 'middle')
                .attr('font-size', 10)
                .html(d => 'Breakeven ' + this.format(d.max[1]) + ' to ' + this.format(d.max[0]));
            enter.append("path") // TODO should be made with a d3 area generator...
                .attr('d', (d) => this.line(d.plot))
                .attr("fill", (d,i) => {
                  let c = d3.color(this.legend(i));
                  c.opacity = 0.6;
                  return c;
                })
                .attr("stroke", d=>this.legend(d.i))
                .on('mouseover', e=>{
                  // let d = d3.select(e.target).datum(); // note: it is possible to obtain the contract from the plot dom element...
                  d3.select(e.target).style('stroke-width', 3);
                })
                .on('mouseleave', 
                    e=>d3.select(e.target).style('stroke-width', 1));
          },
          // update existing plots
          update => {
            update.selectAll('path')
                .attr('d', (d) => this.line(d.plot))
                .attr("fill", (d,i) => {
                  let c = d3.color(this.legend(i));
                  c.opacity = 0.6;
                  return c;
                });
            update.selectAll('text')
                .attr('x', d=>this.x(d.anchor[0]))
                .attr('y', d=>this.y(d.anchor[1])+3)
                .html(d => 'Breakeven ' + this.format(d.max[1]) + ' to ' + this.format(d.max[0]));
          },
          // and just drop missing plots
          exit => { exit.remove(); }
        )
    },

    // display plot coordinates to user
    trace(event) {
      if (event.offsetX < this.margin || event.offsetX > this.width-this.margin
          || event.offsetY < this.margin || event.offsetY > this.height-this.margin) {
        this.coordinate.html('');
      } else {
        let price = this.x.invert(event.offsetX);
        let profit = this.y.invert(event.offsetY);
        this.coordinate.html( this.format(price)+', '+this.format(profit));
      }
    }
  }
}
</script>

<style scoped>

/* I'd like the legend expressed as styles, but it conflicts a bit with D3 idioms...
   maybe I could set 10 CSS vars on mount using a D3 chromatic scale? */
/* div.legend {
  text-align: left;
} */

/* It also appears I can't style SVG elements with vue component styles?  
  definitely something I need to look into. */
/* line.price-ruler {
    stroke-width: 3px;
} */
</style>
