<template>
    <div id="headerview">
        <div class="header">
             <el-button type="primary" @click="goBack" style="margin-right: 10px;">返回</el-button>
             <el-input 
                placeholder="请输入搜索内容" 
                v-model="searchText" 
                class="search-input">
                <i slot="prefix" class="el-input__icon el-icon-search"></i>
                <el-button type="primary" @click="handleSearch" style="text-align: left;">搜索</el-button>
            </el-input>
        </div>
        
        <div class="nav">
            <div class="search-section">
                <el-button type="primary" @click="handleSearch" style="text-align: left;">最近搜索</el-button>
                <div class="history-section">
                    <el-button type="primary" @click="handleSearch" style="text-align: left;">历史记录</el-button>
                </div>
            </div>
            
            <div class="card-grid">
                <div 
                    v-for="(item, index) in items" 
                    :key="index" 
                    class="card"
                >
                    {{ item }}
                </div>
            </div>
        </div>
        
        <!-- 底部导航栏 -->
        <div class="bottom-nav">
            <ul class="nav-list">
                <li><a href="#src\components\views\home.vue" @click.prevent="goToHome">首页</a></li>
                <li><a href="#">分类</a></li>
                <li><a href="#">购物车</a></li>
                <li><a href="#src\components\views\src\components\usercent\usercent.vue" @click.prevent="goToUserCenter">我的</a></li>
            </ul>
        </div>
    </div>
</template>

<script>
export default {
  name: 'headerView',
  data() {
    return {
      searchText: '',
      items: [] // 添加 items 数组以避免错误
    }
  },
  methods: {
    handleSearch() {
      this.$router.push({
        path: '/search',
        query: {
          text: this.searchText
        }
      })
    },
    goBack() {
      this.$router.go(-1);
    },
    goToHome() {
      this.$router.push('/home');
    },
    goToUserCenter() {
      this.$router.push('/usercent');
    }
  }
} 
</script>

<style scoped>
#headerview {
  min-height: 100vh;
  position: relative;
  padding-bottom: 60px; /* 为底部导航栏留出空间 */
  box-sizing: border-box;
}

.header {
  background-color: #309c81;
  padding: 10px 15px;
  position: sticky;
  top: 0;
  width: 100%;
  box-shadow: 0 2px 10px rgba(0,0,0,0.05);
  display: flex;
  align-items: center;
  z-index: 100;
}

.search-input {
  flex: 1;
  display: flex;
  align-items: center;
}

.nav {
  padding: 15px;
  min-height: calc(100vh - 120px); /* 减去头部和底部的高度 */
}

.search-section {
  margin-bottom: 20px;
}

.history-section {
  margin-top: 200px; /* 历史记录在最近搜索下方60px */
}

.card-grid {
  display: grid;
  grid-template-columns: repeat(auto-fill, minmax(150px, 1fr));
  gap: 15px;
  margin-top: 20px;
}

.card {
  border: 1px solid #e0e0e0;
  border-radius: 8px;
  padding: 20px;
  text-align: center;
  background: #f9f9f9;
}

/* 底部导航栏样式 */
.bottom-nav {
  position: fixed;
  bottom: 0;
  left: 0;
  right: 0;
  background-color: #309c81;
  border-top: 1px solid #20a3d6;
  padding: 10px 0;
  box-shadow: 0 -2px 10px rgba(0,0,0,0.1);
  z-index: 1000;
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
  color: rgb(255, 255, 255);
  text-decoration: none;
  padding: 10px 0;
  display: block;
  font-size: 14px;
  transition: color 0.3s;
}

.nav-list li a:hover {
  color: #e0e0e0;
}

/* 响应式设计 */
@media (max-width: 768px) {
  .header {
    padding: 10px;
  }
  
  .nav {
    padding: 10px;
  }
  
  .card-grid {
    grid-template-columns: repeat(auto-fill, minmax(120px, 1fr));
    gap: 10px;
  }
  
  .nav-list li a {
    font-size: 12px;
    padding: 8px 0;
  }
}
</style>