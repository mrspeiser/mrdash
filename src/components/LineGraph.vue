<template>
  <div 
    class="svgctn3 grid1-1 "
    :style="gridStyles"
  >
    <h1 class="newfont">Volume</h1>
    <slot></slot> 
  </div>
</template>

<script>

import * as d3 from "d3";
export default {
  name:'LineGraph',
  props: {
    codename:{
      type:String,
      default:"Linemble"
    },
    msg: String,
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
      svgNode:null
    }
  },
  created(){
    let slicer = (x) => {
      let p1 = x.slice(0,11) 
      let p2 = x.slice(19,28) 
      return String(p1+p2);
    }
    let stringFlattener = (x) => {
      return x.replaceAll(" ", "");
    }
    let grouper = (g, cv,i) => {
      let compress = cv.replaceAll(" ", "");
      let groups = g[0]
      if(groups[compress] !== undefined){
        groups[compress]['count'] = groups[compress]['count']+1
      } else {
        groups[compress] = {count:1,order:i,name:compress};
      }
      return g
    }
    let expander = (ao) => {
      //console.log(ao)
      let arr = []
      for(let k in ao){
        arr.push(ao[k])
      }
      return arr;
    }
    let compare = (a, b) => {
      if (a.order < b.order){
        return -1;
      }
      if (a.order > b.order){
        return 1;
      }
      return 0;
    }

    this.mandata = this.arrayData
      .map(slicer)
      .map(stringFlattener)
      .reduce(grouper, [{}])
      .map(expander)[0]
      .sort(compare)
      .map(x => ({...x,count:300-(x.count*6)}))

    let X = d3.map(this.mandata, x => x['name']);
    let Y = d3.map(this.mandata, x => x['count']);
    let I = d3.range(X.length).filter(i => !isNaN(Y[i]));
    this.X = X
    this.Y = Y
    this.I = I
    let defined;
    if (defined === undefined) defined = (d, i) => !isNaN(X[i]) && !isNaN(Y[i]);
    let D = d3.map(this.mandata, defined);
    this.D = D;

  },
  mounted(){
    let width = 740;
    let height = 240;
    let strokeWidth = 5;
    let strokeLinejoin = "round";
    //let strokeLinecap = "round";

    d3.select(".svgctn3")
      .append("svg")
      .attr("id",this.codename)
      .attr("viewBox", [0, 0, width, height])

    let yAxis = g => g
      .attr("transform",`translate(28,-10)`)
      .call(d3.axisLeft(d3.scaleLinear().domain([100,0]).range([0,210])).ticks(10))

    let xAxis = g => g
      .attr("transform",`translate(40,200)`)
      .call(d3.axisBottom(d3.scaleLinear().range([0,674])).ticks(30))

    d3.select(`#${this.codename}`)
      .append("g")
      .call(yAxis)

    d3.select(`#${this.codename}`)
      .append("g")
      .call(xAxis)

    //console.log(this.X)
    //console.log(this.Y)
    //console.log(this.I)
    //console.log(this.mandata)
    //let xScale = d3.scaleLinear([100,0],[0,600])
    //let yScale = d3.scaleLinear([0,100],[0,200])

    const line = d3.line()
      .curve(d3.curveLinear)
      .x(i => i.order)
      .y(i => i.count)

      d3.select(`#${this.codename}`)
        .append("path")
        .attr("fill", "none")
        .attr("stroke", "orange")
        .attr("stroke-width", strokeWidth)
        .attr("stroke-linejoin", strokeLinejoin)
        //.attr("stroke-linecap", strokeLinecap)
        .attr("d", line(this.mandata))
        .attr("transform", "translate(30,-110)")
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
.newfont {
  font-family:'Varela Round';
}
.svgctn3 {
  filter:blur(3px) opacity(0.4);
}
</style>
