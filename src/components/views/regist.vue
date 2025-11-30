<template>
  <div class="form-container">
    <h1 class="form-title">注册</h1>
    <form @submit.prevent="handleRegister">
      <div class="form-group">
        <label class="form-label">用户名</label>
        <input 
          type="text" 
          class="form-input" 
          v-model="registerForm.username"
          placeholder="请输入用户名"
          required
        >
        <div v-if="errors.username" class="error-message">{{ errors.username }}</div>
      </div>

      <div class="form-group">
        <label class="form-label">邮箱</label>
        <input 
          type="email" 
          class="form-input" 
          v-model="registerForm.email"
          placeholder="请输入邮箱"
          required
        >
        <div v-if="errors.email" class="error-message">{{ errors.email }}</div>
      </div>

      <div class="form-group">
        <label class="form-label">密码</label>
        <input 
          type="password" 
          class="form-input" 
          v-model="registerForm.password"
          placeholder="请输入密码"
          required
        >
        <div v-if="errors.password" class="error-message">{{ errors.password }}</div>
      </div>

      <div class="form-group">
        <label class="form-label">确认密码</label>
        <input 
          type="password" 
          class="form-input" 
          v-model="registerForm.confirmPassword"
          placeholder="请再次输入密码"
          required
        >
        <div v-if="errors.confirmPassword" class="error-message">{{ errors.confirmPassword }}</div>
      </div>

      <button type="submit" class="btn btn-primary" :disabled="loading">
        {{ loading ? '注册中...' : '注册' }}
      </button>

      <div v-if="registerSuccess" class="success-message">
        注册成功！正在跳转到登录页面...
      </div>

      <div class="form-footer">
        已有账号？<router-link to="/login" class="link">立即登录</router-link>
      </div>
    </form>
  </div>
</template>

<script>
export default {
  name: 'UserRegister',
  data() {
    return {
      registerForm: {
        username: '',
        email: '',
        password: '',
        confirmPassword: ''
      },
      errors: {},
      loading: false,
      registerSuccess: false
    }
  },
  methods: {
    validateForm() {
      this.errors = {}
      
      if (!this.registerForm.username.trim()) {
        this.errors.username = '用户名不能为空'
      } else if (this.registerForm.username.length < 3) {
        this.errors.username = '用户名长度不能少于3位'
      }

      if (!this.registerForm.email) {
        this.errors.email = '邮箱不能为空'
      } else if (!this.isValidEmail(this.registerForm.email)) {
        this.errors.email = '请输入有效的邮箱地址'
      }

      if (!this.registerForm.password) {
        this.errors.password = '密码不能为空'
      } else if (this.registerForm.password.length < 6) {
        this.errors.password = '密码长度不能少于6位'
      }

      if (!this.registerForm.confirmPassword) {
        this.errors.confirmPassword = '请确认密码'
      } else if (this.registerForm.password !== this.registerForm.confirmPassword) {
        this.errors.confirmPassword = '两次输入的密码不一致'
      }

      return Object.keys(this.errors).length === 0
    },

    isValidEmail(email) {
      const emailRegex = /^[^\s@]+@[^\s@]+\.[^\s@]+$/
      return emailRegex.test(email)
    },

    handleRegister() {
      if (!this.validateForm()) {
        return
      }

      this.loading = true
      
      // 模拟注册API调用
      setTimeout(() => {
        this.loading = false
        this.registerSuccess = true
        
        console.log('注册信息:', this.registerForm)
        
        // 注册成功后跳转到登录页面
        setTimeout(() => {
          this.$router.push('/login')
        }, 2000)
      }, 1500)
    }
  }
}
</script>
<style scoped> 
</style>