<template>
  <div id="home">
    <el-backtop target=".page-component__scroll .el-scrollbar__wrap"></el-backtop>
    
    <!-- 头部 -->
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
    
    <!-- 主要内容区域 -->
    <div class="main-content">
      <!-- 轮播图部分 -->
      <div class="carousel-container">
        <el-carousel :interval="3000" type="card" height="400px" indicator-position="outside">
          <el-carousel-item v-for="imgItem in carouselImages" :key="imgItem.alt">
            <img 
              :src="imgItem.src"      
              :alt="imgItem.alt"        
              class="carousel-img"
              @load="onImageLoad"
              @error="onImageError"
            />
          </el-carousel-item>
        </el-carousel>
      </div>
      
      <!-- 商品卡片区域 -->
      <div class="products-container">
        <h2 class="products-title">热门商品</h2>
        <div class="products-grid">
          <div 
            v-for="product in products" 
            :key="product.id" 
            class="product-card"
            @mouseenter="onCardHover(product.id)"
            @mouseleave="onCardLeave(product.id)"
          >
            <div class="product-image-wrapper">
              <img 
                :src="product.image" 
                :alt="product.name" 
                class="product-img"
                :class="{ 'loaded': product.imageLoaded }"
                @load="onProductImageLoad(product.id)"
                @error="onProductImageError(product.id)"
              />
            </div>
            <div class="product-info">
              <h3 class="product-name">{{ product.name }}</h3>
              <p class="product-description">{{ product.description }}</p>
              <div class="product-bottom">
                <span class="product-price">¥{{ product.price.toFixed(2) }}</span>
                <el-button 
                  type="primary" 
                  size="small" 
                  @click.stop="addToCart(product)"
                >
                  加入购物车
                </el-button>
              </div>
            </div>
          </div>
        </div>
        
        <!-- 加载更多 -->
        <div v-if="loading" class="loading">加载中...</div>
      </div>
    </div>
    
    <!-- 底部导航 -->
    <div class="footer">
      <ul class="nav-list">
        <li><a href="#" @click.prevent="goToHome">首页</a></li>
        <li><a href="#" @click.prevent="goToCategory">分类</a></li>
        <li><a href="#" @click.prevent="goToCart">购物车</a></li>
        <li><a href="#" @click.prevent="goToUserCenter">我的</a></li>
      </ul>
    </div>
  </div>
</template>

