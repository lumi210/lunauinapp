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
@import '../../styles/design-system.scss';

.page {
  min-height: 100vh;
  background: $bg-base;
  padding-top: 100rpx;
  padding-top: calc(100rpx + env(safe-area-inset-top));
}

/* ============================================
   Navigation Bar - 玻璃态效果
   ============================================ */

.custom-navbar {
  position: fixed;
  top: 0;
  left: 0;
  right: 0;
  height: 100rpx;
  z-index: 100;
  display: flex;
  align-items: center;
  padding: 0 $space-6;
  padding-top: constant(safe-area-inset-top);
  padding-top: env(safe-area-inset-top);
}

.custom-navbar::before {
  content: '';
  position: absolute;
  inset: 0;
  background: rgba(10, 10, 15, 0.8);
  backdrop-filter: blur(30px) saturate(180%);
  -webkit-backdrop-filter: blur(30px) saturate(180%);
}

.navbar-title {
  position: relative;
  color: $brand-primary;
  font-size: 36rpx;
  font-weight: 700;
  margin-right: $space-6;
  letter-spacing: 1rpx;
}

.navbar-search {
  position: relative;
  flex: 1;
  height: 72rpx;
  background: rgba(255, 255, 255, 0.04);
  border-radius: $radius-full;
  display: flex;
  align-items: center;
  padding: 0 $space-5;
  border: 1rpx solid rgba(255, 255, 255, 0.06);
  transition: all $duration-fast $ease-out;
  
  &:active {
    background: rgba(255, 255, 255, 0.06);
    border-color: rgba(225, 29, 72, 0.3);
  }
}

.search-icon {
  width: 32rpx;
  height: 32rpx;
  margin-right: $space-3;
  background-image: $icon-search;
  background-size: contain;
  background-repeat: no-repeat;
  background-position: center;
  opacity: 0.6;
}

.search-placeholder {
  color: $text-muted;
  font-size: 28rpx;
}

/* ============================================
   Hero Banner - 沉浸式设计
   ============================================ */

.hero-banner {
  position: relative;
  width: 100%;
  height: 520rpx;
  overflow: hidden;
  margin-top: -100rpx;
  margin-top: calc(-100rpx - env(safe-area-inset-top));
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
  inset: 0;
  background: linear-gradient(180deg, 
    rgba(0, 0, 0, 0.25) 0%, 
    rgba(0, 0, 0, 0) 15%,
    rgba(0, 0, 0, 0) 40%,
    rgba(0, 0, 0, 0.7) 75%,
    rgba(0, 0, 0, 0.95) 90%,
    $bg-base 100%
  );
  pointer-events: none;
}

.hero-content {
  position: absolute;
  left: $space-6;
  right: $space-6;
  bottom: $space-10;
}

.hero-info {
  margin-bottom: $space-6;
}

.hero-title {
  font-size: 48rpx;
  font-weight: 700;
  color: $text-primary;
  text-shadow: 0 4rpx 20rpx rgba(0, 0, 0, 0.6);
  display: block;
  margin-bottom: $space-4;
  max-width: 85%;
  letter-spacing: 0.5rpx;
  line-height: 1.2;
}

.hero-meta {
  display: flex;
  align-items: center;
  gap: $space-4;
}

.hero-rate {
  display: inline-flex;
  align-items: center;
  gap: $space-1;
  padding: $space-1 $space-4;
  background: $gradient-gold;
  border-radius: $radius-sm;
  font-size: 22rpx;
  font-weight: 700;
  color: $bg-base;
}

.hero-year {
  font-size: 26rpx;
  color: $text-secondary;
  font-weight: 500;
}

.hero-desc {
  color: $text-secondary;
  font-size: 24rpx;
  display: block;
  margin-top: $space-3;
  overflow: hidden;
  text-overflow: ellipsis;
  white-space: nowrap;
  max-width: 85%;
}

.hero-actions {
  display: flex;
  gap: $space-4;
}

