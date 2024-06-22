<template>
  <div>
    <h1>Options Profit Calculator</h1>

    <!-- Chart -->
    <svg :width="width" :height="height">
      <g class="themoney"></g>
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

    // generates lines on the screen from price coordinates
    line: function() {
      return d3.line()
          .x( (d)=>this.x(d[0]) )
          .y( (d)=>this.y(d[1]) );
    },

    // generator for a price axis
    xAxis: function() {
      let w = this.width-(2*this.margin)
      return d3.axisTop(this.x)
          .tickSizeInner(w)
          .tickSizeOuter(w)
          .ticks(5, "$.2f");
    },

    // generator for a profit axis
    yAxis: function() {
      let h = this.width-(2*this.margin);
      return d3.axisLeft(this.y)
          .tickSizeInner(h)
          .tickSizeOuter(h)
          .ticks(5, "$.2f");
    },

    // computes profit areas from the raw contract information
    contracts: function() {
      let contracts = [];
      for (let i=0; i<this.optionsData.length; i++) {
        let option = this.optionsData[i];
        let contract = {};
        contract.mid = (option.bid + option.ask) / 2;
        
        // you can express each contract as a horizontal or vertical reflection
        let v = (option.long_short==='long') ? 1 : -1;
        let h = (option.type==='Call') ? 1 : -1;
        
        // TODO add break-even points? line?
        // let askEven = [h*option.ask,0];
        // let midEven = [h*option.mid,0];
        // let bidEven = [h*option.bid,0];

        // by convention, I start with the plot of a long call
        contract.midLine = [[-this.max*h, -contract.mid*v],[0,-contract.mid*v],[this.max*h, (this.max-contract.mid)*v]];
        contract.range = [[-this.max*h, -option.ask*v],[0,-option.ask*v],[this.max*h, (this.max-option.ask)*v], 
            [this.max*h, (this.max-option.bid)*v],[0,-option.bid*v],[-this.max*h, -option.bid*v]];
        contracts.push(contract);
      }
      return contracts;
    }
    
  },
  methods: {
    // Use D3 to render the chart
    render: function() {
      // TODO might want to override the Vue render function with something like this, but I am unsure of the implications...
      this.svg.select('g.price-axis')
        .attr('transform', `translate(0, ${this.height-this.margin})`)
        .call(this.xAxis);
      this.svg.select('g.profit-axis')
        .attr('transform', `translate(${this.height-this.margin}, 0)`)
        .call(this.yAxis);
      
      let otm = this.svg.select('g.themoney > rect');
      if (otm.size()==0)
        otm = this.svg.select('g.themoney').append('rect');

      otm.style('fill', 'lightgrey')
          .attr('x', this.x(-this.max))
          .attr('y', this.y(0))
          .attr('width', this.x(this.max)-this.x(-this.max))
          .attr('height', this.y(-this.max)-this.y(0));

      this.svg.select('g.contracts')
        .selectAll('g')
        .data(this.contracts)
        .join(
          enter => {
            enter.append("path") // should be made with a d3 area generator
                .attr('d', (d) => this.line(d.range))
                .attr("fill", (d,i) => {
                  let c = d3.color(this.legend(i));
                  c.opacity = 0.5;
                  return c;
                })
                .attr("stroke", "none");
            // enter.append('path')
            //     .attr('d', (d)=>this.line(d.midLine))
            //     .attr('fill', 'none')
            //     .attr('stroke', (d,i) => this.legend(i))
            //     .attr('stroke-width', 3);
          },
          update => {
            update.attr('d', (d) => this.line(d.range))
                .attr("fill", (d,i) => {
                  let c = d3.color(this.legend(i));
                  c.opacity = 0.5;
                  return c;
                });
              // update.attr('d', (d)=>this.line(d.midLine));
          },
          exit => { exit.remove(); }

        )
    }
  },
  // when the component mounts, render the chart
  mounted: function () {
    this.svg = d3.select(this.$el).select('svg');
    this.render();
    // TODO need to invoke on component's refresh hook as well...
  }
}
</script>

<style scoped>

/* I'd like the legend expressed as styles, but it conflicts a bit with D3 idioms...
   maybe I could set 10 CSS vars on mount using a D3 chromatic scale? */

</style>