<script>
import { Carousel, CarouselItem, Input, Button, Message } from 'element-ui';

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
      // 轮播图数据 - 使用本地图片或稳定的图片URL
      carouselImages: [
        { 
          src: 'https://images.unsplash.com/photo-1607082348824-0a96f2a4b9da?ixlib=rb-4.0.3&ixid=M3wxMjA3fDB8MHxwaG90by1wYWdlfHx8fGVufDB8fHx8fA%3D%3D&auto=format&fit=crop&w=1770&q=80', 
          alt: '轮播图1：商品促销' 
        },
        { 
          src: 'https://images.unsplash.com/photo-1607082349566-187342175e2f?ixlib=rb-4.0.3&ixid=M3wxMjA3fDB8MHxwaG90by1wYWdlfHx8fGVufDB8fHx8fA%3D%3D&auto=format&fit=crop&w=1770&q=80', 
          alt: '轮播图2：新品上市' 
        },
        { 
          src: 'https://images.unsplash.com/photo-1607082349396-5b5d0a1c8b6f?ixlib=rb-4.0.3&ixid=M3wxMjA3fDB8MHxwaG90by1wYWdlfHx8fGVufDB8fHx8fA%3D%3D&auto=format&fit=crop&w=1770&q=80', 
          alt: '轮播图3：限时折扣' 
        },
        { 
          src: 'https://images.unsplash.com/photo-1607082350899-7e105aa886ae?ixlib=rb-4.0.3&ixid=M3wxMjA3fDB8MHxwaG90by1wYWdlfHx8fGVufDB8fHx8fA%3D%3D&auto=format&fit=crop&w=1770&q=80', 
          alt: '轮播图4：品牌故事' 
        },
      ],
      
      // 商品数据 - 添加图片加载状态
      products: [
        {
          id: 1,
          name: '苹果 iPhone 14 Pro',
          description: '新款苹果手机，A16芯片，4800万像素摄像头',
          price: 8999.00,
          image: 'https://images.unsplash.com/photo-1679150477802-fb56b5c8bea4?ixlib=rb-4.0.3&auto=format&fit=crop&w=500&q=80',
          imageLoaded: false,
          isHovering: false
        },
        {
          id: 2,
          name: '华为 Mate 50 Pro',
          description: '华为旗舰手机，鸿蒙系统，XMAGE影像',
          price: 6999.00,
          image: 'https://images.unsplash.com/photo-1592750475338-74b7b21085ab?ixlib=rb-4.0.3&auto=format&fit=crop&w=500&q=80',
          imageLoaded: false,
          isHovering: false
        },
        {
          id: 3,
          name: '小米 13 Ultra',
          description: '徕卡影像，第二代骁龙8处理器',
          price: 5999.00,
          image: 'https://images.unsplash.com/photo-1598327105666-5b89351aff97?ixlib=rb-4.0.3&auto=format&fit=crop&w=500&q=80',
          imageLoaded: false,
          isHovering: false
        },
        {
          id: 4,
          name: '三星 Galaxy S23',
          description: '超视觉夜拍系统，第二代骁龙8移动平台',
          price: 5699.00,
          image: 'https://images.unsplash.com/photo-1610945265064-0e34e5519bbf?ixlib=rb-4.0.3&auto=format&fit=crop&w=500&q=80',
          imageLoaded: false,
          isHovering: false
        },
        {
          id: 5,
          name: 'OPPO Find X6 Pro',
          description: '哈苏手机影像，马里亚纳X芯片',
          price: 5999.00,
          image: 'https://images.unsplash.com/photo-1592899677977-9c10ca588bbd?ixlib=rb-4.0.3&auto=format&fit=crop&w=500&q=80',
          imageLoaded: false,
          isHovering: false
        },
        {
          id: 6,
          name: 'vivo X90 Pro+',
          description: '蔡司一英寸主摄，自研芯片V2',
          price: 6499.00,
          image: 'https://images.unsplash.com/photo-1546054451-aa7242c77379?ixlib=rb-4.0.3&auto=format&fit=crop&w=500&q=80',
          imageLoaded: false,
          isHovering: false
        },
        {
          id: 7,
          name: '荣耀 Magic5 Pro',
          description: '青海湖电池，鹰眼相机系统',
          price: 5699.00,
          image: 'https://images.unsplash.com/photo-1574943320219-553eb213f72d?ixlib=rb-4.0.3&auto=format&fit=crop&w=500&q=80',
          imageLoaded: false,
          isHovering: false
        },
        {
          id: 8,
          name: '一加 11',
          description: '哈苏影像，100W超级闪充',
          price: 3999.00,
          image: 'https://images.unsplash.com/photo-1589492477829-5e65395b66cc?ixlib=rb-4.0.3&auto=format&fit=crop&w=500&q=80',
          imageLoaded: false,
          isHovering: false
        }
      ],
      
      user: {
        phone: ''
      },
      isLoggedIn: false,
      searchText: '',
      loading: false,
      hoveredCardId: null
    }
  },
  mounted() {
    // 检查是否已登录
    const savedUser = localStorage.getItem('user');
    if (savedUser) {
      this.user = JSON.parse(savedUser);
      this.isLoggedIn = true;
    }
  },
  methods: {
    handleSearch() {
      if (this.searchText.trim()) {
        Message.success(`搜索: ${this.searchText}`);
        this.$router.push('/header');
      } else {
        Message.warning('请输入搜索内容');
      }
    },
    
    goToLogin() {
      this.$router.push('/login');
    },
    
    logout() {
      this.isLoggedIn = false;
      this.user = {};
      localStorage.removeItem('user');
      Message.success('已退出登录');
    },
    
    goToUserCenter() {
      this.$router.push('/usercent');
    },
    
    addToCart(product) {
      Message.success(`已添加 ${product.name} 到购物车`);
    },
    
    // 图片加载处理
    onImageLoad() {
      // 轮播图图片加载完成
    },
    
    onImageError(event) {
      console.error('轮播图图片加载失败:', event.target.src);
    },
    
    onProductImageLoad(productId) {
      const product = this.products.find(p => p.id === productId);
      if (product) {
        product.imageLoaded = true;
      }
    },
    
    onProductImageError(productId) {
      const product = this.products.find(p => p.id === productId);
      if (product) {
        product.image = 'https://images.unsplash.com/photo-1542291026-7eec264c27ff?ixlib=rb-4.0.3&auto=format&fit=crop&w=500&q=80';
      }
    },
    
    // 卡片悬停效果
    onCardHover(productId) {
      this.hoveredCardId = productId;
    },
    
    loadMoreProducts() {
      if (this.loading) return;
      
      this.loading = true;
      setTimeout(() => {
        const newProducts = [];
        const startId = this.products.length + 1;
        
        for (let i = 0; i < 4; i++) {
          newProducts.push({
            id: startId + i,
            name: `商品名称${startId + i}`,
            description: `商品描述信息 ${startId + i}`,
            price: Math.floor(Math.random() * 1000) + 100,
            image: `https://images.unsplash.com/photo-1542291026-7eec264c27ff?ixlib=rb-4.0.3&auto=format&fit=crop&w=500&q=80`,
            imageLoaded: false,
            isHovering: false
          });
        }
        
        this.products = [...this.products, ...newProducts];
        this.loading = false;
      }, 1500);
    },
    
    // 底部导航方法
    goToHome() {
      // 已经在首页
    },
    
    goToCategory() {
      Message.info('分类页面');
    },
    
    goToCart() {
      Message.info('购物车页面');
    }
  }
}
</script>

