<template>
  <div id="home">
     Scroll down to see the bottom-right button.
  <el-backtop target=".page-component__scroll .el-scrollbar__wrap"></el-backtop>
    <div class="header">
      <div class="logo">
        <div>
          <p v-if="isLoggedIn">ESHOUP|欢迎，{{ user.phone }}</p>
          <p v-else>ESHOUP</p>
        </div>
        <div class="search-container">
          <el-input 
            placeholder="请输入搜索内容" 
            v-model="searchText" 
            class="search-input">
            <i slot="prefix" class="el-input__icon el-icon-search"></i>
          </el-input>
          <el-button type="primary" @click="handleSearch">搜索</el-button>
        </div>
        <div>
          <a v-if="!isLoggedIn" href="#" @click="goToLogin">登录/注册</a>
          <el-button v-else type="text" @click="logout">退出登录</el-button>
        </div>
      </div>
    </div>
    <div class="content">
        <el-carousel :interval="3000" type="card" height="500px" weidth="400px">
           <el-carousel-item v-for="(imgItem, index) in carouselImages" :key="index">
      <!-- 轮播图核心：图片标签 -->
      <img 
        :src="imgItem.src"      
        :alt="imgItem.alt"        
        class="carousel-img"
      >
    </el-carousel-item>
        </el-carousel>
    </div>
    <div class="nav">
       <div class="content" ref="content" @scroll="handleScroll">
      <div class="card-grid">
        <div 
          v-for="(item, index) in items" 
          :key="index" 
          class="card"
        >
          {{ item }}
        </div>
      </div>
      <div v-if="loading" class="loading">加载中...</div>
    </div>
    </div>
    <div class="footer">
        <ul class="D">
            <li><a href="#src\components\views\home.vue">首页</a></li>
            <li><a href="#">分类</a></li>
            <li><a href="#">购物车</a></li>
            <li><a href="#src\components\views\src\components\usercent\usercent.vue" @click="goToUserCenter">我的</a></li>
        </ul>
    </div>
  </div>
</template>
<script>
  import { Carousel, CarouselItem, Input, Button } from 'element-ui';
  import 'element-ui/lib/theme-chalk/index.css';
  
  export default {
    name: 'HomePage',
    components: {
      'el-carousel': Carousel,
      'el-carousel-item': CarouselItem,
      'el-input': Input,
      'el-button': Button
    },
    data() {
      return {
         carouselImages: [
        { src: 'https://example.com/banner1.jpg', alt: '轮播图1：商品促销' },
        { src: 'https://example.com/banner2.jpg', alt: '轮播图2：新品上市' },
        { src: 'https://example.com/banner3.jpg', alt: '轮播图3：限时折扣' },
        { src: 'https://example.com/banner4.jpg', alt: '轮播图4：品牌故事' },
        { src: 'https://example.com/banner5.jpg', alt: '轮播图5：用户好评' },
        { src: 'https://example.com/banner6.jpg', alt: '轮播图6：售后服务' }
      ],

        user: {
          phone: ''
        },
        isLoggedIn: false,
        searchText: '',
        count: 0,
        items: [],
        loading: false,
        page: 1,
        pageSize: 19 // 每次加载3行，每行4个卡片，共12
      }
    },
      mounted() {
      this.loadItems();
      // 检查是否已登录
      const savedUser = localStorage.getItem('user');
      if (savedUser) {
        this.user = JSON.parse(savedUser);
        this.isLoggedIn = true;
      }
    },
    methods: {
      handleSearch() {
         // 修改跳转路径到HeaderView
         this.$router.push('/header');
      },
      loadItems() { 
        this.loading = true;
        setTimeout(() => {
           const newItems = [];
          const startIndex = (this.page - 1) * this.pageSize;
          for (let i = startIndex; i < startIndex + this.pageSize; i++) {
            newItems.push(`商品 ${i + 1}`);
          }
          this.items = [...this.items, ...newItems];
          this.loading = false;
          this.page++;
        }, 1000);
      },
       handleScroll(event) {
        const { scrollTop, clientHeight, scrollHeight } = event.target;
        // 当滚动到底部时加载更多
        if (scrollTop + clientHeight >= scrollHeight - 10) {
          if (!this.loading) {
            this.loadItems();
          }
        }
      },
      goToLogin() {
        this.$router.push('/login');
      },
      logout() {
        this.isLoggedIn = false;
        this.user = {};
        localStorage.removeItem('user');
      },
      goToUserCenter() {
        this.$router.push('/usercent');
      }
    }
  }
</script>
<style>
  .header {
    background-color: rgb(237, 106, 40);
    width: 100%;
    height: 100px;
    position: fixed;
    top: 0;
    left: 0;
    z-index: 100;
  }
  
  .logo {
    width: 100%;
    height: 100px;
    display: flex;
    justify-content: space-between;
    align-items: center;
    padding: 0 20px;
    box-sizing: border-box;
  }
  
  .logo > div:first-child {
    display: flex;
    align-items: center;
    gap: 20px;
  }
  
  .search-container {
    display: flex;
    align-items: center;
    gap: 10px;
  }
  
  .search-input {
    width: 300px;
  }
  
  .el-carousel__item h3 {
    color: #475669;
    font-size: 14px;
    opacity: 0.75;
    line-height: 200px;
    margin: 0;
  }
  
  .el-carousel__item:nth-child(2n) {
    background-color: #260475;
  }
  
  .el-carousel__item:nth-child(2n+1) {
    background-color: #4b9b1d;
  }
  .content {
    margin-top: 100px;
    margin-bottom: 150px;
    min-height: calc(100vh - 250px);
  }
  
  .D
  {
    list-style-type: none;
    padding: 20px;
    margin: 0;
    display: flex;
    overflow: hidden;
    justify-content: space-around;
    align-items: center;
    height: 100%;
  }

  .D li
  {
    text-align: center;
    grid-gap:100px;
  }

  .D li a {
    color: rgb(255, 255, 255);
    text-decoration: none;
    padding: 10px 0;
  }

  .footer {
  background-color: #309c81;
  border-top: 1px solid #20a3d6;
  padding: 10px 0;
  position: sticky;
  bottom: 0;
  width: 100%;
  box-shadow: 0 -2px 10px rgba(0,0,0,0.05);
  }
</style>