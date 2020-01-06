<template>
  <div class="home" ref="relationMap">
    <svg id="svg"></svg>
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
        width: 375, // 总画布svg的宽
        height: 610,
        nodes: [],
        circle:null,
        color:null,
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
  },
  mounted(){
    this.$nextTick(()=>{
      let _this = this;
      let relationMap = this.$refs.relationMap;
      var svg = this.$d3.select("#svg").attr('width',relationMap.offsetWidth).attr("height",relationMap.offsetHeight);
      this.color = this.$d3.scaleOrdinal(this.$d3.schemeCategory10);

      var a = {id: "a"},
          b = {id: "b"},
          c = {id: "c"},
          d = {id: "d"};
          this.nodes = [a, b, c];
          this.links = [{source: b, target: a},{source: c, target: a}];

      this.simulation = this.$d3.forceSimulation(this.nodes)
          .force("charge", this.$d3.forceManyBody().strength(-1000))
          .force("link", this.$d3.forceLink(this.links).distance(200))
          .force("x", this.$d3.forceX())
          .force("y", this.$d3.forceY())
          .alphaTarget(1)
          .on("tick", this.ticked);

      var g = svg.append("g").attr("transform", "translate(" + relationMap.offsetWidth / 2 + "," + relationMap.offsetHeight / 2 + ")");
      this.link = g.append("g").attr("stroke", "#000").attr("stroke-width", 1.5).selectAll(".link");
      this.node = g.append("g").attr("stroke", "#fff").attr("stroke-width", 1.5).selectAll(".node");
      this.restart();


      // this.$d3.interval(function() {
      //   _this.nodes.pop(); // Remove c.
      //   _this.links.pop(); // Remove c-a.
      //   _this.links.pop(); // Remove b-c.
      //   _this.restart();
      // }, 2000, this.$d3.now());

      this.$d3.interval(function() {
        _this.$d3.timeout(function() {
          _this.nodes.push(d)
          _this.links.push({source: d, target: a}); // Add a-b.
          _this.restart();
        }, 1000);
        _this.$d3.timeout(function() {
          _this.nodes.pop()
          _this.links.pop(); // Add a-b.
          _this.restart();
        }, 2000);        
      }, 2000, this.$d3.now() + 1000);
    })
  },
  methods:{
    ticked(){
      this.circle.attr("cx", function(d) { return d.x; }).attr("cy", function(d) { return d.y; })
      // this.node.attr("cx", function(d) { return d.x; }).attr("cy", function(d) { return d.y; })

      this.link.attr("x1", function(d) { return d.source.x; })
          .attr("y1", function(d) { return d.source.y; })
          .attr("x2", function(d) { return d.target.x; })
          .attr("y2", function(d) { return d.target.y; });
    },
    restart(){
        let _this = this;
        let relationMap = this.$refs.relationMap;

        // Apply the general update pattern to the nodes.
        this.node = this.node.data(this.nodes, function(d) { return d.id;});
        this.node.exit().transition().attr("r", 0).remove();
        this.nodgG = this.node.enter().append("g").attr('class','nodeG');
        this.circle = this.nodgG.append("circle")
            .attr("fill", function(d) { return _this.color(d.id); })
            .call(function(d) { d.transition().attr("r", 8); }).merge(this.node)
            
        // Apply the general update pattern to the links.
        this.link = this.link.data(this.links, function(d) { return d.source.id + "-" + d.target.id; });

        // Keep the exiting links connected to the moving remaining nodes.
        this.link.exit().transition()
            .attr("stroke-opacity", 0)
            .attrTween("x1", function(d) { return function() { return d.source.x; }; })
            .attrTween("x2", function(d) { return function() { return d.target.x; }; })
            .attrTween("y1", function(d) { return function() { return d.source.y; }; })
            .attrTween("y2", function(d) { return function() { return d.target.y; }; })
            .remove();
        this.link = this.link.enter().append("line")
          .call(function(d) { d.transition().attr("stroke-opacity", 1); })
          .merge(this.link);

        // Update and restart the simulation.
        this.simulation.nodes(this.nodes);
        this.simulation.force("link").links(this.links);
        // this.simulation.alpha(1).restart();
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
.home{
  width: 100%;
  height: 100%;
}
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