<style scoped>
/* 全局样式 */
#home {
  min-height: 100vh;
  display: flex;
  flex-direction: column;
  background-color: #f8f9fa;
}

/* 头部样式 */
.header {
  background-color: rgb(237, 106, 40);
  width: 100%;
  height: 100px;
  position: fixed;
  top: 0;
  left: 0;
  z-index: 1000;
  box-shadow: 0 2px 10px rgba(0, 0, 0, 0.1);
}

.logo {
  width: 100%;
  height: 100%;
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
  color: white;
  font-size: 24px;
  font-weight: bold;
}

.search-container {
  display: flex;
  align-items: center;
  gap: 10px;
}

.search-input {
  width: 300px;
}

/* 主要内容区域 */
.main-content {
  margin-top: 100px;
  margin-bottom: 80px;
  flex: 1;
  padding: 0;
  background-color: #f8f9fa;
}

/* 轮播图样式 - 控制与下方内容的距离 */
.carousel-container {
  padding: 30px 20px 40px 20px; /* 上 右 下 左 - 下方增加更多间距 */
  background: linear-gradient(135deg, #f5f7fa 0%, #c3cfe2 100%);
  margin-bottom: 10px; /* 减少底部外边距，因为用padding控制了 */
  border-radius: 0;
  box-shadow: 0 4px 12px rgba(0, 0, 0, 0.08);
}

.carousel-img {
  width: 100%;
  height: 100%;
  object-fit: cover;
  border-radius: 12px;
  box-shadow: 0 6px 20px rgba(0, 0, 0, 0.15);
}

/* 商品区域样式 - 控制与轮播图的距离 */
.products-container {
  padding: 30px 20px 50px 20px; /* 上方增加更多间距 */
  max-width: 1200px;
  margin: 0 auto;
  background-color: #f8f9fa;
}

.products-title {
  text-align: center;
  font-size: 32px;
  color: #2c3e50;
  margin-bottom: 40px; /* 增加标题与商品网格的间距 */
  font-weight: bold;
  padding: 25px 0;
  background: white;
  border-radius: 12px;
  box-shadow: 0 4px 12px rgba(0, 0, 0, 0.08);
  position: relative;
}

.products-title::after {
  content: '';
  position: absolute;
  bottom: 0;
  left: 50%;
  transform: translateX(-50%);
  width: 80px;
  height: 4px;
  background: linear-gradient(90deg, #ff6b00, #ff8c00);
  border-radius: 2px;
}

/* 商品网格 - 每行4列 */
.products-grid {
  display: grid;
  grid-template-columns: repeat(4, 1fr);
  gap: 25px; /* 增加卡片间距 */
  margin-bottom: 40px;
}

/* 商品卡片样式 - 修复闪烁问题 */
.product-card {
  background: white;
  border-radius: 16px;
  overflow: hidden;
  box-shadow: 0 4px 15px rgba(0, 0, 0, 0.08);
  transition: all 0.3s cubic-bezier(0.25, 0.46, 0.45, 0.94); /* 使用更平滑的缓动函数 */
  cursor: pointer;
  display: flex;
  flex-direction: column;
  border: 1px solid #e9ecef;
  position: relative;
  /* 添加will-change优化性能 */
  will-change: transform, box-shadow;
  /* 添加3D变换硬件加速 */
  transform: translateZ(0);
  backface-visibility: hidden;
  -webkit-font-smoothing: antialiased;
}

/* 移除可能导致闪烁的:hover效果，改用更稳定的实现 */
.product-card:hover {
  transform: translateY(-8px) translateZ(0); /* 增加Z轴变换以启用硬件加速 */
  box-shadow: 0 15px 30px rgba(0, 0, 0, 0.15);
  border-color: #ff6b00;
}

/* 商品图片容器 - 修复图片闪烁 */
.product-image-wrapper {
  width: 100%;
  height: 220px; /* 增加高度 */
  overflow: hidden;
  position: relative;
  background-color: #f8f9fa; /* 添加背景色防止空白闪烁 */
}

.product-img {
  width: 100%;
  height: 100%;
  object-fit: cover;
  transition: transform 0.6s cubic-bezier(0.25, 0.46, 0.45, 0.94); /* 更平滑的过渡 */
  opacity: 0; /* 初始隐藏 */
  transform: scale(0.98); /* 初始轻微缩小 */
}

.product-img.loaded {
  opacity: 1; /* 加载完成后显示 */
  transform: scale(1); /* 恢复正常大小 */
}

.product-card:hover .product-img {
  transform: scale(1.08); /* 增加缩放幅度 */
}

/* 商品信息 */
.product-info {
  padding: 20px;
  flex: 1;
  display: flex;
  flex-direction: column;
  background: white;
}

.product-name {
  font-size: 16px;
  font-weight: 600;
  color: #2c3e50;
  margin-bottom: 10px;
  line-height: 1.4;
  min-height: 44px;
  overflow: hidden;
  display: -webkit-box;
  -webkit-line-clamp: 2;
  -webkit-box-orient: vertical;
  transition: color 0.3s;
}

.product-card:hover .product-name {
  color: #ff6b00;
}

.product-description {
  font-size: 14px;
  color: #6c757d;
  margin-bottom: 20px;
  line-height: 1.5;
  min-height: 42px;
  overflow: hidden;
  display: -webkit-box;
  -webkit-line-clamp: 2;
  -webkit-box-orient: vertical;
  flex: 1;
}

/* 商品底部（价格和按钮） */
.product-bottom {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-top: 15px;
  padding-top: 15px;
  border-top: 1px solid #e9ecef;
}

.product-price {
  font-size: 20px;
  font-weight: bold;
  color: #ff6b00;
  text-shadow: 0 1px 2px rgba(0, 0, 0, 0.1);
}

/* 加载中样式 */
.loading {
  text-align: center;
  padding: 40px;
  color: #6c757d;
  font-size: 16px;
  background: white;
  border-radius: 12px;
  margin-top: 30px;
  box-shadow: 0 4px 12px rgba(0, 0, 0, 0.08);
  border: 1px solid #e9ecef;
}

/* 底部导航样式 */
.footer {
  background: linear-gradient(135deg, #309c81 0%, #257a65 100%);
  width: 100%;
  height: 80px;
  position: fixed;
  bottom: 0;
  left: 0;
  z-index: 1000;
  box-shadow: 0 -4px 12px rgba(0, 0, 0, 0.1);
  border-top: 1px solid rgba(255, 255, 255, 0.1);
}

.nav-list {
  list-style-type: none;
  padding: 0;
  margin: 0;
  display: flex;
  justify-content: space-around;
  align-items: center;
  height: 100%;
}

.nav-list li {
  text-align: center;
  flex: 1;
}

.nav-list li a {
  color: white;
  text-decoration: none;
  font-size: 14px;
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  height: 100%;
  padding: 10px 0;
  transition: all 0.3s;
  position: relative;
}

.nav-list li a:hover {
  background-color: rgba(255, 255, 255, 0.1);
  transform: translateY(-2px);
}

.nav-list li a::after {
  content: '';
  position: absolute;
  bottom: 0;
  left: 50%;
  transform: translateX(-50%);
  width: 0;
  height: 3px;
  background-color: white;
  transition: width 0.3s;
}

.nav-list li a:hover::after {
  width: 60%;
}

/* 响应式设计 */
@media (max-width: 1200px) {
  .products-grid {
    grid-template-columns: repeat(3, 1fr);
    gap: 20px;
  }
}

@media (max-width: 992px) {
  .carousel-container {
    padding: 20px 15px 30px 15px;
  }
  
  .products-container {
    padding: 20px 15px 40px 15px;
  }
  
  .products-grid {
    grid-template-columns: repeat(3, 1fr);
    gap: 18px;
  }
}

@media (max-width: 768px) {
  .header {
    height: 80px;
  }
  
  .main-content {
    margin-top: 80px;
  }
  
  .logo > div:first-child {
    font-size: 20px;
  }
  
  .search-input {
    width: 200px;
  }
  
  .carousel-container {
    padding: 15px 10px 25px 10px;
  }
  
  .el-carousel {
    height: 300px !important;
  }
  
  .products-container {
    padding: 20px 10px 30px 10px;
  }
  
  .products-title {
    font-size: 28px;
    margin-bottom: 30px;
    padding: 20px 0;
  }
  
  .products-grid {
    grid-template-columns: repeat(2, 1fr);
    gap: 15px;
  }
  
  .product-image-wrapper {
    height: 180px;
  }
  
  .footer {
    height: 70px;
  }
}

@media (max-width: 576px) {
  .products-grid {
    grid-template-columns: 1fr;
    gap: 12px;
  }
  
  .products-title {
    font-size: 24px;
    margin-bottom: 25px;
  }
  
  .product-image-wrapper {
    height: 160px;
  }
  
  .footer {
    height: 60px;
  }
  
  .nav-list li a {
    font-size: 12px;
  }
}
</style>