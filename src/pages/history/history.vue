<template>
  <view class="page">
    <view class="header">
      <view class="back" @click="goBack">
        <view class="back-icon"></view>
      </view>
      <text class="header-title">观看历史</text>
      <view class="clear-all" v-if="list.length > 0" @click="clearAll">
        <text>清空</text>
      </view>
    </view>

    <scroll-view scroll-y class="content">
      <view class="list">
        <view class="item" v-for="(item, index) in list" :key="index" @click="goDetail(item)">
          <image class="cover" :src="getCover(item)" mode="aspectFill" lazy-load @error="onCoverError(item)" />
          <view class="info">
            <text class="title">{{ getTitle(item) }}</text>
            <view class="meta">
              <text class="source" v-if="item.source_name">{{ item.source_name }}</text>
              <text class="episode" v-if="getEpisode(item)">第{{ getEpisode(item) }}集</text>
            </view>
            <view class="progress-row" v-if="item.play_time && item.duration">
              <view class="progress-bar">
                <view class="progress-fill" :style="{ width: getProgressPercent(item) + '%' }"></view>
              </view>
              <text class="progress-text">{{ formatProgress(item.play_time, item.duration) }}</text>
            </view>
            <text class="time">{{ formatTime(getTime(item)) }}</text>
          </view>
          <view class="delete-btn" @click.stop="deleteItem(item, index)">
            <text>删除</text>
          </view>
        </view>
      </view>

      <view class="loading" v-if="loading">
        <view class="loading-spinner"></view>
        <text>加载中...</text>
      </view>

      <view class="empty" v-if="!loading && list.length === 0">
        <view class="empty-icon"></view>
        <text class="empty-text">暂无观看记录</text>
        <text class="empty-tip">快去看看精彩内容吧</text>
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
      list: [],
      loading: false
    }
  },
  onShow() {
    this.loadData()
  },
  methods: {
    goBack() {
      uni.navigateBack()
    },
    
    loadData() {
      this.loading = true
      uni.request({
        url: buildUrl('/api/playrecords'),
        withCredentials: true,
        success: (res) => {
          if (res.statusCode === 200 && res.data) {
            if (Array.isArray(res.data)) {
              this.list = res.data
            } else if (typeof res.data === 'object') {
              const records = res.data.records || res.data.list || res.data.data
              if (Array.isArray(records)) {
                this.list = records
              } else if (records && typeof records === 'object') {
                this.list = Object.entries(records).map(([key, value]) => ({
                  key,
                  ...value
                }))
              } else {
                this.list = Object.entries(res.data).map(([key, value]) => ({
                  key,
                  ...value
                }))
              }
            }
          }
        },
        fail: () => {
          uni.showToast({ title: '加载失败', icon: 'none' })
        },
        complete: () => {
          this.loading = false
        }
      })
    },
    
    getCover(item) {
      const url = item.cover || item.poster || item.pic || item.thumb
      if (!url) {
        return 'data:image/svg+xml;base64,PHN2ZyB4bWxucz0iaHR0cDovL3d3dy53My5vcmcvMjAwMC9zdmciIHdpZHRoPSIxNjAiIGhlaWdodD0iMjIwIiB2aWV3Qm94PSIwIDAgMTYwIDIyMCI+PHJlY3QgZmlsbD0iIzFhMWEyZSIgd2lkdGg9IjE2MCIgaGVpZ2h0PSIyMjAiLz48dGV4dCB4PSI4MCIgeT0iMTEwIiBmaWxsPSIjODg4IiBmb250LXNpemU9IjE0IiB0ZXh0LWFuY2hvcj0ibWlkZGxlIiBkb21pbmFudC1iYXNlbGluZT0ibWlkZGxlIj7ml6DmtITlm77niYc8L3RleHQ+PC9zdmc+'
      }
      
      if (url.includes('/api/image-proxy?url=')) {
        return url
      }
      
      if (url.includes('/api/image-proxy')) {
        return buildUrl(url)
      }
      
      return this.proxyImage(url)
    },
    
    proxyImage(url) {
      if (!url || url.startsWith('data:')) return url
      
      // 使用图片代理
      if (url.includes('doubanio.com')) {
        return buildUrl('/api/image-proxy?url=' + encodeURIComponent(url))
      }
      if (url.startsWith('http://') || url.startsWith('https://')) {
        if (!url.includes('monkeycode-ai.online') && !url.includes('localhost')) {
          return buildUrl('/api/image-proxy?url=' + encodeURIComponent(url))
        }
      }
      
      return url
    },
    
    onCoverError(item) {
      item.cover = 'data:image/svg+xml;base64,PHN2ZyB4bWxucz0iaHR0cDovL3d3dy53My5vcmcvMjAwMC9zdmciIHdpZHRoPSIxNjAiIGhlaWdodD0iMjIwIiB2aWV3Qm94PSIwIDAgMTYwIDIyMCI+PHJlY3QgZmlsbD0iIzFhMWEyZSIgd2lkdGg9IjE2MCIgaGVpZ2h0PSIyMjAiLz48dGV4dCB4PSI4MCIgeT0iMTEwIiBmaWxsPSIjODg4IiBmb250LXNpemU9IjE0IiB0ZXh0LWFuY2hvcj0ibWlkZGxlIiBkb21pbmFudC1iYXNlbGluZT0ibWlkZGxlIj7ml6DmtITlm77niYc8L3RleHQ+PC9zdmc+'
    },
    
    getTitle(item) {
      return item.title || item.name || item.videoTitle || '未知内容'
    },
    
    getEpisode(item) {
      return item.index || item.episode || item.episodeIndex || item.ep
    },
    
    getTime(item) {
      return item.save_time || item.updateTime || item.lastPlayTime || item.time || Date.now()
    },
    
    getProgressPercent(item) {
      if (!item.play_time || !item.duration) return 0
      return Math.min((item.play_time / item.duration) * 100, 100)
    },
    
    clearAll() {
      uni.showModal({
        title: '提示',
        content: '确定清空所有观看记录？',
        success: (res) => {
          if (res.confirm) {
            uni.request({
              url: buildUrl('/api/playrecords'),
              method: 'DELETE',
              withCredentials: true,
              success: () => {
                this.list = []
                uni.showToast({ title: '已清空', icon: 'success' })
              },
              fail: () => {
                uni.showToast({ title: '操作失败', icon: 'none' })
              }
            })
          }
        }
      })
    },
    
    deleteItem(item, index) {
      uni.showModal({
        title: '提示',
        content: '确定删除该记录？',
        success: (res) => {
          if (res.confirm) {
            const key = item.key || item.id || item._id
            if (key) {
              uni.request({
                url: buildUrl('/api/playrecords?key=') + encodeURIComponent(key),
                method: 'DELETE',
                withCredentials: true,
                success: () => {
                  this.list.splice(index, 1)
                  uni.showToast({ title: '已删除', icon: 'none' })
                },
                fail: () => {
                  this.list.splice(index, 1)
                  uni.showToast({ title: '已删除', icon: 'none' })
                }
              })
            } else {
              this.list.splice(index, 1)
              uni.showToast({ title: '已删除', icon: 'none' })
            }
          }
        }
      })
    },
    
    goDetail(item) {
      const title = this.getTitle(item)
      const key = item.key || ''
      
      if (key) {
        const parts = key.split('+')
        if (parts.length >= 2) {
          uni.navigateTo({
            url: '/pages/play/play?title=' + encodeURIComponent(title) + '&q=' + encodeURIComponent(title)
          })
          return
        }
      }
      
      if (item.url) {
        const playData = {
          title: title,
          poster: item.cover,
          episodes: [item.url],
          episodes_titles: ['正片']
        }
        uni.navigateTo({
          url: '/pages/play/play?title=' + encodeURIComponent(title) + '&data=' + encodeURIComponent(JSON.stringify(playData))
        })
        return
      }
      
      uni.navigateTo({
        url: '/pages/play/play?title=' + encodeURIComponent(title) + '&q=' + encodeURIComponent(title)
      })
    },
    
    formatTime(timestamp) {
      if (!timestamp) return ''
      const ts = timestamp > 9999999999 ? timestamp : timestamp * 1000
      const date = new Date(ts)
      const now = new Date()
      const diff = now - date
      
      if (diff < 60000) return '刚刚'
      if (diff < 3600000) return Math.floor(diff / 60000) + '分钟前'
      if (diff < 86400000) return Math.floor(diff / 3600000) + '小时前'
      if (diff < 604800000) return Math.floor(diff / 86400000) + '天前'
      
      const m = String(date.getMonth() + 1).padStart(2, '0')
      const d = String(date.getDate()).padStart(2, '0')
      return `${m}-${d}`
    },
    
    formatProgress(progress, duration) {
      if (!progress) return ''
      const formatNum = (n) => {
        const m = Math.floor(n / 60)
        const s = Math.floor(n % 60)
        return m + ':' + String(s).padStart(2, '0')
      }
      return formatNum(progress) + (duration ? ' / ' + formatNum(duration) : '')
    }
  }
}
</script>

