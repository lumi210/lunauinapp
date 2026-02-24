<template>
  <view class="page">
    <view class="header">
      <view class="back" @click="goBack">
        <view class="back-icon"></view>
      </view>
      <text class="header-title">登录</text>
    </view>

    <view class="content">
      <view class="logo-section">
        <view class="logo-icon"></view>
        <text class="app-name">{{ siteName }}</text>
        <text class="app-desc">影视聚合播放平台</text>
      </view>

      <view class="form">
        <view class="form-item">
          <text class="label">用户名</text>
          <input class="input" v-model="username" placeholder="请输入用户名" type="text" />
        </view>
        <view class="form-item">
          <text class="label">密码</text>
          <input class="input" v-model="password" placeholder="请输入密码" :password="true" />
        </view>

        <button class="login-btn" :loading="loading" @click="handleLogin">登录</button>

        <view class="register-link" @click="goRegister">
          <text>没有账号？立即注册</text>
        </view>
      </view>
    </view>
  </view>
</template>

<script>
import { getApiUrl } from '../../utils/config'

export default {
  data() {
    return {
      siteName: 'LunaTV',
      username: '',
      password: '',
      loading: false
    }
  },
  onShow() {
    const cached = uni.getStorageSync('siteName')
    if (cached) {
      this.siteName = cached
    }
  },
  methods: {
    goBack() {
      uni.navigateBack()
    },
    goRegister() {
      uni.navigateTo({ url: '/pages/register/register' })
    },
    handleLogin() {
      if (!this.username) {
        uni.showToast({ title: '请输入用户名', icon: 'none' })
        return
      }
      if (!this.password) {
        uni.showToast({ title: '请输入密码', icon: 'none' })
        return
      }

      this.loading = true

      uni.request({
        url: getApiUrl('/api/login'),
        method: 'POST',
        data: {
          username: this.username,
          password: this.password
        },
        withCredentials: true,
        success: (res) => {
          console.log('[Login] response:', res.statusCode, res.data)
          if (res.statusCode === 200 && res.data && res.data.ok) {
            uni.setStorageSync('userInfo', { username: this.username })
            const headers = res.header || {}
            const setCookie = headers['set-cookie'] || headers['Set-Cookie']
            if (setCookie) {
              let cookieValue = ''
              if (Array.isArray(setCookie)) {
                cookieValue = setCookie.map(c => c.split(';')[0]).join('; ')
              } else {
                cookieValue = setCookie.split(';')[0]
              }
              uni.setStorageSync('user_cookie', cookieValue)
            } else {
              uni.setStorageSync('user_cookie', 'logged_in')
            }
            uni.showToast({ title: '登录成功', icon: 'success' })
            setTimeout(() => {
              uni.switchTab({ url: '/pages/index/index' })
            }, 1000)
          } else {
            uni.showToast({ title: res.data?.error || '登录失败', icon: 'none' })
          }
        },
        fail: (err) => {
          console.error('[Login] failed:', err)
          uni.showToast({ title: '网络错误', icon: 'none' })
        },
        complete: () => {
          this.loading = false
        }
      })
    }
  }
}
</script>

<style lang="scss">
@import '../../styles/design-system.scss';

.page {
  min-height: 100vh;
  background: $bg-base;
  display: flex;
  flex-direction: column;
}

/* ============================================
   Header - 玻璃态导航
   ============================================ */

.header {
  position: relative;
  display: flex;
  align-items: center;
  padding: $space-6;
  padding-top: calc(#{$space-6} + constant(safe-area-inset-top));
  padding-top: calc(#{$space-6} + env(safe-area-inset-top));
}

.back {
  position: relative;
  width: 72rpx;
  height: 72rpx;
  display: flex;
  align-items: center;
  justify-content: center;
  border-radius: $radius-full;
  transition: all $duration-fast $ease-out;
  
  &:active {
    background: rgba(255, 255, 255, 0.08);
    transform: scale(0.95);
  }
}

.back-icon {
  width: 32rpx;
  height: 32rpx;
  background-image: $icon-back;
  background-size: contain;
  background-repeat: no-repeat;
  background-position: center;
}

.header-title {
  position: relative;
  font-size: 36rpx;
  font-weight: 700;
  color: $text-primary;
  margin-left: $space-2;
}

/* ============================================
   Logo Section
   ============================================ */

.content {
  flex: 1;
  display: flex;
  flex-direction: column;
  padding: $space-10 $space-6;
}

.logo-section {
  text-align: center;
  margin-bottom: $space-16;
}

.logo-icon {
  width: 120rpx;
  height: 120rpx;
  margin: 0 auto $space-6;
  background: $gradient-primary;
  border-radius: $radius-xl;
  display: flex;
  align-items: center;
  justify-content: center;
  box-shadow: $shadow-primary;
  animation: float 3s ease-in-out infinite;
}

.app-name {
  display: block;
  font-size: 52rpx;
  font-weight: 700;
  color: $brand-primary;
  letter-spacing: 2rpx;
}

.app-desc {
  display: block;
  margin-top: $space-4;
  font-size: 26rpx;
  color: $text-muted;
}

@keyframes float {
  0%, 100% { transform: translateY(0); }
  50% { transform: translateY(-8rpx); }
}

/* ============================================
   Form - 玻璃态卡片
   ============================================ */

.form {
  background: rgba(255, 255, 255, 0.02);
  backdrop-filter: blur(20px) saturate(150%);
  -webkit-backdrop-filter: blur(20px) saturate(150%);
  border-radius: $radius-2xl;
  padding: $space-8 $space-6;
  border: 1rpx solid rgba(255, 255, 255, 0.04);
}

.form-item {
  margin-bottom: $space-6;
}

.label {
  display: block;
  margin-bottom: $space-3;
  font-size: 26rpx;
  font-weight: 500;
  color: $text-secondary;
}

.input {
  width: 100%;
  height: 100rpx;
  padding: 0 $space-6;
  background: rgba(255, 255, 255, 0.04);
  border: 2rpx solid transparent;
  border-radius: $radius-lg;
  font-size: 30rpx;
  color: $text-primary;
  box-sizing: border-box;
  transition: all $duration-fast $ease-out;
  
  &::placeholder {
    color: $text-muted;
  }
  
  &:focus {
    background: rgba(255, 255, 255, 0.06);
    border-color: $brand-primary;
    box-shadow: 0 0 0 6rpx rgba(225, 29, 72, 0.15);
  }
}

.login-btn {
  width: 100%;
  height: 100rpx;
  margin-top: $space-4;
  background: $gradient-primary;
  border-radius: $radius-full;
  border: none;
  font-size: 32rpx;
  font-weight: 600;
  color: white;
  box-shadow: $shadow-primary;
  transition: all $duration-fast $ease-out;
  
  &:active {
    transform: scale(0.98);
    box-shadow: 0 4rpx 20rpx rgba(225, 29, 72, 0.5);
  }
}

.register-link {
  text-align: center;
  margin-top: $space-8;
  padding: $space-4;
  border-radius: $radius-lg;
  transition: all $duration-fast $ease-out;
  
  &:active {
    background: rgba(255, 255, 255, 0.04);
  }
}

.register-link text {
  font-size: 28rpx;
  font-weight: 500;
  color: $brand-primary-light;
}
</style>
