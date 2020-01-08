<template>
  <div class="relationMap" id="relationMap" :class="isFullScreen ? 'fullscreen' : ''" ref="relationMap">
    <div class="controlBar">
        <section class="part part1">
          <img class="icon" src="/img/expand.png" alt="">
          <el-checkbox-group @change="selValChange" class="popControl" v-model="checkList">
            <el-checkbox :checked="true" :label="item" v-for="(item,index) in typeArray" :key="index"></el-checkbox>
          </el-checkbox-group>
        </section>
        <section class="part part2">
          <img class="icon" src="/img/shrink.png" alt="">
          <div class="inputW">
            <el-input type="text" v-model="searchVal" @input="searchNode" placeholder="请输入关键字"></el-input>
          </div>
        </section>
        <section class="part part3">
          <img class="icon" src="/img/expand.png" @click="toggleFullScreen" alt="">
        </section>
    </div>
  </div>
</template>

<script>
// @ is an alias to /src
/* eslint-disable*/ 
export default {
  name: 'relationMap',
  components: {
  },
  props:{
    cId:{
      default:'',
      type:String
    }
  },
  data(){
    return {
      searchVal:'',//搜索框的值
      isFullScreen:true,//是否全屏..
      checkList:[],//选中的筛选项.....
      edgesNodes:null,
      nodeGNodes:null,
      simulation:null,
      SVG:null,
      linksGwrap:null,
      nodesGwrap:null,
      relMap_g:null,
      defs:null,
      marker:null,
      rect_g:null,
      rects:null,
      edges:null,
      texts:null,
      links:null,
      nodeCircles:null,
      patterns:null,
      nodeCheckbox:null,
      shrinkExtBtn:null,
      nodeG:null,
      dependsNode:[],// 需要高亮的node
      dependsLinkAndText:[],// 需要高亮的link
      R:30,//画线时候的影响弧度
      typeArray:[],//节点类型....
      config:{
        width: 375, // 总画布svg的宽
        height: 610,
        nodes: [],
        links: [],
        isHighLight: true ,    //是否启动 鼠标 hover 到节点上高亮与节点有关的节点，其他无关节点透明的功能
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
    this.$d3.json("/miserables.json").then((graph)=>{
      graph.nodes.forEach((val)=>{
        val.checked = false;//是否选中
        val.expanded = false;//是否展开
        val.active = false;//是否处于焦点
      })
      // 填充筛选条件
      graph.links.forEach((val)=>{
        this.typeArray.push(val.relation);
      })
      // 去重......
      this.typeArray = new Set(this.typeArray);
      this.config.nodes = graph.nodes;
      this.config.links = graph.links;
      this.init(true);
    });
  },
  methods:{
    initForce(){
      let _this = this;
      let _parentNode = this.$refs.relationMap;
      this.simulation = this.$d3.forceSimulation(this.config.nodes)
      // simulation.force(name,[force])函数，添加某种力
          .force("link", this.$d3.forceLink(this.config.links).id(d => {return d.id}).distance(200))
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
          .on("tick", ()=>this.ticked());
    },
    init(first=false){
      let _this = this;
      let _parentNode = this.$refs.relationMap;
      // 2.创建svg标签
      if( first ){
        this.SVG = this.$d3.select("#relationMap").append("svg")
            .attr("id", "mainSvg")
            .attr("width", _parentNode.offsetWidth)
            .attr("height", _parentNode.offsetHeight)
            // .transition().duration(750).call(this.$d3.zoom().transform, this.$d3.zoomIdentity);
            .call( _this.$d3.zoom().scaleExtent(_this.config.scaleExtent).on("zoom", ()=> {
                if(_this.config.isScale){
                    _this.relMap_g.attr("transform", _this.$d3.event.transform);
                }
            })).on("dblclick.zoom", null); 
            
        // 3.defs  <defs>标签的内容不会显示，只有调用的时候才显示
        this.defs=this.SVG.append('defs');
        // 3.1 添加箭头
        this.marker=this.defs
            .append("marker")
            .attr('id', "marker")
            .attr("markerWidth", 10)    //marker视窗的宽
            .attr("markerHeight", 10)   //marker视窗的高
            .attr("refX", this.config.r + 3*this.config.strokeWidth)            //refX和refY，指的是图形元素和marker连接的位置坐标
            .attr("refY", 4)
            .attr("orient", "auto")     //orient="auto"设置箭头的方向为自动适应线条的方向
            .attr("markerUnits", "userSpaceOnUse")  //marker是否进行缩放 ,默认值是strokeWidth,会缩放
            .append("path")
            .attr("d", "M 0 0 8 4 0 8Z")    //箭头的路径 从 （0,0） 到 （8,4） 到（0,8）
            .attr("fill", "steelblue");
        // 4.放关系图的容器
        this.relMap_g = this.SVG.append("g")
            .attr("id", "relMap_g")
            .attr("width", _parentNode.offsetWidth)
            .attr("height", _parentNode.offsetHeight);              

        this.linksGwrap = this.relMap_g.append("g").attr("id","linksGwrap");
        this.nodesGwrap = this.relMap_g.append("g").attr("id","nodesGwrap");
      }

      // 3.2 添加多个头像图片的 <pattern>
      this.patterns = this.defs.selectAll("pattern.patternclass").data(this.config.nodes,function(d){ return d.id });
      this.patterns.exit().remove();
      this.patterns = this.patterns.enter()
          .append("pattern")
          .attr("class", "patternclass")
          .attr("id", function (d, index) {
            return 'avatar' + d.id;
          })
          // 两个取值userSpaceOnUse  objectBoundingBox
          .attr('patternUnits', 'objectBoundingBox')
          // <pattern>，x、y值的改变决定图案的位置，宽度、高度默认为pattern图案占填充图形的百分比。
          .attr("x", "0")
          .attr("y", "0")
          .attr("width", "1")
          .attr("height", "1")

      // 3.3 向<defs> - <pattern>添加 头像
      this.patterns.append("image")
          .attr("class", "circle")
          .attr("xlink:href", function (d) {
              return d.avatar || "https://static.linkeddb.com/m/images/none.jpg"; // 修改节点头像
          })
          .attr("src", function (d) {
              return d.avatar || "https://static.linkeddb.com/m/images/none.jpg"; // 修改节点头像
          })
          .attr("height", this.config.r*2)
          .attr("width", this.config.r*2)
          .attr("preserveAspectRatio", "xMidYMin slice");

      // 3.4 名字
      this.patterns.append("rect").attr("x", "0").attr("y", 4/3*this.config.r).attr("width", 2*this.config.r).attr("height", 2/3*this.config.r).attr("fill", "black").attr("opacity", "0.5");
      this.patterns.append("text").attr("class", "nodetext")
          .attr("x", this.config.r).attr("y", (5/3*this.config.r))
          .attr('text-anchor', 'middle')
          .attr("fill", "#fff")
          .style("font-size", this.config.r/3)
          .text(function (d) {
              return d.name;
          }).merge(this.patterns); 
          
      // 5.关系图添加线
      this.edgesNodes = this.$d3.select("g#linksGwrap").selectAll("g.edge").data(this.config.links,function(d){return `${d.source.id}-${d.target.id}`});
      console.log("this.edgesNodes",this.edgesNodes);

      this.edgesNodes.exit().transition().style("opacity", 0).remove();
      // 5.1  每条线是个容器，有线 和一个装文字的容器
      console.log("this.edgesNodes",this.edgesNodes);
      
      let _edges = this.edgesNodes
        .enter()
        .append("g")
        .attr("class","edge").attr("id",(d)=>{ return `${_this.cId}linkG${d.target}${d.source}` })
        .on('mouseover', function () {
            _this.$d3.select(this).selectAll('path.links').attr('stroke-width', 4);
        })
        .on('mouseout', function () {
            _this.$d3.select(this).selectAll('path.links').attr('stroke-width', 1);
        }).on('click',(d)=>{

        }).attr('fill', function (d) {
            var str = '#bad4ed';
            if (d.color) {
                str = "#" + d.color;
            }
            return str;
        })
      console.log("this.edges2",_edges);

      // 5.2 添加线
      let _links = _edges.append("path").attr("class","links")
        .attr("d", d => {
            return `M${this.config.linkSrc},0L${this.getDis(d.source,d.target)},0`
        })
        .style("marker-end", "url(#marker)")
        // .attr("refX",this.config.r)
        .attr('stroke', (d)=> {
            var str=d.color ? "#"+d.color : this.config.linkColor ;
            return str;
        });

      // 5.3 添加关系文字的容器
      let _rect_g = _edges.append("g").attr("class", "rect_g");

      // 5.4 添加rect
      let _rects = _rect_g.append("rect")
        .attr("x", 40)
        .attr("y", -10)
        .attr("width", 40)
        .attr("height", 20)
        .attr("fill", "white")
        .attr('stroke',  (d)=> {
            var str=d.color ? "#"+d.color : this.config.linkColor ;
            return str;
        })

      // 5.5 文本标签  坐标（x,y）代表 文本的左下角的点
      let _texts = _rect_g.append("text")
        .attr("x", 40)
        .attr("y", 5)
        .attr("text-anchor", "middle")  // <text>文本中轴对齐方式居中  start | middle | end
        .style("font-size",12).text(d=>{return d.relation}); 

      this.nodeGNodes = this.$d3.select("g#nodesGwrap").selectAll("g.nodeG").data(this.config.nodes,function(d){ return d.id });
      this.nodeGNodes.exit().remove();
      let _nodeG = this.nodeGNodes
        .enter().append("g").attr('class',"nodeG").attr("id",(d)=>{ return `${_this.cId}nodeG${d.id}`})
        .on('mouseover',(d)=>{
          // _this.showTootip(this,d);
        }).on("mouseout",(d)=>{
          // _this.showTootip(this);
        });
      // 6.关系图添加用于显示头像的节点
      let _nodeCircles =  _nodeG.append("circle")
          .attr("class","nodeCircle")
          .style("cursor", "pointer")
          .attr("fill", function (d) {
              return ("url(#avatar" + d.id + ")");
          })
          .attr("stroke", "#ccf1fc")
          .attr("stroke-width", this.config.strokeWidth)
          .attr("r", this.config.r)
          .on('mouseover', function(d){
              if(_this.config.isHighLight){
                  _this.highlightObject(d);
              }
          })
          .on('mouseout', function(d){
              if(_this.config.isHighLight){
                  _this.highlightObject(null);
              }
          })
          // 应用 自定义的 拖拽事件
          .call(this.$d3.drag()
          .on('start', function (d) {
              _this.$d3.event.sourceEvent.stopPropagation();
              // restart()方法重新启动模拟器的内部计时器并返回模拟器。
              // 与simulation.alphaTarget或simulation.alpha一起使用时，此方法可用于在交互
              // 过程中进行“重新加热”模拟，例如在拖动节点时，在simulation.stop暂停之后恢复模拟。
              // 当前alpha值为0，需设置alphaTarget让节点动起来
              if (!_this.$d3.event.active) _this.simulation.alphaTarget(0.3).restart();
              d.fx = d.x;
              d.fy = d.y;
          }).on('drag', function (d) {
              // d.fx属性- 节点的固定x位置
              // 在每次tick结束时，d.x被重置为d.fx ，并将节点 d.vx设置为零
              // 要取消节点，请将节点 .fx和节点 .fy设置为空，或删除这些属性。
              d.fx = _this.$d3.event.x;
              d.fy = _this.$d3.event.y;
          }).on('end', function (d) {
              // 让alpha目标值值恢复为默认值0,停止力模型
              if (!_this.$d3.event.active) _this.simulation.alphaTarget(0);
              d.fx = null;
              d.fy = null;
          }))
      // 6.1 关系图添加用于显示checkbox的按钮
      let _nodeCheckbox = _nodeG.append('image').attr("class","nodeCheckbox")
          .attr("id", (d)=>{ return `${this.cId}nodeCheckbox${d.id}`})
          .attr("xlink:href",(d)=>{ return '/img/expand.png' })
          .style("width",this.config.r * 0.5)
          .style("height",this.config.r * 0.5)
          .on("click",function(d,i){
              _this.setCheckboxStatus(d,i);
          })
      // 6.2 关系图添加用于显示扩展收缩的按钮
      let _shrinkExtBtn = _nodeG.append('image').attr("class","nodeShrinkExtBtn")
          .attr("id", (d)=>{ return `${this.cId}nodeShrinkExtBtn${d.id}`})
          .attr("xlink:href",(d)=>{ return '/img/expand.png' })
          .style("width",this.config.r * 0.5)
          .style("height",this.config.r * 0.5)
          .on("click",function(d,i){
            // 固定节点
              d.fx = d.x;
              d.fy = d.y;
            // 设置节点状态
              _this.setnodeShrinkExtBtnStatus(d,i);
          })
      if( first ){
        // 连线
        this.edges = _edges;
        this.links = _links;
        this.rect_g = _rect_g;
        this.texts = _texts;
        this.rects = _rects;
        // 点
        this.nodeCircles = _nodeCircles;
        this.nodeCheckbox = _nodeCheckbox;
        this.shrinkExtBtn = _shrinkExtBtn;
        this.nodeG = _nodeG;
        this.initForce();
      }else{
        this.edges = _edges.merge(this.edges);
        this.links = _links.merge(this.links);
        this.rect_g = _rect_g.merge(this.rect_g);
        this.texts = _texts.merge(this.texts);
        this.rects = _rects.merge(this.rects);

        this.nodeG = _nodeG.merge(this.nodeG);
        this.nodeCircles = _nodeCircles.merge(this.nodeCircles);
        this.nodeCheckbox = _nodeCheckbox.merge(this.nodeCheckbox);
        this.shrinkExtBtn = _shrinkExtBtn.merge(this.shrinkExtBtn);
        this.simulation.nodes(this.config.nodes);
        this.simulation.force("link").links(this.config.links);
        this.simulation.alpha(1).restart();        
      }
    },
    // 更新checkbox选中状态....
    setCheckboxStatus(d,i){
      d.checked = !d.checked;
      this.$d3.select(`#${this.cId}nodeCheckbox${d.id}`).attr("xlink:href",(d)=>{ 
        if( d.checked ){
          return '/img/shrink.png'
        }else{
          return '/img/expand.png'
        }
      })
    },
    // 更新扩展按钮选中状态....
    setnodeShrinkExtBtnStatus(d,i){
      d.expanded = !d.expanded;
      this.$emit("nodeShrinkExtBtn",d,i);
      this.$d3.select(`#${this.cId}nodeShrinkExtBtn${d.id}`).attr("xlink:href",(d)=>{ 
        if( d.expanded ){
          return '/img/shrink.png'
        }else{
          return '/img/expand.png'
        }
      })
    },    
    showTootip(me,d=null){
        // 展示方式2 ：浮窗展示
        event = this.$d3.event || window.event;
        if(!d){
            this.$d3.selectAll('.tooltip').remove();
            return;
        }
        var pageX = event.pageX ? event.pageX : (event.clientX + (document.body.scrollLeft || document.documentElement.scrollLeft));
        var pageY = event.pageY ? event.pageY : (event.clientY + (document.body.scrollTop || document.documentElement.scrollTop));
        //阻止事件冒泡  阻止事件默认行为
        event.stopPropagation ? (event.stopPropagation()) :  (event.cancelBubble = true);
        event.preventDefault ? (event.preventDefault()) : (event.returnValue = false);
        var _html=`<a href="${d.url}" class="external linkname" data-no-cache="true">${d.name}</a>`;
        if(d.exdata){
            _html+=`&nbsp;&nbsp;(<a class="external linkname" data-no-cache="true" href="${d.exdata.url}" >${d.exdata.name}</a> 饰)`;
        }
        this.$d3.selectAll('.tooltip').remove();
        var _div=this.$d3.select('#relationMap').append('div').attr('class','tooltip').html(_html);

        var _width=parseInt(_div.style('width'));

        //判断浮窗的左上角坐标 ，如果太靠右侧/底部 边缘 ，浮窗位置平移
        if(document.body.clientWidth - pageX < _width){
            pageX=document.body.clientWidth - _width -85;
        }
        if(document.body.clientHeight - pageY < 50){
            pageY=document.body.clientHeight-70;
        }
        _div.style('top',pageY+'px').style('left',pageX+'px');
    },
    // 7.动态改变 线和节点的 位置
    // tick心跳函数    
    ticked(){
        let _this = this;
        // 7.1 修改每条容器edge的位置
        this.edges.attr("transform", function (d) {
            return _this.getTransform(d.source, d.target, _this.getDis(d.source, d.target))
        });
        // 7.2 修改每条线link位置
        this.links.attr("d", d => {
            return "M" + this.config.linkSrc + "," + 0 + " L" + this.getDis(d.source, d.target) + ",0";
        })
    
        // 7.3 修改线中关系文字text的位置 及 文字的反正
        this.texts.attr("x", function (d) {
                // 7.3.1 根据字的长度来更新兄弟元素 rect 的宽度
                var bbox = _this.$d3.select(this).node().getBBox();
                var width = bbox.width;
                // 7.3.2 更新 text 的位置
                return _this.getDis(d.source, d.target) / 2
            }).attr("transform", function (d) {
                // 7.3.3 更新文本反正
                if (d.target.x < d.source.x) {
                    var x = _this.getDis(d.source, d.target) / 2;
                    return 'rotate(180 ' + x + ' ' + 0 + ')';
                } else {
                    return 'rotate(0)';
                }
            });

        // 7.4 修改线中装文本矩形rect的位置
        this.rects.attr("x", function(d){
          return _this.getDis(d.source, d.target) / 2  - _this.$d3.select(this).attr('width')/2
        })    // x 坐标为两点中心距离减去自身长度一半

        // 5.修改节点的位置
        this.nodeCircles.attr("cx", (d) => { return d.x; }).attr("cy", (d) => {
          return d.y; 
        })       
        // 5.修改checkbox的位置
        this.nodeCheckbox.attr("x", function(d){ return d.x - _this.config.r - this.getBBox().width ; }).attr("y", function(d) { return d.y - this.getBBox().height; })       
        // 5.修改checkbox的位置
        this.shrinkExtBtn.attr("x", function(d){ return d.x + _this.config.r/2 + this.getBBox().width ; }).attr("y", function(d) { return d.y - this.getBBox().height; })          
    },
    selValChange(val){
      let _targetArr = [];
      let _this = this;
      this.$d3.select("#linksGwrap").selectAll(".edge").each(function (d) {
        if( val.includes(d.relation) ){
          _this.$d3.select(this).attr("display","inline");
          _this.$d3.select(`#${_this.cId}nodeG${d.target.id}`).attr("display","inline");
        }else{
          _this.$d3.select(this).attr("display","none");
          _this.$d3.select(`#${_this.cId}nodeG${d.target.id}`).attr("display","none");
        }
      })
      // this.nodeG.filter(function(d){
      //   return (d.name.indexOf(val) != -1);
      // }).transition().style('opacity', 1);
    },
    // 搜索节点.....
    searchNode(val=''){
      let _this = this;
      if( val == '' ){
        this.nodeG.transition().style('opacity', 1);
        return;
      }
      this.nodeG.filter(function (d) {
        return (d.name.indexOf(val) == -1);
      }).transition().style('opacity', 0.5);
      this.nodeG.filter(function(d){
        return (d.name.indexOf(val) != -1);
      }).transition().style('opacity', 1);
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
    },
    // 添加节点....
    addAndDeleteNodes(_exData,d,i){
      // 设置焦点节点
      this.config.nodes.forEach((item)=>{
        if( item.id == d.id ){
          item.active = !item.active;
        }else{
          item.active = false;
        }
      })
      if( d.expanded ){//展开
        this.config.links = [...this.config.links,..._exData.links];
        this.config.nodes = [...this.config.nodes,..._exData.nodes];  
      }else{//收起..
        _exData.nodes.forEach((val)=>{
          let _index = this.config.nodes.findIndex((item)=>{ return item.id == val.id });
          if( _index != -1 ){
            this.config.nodes.splice(_index,1);
          }
        })
        _exData.links.forEach((val)=>{
          let _index = this.config.links.findIndex((item)=>{ return (item.target == val.target && item.source == val.source ) });
          if( _index != -1 ){
            this.config.links.splice(_index,1);
          }
        })        
      }
      this.init();
    },
    highlightObject(obj=null){// 高亮和取消高亮
        let _this = this;
        if (obj) {
            // 先清空搜索的对象....
            this.searchNode();

            var objIndex = obj.index;
            this.dependsNode = this.dependsNode.concat([objIndex]);
            this.dependsLinkAndText = this.dependsLinkAndText.concat([objIndex]);
            this.config.links.forEach(function (lkItem) {
              if (objIndex == lkItem['source']['index']) {
                _this.dependsNode = _this.dependsNode.concat([lkItem.target.index]);
                } else if (objIndex == lkItem['target']['index']) {
                  _this.dependsNode = _this.dependsNode.concat([lkItem.source.index]);
                }
            });
            // 隐藏节点
            this.SVG.selectAll('.nodeG').filter(function (d) {
                return (_this.dependsNode.indexOf(d.index) == -1);
            }).transition().style('opacity', 0.1);
            // 隐藏线
            this.SVG.selectAll('.edge').filter(function (d) {
                // return true;
                return ((_this.dependsLinkAndText.indexOf(d.source.index) == -1) && (_this.dependsLinkAndText.indexOf(d.target.index) == -1))
            }).transition().style('opacity', 0.1);
        } else {
            // 取消高亮
            // 恢复隐藏的线
            this.SVG.selectAll('.nodeG').filter(function () {
                return true;
            }).transition().style('opacity', 1);
            // 恢复隐藏的线
            this.SVG.selectAll('.edge').filter(function (d) {
                // return true;
                return ((_this.dependsLinkAndText.indexOf(d.source.index) == -1) && (_this.dependsLinkAndText.indexOf(d.target.index) == -1))
            }).transition().style('opacity', 1);
            this.dependsNode = [],
            this.dependsLinkAndText = [];
            // 若果搜索框有值则执行搜索....
            this.searchNode( this.searchVal );
        }
    },
    /**
     *  关于缩放平移踩的大坑
     *  1. 改变需要缩放平移对象的transform来实现 ，本例子是 this.relMap_g
     *  2. 但是zoom 事件的监听应该放到整个的svg上 ,本例子是 this.SVG
     *  3. 这样 监听出来的 d3.event.transform 的 transform 数值才是正确的
     *  4. 直接把zoom 事件监听 this.relMap_g 得到的 transform 数值会爆炸大，移动非常不平顺，瞬移消失在视野
     *
     *  d3.event.transform{
     *      k:1    // 当前比例尺
     *      x:0    // x 方向移动的大小
     *      y:0    // y 方向移动的大小
     *  }
     * */
    zoom(){
      let _this = this;
      this.$d3.zoom()
        .scaleExtent(this.config.scaleExtent)
        .on("zoom", ()=> {
            if(_this.config.isScale){
                _this.relMap_g.attr("transform", _this.$d3.event.transform);
            }
        });
    },   
    getDis(s, t){// 求两点间的距离
      if( typeof(s)=="object"&&typeof(t)=='object' ){
        return Math.sqrt((s.x - t.x) * (s.x - t.x) + (s.y - t.y) * (s.y - t.y));
      }else{
        return 0;
      }
    },
    /**
     * d3 拖拽事件drag event 参数
     *   target- 相关的拖动行为。
     *   type - 字符串“start”, “drag” or “end”;  请参阅drag .on。
     *   subject- 拖动主题，由drag .subject定义。
     *   x- 主题的新x坐标; 请参阅drag .container。
     *   y- 主题的新y坐标; 请参阅drag .container。
     *   dx- 自上次拖动事件以来x坐标的变化。
     *   dy- 自上次拖动事件以来y坐标的变化。
     *   identifier- 字符串“鼠标”或数字触摸标识符。
     *   active - 当前活动拖动手势的数量（在开始和结束时，不包括这一个）。
     *   sourceEvent - 基础输入事件，如mousemove或touchmove。
     * */
    // d3.drag() 创建一个新的拖动行为。返回的行为拖动既是对象又是函数，通常通过选择 .call应用于选定的元素。
    // start - 新指针变为活动状态后（在mousedown或touchstart上）
    // drag - 活动指针移动后（在mousemove或touchmove上）。
    // end - 活动指针变为非活动状态后（在mouseup，touchend或touchcancel上）。
    drag(){
        let _this = this;
        return this.$d3.drag()
        .on('start', function (d) {
            _this.$d3.event.sourceEvent.stopPropagation();
            // restart()方法重新启动模拟器的内部计时器并返回模拟器。
            // 与simulation.alphaTarget或simulation.alpha一起使用时，此方法可用于在交互
            // 过程中进行“重新加热”模拟，例如在拖动节点时，在simulation.stop暂停之后恢复模拟。
            // 当前alpha值为0，需设置alphaTarget让节点动起来
            if (!_this.$d3.event.active) _this.simulation.alphaTarget(0.3).restart();
            d.fx = d.x;
            d.fy = d.y;
        })
        .on('drag', function (d) {
            // d.fx属性- 节点的固定x位置
            // 在每次tick结束时，d.x被重置为d.fx ，并将节点 d.vx设置为零
            // 要取消节点，请将节点 .fx和节点 .fy设置为空，或删除这些属性。
            d.fx = _this.$d3.event.x;
            d.fy = _this.$d3.event.y;
        })
        .on('end', function (d) {

            // 让alpha目标值值恢复为默认值0,停止力模型
            if (!_this.$d3.event.active) _this.simulation.alphaTarget(0);
            d.fx = null;
            d.fy = null;
        });
    },
    getTransform(source, target, _dis){// 求元素移动到目标位置所需要的 transform 属性值
      var r;
      if (target.x > source.x) {
          if (target.y > source.y) {
              r = Math.asin((target.y - source.y) / _dis)
          } else {
              r = Math.asin((source.y - target.y) / _dis);
              r = -r;
          }
      } else {
          if (target.y > source.y) {
              r = Math.asin((target.y - source.y) / _dis);
              r = Math.PI - r;
          } else {
              r = Math.asin((source.y - target.y) / _dis);
              r -= Math.PI;
          }
      }
      r = r * (180 / Math.PI);
      return "translate(" + source.x + "," + source.y + ")rotate(" + r + ")";      
    },
    // 切换全屏...
    toggleFullScreen(){
      this.isFullScreen = !this.isFullScreen;
      let _parentNode = this.$refs.relationMap;
      this.$nextTick(()=>{
          this.SVG.attr("width",_parentNode.offsetWidth).attr("height",_parentNode.offsetHeight)
          this.$nextTick(()=>{
          this.relMap_g = this.SVG.append("g")
                    .attr("id", "relMap_g")
                    .attr("width", _parentNode.offsetWidth)
                    .attr("height", _parentNode.offsetHeight);           
          this.simulation.force("center", this.$d3.forceCenter(_parentNode.offsetWidth, _parentNode.offsetHeight));
          this.simulation.alpha(1).restart();
        },500)
      })
    }
  }
}
</script>
<style lang="less" scoped>
#relationMap{
  width:500px;
  height: 500px;
  display: flex;
  flex: 1;
  position: relative;
  border: 1px solid red;
  &.fullscreen{
    position: absolute;
    top: 0;
    left: 0;
    width: 100%;
    height: 100%;
    z-index: 2000;
  }
  /deep/.tooltip{
    position: absolute;
    z-index: 10;
  }
  /deep/#mainSvg{
    z-index: 9;
    .nodeG{
      .nodeCheckbox,.nodeShrinkExtBtn{
        transform-origin:center;
        transform-box: fill-box;
        transform: scale(1);
        transition: transform 0.2s;
        cursor: pointer;
        &:hover{
          transform:scale(1.5)
        }
      }     
      .nodeCircle {
        stroke-width:3;
        stroke: #c5dbf0;
        &:hover{
          stroke-width:8;
          stroke: #a3e5f9;
        }
      }
    }
  }
  .controlBar{
    display: flex;
    position: absolute;
    right: 20px;
    top:10px;
    z-index: 10;
    .part{
      display: flex;
      margin: 0 7px;
      position: relative;
      .icon{
        display: flex;
        width: 20px;
        height: 20px;
      }
    }
  }
}
</style>