<style lang="scss">
@import '../../styles/common.scss';

.page {
  height: 100vh;
  background: $color-bg;
  display: flex;
  flex-direction: column;
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
  flex: 1;
  margin-left: 8rpx;
}

.clear-all {
  padding: 12rpx 24rpx;
  border-radius: 20rpx;
  transition: all $transition-fast;
  
  &:active {
    background: rgba($color-primary, 0.1);
  }
  
  text {
    color: $color-primary;
    font-size: 26rpx;
    font-weight: 500;
  }
}

.content {
  flex: 1;
}

.list {
  padding: 24rpx;
}

.item {
  display: flex;
  padding: 24rpx 0;
  border-bottom: 1rpx solid $color-border;
  transition: all $transition-fast;
  
  &:active {
    background: rgba(255, 255, 255, 0.02);
  }
}

.cover {
  flex-shrink: 0;
  width: 160rpx;
  height: 220rpx;
  border-radius: 16rpx;
  background: $color-bg-secondary;
  box-shadow: $shadow-sm;
}

.info {
  flex: 1;
  padding-left: 24rpx;
  min-width: 0;
  display: flex;
  flex-direction: column;
  justify-content: center;
}

.title {
  color: $color-text;
  font-size: 30rpx;
  font-weight: 600;
  display: block;
  overflow: hidden;
  text-overflow: ellipsis;
  white-space: nowrap;
}

