<template>
  <div 
    class="svgctn2 grid1-1"
    :style="gridStyles"
  >
    <h1 class="newfont">Day of Week</h1>
    <slot></slot> 
  </div>
</template>

<script>

import * as d3 from "d3";
export default {
  name:'PieChart',
  props: {
    msg: String,
    codename:{
      type:String,
      default:"PieTasty"
    },
    gridStyles:{
      type:Object,
      default:function(){return {}}
    },
    arrayData:{
      type:Array,
      default:function(){return []}
    }
  },
  data:function(){
    return {
      d3:d3,
      Histogram:null,
      svgNode:null,
      mandata:null
    }
  },
  created(){
    let slicer = (x) => {
      let p1 = x.slice(0,4) 
      return String(p1);
    }
    let stringFlattener = (x) => {
      return x.replaceAll(" ", "");
    }
    let grouper = (g, cv) => {
      let compress = cv.replaceAll(" ", "");
      let groups = g[0]
      if(groups[compress] !== undefined){
        groups[compress]['count'] = groups[compress]['count']+1
      } else {
        groups[compress] = {count:1,name:compress};
      }
      return g
    }
    let expander = (ao) => {
      //console.log(ao)
      let arr = []
      arr.push(ao['Mon'])
      arr.push(ao['Tue'])
      arr.push(ao['Wed'])
      arr.push(ao['Thu'])
      arr.push(ao['Fri'])
      arr.push(ao['Sat'])
      arr.push(ao['Sun'])
      return arr;
    }

    let mandata = this.arrayData
      .map(slicer)
      .map(stringFlattener)
      .reduce(grouper,[{}])
      .map(expander)[0]

    let total = mandata.reduce((acc,cv) => {
      acc+=cv.count;
      return acc;
    },0);

    //console.log(total);
    let percentage = (x) => {
      let dayAmount = x.count;
      let adjusted = Math.round(dayAmount*100/total)
      x.percentage = adjusted;
      return x
    }
    this.mandata = mandata.map(percentage)
    const N = d3.map(mandata, x => x['name']);
    const V = d3.map(mandata, x => x['count']);
    const I = d3.range(N.length).filter(i => !isNaN(V[i]));
    this.N = N
    this.V = V
    this.I = I

  },
  mounted(){

    let width = 450;
    let height = 414;
    var svg = d3.select(".svgctn2")
      .append("svg")
      .attr("id",this.codename)
      .attr("viewBox", [0, 0, width, height])
      .append("g")
      .attr("transform", "translate(228,180)")

    let arcs = d3.pie()(this.mandata.map(d => d.count));
    let arc = d3.arc().innerRadius(0).outerRadius(170)
    let names = this.N;
    //console.log(names)
    let colors = undefined;
    if (colors === undefined) colors = d3.schemeSpectral[new d3.InternSet(names).size];
    //if (colors === undefined) colors = d3.quantize(t => d3.interpolateSpectral(t * 0.8 + 0.1), names.size);

    // Construct scales.
    const color = d3.scaleOrdinal(names, colors);

    svg.selectAll("path")
      .data(arcs)
      .join("path")
        .attr("fill", d => color(this.N[d.index]))
        .attr("d", arc)

   d3.select(`#${this.codename}`)
     .select('g')
     .selectAll('whatever')
     .data(arcs)
     .enter()
     .append('text')
     .text(d => `${this.mandata[d.index]['name']}-${this.mandata[d.index]['percentage']}%`)
     .attr("transform", d => "translate(" + arc.centroid(d) + ")")
     .style("text-anchor", "middle")
     .style("font-size", 16)

     //d3.select(`#${this.codename}`)

  }
}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
.grid1-1 {
  box-sizing:border-box;
  display: grid;
  min-height:100px;
  grid-template-columns: 1fr;
  grid-template-rows: 1fr;
}
.border {
  border: 1px dashed navy;
}
.svgctn2 {
  display:flex;
  flex-direction:column;
}
.newfont {
  font-family:'Varela Round';
}
.svgctn2 {
  filter:blur(3px) opacity(0.4);
}
</style>
