<template>
  <div id="usercenter">
    <div class="user-container">
      <!-- 用户信息概览 -->
      <div class="user-header">
          <div>
            <el-button type="primary" @click="goToHome">返回首页</el-button>
          </div>
        <div style="display: flex; justify-content: space-between; align-items: center;">
          <div class="avatar-section">
            <el-form-item label="头像">
              <el-upload
              class="avatar-uploader"
              action="/api/upload"
              :show-file-list="false"
              :on-success="handleAvatarSuccess"
              :before-upload="beforeAvatarUpload">
              <el-avatar v-if="user.avatar" :size="80" :src="user.avatar"></el-avatar>
              <i v-else class="el-icon-plus avatar-uploader-icon"></i>
            </el-upload>
          </el-form-item>
            <div class="user-info">
          
              <h3>{{ user.nickname }}</h3>
              <p>ID: {{ user.id }}</p>
            </div>
          </div>
        </div>
        <div class="balance-section">
          <p>账户余额</p>
          <h2>¥{{ user.balance }}</h2>
          <el-button type="primary" size="small" @click="activeTab = 'recharge'">立即充值</el-button>
        </div>
      </div>

      <!-- 主内容区域 -->
      <div class="main-content">
        <el-tabs v-model="activeTab">
          <!-- 个人管理 -->
          <el-tab-pane label="个人管理" name="profile">
            <el-form :model="profileForm" :rules="profileRules" ref="profileForm" label-width="100px">
              <el-form-item label="昵称">
                <el-input v-model="profileForm.nickname" placeholder="请输入昵称"></el-input>
              </el-form-item>
              <el-form-item label="手机号" prop="phone">
                <el-input v-model="profileForm.phone" placeholder="请输入手机号"></el-input>
                <el-button type="primary" @click="updatePhone">保存</el-button>
              </el-form-item>
              <el-form-item label="原密码" prop="oldPassword">
                <el-input type="password" v-model="profileForm.oldPassword" autocomplete="off"></el-input>
              </el-form-item>
              <el-form-item label="新密码" prop="newPassword">
                <el-input type="password" v-model="profileForm.newPassword" autocomplete="off"></el-input>
              </el-form-item>
              <el-form-item>
                <el-button type="primary" @click="updatePassword">修改密码</el-button>
              </el-form-item>
            </el-form>
            <el-button type="primary" @click="updateProfile">保存</el-button>
          </el-tab-pane>

          <!-- 我的订单 -->
          <el-tab-pane label="我的订单" name="orders">
            <div class="order-list">
              <el-empty description="暂无订单" v-if="orders.length === 0"></el-empty>
              <div v-else>
                <!-- 订单列表内容 -->
              </div>
            </div>
          </el-tab-pane>

          <!-- 收货地址 -->
          <el-tab-pane label="收货地址" name="address">
            <div class="address-management">
              <div class="address-header">
                <h3>收货地址</h3>
                <el-button type="primary" @click="showAddAddressDialog = true">新增收货地址</el-button>
              </div>
              
              <!-- 地址列表 -->
              <div class="address-list">
                <div v-for="address in addresses" :key="address.id" 
                     class="address-item" 
                     :class="{ 'default-address': address.isDefault }">
                  <div class="address-info">
                    <div class="address-main">
                      <span class="recipient">{{ address.recipient }}</span>
                      <span class="phone">{{ address.phone }}</span>
                      <el-tag v-if="address.isDefault" type="danger" size="small">默认</el-tag>
                    </div>
                    <div class="address-detail">
                      {{ address.province }}{{ address.city }}{{ address.district }}{{ address.detail }}
                    </div>
                  </div>
                  <div class="address-actions">
                    <el-switch
                      v-model="address.isDefault"
                      @change="setDefaultAddress(address.id)"
                      active-text="默认地址">
                    </el-switch>
                    <el-button type="text" @click="editAddress(address)">修改</el-button>
                    <el-button type="text" @click="deleteAddress(address.id)">删除</el-button>
                  </div>
                </div>
              </div>
              
              <!-- 新增/编辑地址对话框 -->
              <el-dialog :title="(editingAddress ? '编辑' : '新增') + '收货地址'" 
                         :visible.sync="showAddAddressDialog" 
                         width="600px">
                <el-form :model="addressForm" :rules="addressRules" ref="addressForm" label-width="100px">
                  <el-form-item label="所在地区" prop="region">
                    <el-cascader
                      v-model="addressForm.region"
                      :options="regionOptions"
                      placeholder="请选择省/市/区"
                      style="width: 100%;"
                    ></el-cascader>
                  </el-form-item>
                  <el-form-item label="详细地址" prop="detail">
                    <el-input v-model="addressForm.detail" placeholder="请输入详细地址"></el-input>
                  </el-form-item>
                  <el-form-item label="收货人" prop="recipient">
                    <el-input v-model="addressForm.recipient" placeholder="请输入收货人姓名"></el-input>
                  </el-form-item>
                  <el-form-item label="手机号码" prop="phone">
                    <el-input v-model="addressForm.phone" placeholder="请输入手机号码"></el-input>
                  </el-form-item>
                  <el-form-item>
                    <el-checkbox v-model="addressForm.isDefault">设置为默认收货地址</el-checkbox>
                  </el-form-item>
                </el-form>
                <div slot="footer" class="dialog-footer">
                  <el-button @click="showAddAddressDialog = false">取 消</el-button>
                  <el-button type="primary" @click="saveAddress">保 存</el-button>
                </div>
              </el-dialog>
            </div>
          </el-tab-pane>

          <!-- 余额充值 -->
          <el-tab-pane label="余额充值" name="recharge">
            <div class="recharge-section">
              <h3>余额充值</h3>
              <div class="recharge-options">
                <div class="amount-options">
                  <div v-for="amount in [50, 100, 200, 500, 1000]" :key="amount"
                       class="amount-option" 
                       :class="{ 'selected': selectedAmount === amount }"
                       @click="selectedAmount = amount">
                    ¥{{ amount }}
                  </div>
                </div>
                <div class="custom-amount">
                  <span>自定义金额：</span>
                  <el-input v-model="customAmount" placeholder="请输入充值金额" style="width: 200px;">
                    <template slot="prepend">¥</template>
                  </el-input>
                </div>
                <div class="recharge-action">
                  <el-button type="primary" size="large" @click="recharge">立即充值</el-button>
                  <el-button @click="selectedAmount = null">退款</el-button>
                </div>
              </div>
            </div>
          </el-tab-pane>

          <!-- 我的收藏 -->
          <el-tab-pane label="我的收藏" name="favorites">
            <div class="favorites-management">
              <div class="favorites-header">
                <h3>我的收藏</h3>
                <div class="batch-actions">
                  <el-button @click="toggleBatchMode" :type="batchMode ? 'info' : ''">
                    {{ batchMode ? '取消批量管理' : '批量管理' }}
                  </el-button>
                  <el-button v-if="batchMode" @click="selectAll">
                    {{ isAllSelected ? '取消全选' : '全选' }}
                  </el-button>
                  <el-button v-if="batchMode" type="danger" @click="batchRemoveFavorites">
                    取消收藏
                  </el-button>
                </div>
              </div>
              
              <!-- 收藏商品列表 -->
              <div class="favorites-list">
                <div v-for="item in favorites" :key="item.id" class="favorite-item">
                  <div class="favorite-check" v-if="batchMode">
                    <el-checkbox v-model="item.checked"></el-checkbox>
                  </div>
                  <div class="favorite-content" @click="goToProductDetail(item.id)">
                    <div class="product-image">
                      <img :src="item.image" :alt="item.name">
                    </div>
                    <div class="product-info">
                      <h4 class="product-name">{{ item.name }}</h4>
                      <p class="product-price">¥{{ item.price }}</p>
                    </div>
                  </div>
                  <div class="favorite-actions">
                    <el-button type="text" @click.stop="removeFavorite(item.id)">取消收藏</el-button>
                    <el-button type="text" @click.stop="goToProductDetail(item.id)">查看详情</el-button>
                  </div>
                </div>
              </div>
              
              <el-empty description="暂无收藏商品" v-if="favorites.length === 0"></el-empty>
            </div>
          </el-tab-pane>

          <!-- 我的浏览 -->
          <el-tab-pane label="我的浏览" name="browse">
            <div class="browse-history">
              <div class="browse-header">
                <h3>最近浏览</h3>
                <el-button type="text" @click="clearBrowseHistory">清空浏览记录</el-button>
              </div>
              <div class="browse-list">
                <div v-for="item in browseHistory" :key="item.id" class="browse-item">
                  <div class="product-image">
                    <img :src="item.image" :alt="item.name">
                  </div>
                  <div class="product-info">
                    <h4 class="product-name">{{ item.name }}</h4>
                    <p class="product-price">¥{{ item.price }}</p>
                  </div>
                  <div class="browse-actions">
                    <el-button size="mini" @click="goToProductDetail(item.id)">查看详情</el-button>
                    <el-button size="mini" type="primary" @click="addToCart(item)">加入购物车</el-button>
                  </div>
                </div>
              </div>
              <el-empty description="暂无浏览记录" v-if="browseHistory.length === 0"></el-empty>
            </div>
          </el-tab-pane>
        </el-tabs>
      </div>
    </div>
  </div>
