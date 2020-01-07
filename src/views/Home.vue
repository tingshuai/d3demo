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
      },
      {
          "id": 832,
          "oid": "5a56e24b93cc2b42a6bf0336",
          "type": "role",
          "name": "\u6c88\u592b\u4eba",
          "avatar": "https://i.linkeddb.com/upload/05db/a955/dda11486be4f1b94717ea8bb.png",
          "url": "/movie/role/5a56e24b93cc2b42a6bf0336/",
          "exdata": {
              "id": 76304,
              "oid": "59fa729b18521569b6636eb9",
              "name": "\u5218\u6d01",
              "avatar": "https://i.linkeddb.com/person2/e8ac/f7a2/cf19348ed16ab3700bee93b9.jpg"
          }
      },     
      {
          "id": 833,
          "oid": "5a56e24b93cc2b42a6bf0337",
          "type": "role",
          "name": "\u6c88\u8001\u592b\u4eba",
          "avatar": "https://i.linkeddb.com/upload/742a/72f2/f933743cc2563895cb28a4ba.png",
          "url": "/movie/role/5a56e24b93cc2b42a6bf0337/",
          "exdata": {
              "id": 74843,
              "oid": "59fa729018521569b66364a3",
              "name": "\u5510\u7fa4",
              "avatar": "https://i.linkeddb.com/person2/97c3/9c83/fee3e8039a4c976f192447bf.jpg"
          }
      }
  ],
  links:[
      {
          "source": 41,
          "target": 832,
          "relation": "\u517b\u5973",
          "color": "f2826a"
      },
      {
          "source": 41,
          "target": 833,
          "relation": "\u4e3b\u4ec6",
          "color": ""
      },
      {
          "source": 41,
          "target": 831,
          "relation": "\u4ec7\u4eba",
          "color": ""
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
      this.$refs.zstp.addAndDeleteNodes(_extData,d,i);
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