<template>
  <div>
    <h1>Options Profit Calculator</h1>

    <!-- Chart -->
    <svg :width="width" :height="height" xmlns="http://www.w3.org/2000/svg">
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
        <line class="price-ruler"></line>
        <line class="profit-ruler"></line>
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
    <span class=""></span>
    <!-- Legend -->
    <div class="legend">
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
    <!-- TODO would have more design freedom is we broke the legend out into it's own component -->

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

    // computes profit areas from the raw contract information
    contracts: function() {
      let contracts = [];
      for (let i=0; i<this.optionsData.length; i++) {
        let option = this.optionsData[i];
        let contract = {option};
        
        // you can express each contract as a horizontal or vertical reflection
        let v = (option.long_short==='long') ? 1 : -1;
        let h = (option.type==='Call') ? 1 : -1;
        contract.v = v;
        contract.h = h;
        
        // by convention, I start with the plot of a long call
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

  // when the component mounts, render the chart
  mounted: function () {
    this.svg = d3.select(this.$el).select('svg');
    this.render();
    // TODO need to invoke on component's refresh hook as well...
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
      
      // shade the area where an option would lose money
      let out = this.svg.select('g.background > rect');
      if (out.size()==0)
        out = this.svg.select('g.background').append('rect');
      out.style('fill', 'lightgrey')
          .attr('x', this.x(-this.max))
          .attr('y', this.y(0))
          .attr('width', this.x(this.max)-this.x(-this.max))
          .attr('height', this.y(-this.max)-this.y(0));

      // draw/update a plot for each contract
      this.svg.select('g.contracts')
        .selectAll('g')
        .data(this.contracts)
        .join(
          enter => {
            enter.append("path") // should be made with a d3 area generator
                .attr('d', (d) => this.line(d.plot))
                .attr("fill", (d,i) => {
                  let c = d3.color(this.legend(i));
                  c.opacity = 0.6;
                  return c;
                })
                .attr("stroke", "none")
                .on('mousemove', e=>{
                  let d = d3.select(e.target).datum();
                  this.trace(e, d);
                });
            enter.append('text')
                  .attr('x', d=>this.x(d.anchor[0]))
                  .attr('y', d=>this.y(d.anchor[1])+3)
                  .attr('text-anchor', 'middle')
                  .attr('font-size', 10)
                  .html(d => 'Breakeven ' + this.format(d.max[1]) + ' to ' + this.format(d.max[0]));
          },
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
          exit => { exit.remove(); }
        )
    },
    trace(event, contract) {
      console.log(event, contract);
      // console.log(event.offsetX, event.offsetY);
      let price = this.x.invert(event.offsetX);
      let profit = this.y.invert(event.offsetY);
      console.log(price, profit);
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
</style>
