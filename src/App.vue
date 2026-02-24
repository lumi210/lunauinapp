<script>
import { checkAndUpdateOnLaunch } from './utils/hot-update'

export default {
  onLaunch() {
    console.log('App Launch')
    // H5 环境下不设置导航栏颜色
    // #ifndef H5
    uni.setNavigationBarColor({
      frontColor: '#ffffff',
      backgroundColor: '#0f0f1a'
    })
    
    // 检查热更新
    checkAndUpdateOnLaunch()
    // #endif
  },
  onShow() {
    console.log('App Show')
  },
  onHide() {
    console.log('App Hide')
  }
}
</script>

<style>
/* ============================================
   LunaTV Design System v2.0
   Global Styles
   ============================================ */

/* 全局样式 */
page {
  background-color: #000000;
  color: #ffffff;
  font-family: -apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto, 'Helvetica Neue', Arial, 'PingFang SC', 'Microsoft YaHei', sans-serif;
  font-size: 28rpx;
  line-height: 1.6;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-rendering: optimizeLegibility;
}

/* 安全区域适配 */
.safe-area-top {
  padding-top: constant(safe-area-inset-top);
  padding-top: env(safe-area-inset-top);
}

.safe-area-bottom {
  padding-bottom: constant(safe-area-inset-bottom);
  padding-bottom: env(safe-area-inset-bottom);
}

/* 文字省略 */
.text-ellipsis {
  overflow: hidden;
  text-overflow: ellipsis;
  white-space: nowrap;
}

.text-ellipsis-2 {
  overflow: hidden;
  text-overflow: ellipsis;
  display: -webkit-box;
  -webkit-line-clamp: 2;
  -webkit-box-orient: vertical;
}

.text-ellipsis-3 {
  overflow: hidden;
  text-overflow: ellipsis;
  display: -webkit-box;
  -webkit-line-clamp: 3;
  -webkit-box-orient: vertical;
}

/* 清除按钮默认样式 */
button {
  margin: 0;
  padding: 0;
  background: none;
  border: none;
  line-height: inherit;
  font-weight: inherit;
}

button::after {
  border: none;
}

/* 输入框默认样式 */
input {
  box-sizing: border-box;
}

input::placeholder {
  color: #52525b;
}

/* 滚动条隐藏 */
::-webkit-scrollbar {
  display: none;
  width: 0;
  height: 0;
  color: transparent;
}

/* 图片默认样式 */
image {
  background-color: #0a0a0f;
}

/* 触摸反馈优化 */
view, button, text {
  -webkit-tap-highlight-color: transparent;
}

/* 选中文字样式 */
::selection {
  background-color: rgba(225, 29, 72, 0.3);
  color: #ffffff;
}

/* 全局过渡效果 */
view, text, image {
  transition-timing-function: cubic-bezier(0.16, 1, 0.3, 1);
}

/* 减少动画偏好 */
@media (prefers-reduced-motion: reduce) {
  *,
  *::before,
  *::after {
    animation-duration: 0.01ms !important;
    animation-iteration-count: 1 !important;
    transition-duration: 0.01ms !important;
    scroll-behavior: auto !important;
  }
}

/* Focus 样式 */
:focus-visible {
  outline: 2rpx solid #E11D48;
  outline-offset: 2rpx;
}

/* 高对比度模式 */
@media (prefers-contrast: high) {
  page {
    background-color: #000000;
  }
}
</style>
