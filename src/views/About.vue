<template>
  <div class="home" ref="relationMap">
     <div class="button" @click="addData">添加数据</div>
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
        nodeCircle:null,
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
      this.svg = svg;
      this.color = this.$d3.scaleOrdinal(this.$d3.schemeCategory10);

      var a = {id: "a"},
          b = {id: "b"},
          c = {id: "c"};
          this.b= b;
          this.nodes = [a, b, c];
          this.links = [];

      // this.simulation = this.$d3.forceSimulation(this.nodes)
      //     .force("charge", this.$d3.forceManyBody().strength(-1000))
      //     .force("link", this.$d3.forceLink(this.links).distance(200))
      //     .force("center", this.$d3.forceCenter(relationMap.offsetWidth/2, relationMap.offsetHeight/2))
      //     .alphaTarget(1)
      //     .on("tick", this.ticked);
      this.simulation = this.$d3.forceSimulation(this.nodes)
      // simulation.force(name,[force])函数，添加某种力
          .force("link", this.$d3.forceLink(this.links).id(d => {return d.id}).distance(200))
          // 万有引力
          .force("charge", this.$d3.forceManyBody().strength(this.chargeStrength))
          // this.$d3.forceCenter()用指定的x坐标和y坐标创建一个新的居中力。
          .force("center", this.$d3.forceCenter(relationMap.offsetWidth/2, relationMap.offsetHeight/2))
          // 碰撞作用力，为节点指定一个radius区域来防止节点重叠，设置碰撞力的强度，范围[0,1], 默认为0.7。设置迭代次数，默认为1，迭代次数越多最终的布局效果越好，但是计算复杂度更高
          .force("collide", this.$d3.forceCollide(this.collide).strength(0.2).iterations(5))
          // 在计时器的每一帧中，仿真的alpha系数会不断削减,当alpha到达一个系数时，仿真将会停止，也就是alpha的目标系数alphaTarget，该值区间为[0,1]. 默认为0，
          // 控制力学模拟衰减率，[0-1] ,设为0则不停止 ， 默认0.0228，直到0.001
          .alphaDecay(this.alphaDecay)
          // 监听事件 ，tick|end ，例如监听 tick 滴答事件
          .on("tick", ()=>this.ticked());
      this.link = svg.append("g").attr("stroke", "#000").attr("stroke-width", 1.5).selectAll(".link");
      this.node = svg.append("g").attr("stroke", "#fff").attr("stroke-width", 1.5).selectAll(".node");
        _this.links.push({source: a, target: b}); // Add a-b.
        _this.links.push({source: b, target: c}); // Add b-c.
        _this.links.push({source: c, target: a}); // Add c-a.
      this.restart();


      // this.$d3.interval(function() {
      //   _this.nodes.pop(); // Remove c.
      //   _this.links.pop(); // Remove c-a.
      //   _this.links.pop(); // Remove b-c.
      //   _this.restart();
      // }, 2000, this.$d3.now());

      // this.$d3.interval(function() {
      //   _this.nodes.push(c); // Re-add c.
      //   _this.links.push({source: b, target: c}); // Re-add b-c.
      //   _this.links.push({source: c, target: a}); // Re-add c-a.
      //   _this.restart();
      // }, 2000, this.$d3.now() + 1000);
    })
  },
  methods:{
    ticked(){
      this.nodeCircle.attr("cx", function(d) { return d.x; }).attr("cy", function(d) { return d.y; })
      this.link.attr("x1", function(d) { return d.source.x; })
          .attr("y1", function(d) { return d.source.y; })
          .attr("x2", function(d) { return d.target.x; })
          .attr("y2", function(d) { return d.target.y; });
    },
    addData(){
      var _this = this;
      var data = { id: ('id_'+ Math.random()).replace('0.', '') };
      this.nodes.push(data); // Re-add c.
      this.links.push({source: this.b, target: data}); 

      
      this.nodeG= this.nodeG.data(this.nodes);
      this.nodeG.exit().remove();
      let _nodeG = this.nodeG.enter().append("g").attr("class","nodeG");
      this.nodeCircle = _nodeG.append("circle").attr("class","nodeCircle")
            .attr("fill", function(d) { return _this.color(d.id); })
            .call(function(d) { d.transition().attr("r", 8); }).merge(this.nodeCircle);
      let _link= this.link.data(this.links, function(d) { return d.source.id + "-" + d.target.id; });

      // Keep the exiting links connected to the moving remaining nodes.
      this.link = _link.exit().transition()
          .attr("stroke-opacity", 0)
          .attrTween("x1", function(d) { return function() { return d.source.x; }; })
          .attrTween("x2", function(d) { return function() { return d.target.x; }; })
          .attrTween("y1", function(d) { return function() { return d.source.y; }; })
          .attrTween("y2", function(d) { return function() { return d.target.y; }; })
          .remove();

      this.link = _link.enter().append("line")
        .call(function(d) { d.transition().attr("stroke-opacity", 1); })
        .merge(_link);     
      this.simulation.nodes(this.nodes);
      this.simulation.force("link").links(this.links);
      this.simulation.alpha(1).restart()       
    },
    restart(){
        let _this = this;
        let relationMap = this.$refs.relationMap;

        // Apply the general update pattern to the nodes.
        this.node = this.node.data(this.nodes, function(d) { return d.id;});

        this.node.exit().transition().attr("r", 0).remove();

        this.nodeG = this.node.enter().append('g').attr("class","nodeG").merge(this.node);
        this.nodeCircle = this.nodeG.append("circle").attr("class","nodeCircle")
            .attr("fill", function(d) { return _this.color(d.id); })
            .call(function(d) { d.transition().attr("r", 8); })
            
        
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
        this.simulation.alpha(1).restart();
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
.button{
    position: absolute;
    z-index: 100;;
    padding:12px;
    border: 1px solid #4099EF;
    background: #4099EF;
    font-size: 12px;
    line-height: 12px;
    cursor: pointer;
}
</style>