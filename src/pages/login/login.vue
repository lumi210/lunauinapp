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
@import '../../styles/common.scss';

.page {
  min-height: 100vh;
  background: $color-bg;
}

.header {
  display: flex;
  align-items: center;
  padding: 24rpx;
  padding-top: calc(24rpx + constant(safe-area-inset-top));
  padding-top: calc(24rpx + env(safe-area-inset-top));
  background: rgba($color-bg-secondary, 0.85);
  backdrop-filter: blur(20px);
  -webkit-backdrop-filter: blur(20px);
  border-bottom: 1rpx solid $color-border;
}

.back {
  width: 64rpx;
  height: 64rpx;
  display: flex;
  align-items: center;
  justify-content: center;
  border-radius: 50%;
  transition: all $transition-fast;
  
  &:active {
    background: rgba(255, 255, 255, 0.1);
  }
}

.back-icon {
  width: 28rpx;
  height: 28rpx;
  background-image: $icon-back;
  background-size: contain;
  background-repeat: no-repeat;
  background-position: center;
}

.header-title {
  color: $color-text;
  font-size: 36rpx;
  font-weight: 700;
  margin-left: 8rpx;
}

.content {
  padding: 64rpx 32rpx;
}

.logo-section {
  text-align: center;
  margin-bottom: 72rpx;
}

.logo-icon {
  width: 120rpx;
  height: 120rpx;
  margin: 0 auto 24rpx;
  background: $gradient-primary;
  border-radius: 28rpx;
  display: flex;
  align-items: center;
  justify-content: center;
  box-shadow: $shadow-primary;
}

.app-name {
  color: $color-primary;
  font-size: 52rpx;
  font-weight: 700;
  letter-spacing: 2rpx;
  display: block;
}

.app-desc {
  color: $color-text-muted;
  font-size: 26rpx;
  margin-top: 16rpx;
  display: block;
}

.form {
  background: rgba($color-bg-secondary, 0.5);
  backdrop-filter: blur(12px);
  -webkit-backdrop-filter: blur(12px);
  border-radius: 24rpx;
  padding: 40rpx 32rpx;
  border: 1rpx solid $color-border;
  box-shadow: $shadow-card;
}

.form-item {
  margin-bottom: 36rpx;
}

.label {
  color: $color-text-secondary;
  font-size: 28rpx;
  font-weight: 500;
  margin-bottom: 16rpx;
  display: block;
}

.input {
  width: 100%;
  height: 96rpx;
  padding: 0 28rpx;
  background: $color-bg;
  border-radius: 16rpx;
  color: $color-text;
  font-size: 30rpx;
  box-sizing: border-box;
  border: 2rpx solid transparent;
  transition: all $transition-fast;
  
  &:focus {
    border-color: $color-primary;
    box-shadow: 0 0 0 4rpx rgba($color-primary, 0.15);
  }
}

.login-btn {
  width: 100%;
  height: 100rpx;
  background: $gradient-primary;
  border-radius: 50rpx;
  color: $color-text;
  font-size: 32rpx;
  font-weight: 600;
  border: none;
  margin-top: 24rpx;
  box-shadow: $shadow-primary;
  transition: all $transition-normal;
  
  &:active {
    transform: scale(0.98);
  }
}

.register-link {
  text-align: center;
  margin-top: 36rpx;
  padding: 16rpx;
  border-radius: 16rpx;
  transition: all $transition-fast;
  
  &:active {
    background: rgba(255, 255, 255, 0.05);
  }
}

.register-link text {
  color: $color-primary;
  font-size: 28rpx;
  font-weight: 500;
}
</style>