.hero-btn {
  display: inline-flex;
  align-items: center;
  justify-content: center;
  gap: $space-2;
  padding: $space-4 $space-6;
  border-radius: $radius-full;
  font-size: 28rpx;
  font-weight: 600;
  transition: all $duration-fast $ease-out;
  
  text {
    color: white;
  }
}

.hero-btn-icon {
  width: 28rpx;
  height: 28rpx;
  background-size: contain;
  background-repeat: no-repeat;
  background-position: center;
  
  &.icon-play-sm {
    background-image: $icon-play;
  }
  
  &.icon-info-sm {
    background-image: $icon-info;
  }
}

.hero-btn-play {
  background: $gradient-primary;
  box-shadow: $shadow-primary;
  
  &:active {
    transform: scale(0.95);
    box-shadow: 0 4rpx 20rpx rgba(225, 29, 72, 0.5);
  }
}

.hero-btn-info {
  background: rgba(255, 255, 255, 0.12);
  backdrop-filter: blur(10px);
  -webkit-backdrop-filter: blur(10px);
  border: 1rpx solid rgba(255, 255, 255, 0.1);
  
  &:active {
    background: rgba(255, 255, 255, 0.2);
    transform: scale(0.95);
  }
}

.hero-indicators {
  position: absolute;
  right: $space-6;
  bottom: $space-6;
  display: flex;
  gap: $space-2;
}

.hero-indicator {
  width: 12rpx;
  height: 12rpx;
  border-radius: $radius-full;
  background: rgba(255, 255, 255, 0.25);
  transition: all $duration-normal $ease-out;
  
  &.active {
    width: 32rpx;
    border-radius: 6rpx;
    background: $brand-primary;
  }
}

/* ============================================
   Category Tabs
   ============================================ */

.quick-cats {
  display: flex;
  gap: $space-3;
  padding: $space-4 $space-6;
  overflow-x: auto;
  white-space: nowrap;
  
  &::-webkit-scrollbar {
    display: none;
  }
}

.quick-cat-item {
  flex-shrink: 0;
  padding: $space-3 $space-6;
  font-size: 28rpx;
  font-weight: 500;
  color: $text-muted;
  background: rgba(255, 255, 255, 0.03);
  border-radius: $radius-full;
  border: 1rpx solid transparent;
  transition: all $duration-fast $ease-out;
  
  &:active {
    transform: scale(0.95);
  }
  
  &.active {
    background: $gradient-primary;
    color: white;
    box-shadow: $shadow-primary;
  }
}

/* ============================================
   Content & Sections
   ============================================ */

.content {
  padding-top: $space-2;
}

.section {
  margin-bottom: $space-10;
}

.section-header {
  display: flex;
  align-items: center;
  justify-content: space-between;
  padding: $space-4 $space-6 $space-3;
}

.section-title {
  font-size: 32rpx;
  font-weight: 700;
  color: $text-primary;
  letter-spacing: 0.3rpx;
}

.section-more {
  display: flex;
  align-items: center;
  gap: $space-1;
  font-size: 24rpx;
  color: $text-muted;
  padding: $space-2 $space-3;
  border-radius: $radius-md;
  transition: all $duration-fast $ease-out;
  
  &:active {
    background: rgba(255, 255, 255, 0.04);
  }
}

.more-arrow {
  width: 24rpx;
  height: 24rpx;
  background-image: $icon-arrow-right;
  background-size: contain;
  background-repeat: no-repeat;
  background-position: center;
  opacity: 0.5;
}

/* ============================================
   Video Cards
   ============================================ */

.scroll-x {
  white-space: nowrap;
  overflow-x: auto;
  overflow-y: hidden;
  -webkit-overflow-scrolling: touch;
  padding: 0 $space-6;
  
  &::-webkit-scrollbar {
    display: none;
  }
}

.video-list {
  display: inline-flex;
  gap: $space-4;
}

.video-card {
  position: relative;
  width: 220rpx;
  border-radius: $radius-lg;
  overflow: hidden;
  background: $bg-elevated;
  flex-shrink: 0;
  transition: transform $duration-normal $ease-out;
  
  &:active {
    transform: scale(0.96);
  }
}

