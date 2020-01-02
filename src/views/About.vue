<template>
  <div class="home" ref="relationMap">
    <svg id="svg" width="960" height="600"></svg>
  </div>
</template>

<script>
// @ is an alias to /src
/* eslint-disable*/ 

export default {
  name: 'home',
  components: {
  },
  data(){
    return {
      simulation:null,
      config:{
        width: 375, // 总画布svg的宽
        height: 610,
        nodes: [],
        links: [],
        graph:null,
        link:null,
        node:null,
        isHighLight: false ,    //是否启动 鼠标 hover 到节点上高亮与节点有关的节点，其他无关节点透明的功能
        isScale: true,      //是否启用缩放平移zoom功能
        scaleExtent: [0.5, 1.5],  //缩放的比例尺
        chargeStrength:-300,    //万有引力
        collide:60,        //碰撞力的大小 （节点之间的间距）
        nodeWidth: 120,     // 每个node节点所占的宽度，正方形
        margin: 20,     // node节点距离父亲div的margin
        alphaDecay:0.0228,  //控制力学模拟衰减率
        r: 30,      // 头像的半径 [30 - 45]
        relFontSize: 12 ,   //关系文字字体大小
        linkSrc: 30, // 划线时候的弧度
        linkColor:'#bad4ed',    //链接线默认的颜色
        strokeColor: '#7ecef4', // 头像外围包裹的颜色
        strokeWidth: 3, // 头像外围包裹的宽度        
      }
    }
  },
  mounted(){
    var svg = this.$d3.select("svg"),
    width = +svg.attr("width"),
    height = +svg.attr("height");
    let _this = this;
    debugger
    console.log('d3',this.$d3);
    // this.simulation = this.$d3.forceSimulation()
    //     .force("link", this.$d3.forceLink().id((d)=> { return d.id; }))
    //     .force("charge", this.$d3.forceManyBody())
    //     .force("center", this.$d3.forceCenter(width / 2, height / 2));
    this.$d3.json("/data.json").then((graph)=>{
      this.graph = graph;
      this.link = svg.append("g")
        .attr("class", "links")
        .selectAll("line")
        .data(graph.links)
        .enter().append("line");
      
      this.node = svg.append("g")
        .attr("class", "nodes")
        .selectAll("circle")
        .data(graph.nodes)
        .enter().append("circle")
        .attr("r", 12.5)
        .call(_this.$d3.drag()
              .on("start", (d)=>{_this.dragstarted(d)})
              .on("drag", (d)=>{_this.dragged(d)})
              .on("end", (d)=>{_this.dragended(d)}));
      this.initForce();
    });
  },
  methods:{
    initForce(){
      let _parentNode = this.$refs.relationMap;

      this.simulation = this.$d3.forceSimulation()
      // simulation.force(name,[force])函数，添加某种力
          .force("link", this.$d3.forceLink().id(d => {return d.id}))
          // 万有引力
          .force("charge", this.$d3.forceManyBody().strength(this.config.chargeStrength))
          // this.$d3.forceCenter()用指定的x坐标和y坐标创建一个新的居中力。
          .force("center", this.$d3.forceCenter(_parentNode.offsetWidth/2, _parentNode.offsetHeight/2))
          // 碰撞作用力，为节点指定一个radius区域来防止节点重叠，设置碰撞力的强度，范围[0,1], 默认为0.7。设置迭代次数，默认为1，迭代次数越多最终的布局效果越好，但是计算复杂度更高
          .force("collide", this.$d3.forceCollide(this.config.collide).strength(0.2).iterations(5))
          // 在计时器的每一帧中，仿真的alpha系数会不断削减,当alpha到达一个系数时，仿真将会停止，也就是alpha的目标系数alphaTarget，该值区间为[0,1]. 默认为0，
          // 控制力学模拟衰减率，[0-1] ,设为0则不停止 ， 默认0.0228，直到0.001
          .alphaDecay(this.config.alphaDecay)
          // 监听事件 ，tick|end ，例如监听 tick 滴答事件        
      this.simulation.nodes(this.graph.nodes).on("tick", ()=>{
        this.ticked(this.link,this.node)
      });
      
      this.simulation.force("link").links(this.graph.links);
    },
    ticked(link,node){
        link
            .attr("x1", function(d) { return d.source.x; })
            .attr("y1", function(d) { return d.source.y; })
            .attr("x2", function(d) { return d.target.x; })
            .attr("y2", function(d) { return d.target.y; });
        node
            .attr("cx", function(d) { return d.x; })
            .attr("cy", function(d) { return d.y; });
    },
    dragended(d){
      if (!this.$d3.event.active) this.simulation.alphaTarget(0);
      d.fx = null;
      d.fy = null;
    },
    dragged(d){
      d.fx = this.$d3.event.x;
      d.fy = this.$d3.event.y;
    },
    dragstarted(d){
      if (!this.$d3.event.active) this.simulation.alphaTarget(0.3).restart();
      d.fx = d.x;
      d.fy = d.y;
    }
  }
}
</script>
<style lang="less" scoped>
#svg{

  /deep/.links line {
    stroke: #aaa;
  }
  
  /deep/.nodes circle {
    pointer-events: all;
    stroke: none;
    stroke-width: 40px;
  }
}
</style>