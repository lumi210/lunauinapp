<template>
  <view class="page">
    <view class="header">
      <text class="header-title">我的</text>
    </view>

    <scroll-view scroll-y class="content">
      <view class="user-card" v-if="userInfo">
        <view class="user-header">
          <text class="username">{{ userInfo.username }}</text>
          <view class="vip-badge" v-if="cardKeyInfo && !cardKeyInfo.isExpired">
            <text>VIP</text>
          </view>
        </view>
        
        <view class="cardkey-info" v-if="cardKeyInfo">
          <view class="cardkey-row">
            <text class="cardkey-label">会员状态</text>
            <text class="cardkey-value" :class="{ active: !cardKeyInfo.isExpired }">
              {{ cardKeyInfo.isExpired ? '已过期' : '有效' }}
            </text>
          </view>
          <view class="cardkey-row" v-if="cardKeyInfo.expiresAt">
            <text class="cardkey-label">到期时间</text>
            <text class="cardkey-value">{{ formatTime(cardKeyInfo.expiresAt) }}</text>
          </view>
          <view class="cardkey-row" v-if="cardKeyInfo.daysRemaining !== undefined && !cardKeyInfo.isExpired">
            <text class="cardkey-label">剩余天数</text>
            <text class="cardkey-value">{{ cardKeyInfo.daysRemaining }} 天</text>
          </view>
        </view>
        
        <view class="stats">
          <view class="stat-item" @click="goPage('/pages/points/points')">
            <text class="stat-value">{{ points }}</text>
            <text class="stat-label">积分</text>
          </view>
          <view class="stat-item" @click="goPage('/pages/favorites/favorites')">
            <text class="stat-value">{{ favoriteCount }}</text>
            <text class="stat-label">收藏</text>
          </view>
          <view class="stat-item" @click="goPage('/pages/history/history')">
            <text class="stat-value">{{ playCount }}</text>
            <text class="stat-label">观看</text>
          </view>
        </view>
      </view>
      <view class="user-card" v-else @click="goLogin">
        <text class="login-text">点击登录</text>
      </view>

      <view class="menu-section">
        <text class="section-title">会员中心</text>
        <view class="menu-list">
          <view class="menu-item" @click="goPage('/pages/points/points')">
            <view class="menu-icon menu-icon-coin"></view>
            <text>积分兑换</text>
            <view class="item-extra">
              <view class="arrow-icon"></view>
            </view>
          </view>
          <view class="menu-item" @click="goPage('/pages/invitation/invitation')">
            <view class="menu-icon menu-icon-share"></view>
            <text>邀请好友</text>
            <view class="item-extra">
              <view class="arrow-icon"></view>
            </view>
          </view>
          <view class="menu-item" @click="goPage('/pages/cardkey/cardkey')">
            <view class="menu-icon menu-icon-key"></view>
            <text>卡密充值</text>
            <view class="item-extra">
              <view class="arrow-icon"></view>
            </view>
          </view>
        </view>
      </view>

      <view class="menu-section">
        <text class="section-title">我的内容</text>
        <view class="menu-list">
          <view class="menu-item" @click="goPage('/pages/favorites/favorites')">
            <view class="menu-icon menu-icon-bookmark"></view>
            <text>我的收藏</text>
            <view class="item-extra">
              <text class="item-count" v-if="favoriteCount > 0">{{ favoriteCount }}</text>
              <view class="arrow-icon"></view>
            </view>
          </view>
          <view class="menu-item" @click="goPage('/pages/history/history')">
            <view class="menu-icon menu-icon-clock"></view>
            <text>观看历史</text>
            <view class="item-extra">
              <text class="item-count" v-if="playCount > 0">{{ playCount }}</text>
              <view class="arrow-icon"></view>
            </view>
          </view>
        </view>
      </view>

      <view class="menu-section">
        <text class="section-title">设置</text>
        <view class="menu-list">
          <view class="menu-item" @click="goPage('/pages/settings/settings')">
            <view class="menu-icon menu-icon-settings"></view>
            <text>设置</text>
            <view class="item-extra">
              <view class="arrow-icon"></view>
            </view>
          </view>
          <view class="menu-item" v-if="userInfo" @click="changePassword">
            <view class="menu-icon menu-icon-lock"></view>
            <text>修改密码</text>
            <view class="item-extra">
              <view class="arrow-icon"></view>
            </view>
          </view>
        </view>
      </view>

      <view class="logout-btn" v-if="userInfo" @click="logout">
        <view class="logout-icon"></view>
        <text>退出登录</text>
      </view>
      
      <view class="safe-area-bottom"></view>
    </scroll-view>
  </view>