</template>

<script>
export default {
  name: 'UserCenter',
  data() {
    return {
      activeTab: 'profile',
      user: {
        id: '10001',
        nickname: '用户昵称',
        avatar: '',
        balance: 0,
        phone: '138****5678'
      },
      profileForm: {
        phone: '',
        oldPassword: '',
        newPassword: ''
      },
      profileRules: {
        phone: [
          { required: true, message: '请输入手机号', trigger: 'blur' },
          { pattern: /^1[3-9]\d{9}$/, message: '手机号格式不正确', trigger: 'blur' }
        ],
        oldPassword: [
          { required: true, message: '请输入原密码', trigger: 'blur' }
        ],
        newPassword: [
          { required: true, message: '请输入新密码', trigger: 'blur' },
          { min: 6, message: '密码长度不能少于6位', trigger: 'blur' }
        ]
      },
      // 地址管理相关数据
      addresses: [],
      showAddAddressDialog: false,
      editingAddress: null,
      addressForm: {
        region: [],
        detail: '',
        recipient: '',
        phone: '',
        isDefault: false
      },
      addressRules: {
        region: [
          { required: true, message: '请选择所在地区', trigger: 'change' }
        ],
        detail: [
          { required: true, message: '请输入详细地址', trigger: 'blur' }
        ],
        recipient: [
          { required: true, message: '请输入收货人姓名', trigger: 'blur' }
        ],
        phone: [
          { required: true, message: '请输入手机号码', trigger: 'blur' },
          { pattern: /^1[3-9]\d{9}$/, message: '手机号格式不正确', trigger: 'blur' }
        ]
      },
      regionOptions: [
        {
          value: 'guangdong',
          label: '广东省',
          children: [
            {
              value: 'guangzhou',
              label: '广州市',
              children: [
                { value: 'haizhu', label: '海珠区' },
                { value: 'liwan', label: '荔湾区' },
                { value: 'tianhe', label: '天河区' }
              ]
            },
            {
              value: 'shenzhen',
              label: '深圳市',
              children: [
                { value: 'nanshan', label: '南山区' },
                { value: 'futian', label: '福田区' },
                { value: 'luohu', label: '罗湖区' }
              ]
            }
          ]
        },
        {
          value: 'jiangsu',
          label: '江苏省',
          children: [
            {
              value: 'nanjing',
              label: '南京市',
              children: [
                { value: 'qianjiang', label: '前江' },
                { value: 'xinhua', label: '秦淮' },
                { value: 'yuhuai', label: '雨花' }
              ]
            }
          ]
        },
        {
          value: 'anhui',
          label: '安徽省',
          children: [
            {
              value: 'hefei',
              label: '合肥市',
              children: [
                { value: 'xinhua', label: '秦淮' },
                { value: 'yuhuai', label: '雨花' },
                { value: 'qianjiang', label: '前江' }
              ]
            }
          ]
        }
      ],
      // 充值相关数据
      selectedAmount: 0,
      customAmount: '',
      // 收藏相关数据
      favorites: [],
      batchMode: false,
      // 浏览历史
      browseHistory: [],
      // 订单数据
      orders: []
    };
  },
  computed: {
    isAllSelected() {
      return this.favorites.length > 0 && this.favorites.every(item => item.checked);
    }
  },
  mounted() {
    this.loadUserData();
    this.loadAddresses();
    this.loadFavorites();
    this.loadBrowseHistory();
  },
  methods: {
    // 返回首页
    goToHome() {
      this.$router.push('/home');
    },
    
    // 加载用户数据
    loadUserData() {
      // 模拟API调用
      this.profileForm.phone = this.user.phone;
    },
    
    // 更新手机号
    updatePhone() {
      this.$refs.profileForm.validateField('phone', (valid) => {
        if (valid) {
          // 调用API更新手机号
          this.$message.success('手机号更新成功');
        }
      });
    },
    
    // 更新密码
    updatePassword() {
      this.$refs.profileForm.validate((valid) => {
        if (valid) {
          // 调用API更新密码
          this.$message.success('密码修改成功');
          this.profileForm.oldPassword = '';
          this.profileForm.newPassword = '';
        }
      });
    },
    
    // 地址管理方法
    loadAddresses() {
      // 模拟API调用获取地址列表
      this.addresses = [
        {
          id: 1,
          recipient: '张三',
          phone: '13800138000',
          province: '广东省',
          city: '广州市',
          district: '天河区',
          detail: '某某街道某某号',
          isDefault: false
        },
        {
          id: 2,
          recipient: '李四',
          phone: '13900139000',
          province: '广东省',
          city: '深圳市',
          district: '南山区',
          detail: '科技园南区',
          isDefault: false
        }
      ];
    },
    closeAddressDialog() {
      this.showAddAddressDialog = false;
      this.resetAddressForm();
    },
    
    // 保存地址
    saveAddress() {
  this.$refs.addressForm.validate((valid) => {
    if (valid) {
      // 验证地区选择
      if (this.addressForm.region.length !== 3) {
        this.$message.error('请选择完整的省市区');
        return;
      }
      
      // 调用API保存地址
      if (this.editingAddress) {
        // 更新地址
        this.$message.success('地址更新成功');
      } else {
        // 新增地址
        this.$message.success('地址添加成功');
      }
      this.closeAddressDialog();
      this.loadAddresses(); // 重新加载地址列表
    }
  });
},
    
    // 编辑地址
    editAddress(address) {
      this.editingAddress = address;
      this.addressForm = {
        region: [address.province, address.city, address.district],
        detail: address.detail,
        recipient: address.recipient,
        phone: address.phone,
        isDefault: address.isDefault
      };
      this.showAddAddressDialog = true;
    },
    
    // 删除地址
    deleteAddress(id) {
      this.$confirm('确定要删除这个地址吗？', '提示', {
        confirmButtonText: '确定',
        cancelButtonText: '取消',
        type: 'warning'
      }).then(() => {
        // 调用API删除地址
        console.log(`删除地址：${id}`);
        this.$message.success('地址删除成功');
        this.loadAddresses();
      });
    },
    
    // 设置默认地址
    setDefaultAddress(id) {
      // 调用API设置默认地址
      console.log(`设置默认地址：${id}`);
      this.$message.success('默认地址设置成功');
    },
    
    // 重置地址表单
    resetAddressForm() {
      this.addressForm = {
        region: [],
        detail: '',
        recipient: '',
        phone: '',
        isDefault: false
      };
      this.editingAddress = null;
    },
    
    // 充值
    recharge() {
      const amount = this.customAmount || this.selectedAmount;
      if (!amount || amount <= 0) {
        this.$message.error('请输入正确的充值金额');
        return;
      }
       // 转换为数字类型
  const rechargeAmount = parseFloat(amount);
  
  // 调用充值API
  this.$message.success(`成功充值¥${rechargeAmount}`);
  
  // 更新账户余额 - 添加这行代码
  this.user.balance = (parseFloat(this.user.balance) + rechargeAmount).toFixed(2);
  
  // 重置表单
  this.customAmount = '';
  this.selectedAmount = 0;
    },
    
    // 收藏管理方法
    loadFavorites() {
      // 模拟API调用获取收藏列表
      this.favorites = [
        {
          id: 1,
          name: '商品名称1',
          price: 199.00,
          image: 'https://img.yzcdn.cn/vant/t-thirt.jpg',
          checked: false
        },
        {
          id: 2,
          name: '商品名称2',
          price: 299.00,
          image: 'https://img.yzcdn.cn/vant/t-thirt.jpg',
          checked: false
        },
        {
          id: 3,
          name: '商品名称3',
          price: 399.00,
          image: 'https://img.yzcdn.cn/vant/t-thirt.jpg',
          checked: false
        }
      ];
    },
    
    // 切换批量管理模式
    toggleBatchMode() {
      this.batchMode = !this.batchMode;
      if (!this.batchMode) {
        // 退出批量模式时取消所有选中
        this.favorites.forEach(item => {
          item.checked = false;
        });
      }
    },
    
    // 全选/取消全选
    selectAll() {
      const selected = !this.isAllSelected;
      this.favorites.forEach(item => {
        item.checked = selected;
      });
    },
    
    // 批量取消收藏
    batchRemoveFavorites() {
      const selectedItems = this.favorites.filter(item => item.checked);
      if (selectedItems.length === 0) {
        this.$message.warning('请选择要取消收藏的商品');
        return;
      }
      
      this.$confirm(`确定要取消收藏这${selectedItems.length}件商品吗？`, '提示', {
        confirmButtonText: '确定',
        cancelButtonText: '取消',
        type: 'warning'
      }).then(() => {
        // 调用API批量取消收藏
        this.favorites = this.favorites.filter(item => !item.checked);
        this.$message.success('取消收藏成功');
      });
    },
    
    // 单个取消收藏
    removeFavorite(id) {
      this.$confirm('确定要取消收藏这个商品吗？', '提示', {
        confirmButtonText: '确定',
        cancelButtonText: '取消',
        type: 'warning'
      }).then(() => {
        // 调用API取消收藏
        console.log(`取消收藏商品ID：${id}`);
        this.favorites = this.favorites.filter(item => item.id !== id);
        this.$message.success('取消收藏成功');
      });
    },
    
    // 加载浏览历史
    loadBrowseHistory() {
      // 模拟API调用获取浏览历史
      this.browseHistory = [
        {
          id: 1,
          name: '浏览商品1',
          price: 199.00,
          image: 'https://img.yzcdn.cn/vant/t-thirt.jpg'
        },
        {
          id: 2,
          name: '浏览商品2',
          price: 299.00,
          image: 'https://img.yzcdn.cn/vant/t-thirt.jpg'
        }
      ];
    },
    
    // 清空浏览历史
    clearBrowseHistory() {
      this.$confirm('确定要清空浏览记录吗？', '提示', {
        confirmButtonText: '确定',
        cancelButtonText: '取消',
        type: 'warning'
      }).then(() => {
        // 调用API清空浏览历史
        this.browseHistory = [];
        this.$message.success('浏览记录已清空');
      });
    },
    
    // 跳转到商品详情
    goToProductDetail(id) {
      this.$router.push(`/product/${id}`);
    },
    
    // 加入购物车
    addToCart(product) {
      // 调用API加入购物车
      console.log('加入购物车', product);
      this.$message.success('商品已加入购物车');
    }
  }
};
</script>