.video-poster-wrap {
  position: relative;
  width: 100%;
  height: 308rpx;
  overflow: hidden;
}

.video-poster {
  width: 100%;
  height: 100%;
  object-fit: cover;
  transition: transform $duration-slow $ease-out;
}

.video-card:active .video-poster {
  transform: scale(1.05);
}

.video-rate {
  position: absolute;
  top: $space-3;
  right: $space-3;
  padding: $space-1 $space-3;
  background: $gradient-gold;
  border-radius: $radius-sm;
  font-size: 20rpx;
  font-weight: 700;
  color: $bg-base;
  box-shadow: $shadow-gold;
}

.video-episodes {
  position: absolute;
  bottom: $space-3;
  left: $space-3;
  padding: $space-1 $space-3;
  background: rgba(0, 0, 0, 0.75);
  backdrop-filter: blur(8px);
  -webkit-backdrop-filter: blur(8px);
  border-radius: $radius-sm;
  font-size: 20rpx;
  font-weight: 600;
  color: $accent-cyan;
}

.video-info {
  padding: $space-4;
}

.video-title {
  font-size: 26rpx;
  font-weight: 500;
  color: $text-primary;
  white-space: nowrap;
  overflow: hidden;
  text-overflow: ellipsis;
  line-height: 1.4;
}

/* ============================================
   Login Tip Card
   ============================================ */

.login-tip {
  display: flex;
  align-items: center;
  margin: $space-6;
  padding: $space-6;
  background: rgba(255, 255, 255, 0.02);
  backdrop-filter: blur(20px) saturate(150%);
  -webkit-backdrop-filter: blur(20px) saturate(150%);
  border-radius: $radius-xl;
  border: 1rpx solid rgba(255, 255, 255, 0.04);
  transition: all $duration-fast $ease-out;
  
  &:active {
    transform: scale(0.98);
    background: rgba(255, 255, 255, 0.04);
  }
}

.login-tip-icon {
  width: 80rpx;
  height: 80rpx;
  border-radius: $radius-full;
  background: rgba(225, 29, 72, 0.12);
  display: flex;
  align-items: center;
  justify-content: center;
  
  .icon-user-circle {
    width: 40rpx;
    height: 40rpx;
    background-image: $icon-user;
    background-size: contain;
    background-repeat: no-repeat;
    background-position: center;
    opacity: 0.8;
    filter: brightness(0) saturate(100%) invert(46%) sepia(93%) saturate(5737%) hue-rotate(337deg) brightness(97%) contrast(94%);
  }
}

.login-tip-text {
  flex: 1;
  margin-left: $space-5;
  font-size: 28rpx;
  font-weight: 500;
  color: $text-secondary;
}

.login-tip-btn {
  padding: $space-4 $space-6;
  background: $gradient-primary;
  border-radius: $radius-full;
  font-size: 26rpx;
  font-weight: 600;
  color: white;
  box-shadow: $shadow-primary;
  transition: all $duration-fast $ease-out;
  
  &:active {
    transform: scale(0.95);
  }
}

/* ============================================
   Loading & States
   ============================================ */

.loading {
  display: flex;
  flex-direction: column;
  align-items: center;
  padding: $space-16 $space-10;
  
  text {
    margin-top: $space-5;
    font-size: 26rpx;
    color: $text-muted;
  }
}

.loading-spinner {
  width: 56rpx;
  height: 56rpx;
  border: 4rpx solid rgba(255, 255, 255, 0.1);
  border-top-color: $brand-primary;
  border-radius: $radius-full;
  animation: spin 0.8s linear infinite;
}

.error-state {
  display: flex;
  flex-direction: column;
  align-items: center;
  padding: $space-16 $space-10;
}

