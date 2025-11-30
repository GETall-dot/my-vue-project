<template>
    <div class="login">
      <h1>登录</h1>
      <form @submit.prevent="handleLogin">
        <div class="form-group">
          <label class="form-label">用户名</label>
          <input 
            type="text" 
            class="form-input" 
            placeholder="用户名" 
            v-model="loginForm.username" 
            required
          >
          <div v-if="errors.username" class="error-message">
            {{ errors.username }}
          </div>
        </div>
       <div class="form-group">
        <label class="form-label">密码</label>
        <input 
          type="password" 
          class="form-input" 
          v-model="loginForm.password"
          placeholder="请输入密码"
          required
        >
        <div v-if="errors.password" class="error-message">{{ errors.password }}</div>
      </div>
      <button type="submit" class="btn btn-primary" :disabled="loading">
        {{ loading ? '登录中...' : '登录' }}
      </button>

      <div v-if="loginSuccess" class="success-message">
        登录成功！
      </div>

      <div class="form-footer">
        还没有账号？<router-link to="/regist" class="link">立即注册</router-link>
      </div>
    </form>
    </div>
</template>

<script>
export default {
  name: 'UserLogin',
  data() {
    return {
      loginForm: {
        username: '',
        password: ''
      },
      errors: {},
      loading: false,
      loginSuccess: false
    }
  },
  methods: {
    validateForm() {
      this.errors = {}
      
      if (!this.loginForm.username.trim()) {
        this.errors.username = '用户名不能为空'
      }

      if (!this.loginForm.password) {
        this.errors.password = '密码不能为空'
      } else if (this.loginForm.password.length < 6) {
        this.errors.password = '密码长度不能少于6位'
      }

      return Object.keys(this.errors).length === 0
    },

    handleLogin() {
      if (!this.validateForm()) {
        return
      }

      this.loading = true
      
      // 模拟登录API调用
      setTimeout(() => {
        this.loading = false
        this.loginSuccess = true
        
        // 保存用户信息到localStorage
        const user = {
          phone: this.loginForm.username + '的手机',
          username: this.loginForm.username
        };
        localStorage.setItem('user', JSON.stringify(user));
        
        // 跳转到主页
        console.log('登录信息:', this.loginForm)
        
        // 实际项目中跳转到主页
        setTimeout(() => {
          this.$router.push('/');
        }, 1000);
      }, 1500)
    }
  }
}
</script>