<style scoped>
#usercenter {
  background-color: #f5f7fa;
  min-height: 100vh;
  padding: 20px 0;
}

.user-container {
  max-width: 1200px;
  margin: 0 auto;
  background: #fff;
  border-radius: 8px;
  box-shadow: 0 2px 12px 0 rgba(0, 0, 0, 0.1);
  overflow: hidden;
}

.user-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding: 30px;
  background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
  color: white;
}

.avatar-section {
  display: flex;
  align-items: center;
}

.user-info {
  margin-left: 20px;
}

.user-info h3 {
  margin: 0 0 5px 0;
  font-size: 24px;
}

.user-info p {
  margin: 0;
  opacity: 0.8;
}

.balance-section {
  text-align: right;
}

.balance-section p {
  margin: 0 0 10px 0;
  opacity: 0.8;
}

.balance-section h2 {
  margin: 0 0 15px 0;
  font-size: 32px;
}

.main-content {
  padding: 20px;
}

/* 地址管理样式 */
.address-management {
  padding: 0 10px;
}

.address-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 20px;
}

.address-list {
  display: grid;
  grid-template-columns: repeat(auto-fill, minmax(300px, 1fr));
  gap: 20px;
}

.address-item {
  border: 1px solid #e0e0e0;
  border-radius: 8px;
  padding: 15px;
  position: relative;
}