</template>

<script>
import { buildUrl } from "../../utils/request"
export default {
  data() {
    return {
      userInfo: null,
      points: 0,
      favoriteCount: 0,
      playCount: 0,
      cardKeyInfo: null
    }
  },
  onShow() {
    this.userInfo = uni.getStorageSync('userInfo')
    if (this.userInfo) {
      this.loadUserData()
    }
  },
  methods: {
    goLogin() {
      uni.navigateTo({ url: '/pages/login/login' })
    },
    goPage(url) {
      if (!this.userInfo && url !== '/pages/settings/settings') {
        uni.navigateTo({ url: '/pages/login/login' })
        return
      }
      uni.navigateTo({ url: url })
    },
    loadUserData() {
      // 加载用户统计
      uni.request({
        url: buildUrl('/api/user/my-stats'),
        withCredentials: true,
        success: (res) => {
          console.log('my-stats:', res.data)
          if (res.data && !res.data.error) {
            this.playCount = res.data.totalPlays || 0
          }
        }
      })
      
      // 加载积分
      uni.request({
        url: buildUrl('/api/points/balance'),
        withCredentials: true,
        success: (res) => {
          console.log('points-balance:', res.data)
          if (res.data && res.data.balance !== undefined) {
            this.points = res.data.balance
          }
        }
      })
      
      // 加载收藏
      uni.request({
        url: buildUrl('/api/favorites'),
        withCredentials: true,
        success: (res) => {
          console.log('favorites:', res.data)
          if (res.data) {
            if (Array.isArray(res.data)) {
              this.favoriteCount = res.data.length
            } else if (res.data.list) {
              this.favoriteCount = res.data.list.length
            } else if (res.data.favorites) {
              this.favoriteCount = res.data.favorites.length
            } else {
              this.favoriteCount = Object.keys(res.data).length
            }
          }
        }
      })
      
      // 加载观看记录数量
      uni.request({
        url: buildUrl('/api/playrecords'),
        withCredentials: true,
        success: (res) => {
          console.log('playrecords:', res.data)
          if (res.data) {
            if (Array.isArray(res.data)) {
              this.playCount = res.data.length
            } else if (res.data.list) {
              this.playCount = res.data.list.length
            } else if (res.data.records) {
              this.playCount = Array.isArray(res.data.records) ? res.data.records.length : Object.keys(res.data.records).length
            } else if (res.data.data && Array.isArray(res.data.data)) {
              this.playCount = res.data.data.length
            } else {
              this.playCount = Object.keys(res.data).length
            }
          }
        }
      })
      
      // 加载卡密信息
      uni.request({
        url: buildUrl('/api/user/cardkey'),
        withCredentials: true,
        success: (res) => {
          console.log('cardkey:', res.data)
          if (res.data && res.data.hasCardKey && res.data.cardKeyInfo) {
            this.cardKeyInfo = res.data.cardKeyInfo
          }
        }
      })
    },
    formatTime(timestamp) {
      if (!timestamp) return ''
      const ts = timestamp > 9999999999 ? timestamp : timestamp * 1000
      const date = new Date(ts)
      const y = date.getFullYear()
      const m = String(date.getMonth() + 1).padStart(2, '0')
      const d = String(date.getDate()).padStart(2, '0')
      const h = String(date.getHours()).padStart(2, '0')
      const min = String(date.getMinutes()).padStart(2, '0')
      return `${y}-${m}-${d} ${h}:${min}`
    },
    changePassword() {
      uni.showModal({
        title: '修改密码',
        editable: true,
        placeholderText: '请输入新密码',
        success: (modalRes) => {
          if (modalRes.confirm && modalRes.content) {
            uni.request({
              url: buildUrl('/api/change-password'),
              method: 'POST',
              data: { newPassword: modalRes.content },
              withCredentials: true,
              success: (res) => {
                if (res.data && res.data.ok) {
                  uni.showToast({ title: '修改成功', icon: 'success' })
                } else {
                  uni.showToast({ title: res.data?.error || '修改失败', icon: 'none' })
                }
              },
              fail: () => {
                uni.showToast({ title: '请求失败', icon: 'none' })
              }
            })
          }
        }
      })
    },
    logout() {
      uni.showModal({
        title: '提示',
        content: '确定退出登录？',
        success: (res) => {
          if (res.confirm) {
            uni.removeStorageSync('userInfo')
            uni.removeStorageSync('user_cookie')
            this.userInfo = null
            uni.showToast({ title: '已退出' })
            setTimeout(() => {
              uni.navigateTo({ url: '/pages/login/login' })
            }, 500)
          }
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
   Header
   ============================================ */

.header {
  position: relative;
  padding: $space-6;
  padding-top: calc(#{$space-6} + constant(safe-area-inset-top));
  padding-top: calc(#{$space-6} + env(safe-area-inset-top));
}

.header::before {
  content: '';
  position: absolute;
  inset: 0;
  background: rgba(10, 10, 15, 0.8);
  backdrop-filter: blur(30px) saturate(180%);
  -webkit-backdrop-filter: blur(30px) saturate(180%);
}

.header-title {
  position: relative;
  font-size: 38rpx;
  font-weight: 700;
  color: $text-primary;
}

/* ============================================
   Content
   ============================================ */

.content {
  flex: 1;
  padding: $space-6;
}

/* ============================================
   User Card
   ============================================ */

.user-card {
  background: rgba(255, 255, 255, 0.02);
  backdrop-filter: blur(20px) saturate(150%);
  -webkit-backdrop-filter: blur(20px) saturate(150%);
  border-radius: $radius-2xl;
  padding: $space-8;
  margin-bottom: $space-8;
  border: 1rpx solid rgba(255, 255, 255, 0.04);
}

.user-header {
  display: flex;
  align-items: center;
  margin-bottom: $space-5;
}

.username {
  font-size: 40rpx;
  font-weight: 700;
  color: $text-primary;
  letter-spacing: 0.5rpx;
}

.vip-badge {
  margin-left: $space-4;
  padding: $space-1 $space-4;
  background: $gradient-gold;
  border-radius: $radius-full;
  box-shadow: $shadow-gold;
  
  text {
    font-size: 22rpx;
    font-weight: 700;
    color: $bg-base;
  }
}

.cardkey-info {
  padding: $space-5;
  background: rgba(251, 191, 36, 0.06);
  border-radius: $radius-lg;
  margin-bottom: $space-5;
  border: 1rpx solid rgba(251, 191, 36, 0.1);
}

.cardkey-row {
  display: flex;
  justify-content: space-between;
  padding: $space-2 0;
}

.cardkey-label {
  font-size: 26rpx;
  color: $text-secondary;
}

.cardkey-value {
  font-size: 26rpx;
  font-weight: 500;
  color: $text-primary;
  
  &.active {
    color: $accent-emerald;
  }
}

.login-text {
  display: block;
  padding: $space-5;
  font-size: 32rpx;
  font-weight: 500;
  color: $brand-primary-light;
  text-align: center;
}

/* ============================================
   Stats
   ============================================ */

.stats {
  display: flex;
  margin-top: $space-4;
  background: rgba(255, 255, 255, 0.02);
  border-radius: $radius-lg;
  padding: $space-2;
}

.stat-item {
  flex: 1;
  text-align: center;
  padding: $space-5 $space-4;
  border-radius: $radius-md;
  transition: all $duration-fast $ease-out;
  
  &:active {
    background: rgba(255, 255, 255, 0.04);
  }
}

.stat-value {
  display: block;
  font-size: 40rpx;
  font-weight: 700;
  color: $text-primary;
}

.stat-label {
  display: block;
  margin-top: $space-1;
  font-size: 24rpx;
  color: $text-muted;
}

/* ============================================
   Menu Sections
   ============================================ */

.menu-section {
  margin-bottom: $space-8;
}

.section-title {
  display: block;
  margin-bottom: $space-4;
  margin-left: $space-2;
  font-size: 24rpx;
  font-weight: 500;
  color: $text-muted;
  letter-spacing: 0.5rpx;
}

.menu-list {
  background: rgba(255, 255, 255, 0.02);
  backdrop-filter: blur(10px);
  -webkit-backdrop-filter: blur(10px);
  border-radius: $radius-xl;
  border: 1rpx solid rgba(255, 255, 255, 0.04);
  overflow: hidden;
}

.menu-item {
  display: flex;
  align-items: center;
  padding: $space-5 $space-6;
  border-bottom: 1rpx solid rgba(255, 255, 255, 0.03);
  transition: all $duration-fast $ease-out;
  
  &:last-child {
    border-bottom: none;
  }
  
  &:active {
    background: rgba(255, 255, 255, 0.03);
  }
  
  text {
    font-size: 30rpx;
    font-weight: 500;
    color: $text-primary;
  }
}

.menu-icon {
  width: 44rpx;
  height: 44rpx;
  margin-right: $space-5;
  background-size: contain;
  background-repeat: no-repeat;
  background-position: center;
  opacity: 0.85;
  
  &.menu-icon-coin {
    background-image: url("data:image/svg+xml,%3Csvg xmlns='http://www.w3.org/2000/svg' width='24' height='24' viewBox='0 0 24 24' fill='none' stroke='%23FBBF24' stroke-width='2' stroke-linecap='round' stroke-linejoin='round'%3E%3Ccircle cx='12' cy='12' r='10'/%3E%3Cpath d='M16 8h-6a2 2 0 1 0 0 4h4a2 2 0 1 1 0 4H8'/%3E%3Cpath d='M12 18V6'/%3E%3C/svg%3E");
  }
  
  &.menu-icon-share {
    background-image: url("data:image/svg+xml,%3Csvg xmlns='http://www.w3.org/2000/svg' width='24' height='24' viewBox='0 0 24 24' fill='none' stroke='%2322D3EE' stroke-width='2' stroke-linecap='round' stroke-linejoin='round'%3E%3Ccircle cx='18' cy='5' r='3'/%3E%3Ccircle cx='6' cy='12' r='3'/%3E%3Ccircle cx='18' cy='19' r='3'/%3E%3Cline x1='8.59' y1='13.51' x2='15.42' y2='17.49'/%3E%3Cline x1='15.41' y1='6.51' x2='8.59' y2='10.49'/%3E%3C/svg%3E");
  }
  
  &.menu-icon-key {
    background-image: url("data:image/svg+xml,%3Csvg xmlns='http://www.w3.org/2000/svg' width='24' height='24' viewBox='0 0 24 24' fill='none' stroke='%23E11D48' stroke-width='2' stroke-linecap='round' stroke-linejoin='round'%3E%3Cpath d='M21 2l-2 2m-7.61 7.61a5.5 5.5 0 1 1-7.778 7.778 5.5 5.5 0 0 1 7.777-7.777zm0 0L15.5 7.5m0 0l3 3L22 7l-3-3m-3.5 3.5L19 4'/%3E%3C/svg%3E");
  }
  
  &.menu-icon-bookmark {
    background-image: url("data:image/svg+xml,%3Csvg xmlns='http://www.w3.org/2000/svg' width='24' height='24' viewBox='0 0 24 24' fill='none' stroke='%23E11D48' stroke-width='2' stroke-linecap='round' stroke-linejoin='round'%3E%3Cpath d='M19 21l-7-5-7 5V5a2 2 0 0 1 2-2h10a2 2 0 0 1 2 2z'/%3E%3C/svg%3E");
  }
  
  &.menu-icon-clock {
    background-image: url("data:image/svg+xml,%3Csvg xmlns='http://www.w3.org/2000/svg' width='24' height='24' viewBox='0 0 24 24' fill='none' stroke='%2322D3EE' stroke-width='2' stroke-linecap='round' stroke-linejoin='round'%3E%3Ccircle cx='12' cy='12' r='10'/%3E%3Cpolyline points='12 6 12 12 16 14'/%3E%3C/svg%3E");
  }
  
  &.menu-icon-settings {
    background-image: url("data:image/svg+xml,%3Csvg xmlns='http://www.w3.org/2000/svg' width='24' height='24' viewBox='0 0 24 24' fill='none' stroke='%2371717A' stroke-width='2' stroke-linecap='round' stroke-linejoin='round'%3E%3Ccircle cx='12' cy='12' r='3'/%3E%3Cpath d='M19.4 15a1.65 1.65 0 0 0 .33 1.82l.06.06a2 2 0 0 1 0 2.83 2 2 0 0 1-2.83 0l-.06-.06a1.65 1.65 0 0 0-1.82-.33 1.65 1.65 0 0 0-1 1.51V21a2 2 0 0 1-2 2 2 2 0 0 1-2-2v-.09A1.65 1.65 0 0 0 9 19.4a1.65 1.65 0 0 0-1.82.33l-.06.06a2 2 0 0 1-2.83 0 2 2 0 0 1 0-2.83l.06-.06a1.65 1.65 0 0 0 .33-1.82 1.65 1.65 0 0 0-1.51-1H3a2 2 0 0 1-2-2 2 2 0 0 1 2-2h.09A1.65 1.65 0 0 0 4.6 9a1.65 1.65 0 0 0-.33-1.82l-.06-.06a2 2 0 0 1 0-2.83 2 2 0 0 1 2.83 0l.06.06a1.65 1.65 0 0 0 1.82.33H9a1.65 1.65 0 0 0 1-1.51V3a2 2 0 0 1 2-2 2 2 0 0 1 2 2v.09a1.65 1.65 0 0 0 1 1.51 1.65 1.65 0 0 0 1.82-.33l.06-.06a2 2 0 0 1 2.83 0 2 2 0 0 1 0 2.83l-.06.06a1.65 1.65 0 0 0-.33 1.82V9a1.65 1.65 0 0 0 1.51 1H21a2 2 0 0 1 2 2 2 2 0 0 1-2 2h-.09a1.65 1.65 0 0 0-1.51 1z'/%3E%3C/svg%3E");
  }
  
  &.menu-icon-lock {
    background-image: url("data:image/svg+xml,%3Csvg xmlns='http://www.w3.org/2000/svg' width='24' height='24' viewBox='0 0 24 24' fill='none' stroke='%2371717A' stroke-width='2' stroke-linecap='round' stroke-linejoin='round'%3E%3Crect x='3' y='11' width='18' height='11' rx='2' ry='2'/%3E%3Cpath d='M7 11V7a5 5 0 0 1 10 0v4'/%3E%3C/svg%3E");
  }
}

.item-extra {
  display: flex;
  align-items: center;
  margin-left: auto;
}

.item-count {
  margin-right: $space-4;
  padding: $space-1 $space-3;
  background: rgba(255, 255, 255, 0.06);
  border-radius: $radius-full;
  font-size: 24rpx;
  font-weight: 500;
  color: $text-muted;
}

.arrow-icon {
  width: 28rpx;
  height: 28rpx;
  background-image: $icon-arrow-right;
  background-size: contain;
  background-repeat: no-repeat;
  background-position: center;
  opacity: 0.4;
}

/* ============================================
   Logout Button
   ============================================ */

.logout-btn {
  display: flex;
  align-items: center;
  justify-content: center;
  gap: $space-3;
  margin-top: $space-12;
  padding: $space-5;
  background: rgba(255, 255, 255, 0.02);
  backdrop-filter: blur(10px);
  -webkit-backdrop-filter: blur(10px);
  border-radius: $radius-xl;
  border: 1rpx solid rgba(255, 255, 255, 0.04);
  transition: all $duration-fast $ease-out;
  
  &:active {
    background: rgba(225, 29, 72, 0.08);
    transform: scale(0.98);
  }
  
  text {
    font-size: 30rpx;
    font-weight: 500;
    color: $brand-primary-light;
  }
}

.logout-icon {
  width: 36rpx;
  height: 36rpx;
  background-image: url("data:image/svg+xml,%3Csvg xmlns='http://www.w3.org/2000/svg' width='24' height='24' viewBox='0 0 24 24' fill='none' stroke='%23FB7185' stroke-width='2' stroke-linecap='round' stroke-linejoin='round'%3E%3Cpath d='M9 21H5a2 2 0 0 1-2-2V5a2 2 0 0 1 2-2h4'/%3E%3Cpolyline points='16 17 21 12 16 7'/%3E%3Cline x1='21' y1='12' x2='9' y2='12'/%3E%3C/svg%3E");
  background-size: contain;
  background-repeat: no-repeat;
  background-position: center;
}

/* ============================================
   Responsive
   ============================================ */

@media screen and (min-width: 768px) {
  .content {
    max-width: 750rpx;
    margin: 0 auto;
  }
}
</style>
