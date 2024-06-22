<template>
  <div>
    <h1>Options Profit Calculator</h1>

    <svg :width="width" :height="height">
      <g class="contracts"></g>
      <g class="price-axis"></g>
      <g class="profit-axis"></g>
    </svg>

    <div v-for="(option, index) in optionsData" :key="index" :class="'legend'+index">
      {{ option.long_short }} 
      {{ option.type }}
      strike:{{ option.strike_price }}
      bid:{{ option.bid }}
      ask:{{ option.ask }}
      expires: {{ option.expiration_date }}
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
  computed: {
    max: function() {
      let v = 0;
      for (let option of this.optionsData) {
        if (option.bid > v) v = option.bid;
        if (option.ask > v) v = option.ask;
      }
      return v*1.1;
    },
    x: function() {
      return d3.scaleLinear()
        .domain([-this.max,this.max])
        .range([this.margin, this.width-this.margin]);
    },
    y: function() {
      return d3.scaleLinear()
        .domain([-this.max,this.max])
        .range([this.height-this.margin, this.margin]);
    },
    legend: function() {
      return d3.scaleOrdinal(d3.schemeCategory10);
    },
    line: function() {
      return d3.line()
          .x( (d)=>this.x(d[0]) )
          .y( (d)=>this.y(d[1]) );
    },
    top: function() {
      return d3.axisTop(this.x).ticks(5, "$.2f");
    },
    left: function() {
      return d3.axisLeft(this.y).ticks(5, "$.2f");
    },
    contracts: function() {
      let contracts = [];
      for (let i=0; i<this.optionsData.length; i++) {
        let option = this.optionsData[i];
        let contract = {};
        contract.mid = (option.bid + option.ask) / 2;
        
        let v = (option.long_short==='long') ? 1 : -1;
        let h = (option.type==='Call') ? 1 : -1;
        
        // TODO add break-even points? line?
        // let askEven = [h*option.ask,0];
        // let midEven = [h*option.mid,0];
        // let bidEven = [h*option.bid,0];

        contract.midLine = [[-this.max*h, -contract.mid*v],[0,-contract.mid*v],[this.max*h, (this.max-contract.mid)*v]];
        contract.range = [[-this.max*h, -option.ask*v],[0,-option.ask*v],[this.max*h, (this.max-option.ask)*v], 
            [this.max*h, (this.max-option.bid)*v],[0,-option.bid*v],[-this.max*h, -option.bid*v]];
        contracts.push(contract);
      }
      return contracts;
    }
    
  },
  methods: {
    render: function() {
      this.svg.select('g.price-axis')
        .attr('transform', `translate(0, ${this.margin})`)
        .call(this.top);
      this.svg.select('g.profit-axis')
        .attr('transform', `translate(${this.margin}, 0)`)
        .call(this.left);
      
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
            enter.append('path')
                .attr('d', (d)=>this.line(d.midLine))
                .attr('fill', 'none')
                .attr('stroke', (d,i) => this.legend(i))
                .attr('stroke-width', 3);
          },
          exit => { exit.remove(); }

        )
    }
    // updateContracts: function() {
    //   for (let option of this.optionsData) {
    //     option.mid = (option.bid + option.ask)/2;

    //     if (option.long_short==='long') {
    //       if (option.type==='Call') {
    //         option.maxLoss = option.mid;
    //       } else if (option.type==='Put') {
    //         option.maxLoss = option.mid;
    //       }
    //     } else if (option.long_short==='short') {
    //       if (option.type==='Call') {
    //         option.maxProfit = option.mid;
    //       } else if (option.type==='Put') {
    //         option.maxProfit = option.mid;
    //       }
    //     }
    //   }
    //   console.log( this.optionsData );
    // }
  },
  mounted: function () {
    this.svg = d3.select(this.$el).select('svg');
    this.render();
  }
}
</script>

<style scoped>
/* Your Code Here */

</style>