.default-address {
  border-color: #ff6b6b;
  background-color: #fff5f5;
}

.address-info {
  margin-bottom: 15px;
}

.address-main {
  display: flex;
  align-items: center;
  margin-bottom: 8px;
}

.recipient {
  font-weight: bold;
  margin-right: 10px;
}

.phone {
  margin-right: 10px;
}

.address-detail {
  color: #666;
  line-height: 1.5;
}

.address-actions {
  display: flex;
  justify-content: space-between;
  align-items: center;
}

/* 收藏管理样式 */
.favorites-management {
  padding: 0 10px;
}

.favorites-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 20px;
}

.batch-actions {
  display: flex;
  gap: 10px;
}

.favorites-list {
  display: grid;
  grid-template-columns: repeat(auto-fill, minmax(200px, 1fr));
  gap: 20px;
}

.favorite-item {
  border: 1px solid #e0e0e0;
  border-radius: 8px;
  overflow: hidden;
  position: relative;
}

.favorite-check {
  position: absolute;
  top: 10px;
  left: 10px;
  z-index: 1;
}

.favorite-content {
  cursor: pointer;
}

.product-image {
  width: 100%;
  height: 180px;
  overflow: hidden;
}

.product-image img {
  width: 100%;
  height: 100%;
  object-fit: cover;
}

