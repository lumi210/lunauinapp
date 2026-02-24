<template>
  <view class="page">
    <!-- Custom Navigation Bar -->
    <view class="custom-navbar">
      <text class="navbar-title">{{ siteName }}</text>
      <view class="navbar-search" @click="goSearch">
        <view class="search-icon"></view>
        <text class="search-placeholder">搜索影片...</text>
      </view>
    </view>
    
    <!-- Hero Banner -->
    <view class="hero-banner" v-if="bannerItems.length > 0" @touchstart="onTouchStart" @touchend="onTouchEnd">
      <swiper 
        class="hero-swiper" 
        :current="currentBanner" 
        :autoplay="!isSwiping" 
        :interval="8000" 
        :duration="800"
        circular
        @change="onBannerChange"
      >
        <swiper-item v-for="(item, index) in bannerItems" :key="index">
          <view class="hero-slide">
            <image 
              class="hero-backdrop" 
              :src="getBackdrop(item)" 
              mode="aspectFill"
              lazy-load
            />
            <view class="hero-gradient"></view>
            <view class="hero-content">
              <view class="hero-info">
                <text class="hero-title">{{ item.title || item.name }}</text>
                <view class="hero-meta" v-if="item.rate || item.year">
                  <text class="hero-rate" v-if="item.rate">{{ item.rate }}分</text>
                  <text class="hero-year" v-if="item.year">{{ item.year }}</text>
                </view>
                <text class="hero-desc" v-if="item.description">{{ item.description }}</text>
              </view>
              <view class="hero-actions">
                <view class="hero-btn hero-btn-play" @click="playBanner(item)">
                  <view class="hero-btn-icon icon-play-sm"></view>
                  <text>播放</text>
                </view>
                <view class="hero-btn hero-btn-info" @click="showDetail(item)">
                  <view class="hero-btn-icon icon-info-sm"></view>
                  <text>详情</text>
                </view>
              </view>
            </view>
          </view>
        </swiper-item>
      </swiper>
      
      <!-- Banner Indicators -->
      <view class="hero-indicators">
        <view 
          v-for="(item, index) in bannerItems" 
          :key="index"
          class="hero-indicator"
          :class="{ active: index === currentBanner }"
          @click="currentBanner = index"
        ></view>
      </view>
    </view>

    <!-- Main Content -->
    <scroll-view 
      scroll-y 
      class="content" 
      :refresher-enabled="true" 
      :refresher-triggered="refreshing"
      @refresherrefresh="onRefresh"
      @scrolltolower="loadMore"
    >
      <!-- Quick Categories -->
      <view class="quick-cats">
        <view 
          class="quick-cat-item" 
          :class="{ active: activeCategory === 'all' }"
          @click="activeCategory = 'all'"
        >
          <text>推荐</text>
        </view>
        <view 
          class="quick-cat-item" 
          :class="{ active: activeCategory === 'movie' }"
          @click="activeCategory = 'movie'"
        >
          <text>电影</text>
        </view>
        <view 
          class="quick-cat-item" 
          :class="{ active: activeCategory === 'tv' }"
          @click="activeCategory = 'tv'"
        >
          <text>电视剧</text>
        </view>
        <view 
          class="quick-cat-item" 
          :class="{ active: activeCategory === 'variety' }"
          @click="activeCategory = 'variety'"
        >
          <text>综艺</text>
        </view>
        <view 
          class="quick-cat-item" 
          :class="{ active: activeCategory === 'anime' }"
          @click="activeCategory = 'anime'"
        >
          <text>动漫</text>
        </view>
        <view 
          class="quick-cat-item" 
          :class="{ active: activeCategory === 'shortdrama' }"
          @click="activeCategory = 'shortdrama'"
        >
          <text>短剧</text>
        </view>
      </view>

      <!-- Hot Movies -->
      <view class="section" v-if="movies.length > 0 && (activeCategory === 'all' || activeCategory === 'movie')">
        <view class="section-header">
          <text class="section-title">热门电影</text>
          <text class="section-more" @click="goMore('movie', '热门电影')">
            更多 <view class="more-arrow"></view>
          </text>
        </view>
        <scroll-view scroll-x class="scroll-x" :show-scrollbar="false" enable-flex>
          <view class="video-list">
            <view class="video-card" v-for="(item, index) in movies" :key="index" @click="searchAndPlay(item)">
              <view class="video-poster-wrap">
                <image 
                  class="video-poster" 
                  :src="getPoster(item)" 
                  mode="aspectFill" 
                  lazy-load
                  @error="onImageError($event, item)"
                />
                <view class="video-rate" v-if="item.rate">
                  <text>{{ item.rate }}</text>
                </view>
              </view>
              <view class="video-info">
                <text class="video-title">{{ item.title || item.name }}</text>
              </view>
            </view>
          </view>
        </scroll-view>
      </view>

      <!-- Hot TV Shows -->
      <view class="section" v-if="tvShows.length > 0 && (activeCategory === 'all' || activeCategory === 'tv')">
        <view class="section-header">
          <text class="section-title">热门电视剧</text>
          <text class="section-more" @click="goMore('tv', '热门电视剧')">
            更多 <view class="more-arrow"></view>
          </text>
        </view>
        <scroll-view scroll-x class="scroll-x" :show-scrollbar="false" enable-flex>
          <view class="video-list">
            <view class="video-card" v-for="(item, index) in tvShows" :key="index" @click="searchAndPlay(item)">
              <view class="video-poster-wrap">
                <image 
                  class="video-poster" 
                  :src="getPoster(item)" 
                  mode="aspectFill" 
                  lazy-load
                  @error="onImageError($event, item)"
                />
                <view class="video-rate" v-if="item.rate">
                  <text>{{ item.rate }}</text>
                </view>
              </view>
              <view class="video-info">
                <text class="video-title">{{ item.title || item.name }}</text>
              </view>
            </view>
          </view>
        </scroll-view>
      </view>

      <!-- Hot Variety Shows -->
      <view class="section" v-if="varietyShows.length > 0 && (activeCategory === 'all' || activeCategory === 'variety')">
        <view class="section-header">
          <text class="section-title">热门综艺</text>
          <text class="section-more" @click="goMore('variety', '热门综艺')">
            更多 <view class="more-arrow"></view>
          </text>
        </view>
        <scroll-view scroll-x class="scroll-x" :show-scrollbar="false" enable-flex>
          <view class="video-list">
            <view class="video-card" v-for="(item, index) in varietyShows" :key="index" @click="searchAndPlay(item)">
              <view class="video-poster-wrap">
                <image 
                  class="video-poster" 
                  :src="getPoster(item)" 
                  mode="aspectFill" 
                  lazy-load
                  @error="onImageError($event, item)"
                />
                <view class="video-rate" v-if="item.rate">
                  <text>{{ item.rate }}</text>
                </view>
              </view>
              <view class="video-info">
                <text class="video-title">{{ item.title || item.name }}</text>
              </view>
            </view>
          </view>
        </scroll-view>
      </view>

      <!-- Hot Anime - 今日番剧 -->
      <view class="section" v-if="todayAnimes.length > 0 && (activeCategory === 'all' || activeCategory === 'anime')">
        <view class="section-header">
          <text class="section-title">今日番剧</text>
          <text class="section-more" @click="goMore('anime', '今日番剧')">
            更多 <view class="more-arrow"></view>
          </text>
        </view>
        <scroll-view scroll-x class="scroll-x" :show-scrollbar="false" enable-flex>
          <view class="video-list">
            <view class="video-card" v-for="(item, index) in todayAnimes" :key="index" @click="playBangumi(item)">
              <view class="video-poster-wrap">
                <image 
                  class="video-poster" 
                  :src="getBangumiPoster(item)" 
                  mode="aspectFill" 
                  lazy-load
                />
                <view class="video-rate" v-if="item.rating && item.rating.score">
                  <text>{{ item.rating.score.toFixed(1) }}</text>
                </view>
              </view>
              <view class="video-info">
                <text class="video-title">{{ item.name_cn || item.name }}</text>
              </view>
            </view>
          </view>
        </scroll-view>
      </view>

      <!-- Hot Short Dramas -->
      <view class="section" v-if="shortDramas.length > 0 && (activeCategory === 'all' || activeCategory === 'shortdrama')">
        <view class="section-header">
          <text class="section-title">热门短剧</text>
          <text class="section-more" @click="goShortDrama">
            更多 <view class="more-arrow"></view>
          </text>
        </view>
        <scroll-view scroll-x class="scroll-x" :show-scrollbar="false" enable-flex>
          <view class="video-list">
            <view class="video-card" v-for="(item, index) in shortDramas" :key="index" @click="playShortDrama(item)">
              <view class="video-poster-wrap">
                <image 
                  class="video-poster" 
                  :src="item.cover" 
                  mode="aspectFill" 
                  lazy-load
                />
                <view class="video-rate" v-if="item.score || item.vote_average">
                  <text>{{ (item.score || item.vote_average).toFixed(1) }}</text>
                </view>
                <view class="video-episodes" v-if="item.episode_count && item.episode_count > 1">
                  <text>{{ item.episode_count }}集</text>
                </view>
              </view>
              <view class="video-info">
                <text class="video-title">{{ item.name }}</text>
              </view>
            </view>
          </view>
        </scroll-view>
      </view>

      <!-- Login Tip -->
      <view class="login-tip" v-if="!isLoggedIn" @click="goLogin">
        <view class="login-tip-icon">
          <view class="icon-user-circle"></view>
        </view>
        <text class="login-tip-text">登录后查看个性化推荐</text>
        <text class="login-tip-btn">立即登录</text>
      </view>

      <!-- Loading -->
      <view class="loading" v-if="loading">
        <view class="loading-spinner"></view>
        <text>加载中...</text>
      </view>
      
      <!-- Error State -->
      <view class="error-state" v-if="!loading && loadError">
        <view class="error-icon"></view>
        <text class="error-text">请登录账号使用</text>
      </view>
      
      <!-- Empty State -->
      <view class="empty-state" v-if="!loading && !loadError && movies.length === 0 && tvShows.length === 0 && varietyShows.length === 0 && todayAnimes.length === 0 && shortDramas.length === 0">
        <view class="empty-icon"></view>
        <text class="empty-text">暂无影片数据</text>
        <text class="empty-tip">下拉刷新试试</text>
      </view>
      
      <view class="safe-area-bottom"></view>
    </scroll-view>

    <!-- Floating Search Button -->
    <view class="float-search" @click="goSearch">
      <view class="float-search-icon"></view>
    </view>
  </view>
