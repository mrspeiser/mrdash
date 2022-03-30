<template>
  <div 
    class="svgctn grid1-1"
    :style="gridStyles"
  >
    <h1 class="newfont">Total Traffic</h1>
    <slot></slot> 
  </div>
</template>

<script>

import * as d3 from "d3";

export default {
  name:'Histogram',
  props: {
    codename:{
      type:String,
      default:"HistoTasty"
    },
    id:{
      type: String,
      default:'histogram-svg'
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
      svgNode:null
    }
  },
  created(){
    let slicer = (x) => {
      let p1 = String(x).slice(0,11) 
      let p2 = String(x).slice(19,28) 
      return String(p1+p2);
    }
    let slicer2 = (x) => {
      //console.log(typeof(x))
      //console.log(String(x))
      //console.log(typeof(String(x)))
      let s = String(x).split(" ")
      let trueday = s[2] < 10 ? `${s[2].slice(1,2)}` : s[2];
      let news = `${s[0]}${s[1]}${trueday}MST${s[3]}`
      return news
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
    let setm = (x) => {
      return x.map((v,i) => {
        let xax = i*1.37
        return {...v,m:`${xax},700`}
      })
    }

    let setl = (x) => {
      console.log(Array.isArray(x))
      return x.map((v) => {
        let lc = 700-(v.count*18)
        //let xax = i+1
        //return {...v,l:`${xax},${lc}`}
        return {...v,l:`${lc}`}
      })
    }

    let totalDateRange = d3.timeDay.every(1).range(new Date(2019, 10, 20), new Date(2021, 11, 2))
    this.xDomain = totalDateRange.map(slicer2)

    this.mandata = this.arrayData
      .map(slicer)
      .map(stringFlattener)
      .reduce(grouper, [{}])
      .map(expander)
      .sort(compare)
      .map(setm)
      .map(setl)
    
    //console.log(totalDateRange[0]);
    //console.log(reducedRange[0])

    this.attrd = this.mandata[0].map(x => {
      return `M ${x.m} V ${x.l}`
    }).join(" ");

    this.finalData = this.xDomain.map((x,i) => {
      let o = this.mandata[0].find(y => y['name'] === x)
      if(!o){
        //console.log(o)
        //console.log(x)
        let xax = `${i*1.37},700`
        return {
          name:x,
          count:0,
          order:i,
          m:xax,
          l:"698"
        }
      } else {
        o.order = i;
        let xax = i*1.37;
        o.m=`${xax},700`;
        let lc = 700-(o.count*16.75)
        o.l = `${lc}`;
      }
      return o
    })
    console.log(this.xDomain.length)
    // console.log(this.mandata)
    this.maximum = this.finalData.reduce((acc,cv) => {
      if(cv.count > acc.count){
        return cv;
      }
      return acc;
    },{count:0})
    this.totalEmptyDays = this.finalData.reduce((acc,cv) => {
      if(cv.count === 0){
        acc+=1
      }
      return acc;
    },0)
    this.totalPDs = this.finalData.reduce((acc,cv) => {
      acc+=cv.count
      return acc;
    },0)
    //console.log(`max: ${maximum}`)
    //console.log(maximum)
    //console.log(totalEmptyDays)
    //console.log(Math.floor(212/713*100))
    //console.log(totalPDs)

  },
  mounted(){
    // create svg

    //console.log(this.finalData)
    //console.log(Array.isArray(this.finalData))

    let width = 1080;
    let height = 450;
    d3.select(".svgctn")
      .append("svg")
      .attr("id",this.codename)
      .attr("viewBox", [0, 0, width, height])


    let yAxis = g => g
      .attr("transform",`translate(28,-10)`)
      .call(d3.axisLeft(d3.scaleLinear().domain([24,0]).range([0,400])).ticks(10))

    let xAxis = g => g
      .attr("transform",`translate(40,400)`)
      .call(d3.axisBottom(
          d3.scaleTime()
            .domain([new Date(2019,10,20,0), new Date(2021,11,2)])
            .range([0,1017])
        )
        .ticks(d3.timeMonth,1).tickFormat(d3.timeFormat('%b-%y'))
      )
    //console.log(this.mandata[0])
    //console.log(d3.timeDay.every(1).range(new Date(2019, 10, 20), new Date(2021, 10, 2)))


    d3.select(`#${this.codename}`)
      .append("g")
      .call(yAxis)

    d3.select(`#${this.codename}`)
      .append("g")
      .call(xAxis)

    // create each line as path inside svg
    this.finalData.forEach(x => {
      let attrd = `M ${x.m} V ${x.l}`
      let color = x.count >= 9 ? "#00bf00" : "blue";
      if(x.count === 0) color = "red";
      d3.select(`#${this.codename}`)
        .insert("path")
        .attr("stroke",color)
        .attr("fill","none")
        .attr("d", attrd)
        .attr("transform", "translate(40,-310)")
        .on('mouseover',() => {console.log(x)})
        //.attr("style", "transform:translateY(-380px)")
    });

    //d3.select(`#${this.codename}`)
    //  .insert("path")
    //  .attr("stroke","navy")
    //  .attr("fill","none")
    //  .attr("d", this.attrd)
    //  .attr("style", "transform:translateY(-360px)")

  },
  methods:{
    getSvgNode(){
      //const svg = d3.create("svg")
      //return svg.node()
    } 
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
.svgctn {
  filter:blur(3px) opacity(0.4);
}
</style>
