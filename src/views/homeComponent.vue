<template>
  <div class="relationMap" id="relationMap" ref="relationMap">
  </div>
</template>

<script>
// @ is an alias to /src
/* eslint-disable*/ 
import RelationChart from 'relation-chart'
export default {
  name: 'relationMap',
  components: {
  },
  data(){
    return {

    }
  },
  mounted(){
    this.$d3.json("/miserables.json").then((data)=>{
        new RelationChart(this.$refs.relationMap, data)
    })
  },
  methods:{

  }
}
</script>
<style lang="less" scoped>
#relationMap{
  width:100%;
  height: 100%;
  display: flex;
  flex: 1;
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