.meta {
  display: flex;
  align-items: center;
  margin-top: 12rpx;
  gap: 16rpx;
}

.source {
  color: $color-secondary;
  font-size: 22rpx;
  font-weight: 500;
  padding: 4rpx 14rpx;
  background: rgba($color-secondary, 0.15);
  border-radius: 10rpx;
}

.episode {
  color: $color-warning;
  font-size: 24rpx;
  font-weight: 500;
}

.time {
  color: $color-text-muted;
  font-size: 24rpx;
  margin-top: 12rpx;
  display: block;
}

.progress-row {
  margin-top: 12rpx;
}

.progress-bar {
  width: 100%;
  height: 8rpx;
  background: rgba($color-bg-tertiary, 0.5);
  border-radius: 4rpx;
  overflow: hidden;
}

.progress-fill {
  height: 100%;
  background: $gradient-primary;
  border-radius: 4rpx;
  transition: width 0.3s ease;
}

.progress-text {
  color: $color-text-muted;
  font-size: 20rpx;
  margin-top: 8rpx;
  display: block;
}

.delete-btn {
  flex-shrink: 0;
  display: flex;
  align-items: center;
  padding: 16rpx;
  border-radius: 12rpx;
  transition: all $transition-fast;
  
  &:active {
    background: rgba($color-primary, 0.1);
  }
  
  text {
    color: $color-primary;
    font-size: 24rpx;
    font-weight: 500;
  }
}

.loading {
  display: flex;
  flex-direction: column;
  align-items: center;
  padding: 48rpx;
  
  text {
    color: $color-text-muted;
    font-size: 26rpx;
    margin-top: 20rpx;
  }
}

.loading-spinner {
  width: 52rpx;
  height: 52rpx;
  border: 4rpx solid rgba($color-bg-tertiary, 0.5);
  border-top-color: $color-primary;
  border-radius: 50%;
  animation: spin 0.8s linear infinite;
}

@keyframes spin {
  to {
    transform: rotate(360deg);
  }
}

.empty {
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  padding: 100rpx 40rpx;
}

.empty-icon {
  width: 100rpx;
  height: 100rpx;
  margin-bottom: 28rpx;
  background-image: url("data:image/svg+xml,%3Csvg xmlns='http://www.w3.org/2000/svg' width='48' height='48' viewBox='0 0 24 24' fill='none' stroke='%23888898' stroke-width='2' stroke-linecap='round' stroke-linejoin='round'%3E%3Crect x='2' y='3' width='20' height='14' rx='2' ry='2'/%3E%3Cline x1='8' y1='21' x2='16' y2='21'/%3E%3Cline x1='12' y1='17' x2='12' y2='21'/%3E%3C/svg%3E");
  background-size: contain;
  background-repeat: no-repeat;
  background-position: center;
  opacity: 0.5;
}

.empty-text {
  color: $color-text;
  font-size: 32rpx;
  font-weight: 600;
}

.empty-tip {
  color: $color-text-muted;
  font-size: 26rpx;
  margin-top: 12rpx;
}
</style>