.product-info {
  padding: 10px;
}

.product-name {
  margin: 0 0 8px 0;
  font-size: 14px;
  height: 40px;
  overflow: hidden;
  display: -webkit-box;
  -webkit-line-clamp: 2;
  -webkit-box-orient: vertical;
}

.product-price {
  margin: 0;
  color: #ff6b6b;
  font-weight: bold;
}

.favorite-actions {
  display: flex;
  justify-content: space-between;
  padding: 10px;
  border-top: 1px solid #f0f0f0;
}

/* 浏览历史样式 */
.browse-history {
  padding: 0 10px;
}

.browse-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 20px;
}

.browse-list {
  display: grid;
  grid-template-columns: repeat(auto-fill, minmax(200px, 1fr));
  gap: 20px;
}

.browse-item {
  border: 1px solid #e0e0e0;
  border-radius: 8px;
  overflow: hidden;
}

.browse-actions {
  display: flex;
  justify-content: space-between;
  padding: 10px;
  border-top: 1px solid #f0f0f0;
}

/* 充值样式 */
.recharge-section {
  padding: 0 10px;
}

.recharge-options {
  max-width: 600px;
}

.amount-options {
  display: grid;
  grid-template-columns: repeat(3, 1fr);
  gap: 15px;
  margin-bottom: 20px;
}

.amount-option {
  border: 1px solid #e0e0e0;
  border-radius: 8px;
  padding: 15px;
  text-align: center;
  cursor: pointer;
  transition: all 0.3s;
}

.amount-option:hover {
  border-color: #409eff;
}

.amount-option.selected {
  border-color: #409eff;
  background-color: #ecf5ff;
}

.custom-amount {
  margin-bottom: 20px;
  display: flex;
  align-items: center;
  gap: 10px;
}

.recharge-action {
  text-align: center;
}

/* 响应式调整 */
@media (max-width: 768px) {
  .user-header {
    flex-direction: column;
    text-align: center;
  }
  
  .avatar-section {
    flex-direction: column;
    margin-bottom: 20px;
  }
  
  .user-info {
    margin-left: 0;
    margin-top: 15px;
  }
  
  .address-list,
  .favorites-list,
  .browse-list {
    grid-template-columns: repeat(auto-fill, minmax(150px, 1fr));
  }
  
  .amount-options {
    grid-template-columns: repeat(2, 1fr);
  }
}
</style>