<template>
 <div id="mypage">
    <el-tabs v-model="activeName" @tab-click="handleClick">
    <el-tab-pane label="待付款" name="first">

    </el-tab-pane>

    <el-tab-pane label="待发货" name="second">

    </el-tab-pane>

    <el-tab-pane label="待收货" name="third">

    </el-tab-pane>

    <el-tab-pane label="待评价" name="fourth">

    </el-tab-pane>

    <el-tab-pane label="已完成" name="fifth">

    </el-tab-pane>

  </el-tabs>
 </div>
</template>
<script>
  export default {
    name: 'MyPage',
    props: {
        product:
        {
            type: Object,
            default: ()=>({})
        }
    },
    data() {
      return {
        activeName: 'first',
        from:
        {
            type:'',
            description:''
        }
      };
    },
    methods: {
      handleClick(tab, event) {
        console.log(tab, event);
      }
    }
  };
</script>