.error-icon {
  width: 100rpx;
  height: 100rpx;
  margin-bottom: $space-6;
  background-image: url("data:image/svg+xml,%3Csvg xmlns='http://www.w3.org/2000/svg' width='48' height='48' viewBox='0 0 24 24' fill='none' stroke='%23FBBF24' stroke-width='2' stroke-linecap='round' stroke-linejoin='round'%3E%3Cpath d='M10.29 3.86L1.82 18a2 2 0 0 0 1.71 3h16.94a2 2 0 0 0 1.71-3L13.71 3.86a2 2 0 0 0-3.42 0z'/%3E%3Cline x1='12' y1='9' x2='12' y2='13'/%3E%3Cline x1='12' y1='17' x2='12.01' y2='17'/%3E%3C/svg%3E");
  background-size: contain;
  background-repeat: no-repeat;
  background-position: center;
  opacity: 0.6;
}

.error-text {
  font-size: 32rpx;
  font-weight: 600;
  color: $text-primary;
}

.empty-state {
  display: flex;
  flex-direction: column;
  align-items: center;
  padding: $space-16 $space-10;
}

.empty-icon {
  width: 120rpx;
  height: 120rpx;
  margin-bottom: $space-6;
  background-image: url("data:image/svg+xml,%3Csvg xmlns='http://www.w3.org/2000/svg' width='48' height='48' viewBox='0 0 24 24' fill='none' stroke='%2371717A' stroke-width='2' stroke-linecap='round' stroke-linejoin='round'%3E%3Cpath d='M22 19a2 2 0 0 1-2 2H4a2 2 0 0 1-2-2V5a2 2 0 0 1 2-2h5l2 3h9a2 2 0 0 1 2 2z'/%3E%3C/svg%3E");
  background-size: contain;
  background-repeat: no-repeat;
  background-position: center;
  opacity: 0.4;
}

.empty-text {
  font-size: 32rpx;
  font-weight: 600;
  color: $text-primary;
  margin-bottom: $space-3;
}

.empty-tip {
  font-size: 26rpx;
  color: $text-muted;
}

/* ============================================
   Floating Action Button
   ============================================ */

.float-search {
  position: fixed;
  right: $space-6;
  bottom: calc(200rpx + constant(safe-area-inset-bottom));
  bottom: calc(200rpx + env(safe-area-inset-bottom));
  width: 108rpx;
  height: 108rpx;
  border-radius: $radius-full;
  background: $gradient-primary;
  display: flex;
  align-items: center;
  justify-content: center;
  box-shadow: $shadow-primary, 0 0 48rpx rgba(225, 29, 72, 0.25);
  z-index: 90;
  transition: all $duration-normal $ease-out;
  animation: breathe 3s ease-in-out infinite;
  
  &:active {
    transform: scale(0.9);
    box-shadow: 0 4rpx 20rpx rgba(225, 29, 72, 0.5);
    animation: none;
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

@keyframes breathe {
  0%, 100% { 
    transform: scale(1);
    box-shadow: $shadow-primary, 0 0 48rpx rgba(225, 29, 72, 0.25);
  }
  50% { 
    transform: scale(1.05);
    box-shadow: $shadow-primary, 0 0 64rpx rgba(225, 29, 72, 0.4);
  }
}

/* ============================================
   Safe Area
   ============================================ */

.safe-area-bottom {
  padding-bottom: constant(safe-area-inset-bottom);
  padding-bottom: env(safe-area-inset-bottom);
}

/* ============================================
   Responsive
   ============================================ */

@media screen and (max-width: 375px) {
  .video-card { width: 180rpx; }
  .video-poster-wrap { height: 252rpx; }
  .hero-banner { height: 440rpx; }
  .hero-title { font-size: 40rpx; }
}

@media screen and (min-width: 414px) {
  .video-card { width: 240rpx; }
  .video-poster-wrap { height: 336rpx; }
  .hero-banner { height: 580rpx; }
  .hero-title { font-size: 52rpx; }
}

@media screen and (min-width: 768px) {
  .video-card { width: 280rpx; }
  .video-poster-wrap { height: 392rpx; }
  .video-list { gap: $space-6; }
  .hero-banner { height: 680rpx; }
}
</style>
