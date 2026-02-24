<template>
  <view class="page">
    <view class="header">
      <view class="back" @click="goBack">
        <text>&lt;</text>
      </view>
      <text class="header-title">{{ title }}</text>
      <view class="placeholder"></view>
    </view>

    <scroll-view 
      scroll-y 
      class="content"
      :refresher-enabled="true"
      :refresher-triggered="refreshing"
      @refresherrefresh="onRefresh"
      @scrolltolower="loadMore"
    >
      <view class="grid">
        <view 
          class="item" 
          v-for="(item, index) in list" 
          :key="index"
          @click="searchAndPlay(item)"
        >
          <image 
            class="item-cover" 
            :src="getPoster(item)" 
            mode="aspectFill"
            lazy-load
            @error="onImageError($event, item)"
          />
          <view class="item-info">
            <text class="item-title">{{ item.title || item.name }}</text>
            <text class="item-rate" v-if="item.rate || item.rating">{{ item.rate || item.rating }}分</text>
          </view>
        </view>
      </view>

      <view class="loading" v-if="loading">
        <text>加载中...</text>
      </view>

      <view class="no-more" v-if="!hasMore && list.length > 0">
        <text>没有更多了</text>
      </view>

      <view class="empty" v-if="!loading && list.length === 0">
        <text>暂无内容</text>
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
      type: 'movie',
      title: '',
      list: [],
      loading: false,
      refreshing: false,
      hasMore: true,
      pageStart: 0,
      pageSize: 24
    }
  },
  onLoad(options) {
    this.type = options.type || 'movie'
    this.title = decodeURIComponent(options.title || '更多')
    this.loadData()
  },
  methods: {
    goBack() {
      uni.navigateBack()
    },
    
    getPoster(item) {
      if (!item.poster && !item.cover && !item.pic) {
        return 'data:image/svg+xml;base64,PHN2ZyB4bWxucz0iaHR0cDovL3d3dy53My5vcmcvMjAwMC9zdmciIHdpZHRoPSIyMDAiIGhlaWdodD0iMjgwIiB2aWV3Qm94PSIwIDAgMjAwIDI4MCI+PHJlY3QgZmlsbD0iIzFhMWEyZSIgd2lkdGg9IjIwMCIgaGVpZ2h0PSIyODAiLz48dGV4dCB4PSIxMDAiIHk9IjE0MCIgZmlsbD0iIzg4OCIgZm9udC1zaXplPSIxNCIgdGV4dC1hbmNob3I9Im1pZGRsZSIgZG9taW5hbnQtYmFzZWxpbmU9Im1pZGRsZSI+5peg5rSE5Zu+54mHPC90ZXh0Pjwvc3ZnPg=='
      }
      const url = item.poster || item.cover || item.pic
      return this.proxyImage(url)
    },
    
    proxyImage(url) {
      if (!url || url.startsWith('data:')) return url
      
      // #ifdef H5
      // H5 端使用代理避免跨域
      if (url.includes('doubanio.com')) {
        return buildUrl('/api/image-proxy?url=' + encodeURIComponent(url))
      }
      if (url.startsWith('http://') || url.startsWith('https://')) {
        if (!url.includes('monkeycode-ai.online') && !url.includes('localhost')) {
          return buildUrl('/api/image-proxy?url=' + encodeURIComponent(url))
        }
      }
      // #endif
      
      // #ifndef H5
      // APP 端直接使用原始 URL
      // #endif
      
      return url
    },
    
    onImageError(e, item) {
      console.log('image load error:', item.title)
      item.poster = 'data:image/svg+xml;base64,PHN2ZyB4bWxucz0iaHR0cDovL3d3dy53My5vcmcvMjAwMC9zdmciIHdpZHRoPSIyMDAiIGhlaWdodD0iMjgwIiB2aWV3Qm94PSIwIDAgMjAwIDI4MCI+PHJlY3QgZmlsbD0iIzFhMWEyZSIgd2lkdGg9IjIwMCIgaGVpZ2h0PSIyODAiLz48dGV4dCB4PSIxMDAiIHk9IjE0MCIgZmlsbD0iIzg4OCIgZm9udC1zaXplPSIxNCIgdGV4dC1hbmNob3I9Im1pZGRsZSIgZG9taW5hbnQtYmFzZWxpbmU9Im1pZGRsZSI+5peg5rSE5Zu+54mHPC90ZXh0Pjwvc3ZnPg=='
      this.$forceUpdate()
    },
    
    onRefresh() {
      this.refreshing = true
      this.pageStart = 0
      this.hasMore = true
      this.loadData(true).finally(() => {
        this.refreshing = false
      })
    },
    
    async loadData(refresh = false) {
      if (this.loading) return
      if (!refresh && !this.hasMore) return
      
      this.loading = true
      if (refresh) {
        this.pageStart = 0
        this.hasMore = true
      }
      
      try {
        const res = await new Promise((resolve, reject) => {
          uni.request({
            url: buildUrl(`/api/douban?type=${this.getDoubanType()}&tag=${this.getTag()}&pageStart=${this.pageStart}&pageSize=${this.pageSize}`),
            withCredentials: true,
            success: resolve,
            fail: reject
          })
        })
        
        if (res.statusCode === 200 && res.data && res.data.list) {
          const items = res.data.list.map(item => ({
            id: item.id,
            title: item.title,
            poster: item.poster,
            rate: item.rate
          }))
          
          if (refresh || this.pageStart === 0) {
            this.list = items
          } else {
            this.list = [...this.list, ...items]
          }
          
          this.hasMore = items.length >= this.pageSize
          if (items.length > 0) {
            this.pageStart += this.pageSize
          }
        } else {
          this.hasMore = false
        }
      } catch (error) {
        console.error('加载失败:', error)
        uni.showToast({ title: '加载失败', icon: 'none' })
      } finally {
        this.loading = false
      }
    },
    
    loadMore() {
      if (!this.hasMore || this.loading) return
      this.loadData()
    },
    
    getDoubanType() {
      return this.type === 'movie' ? 'movie' : 'tv'
    },
    
    getTag() {
      const tagMap = {
        movie: '热门',
        tv: '热门',
        variety: '综艺',
        anime: '动漫'
      }
      return encodeURIComponent(tagMap[this.type] || '热门')
    },
    
    searchAndPlay(item) {
      if (!this.checkLogin()) {
        return
      }
      
      const userCookie = uni.getStorageSync('user_cookie') || ''
      const headers = {
        'Content-Type': 'application/json'
      }
      if (userCookie) {
        headers['Cookie'] = userCookie
      }
      
      uni.showLoading({ title: '搜索中...' })
      uni.request({
        url: buildUrl('/api/search?q=') + encodeURIComponent(item.title || item.name),
        withCredentials: true,
        header: headers,
        success: (res) => {
          uni.hideLoading()
          if (res.data && res.data.results && res.data.results.length > 0) {
            const first = res.data.results[0]
            uni.navigateTo({
              url: '/pages/play/play?title=' + encodeURIComponent(first.title) + '&data=' + encodeURIComponent(JSON.stringify(first))
            })
          } else {
            uni.showToast({ title: '未找到播放源', icon: 'none' })
          }
        },
        fail: () => {
          uni.hideLoading()
          uni.showToast({ title: '搜索失败', icon: 'none' })
        }
      })
    },
    
    checkLogin() {
      const userInfo = uni.getStorageSync('userInfo')
      if (!userInfo) {
        uni.showModal({
          title: '提示',
          content: '请登录账号后观看影片',
          showCancel: true,
          confirmText: '去登录',
          cancelText: '取消',
          success: (res) => {
            if (res.confirm) {
              uni.navigateTo({ url: '/pages/login/login' })
            }
          }
        })
        return false
      }
      return true
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
  display: flex;
  align-items: center;
  justify-content: space-between;
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

.back text {
  color: $text-primary;
  font-size: 36rpx;
}

.placeholder {
  width: 72rpx;
}

.header-title {
  position: relative;
  font-size: 36rpx;
  font-weight: 700;
  color: $text-primary;
}

/* ============================================
   Content
   ============================================ */

.content {
  flex: 1;
  height: 0;
}

.grid {
  display: flex;
  flex-wrap: wrap;
  padding: $space-4;
  gap: $space-4;
}

/* ============================================
   Video Card
   ============================================ */

.item {
  width: calc(50% - 8rpx);
}

.item-cover {
  width: 100%;
  height: 280rpx;
  border-radius: $radius-lg;
  background: $bg-elevated;
  transition: all $duration-normal $ease-out;
}

.item:active .item-cover {
  transform: scale(0.98);
}

.item-info {
  padding: $space-3 $space-2;
}

.item-title {
  display: block;
  font-size: 26rpx;
  font-weight: 500;
  color: $text-primary;
  overflow: hidden;
  text-overflow: ellipsis;
  white-space: nowrap;
}

.item-rate {
  display: block;
  margin-top: $space-1;
  font-size: 22rpx;
  font-weight: 600;
  color: $accent-gold;
}

/* ============================================
   States
   ============================================ */

.loading {
  display: flex;
  flex-direction: column;
  align-items: center;
  padding: $space-8;
  
  text {
    color: $text-muted;
    font-size: 26rpx;
  }
}

.loading-spinner {
  width: 48rpx;
  height: 48rpx;
  border: 4rpx solid rgba(255, 255, 255, 0.1);
  border-top-color: $brand-primary;
  border-radius: $radius-full;
  animation: spin 0.8s linear infinite;
}

@keyframes spin {
  to { transform: rotate(360deg); }
}

.no-more {
  text-align: center;
  padding: $space-8;
  
  text {
    color: $text-muted;
    font-size: 26rpx;
  }
}

.empty {
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  padding: $space-20;
  
  text {
    color: $text-muted;
    font-size: 30rpx;
  }
}

/* ============================================
   Responsive
   ============================================ */

@media screen and (min-width: 414px) {
  .item-cover { height: 320rpx; }
}

@media screen and (min-width: 768px) {
  .item {
    width: calc(33.33% - 11rpx);
  }
  .item-cover { height: 280rpx; }
}

.safe-area-bottom {
  padding-bottom: constant(safe-area-inset-bottom);
  padding-bottom: env(safe-area-inset-bottom);
}
</style>