</template>

<script>
import request from '../../utils/request'
import { getApiUrl } from '../../utils/config'

export default {
  data() {
    return {
      siteName: 'LunaTV',
      isLoggedIn: false,
      loading: false,
      refreshing: false,
      activeCategory: 'all',
      currentBanner: 0,
      isSwiping: false,
      touchStartX: 0,
      bannerItems: [],
      movies: [],
      tvShows: [],
      varietyShows: [],
      todayAnimes: [],
      shortDramas: [],
      loadError: false
    }
  },
  onShow() {
    this.isLoggedIn = !!uni.getStorageSync('userInfo')
    this.loadSiteConfig()
    this.loadData()
  },
  methods: {
    loadSiteConfig() {
      uni.request({
        url: getApiUrl('/api/server-config'),
        withCredentials: true,
        success: (res) => {
          if (res.statusCode === 200 && res.data && res.data.SiteName) {
            this.siteName = res.data.SiteName
            uni.setStorageSync('siteName', res.data.SiteName)
          }
        },
        fail: () => {
          const cached = uni.getStorageSync('siteName')
          if (cached) {
            this.siteName = cached
          }
        }
      })
    },
    getPoster(item) {
      if (!item.poster && !item.cover && !item.pic) {
        return 'data:image/svg+xml;base64,PHN2ZyB4bWxucz0iaHR0cDovL3d3dy53My5vcmcvMjAwMC9zdmciIHdpZHRoPSIyMDAiIGhlaWdodD0iMjgwIiB2aWV3Qm94PSIwIDAgMjAwIDI4MCI+PHJlY3QgZmlsbD0iIzFhMWEyZSIgd2lkdGg9IjIwMCIgaGVpZ2h0PSIyODAiLz48dGV4dCB4PSIxMDAiIHk9IjE0MCIgZmlsbD0iIzg4OCIgZm9udC1zaXplPSIxNCIgdGV4dC1hbmNob3I9Im1pZGRsZSIgZG9taW5hbnQtYmFzZWxpbmU9Im1pZGRsZSI+5peg5rSE5Zu+54mHPC90ZXh0Pjwvc3ZnPg=='
      }
      const url = item.poster || item.cover || item.pic
      return this.proxyImage(url)
    },
    
    getBackdrop(item) {
      if (!item.backdrop && !item.poster && !item.cover) {
        return 'data:image/svg+xml;base64,PHN2ZyB4bWxucz0iaHR0cDovL3d3dy53My5vcmcvMjAwMC9zdmciIHdpZHRoPSI4MDAiIGhlaWdodD0iNDUwIiB2aWV3Qm94PSIwIDAgODAwIDQ1MCI+PHJlY3QgZmlsbD0iIzBmMGYxYSIgd2lkdGg9IjgwMCIgaGVpZ2h0PSI0NTAiLz48dGV4dCB4PSI0MDAiIHk9IjIyNSIgZmlsbD0iIzg4OCIgZm9udC1zaXplPSIyMCIgdGV4dC1hbmNob3I9Im1pZGRsZSIgZG9taW5hbnQtYmFzZWxpbmU9Im1pZGRsZSI+5peg5rSE5Zu+54mHPC90ZXh0Pjwvc3ZnPg=='
      }
      const url = item.backdrop || item.poster || item.cover
      return this.proxyImage(url)
    },
    
    proxyImage(url) {
      if (!url || url.startsWith('data:')) return url
      
      // 使用图片代理（H5 避免跨域，APP 端避免防盗链）
      if (url.includes('doubanio.com')) {
        return getApiUrl('/api/image-proxy?url=' + encodeURIComponent(url))
      }
      if (url.startsWith('http://') || url.startsWith('https://')) {
        if (!url.includes('monkeycode-ai.online') && !url.includes('localhost')) {
          return getApiUrl('/api/image-proxy?url=' + encodeURIComponent(url))
        }
      }
      
      return url
    },
    
    onImageError(e, item) {
      console.log('image load error:', item.title, item.poster)
      item.poster = 'data:image/svg+xml;base64,PHN2ZyB4bWxucz0iaHR0cDovL3d3dy53My5vcmcvMjAwMC9zdmciIHdpZHRoPSIyMDAiIGhlaWdodD0iMjgwIiB2aWV3Qm94PSIwIDAgMjAwIDI4MCI+PHJlY3QgZmlsbD0iIzFhMWEyZSIgd2lkdGg9IjIwMCIgaGVpZ2h0PSIyODAiLz48dGV4dCB4PSIxMDAiIHk9IjE0MCIgZmlsbD0iIzg4OCIgZm9udC1zaXplPSIxNCIgdGV4dC1hbmNob3I9Im1pZGRsZSIgZG9taW5hbnQtYmFzZWxpbmU9Im1pZGRsZSI+5peg5rSE5Zu+54mHPC90ZXh0Pjwvc3ZnPg=='
      this.$forceUpdate()
    },
    
    onTouchStart(e) {
      this.touchStartX = e.touches[0].clientX
      this.isSwiping = true
    },
    
    onTouchEnd(e) {
      this.isSwiping = false
    },
    
    onBannerChange(e) {
      this.currentBanner = e.detail.current
    },
    
    goLogin() {
      uni.navigateTo({ url: '/pages/login/login' })
    },
    
    goSearch() {
      uni.navigateTo({ url: '/pages/search/search' })
    },
    
    goMore(type, title) {
      uni.navigateTo({ 
        url: '/pages/more/more?type=' + type + '&title=' + encodeURIComponent(title)
      })
    },
    
    playBanner(item) {
      this.searchAndPlay(item)
    },
    
    showDetail(item) {
      this.searchAndPlay(item)
    },
    
    searchAndPlay(item) {
      if (!this.checkLogin()) {
        return
      }
      const title = item.title || item.name
      uni.navigateTo({
        url: '/pages/play/play?q=' + encodeURIComponent(title) + '&title=' + encodeURIComponent(title)
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
    },
    
    loadData() {
      this.loading = true
      this.loadError = false
      Promise.all([
        this.loadMovies(),
        this.loadTvShows(),
        this.loadVariety(),
        this.loadBangumiCalendar(),
        this.loadShortDramas()
      ]).then((results) => {
        const hasError = results.some(r => r === 'error')
        if (hasError) {
          this.loadError = true
        }
      }).catch(() => {
        this.loadError = true
      }).finally(() => {
        this.loading = false
        this.updateBannerItems()
      })
    },
    
    updateBannerItems() {
      const allItems = [...this.movies.slice(0, 3), ...this.tvShows.slice(0, 2)]
      if (allItems.length > 0) {
        this.bannerItems = allItems.map(item => ({
          ...item,
          backdrop: item.poster
        }))
      }
    },
    
    loadMovies() {
      return new Promise((resolve) => {
        uni.request({
          url: getApiUrl('/api/douban?type=movie&tag=%E7%83%AD%E9%97%A8&pageStart=0&pageSize=12'),
          withCredentials: true,
          success: (res) => {
            if (res.statusCode === 200 && res.data && res.data.list) {
              this.movies = res.data.list.map(item => ({
                id: item.id,
                title: item.title,
                poster: item.poster,
                rate: item.rate,
                year: item.year
              }))
              resolve('success')
            } else {
              resolve('error')
            }
          },
          fail: () => {
            resolve('error')
          },
          complete: () => {}
        })
      })
    },
    
    loadTvShows() {
      return new Promise((resolve) => {
        uni.request({
          url: getApiUrl('/api/douban?type=tv&tag=%E7%83%AD%E9%97%A8&pageStart=0&pageSize=12'),
          withCredentials: true,
          success: (res) => {
            if (res.statusCode === 200 && res.data && res.data.list) {
              this.tvShows = res.data.list.map(item => ({
                id: item.id,
                title: item.title,
                poster: item.poster,
                rate: item.rate,
                year: item.year
              }))
              resolve('success')
            } else {
              resolve('error')
            }
          },
          fail: () => {
            resolve('error')
          },
          complete: () => {}
        })
      })
    },
    
    loadVariety() {
      return new Promise((resolve) => {
        uni.request({
          url: getApiUrl('/api/douban?type=tv&tag=%E7%BB%BC%E8%89%BA&pageStart=0&pageSize=12'),
          withCredentials: true,
          success: (res) => {
            if (res.statusCode === 200 && res.data && res.data.list) {
              this.varietyShows = res.data.list.map(item => ({
                id: item.id,
                title: item.title,
                poster: item.poster,
                rate: item.rate,
                year: item.year
              }))
              resolve('success')
            } else {
              resolve('error')
            }
          },
          fail: () => {
            resolve('error')
          },
          complete: () => {}
        })
      })
    },
    
    loadBangumiCalendar() {
      return new Promise((resolve) => {
        uni.request({
          url: getApiUrl('/api/proxy/bangumi?path=calendar'),
          withCredentials: true,
          success: (res) => {
            console.log('[Index] bangumi calendar response:', res.statusCode, res.data?.length)
            if (res.statusCode === 200 && Array.isArray(res.data)) {
              const weekdays = ['Sun', 'Mon', 'Tue', 'Wed', 'Thu', 'Fri', 'Sat']
              const today = new Date()
              const currentWeekday = weekdays[today.getDay()]
              
              const todayData = res.data.find(item => item.weekday && item.weekday.en === currentWeekday)
              if (todayData && todayData.items) {
                this.todayAnimes = todayData.items.slice(0, 12)
                console.log('[Index] today animes:', this.todayAnimes.length)
              }
              resolve('success')
            } else {
              console.error('[Index] bangumi calendar load failed')
              resolve('error')
            }
          },
          fail: (err) => {
            console.error('[Index] bangumi calendar request failed:', err)
            resolve('error')
          }
        })
      })
    },
    
    getBangumiPoster(item) {
      if (item.images) {
        return item.images.large || item.images.common || item.images.medium || item.images.small || item.images.grid || ''
      }
      return ''
    },
    
    playBangumi(item) {
      if (!this.checkLogin()) {
        return
      }
      const title = item.name_cn || item.name
      uni.navigateTo({
        url: '/pages/play/play?q=' + encodeURIComponent(title) + '&title=' + encodeURIComponent(title)
      })
    },
    
    loadShortDramas() {
      return new Promise((resolve) => {
        uni.request({
          url: getApiUrl('/api/shortdrama/list?categoryId=1&page=1&size=12'),
          withCredentials: true,
          success: (res) => {
            if (res.statusCode === 200 && res.data) {
              let list = []
              if (Array.isArray(res.data)) {
                list = res.data
              } else if (res.data.list) {
                list = res.data.list
              }
              this.shortDramas = list.slice(0, 12)
              resolve('success')
            } else {
              resolve('error')
            }
          },
          fail: () => {
            resolve('error')
          }
        })
      })
    },
    
    playShortDrama(item) {
      if (!this.checkLogin()) {
        return
      }
      uni.navigateTo({
        url: '/pages/play/play?id=' + item.id + '&type=shortdrama&title=' + encodeURIComponent(item.name)
      })
    },
    
    goShortDrama() {
      uni.switchTab({ url: '/pages/shortdrama/shortdrama' })
    },
    
    loadMore() {
      // 可扩展加载更多逻辑
    },
    
    onRefresh() {
      this.refreshing = true
      this.loadData()
      setTimeout(() => {
        this.refreshing = false
      }, 1000)
    }
  }
}
</script>

<style lang="scss">
@import '../../styles/common.scss';

.page {
  min-height: 100vh;
  background: $color-bg;
  padding-top: 88rpx;
}

/* Custom Navigation Bar - 玻璃态效果 */
.custom-navbar {
  position: fixed;
  top: 0;
  left: 0;
  right: 0;
  height: 88rpx;
  background: rgba($color-bg-secondary, 0.85);
  backdrop-filter: blur(20px);
  -webkit-backdrop-filter: blur(20px);
  display: flex;
  align-items: center;
  padding: 0 24rpx;
  padding-top: constant(safe-area-inset-top);
  padding-top: env(safe-area-inset-top);
  z-index: 100;
  border-bottom: 1rpx solid $color-border;
}

.navbar-title {
  color: $color-primary;
  font-size: 36rpx;
  font-weight: 700;
  margin-right: 24rpx;
  letter-spacing: 0.5rpx;
}

.navbar-search {
  flex: 1;
  height: 64rpx;
  background: rgba($color-bg, 0.6);
  border-radius: 32rpx;
  display: flex;
  align-items: center;
  padding: 0 24rpx;
  border: 1rpx solid $color-border;
  transition: all $transition-fast;
  
  &:active {
    background: rgba($color-bg, 0.8);
    border-color: rgba($color-primary, 0.3);
  }
}

.search-icon {
  width: 36rpx;
  height: 36rpx;
  margin-right: 12rpx;
  background-image: $icon-search;
  background-size: contain;
  background-repeat: no-repeat;
  background-position: center;
  opacity: 0.7;
}

.search-placeholder {
  color: $color-text-muted;
  font-size: 26rpx;
}

/* Hero Banner - 增强视觉效果 */
.hero-banner {
  position: relative;
  width: 100%;
  height: 50vw;
  max-height: 420rpx;
  min-height: 300rpx;
  overflow: hidden;
}

.hero-swiper {
  width: 100%;
  height: 100%;
}

.hero-slide {
  position: relative;
  width: 100%;
  height: 100%;
}

.hero-backdrop {
  width: 100%;
  height: 100%;
  object-fit: cover;
}

.hero-gradient {
  position: absolute;
  left: 0;
  right: 0;
  bottom: 0;
  height: 320rpx;
  background: $gradient-hero;
}

.hero-gradient::before {
  content: '';
  position: absolute;
  left: 0;
  right: 0;
  top: 0;
  height: 180rpx;
  background: linear-gradient(to bottom, rgba($color-bg, 0.5) 0%, transparent 100%);
}

.hero-content {
  position: absolute;
  left: 0;
  right: 0;
  bottom: 0;
  padding: 28rpx;
  padding-bottom: 48rpx;
}

.hero-info {
  margin-bottom: 28rpx;
}

.hero-title {
  color: $color-text;
  font-size: 44rpx;
  font-weight: 700;
  text-shadow: 0 4rpx 12rpx rgba(0, 0, 0, 0.6);
  display: block;
  margin-bottom: 16rpx;
  overflow: hidden;
  text-overflow: ellipsis;
  white-space: nowrap;
  max-width: 90%;
  letter-spacing: 0.5rpx;
}

.hero-meta {
  display: flex;
  align-items: center;
  gap: 16rpx;
  margin-bottom: 12rpx;
}

.hero-rate {
  background: $gradient-gold;
  color: $color-bg;
  font-size: 22rpx;
  font-weight: 700;
  padding: 6rpx 14rpx;
  border-radius: 8rpx;
  box-shadow: 0 2rpx 8rpx rgba(#ffd700, 0.3);
}

.hero-year {
  color: $color-text-secondary;
  font-size: 26rpx;
  font-weight: 500;
}

.hero-desc {
  color: $color-text-secondary;
  font-size: 24rpx;
  display: block;
  overflow: hidden;
  text-overflow: ellipsis;
  white-space: nowrap;
  max-width: 85%;
}

.hero-actions {
  display: flex;
  gap: 16rpx;
}

.hero-btn {
  display: flex;
  align-items: center;
  justify-content: center;
  gap: 8rpx;
  padding: 18rpx 36rpx;
  border-radius: 12rpx;
  font-size: 28rpx;
  font-weight: 600;
  transition: all $transition-normal;
  
  text {
    color: $color-text;
  }
}

.hero-btn-icon {
  width: 32rpx;
  height: 32rpx;
  
  &.icon-play-sm {
    background-image: $icon-play;
    background-size: contain;
    background-repeat: no-repeat;
    background-position: center;
  }
  
  &.icon-info-sm {
    background-image: $icon-info;
    background-size: contain;
    background-repeat: no-repeat;
    background-position: center;
  }
}

.hero-btn-play {
  background: $gradient-primary;
  box-shadow: $shadow-primary;
  
  &:active {
    transform: scale(0.96);
    box-shadow: 0 2rpx 12rpx rgba($color-primary, 0.4);
  }
}

.hero-btn-info {
  background: rgba(255, 255, 255, 0.15);
  backdrop-filter: blur(10px);
  -webkit-backdrop-filter: blur(10px);
  border: 1rpx solid rgba(255, 255, 255, 0.1);
  
  &:active {
    background: rgba(255, 255, 255, 0.25);
    transform: scale(0.96);
  }
}

.hero-indicators {
  position: absolute;
  right: 28rpx;
  bottom: 28rpx;
  display: flex;
  gap: 10rpx;
}

.hero-indicator {
  width: 16rpx;
  height: 16rpx;
  border-radius: 50%;
  background: rgba(255, 255, 255, 0.35);
  transition: all $transition-normal;
  
  &.active {
    width: 36rpx;
    border-radius: 8rpx;
    background: $color-primary;
    box-shadow: 0 2rpx 8rpx rgba($color-primary, 0.4);
  }
}

/* Quick Categories - 增强交互 */
.quick-cats {
  display: flex;
  padding: 20rpx 24rpx;
  gap: 16rpx;
  background: $color-bg;
  overflow-x: auto;
  white-space: nowrap;
  
  &::-webkit-scrollbar {
    display: none;
  }
}

.quick-cat-item {
  flex-shrink: 0;
  padding: 14rpx 32rpx;
  border-radius: 32rpx;
  font-size: 26rpx;
  font-weight: 500;
  background: rgba($color-bg-secondary, 0.6);
  color: $color-text-muted;
  border: 1rpx solid transparent;
  transition: all $transition-normal;
  
  &:active {
    transform: scale(0.96);
  }
  
  &.active {
    background: $gradient-primary;
    color: $color-text;
    box-shadow: $shadow-primary;
    border-color: transparent;
  }
}

/* Content */
.content {
  flex: 1;
  height: calc(100vh - 88rpx - constant(safe-area-inset-top));
  height: calc(100vh - 88rpx - env(safe-area-inset-top));
}

/* Section */
.section {
  margin-top: 20rpx;
}

.section-header {
  display: flex;
  align-items: center;
  justify-content: space-between;
  padding: 20rpx 24rpx;
}

.section-title {
  color: $color-text;
  font-size: 32rpx;
  font-weight: 700;
  letter-spacing: 0.5rpx;
}

.section-more {
  font-size: 24rpx;
  color: $color-text-muted;
  display: flex;
  align-items: center;
  gap: 4rpx;
  padding: 8rpx 16rpx;
  border-radius: 16rpx;
  transition: all $transition-fast;
  
  &:active {
    background: rgba(255, 255, 255, 0.05);
  }
}

.more-arrow {
  width: 24rpx;
  height: 24rpx;
  background-image: $icon-arrow-right;
  background-size: contain;
  background-repeat: no-repeat;
  background-position: center;
  opacity: 0.6;
}

/* Scroll X */
.scroll-x {
  width: 100%;
  white-space: nowrap;
}

/* Video List */
.video-list {
  display: flex;
  flex-direction: row;
  padding: 0 24rpx;
  gap: 20rpx;
}

/* Video Card - 增强卡片效果 */
.video-card {
  flex-shrink: 0;
  width: 200rpx;
  transition: transform $transition-normal;
}

.video-poster-wrap {
  position: relative;
  width: 200rpx;
  height: 280rpx;
  border-radius: 16rpx;
  overflow: hidden;
  background: $color-bg-secondary;
  box-shadow: $shadow-card;
  transition: all $transition-normal;
}

.video-poster {
  width: 100%;
  height: 100%;
  object-fit: cover;
  transition: transform $transition-slow;
}

.video-card:active .video-poster-wrap {
  transform: scale(1.02);
  box-shadow: 0 12rpx 32rpx rgba(0, 0, 0, 0.4);
}

.video-rate {
  position: absolute;
  top: 10rpx;
  right: 10rpx;
  background: $gradient-gold;
  color: $color-bg;
  font-size: 20rpx;
  font-weight: 700;
  padding: 4rpx 12rpx;
  border-radius: 8rpx;
  box-shadow: 0 2rpx 8rpx rgba(0, 0, 0, 0.3);
}

.video-episodes {
  position: absolute;
  bottom: 10rpx;
  left: 10rpx;
  background: rgba(0, 0, 0, 0.75);
  backdrop-filter: blur(4px);
  -webkit-backdrop-filter: blur(4px);
  color: $color-secondary;
  font-size: 18rpx;
  font-weight: 600;
  padding: 4rpx 12rpx;
  border-radius: 6rpx;
}

.video-info {
  padding: 14rpx 4rpx;
}

.video-title {
  color: $color-text;
  font-size: 26rpx;
  font-weight: 500;
  display: block;
  overflow: hidden;
  text-overflow: ellipsis;
  white-space: nowrap;
  line-height: 1.4;
}

/* Login Tip - 玻璃态卡片 */
.login-tip {
  display: flex;
  align-items: center;
  justify-content: space-between;
  padding: 28rpx;
  margin: 24rpx;
  background: rgba($color-bg-secondary, 0.5);
  backdrop-filter: blur(12px);
  -webkit-backdrop-filter: blur(12px);
  border-radius: 20rpx;
  border: 1rpx solid $color-border;
  box-shadow: $shadow-card;
  transition: all $transition-fast;
  
  &:active {
    transform: scale(0.98);
    background: rgba($color-bg-secondary, 0.7);
  }
}

.login-tip-icon {
  width: 80rpx;
  height: 80rpx;
  border-radius: 50%;
  background: rgba($color-primary, 0.15);
  display: flex;
  align-items: center;
  justify-content: center;
  
  .icon-user-circle {
    width: 44rpx;
    height: 44rpx;
    background-image: $icon-user;
    background-size: contain;
    background-repeat: no-repeat;
    background-position: center;
    opacity: 0.9;
  }
}

.login-tip-text {
  flex: 1;
  margin-left: 20rpx;
  color: $color-text-secondary;
  font-size: 28rpx;
  font-weight: 500;
}

.login-tip-btn {
  background: $gradient-primary;
  color: $color-text;
  font-size: 26rpx;
  font-weight: 600;
  padding: 18rpx 36rpx;
  border-radius: 32rpx;
  box-shadow: 0 4rpx 12rpx rgba($color-primary, 0.25);
}

/* Loading - 优化动画 */
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

/* Error State */
.error-state {
  display: flex;
  flex-direction: column;
  align-items: center;
  padding: 100rpx 48rpx;
}

.error-icon {
  width: 100rpx;
  height: 100rpx;
  margin-bottom: 28rpx;
  background-image: url("data:image/svg+xml,%3Csvg xmlns='http://www.w3.org/2000/svg' width='48' height='48' viewBox='0 0 24 24' fill='none' stroke='%23f5a623' stroke-width='2' stroke-linecap='round' stroke-linejoin='round'%3E%3Cpath d='M10.29 3.86L1.82 18a2 2 0 0 0 1.71 3h16.94a2 2 0 0 0 1.71-3L13.71 3.86a2 2 0 0 0-3.42 0z'/%3E%3Cline x1='12' y1='9' x2='12' y2='13'/%3E%3Cline x1='12' y1='17' x2='12.01' y2='17'/%3E%3C/svg%3E");
  background-size: contain;
  background-repeat: no-repeat;
  background-position: center;
  opacity: 0.8;
}

.error-text {
  color: $color-text;
  font-size: 32rpx;
  font-weight: 700;
  margin-bottom: 12rpx;
}

.error-tip {
  color: $color-text-muted;
  font-size: 26rpx;
  margin-bottom: 32rpx;
}

.error-btn {
  background: $gradient-primary;
  color: $color-text;
  font-size: 28rpx;
  font-weight: 600;
  padding: 20rpx 48rpx;
  border-radius: 40rpx;
  box-shadow: $shadow-primary;
  
  &:active {
    transform: scale(0.96);
  }
}

/* Empty State */
.empty-state {
  display: flex;
  flex-direction: column;
  align-items: center;
  padding: 100rpx 48rpx;
}

.empty-icon {
  width: 100rpx;
  height: 100rpx;
  margin-bottom: 28rpx;
  background-image: url("data:image/svg+xml,%3Csvg xmlns='http://www.w3.org/2000/svg' width='48' height='48' viewBox='0 0 24 24' fill='none' stroke='%23888898' stroke-width='2' stroke-linecap='round' stroke-linejoin='round'%3E%3Cpath d='M22 19a2 2 0 0 1-2 2H4a2 2 0 0 1-2-2V5a2 2 0 0 1 2-2h5l2 3h9a2 2 0 0 1 2 2z'/%3E%3C/svg%3E");
  background-size: contain;
  background-repeat: no-repeat;
  background-position: center;
  opacity: 0.5;
}

.empty-text {
  color: $color-text;
  font-size: 32rpx;
  font-weight: 600;
  margin-bottom: 12rpx;
}

.empty-tip {
  color: $color-text-muted;
  font-size: 26rpx;
}

/* Floating Search Button - 增强浮动效果 */
.float-search {
  position: fixed;
  right: 28rpx;
  bottom: calc(180rpx + constant(safe-area-inset-bottom));
  bottom: calc(180rpx + env(safe-area-inset-bottom));
  width: 100rpx;
  height: 100rpx;
  border-radius: 50%;
  background: $gradient-primary;
  display: flex;
  align-items: center;
  justify-content: center;
  box-shadow: $shadow-float, 0 0 40rpx rgba($color-primary, 0.2);
  z-index: 100;
  transition: all $transition-normal;
  
  &:active {
    transform: scale(0.9);
    box-shadow: 0 6rpx 20rpx rgba($color-primary, 0.4);
  }
}

.float-search-icon {
  width: 44rpx;
  height: 44rpx;
  background-image: $icon-search;
  background-size: contain;
  background-repeat: no-repeat;
  background-position: center;
  filter: brightness(0) invert(1);
}

/* Safe Area */
.safe-area-bottom {
  padding-bottom: constant(safe-area-inset-bottom);
  padding-bottom: env(safe-area-inset-bottom);
}

/* Responsive */
@media screen and (max-width: 375px) {
  .video-card { width: 160rpx; }
  .video-poster-wrap { width: 160rpx; height: 224rpx; }
  .hero-banner { height: 480rpx; }
}

@media screen and (min-width: 414px) {
  .video-card { width: 220rpx; }
  .video-poster-wrap { width: 220rpx; height: 308rpx; }
  .hero-banner { height: 600rpx; }
}

@media screen and (min-width: 768px) {
  .video-card { width: 240rpx; }
  .video-poster-wrap { width: 240rpx; height: 336rpx; }
  .video-list { gap: 24rpx; }
  .hero-banner { height: 720rpx; }
}
</style>
