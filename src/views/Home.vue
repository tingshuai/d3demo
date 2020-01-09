<template>
  <div class="helloPage" id="helloPage" ref="helloPage">
    <hello ref="zstp" @nodeShrinkExtBtn="nodeShrinkExtBtn"></hello>
  </div>
</template>

<script>
// @ is an alias to /src
/* eslint-disable*/ 
let _extData = {
  nodes:[
          {
          "id": 831,
          "oid": "5a56e24b93cc2b42a6bf0335",
          "type": "role",
          "name": "\u67e5\u5764",
          "avatar": "https://i.linkeddb.com/upload/48e2/810a/3675c7b3b157438ce5a42dbf.png",
          "url": "/movie/role/5a56e24b93cc2b42a6bf0335/",
          "exdata": {
              "id": 182836,
              "oid": "5a150881b0f85226cf8678e4",
              "name": "\u674e\u827a\u79d1",
              "avatar": "https://i.linkeddb.com/upload/4a66/509d/39a25ad4a12d1b6486a0ff91.jpg"
          }
      }
  ],
  links:[
      {
          "source": 41,
          "target": 831,
          "relation": "\u517b\u5973",
          "color": "f2826a"
      }
  ]
} 
let _extData2 = {
  nodes:[      {
          "id": 56,
          "oid": "5a56e22893cc2b42a6bf002e",
          "type": "role",
          "name": "\u5434\u6cfd",
          "avatar": "https://i.linkeddb.com/upload/image/2017920/165926893902.jpg",
          "url": "/movie/role/5a56e22893cc2b42a6bf002e/",
          "exdata": {
              "id": 156844,
              "oid": "59fa7af418521569b66b44e1",
              "name": "\u5f20\u5929\u9633",
              "avatar": "https://i.linkeddb.com/person2/75f6/f682/2dbdb23443d37793c5b44250.jpg"
          }
      }],
  links:[
      {
          "source": 46,
          "target": 56,
          "relation": "\u517b\u5973",
          "color": "f2826a"
      }   
  ]
}
import hello from '@/components/HelloWorld.vue'
export default {
  name: 'relationMap',
  components: { hello },
  props:{},
  data(){
    return {}
  },
  mounted(){
   
  },
  methods:{
    nodeShrinkExtBtn(d,i){
      this.$refs.zstp.addAndDeleteNodes(i == 0 ?_extData : _extData2,d,i);
    }
  }
}
</script>
<style lang="less" scoped>
  .helloPage{
    width: 100%;
    height: 100%;
  }
</style>