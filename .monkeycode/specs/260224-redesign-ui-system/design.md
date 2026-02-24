# LunaTV 全新设计系统

**项目:** LunaTV 影视聚合平台
**更新日期:** 2026-02-24
**版本:** 2.0

---

## 一、设计理念

### 核心原则

1. **沉浸式体验** - 内容优先，界面后退
2. **流畅动效** - 微交互无处不在
3. **视觉层次** - 清晰的信息架构
4. **触感反馈** - 每次交互都有回应

### 风格定位

- **Netflix Premium** - 高端流媒体质感
- **Glassmorphism 2.0** - 现代毛玻璃效果
- **Micro-interactions** - 精致的微动效
- **Dark Mode OLED** - 纯黑省电高对比

---

## 二、色彩系统

### 主色板

```scss
// 品牌色
$brand-primary: #E11D48;        // 玫红 - 主行动色
$brand-primary-light: #FB7185;  // 浅玫红
$brand-primary-dark: #BE123C;   // 深玫红

// 背景色 (OLED 纯黑系)
$bg-base: #000000;              // 纯黑 - 主背景
$bg-elevated: #0A0A0F;          // 微蓝黑 - 卡片
$bg-surface: #12121A;           // 表面层
$bg-overlay: rgba(0,0,0,0.85);  // 遮罩层

// 文字色
$text-primary: #FFFFFF;         // 主文字
$text-secondary: #A1A1AA;       // 次要文字
$text-muted: #71717A;           // 弱化文字
$text-accent: #FB7185;          // 强调文字

// 功能色
$accent-gold: #FBBF24;          // 评分/会员
$accent-cyan: #22D3EE;          // 直播/更新
$accent-emerald: #34D399;       // 成功
$accent-violet: #A78BFA;        // 特殊标签

// 渐变色
$gradient-primary: linear-gradient(135deg, #E11D48 0%, #BE123C 100%);
$gradient-gold: linear-gradient(135deg, #FBBF24 0%, #F59E0B 100%);
$gradient-shine: linear-gradient(90deg, transparent, rgba(255,255,255,0.1), transparent);
```

### 语义色

```scss
$color-success: #34D399;
$color-warning: #FBBF24;
$color-error: #EF4444;
$color-info: #22D3EE;
```

---

## 三、字体系统

### 字体家族

```scss
// 主字体 - 现代无衬线
$font-display: 'Satoshi', -apple-system, sans-serif;
$font-body: 'General Sans', 'PingFang SC', sans-serif;
$font-mono: 'JetBrains Mono', monospace;

// 回退字体
$font-fallback: -apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto, 'Helvetica Neue', Arial, 'PingFang SC', 'Microsoft YaHei', sans-serif;
```

### 字体层级

```scss
// 标题
$font-hero: 56rpx / 1.1 $font-display;        // 超大标题
$font-h1: 44rpx / 1.15 $font-display;          // 一级标题
$font-h2: 36rpx / 1.2 $font-display;           // 二级标题
$font-h3: 30rpx / 1.25 $font-display;          // 三级标题

// 正文
$font-body-lg: 30rpx / 1.5 $font-body;         // 大正文
$font-body: 28rpx / 1.6 $font-body;            // 正文
$font-body-sm: 26rpx / 1.5 $font-body;         // 小正文

// 辅助
$font-caption: 22rpx / 1.4 $font-body;         // 说明文字
$font-overline: 20rpx / 1.3 $font-body;        // 标签文字

// 字重
$font-regular: 400;
$font-medium: 500;
$font-semibold: 600;
$font-bold: 700;
```

---

## 四、间距系统

```scss
$space-0: 0;
$space-1: 4rpx;      // 极小间距
$space-2: 8rpx;      // 微小间距
$space-3: 12rpx;     // 小间距
$space-4: 16rpx;     // 标准间距
$space-5: 20rpx;     // 中等间距
$space-6: 24rpx;     // 大间距
$space-8: 32rpx;     // 区块间距
$space-10: 40rpx;    // 大区块
$space-12: 48rpx;    // 章节间距
$space-16: 64rpx;    // 大章节
$space-20: 80rpx;    // 页面间距
```

---

## 五、圆角系统

```scss
$radius-none: 0;
$radius-sm: 8rpx;      // 小元素
$radius-md: 12rpx;     // 按钮、输入框
$radius-lg: 16rpx;     // 卡片
$radius-xl: 24rpx;     // 大卡片、弹窗
$radius-2xl: 32rpx;    // 底部弹窗
$radius-full: 9999rpx; // 圆形
```

---

## 六、阴影系统

```scss
// 基础阴影
$shadow-sm: 0 2rpx 8rpx rgba(0, 0, 0, 0.3);
$shadow-md: 0 4rpx 16rpx rgba(0, 0, 0, 0.4);
$shadow-lg: 0 8rpx 32rpx rgba(0, 0, 0, 0.5);
$shadow-xl: 0 16rpx 48rpx rgba(0, 0, 0, 0.6);

// 彩色阴影
$shadow-primary: 0 8rpx 32rpx rgba(225, 29, 72, 0.35);
$shadow-gold: 0 8rpx 32rpx rgba(251, 191, 36, 0.35);

// 内阴影
$shadow-inner: inset 0 2rpx 8rpx rgba(0, 0, 0, 0.3);
```

---

## 七、动效系统

### 过渡时间

```scss
$duration-instant: 50ms;    // 瞬间反馈
$duration-fast: 150ms;      // 快速过渡
$duration-normal: 250ms;    // 标准过渡
$duration-slow: 400ms;      // 慢速过渡
$duration-slower: 600ms;    // 更慢过渡
```

### 缓动函数

```scss
$ease-out: cubic-bezier(0.16, 1, 0.3, 1);      // 快出慢停
$ease-in-out: cubic-bezier(0.65, 0, 0.35, 1);  // 平滑进出
$ease-spring: cubic-bezier(0.34, 1.56, 0.64, 1); // 弹性效果
$ease-bounce: cubic-bezier(0.68, -0.55, 0.265, 1.55); // 回弹效果
```

### 动画关键帧

```scss
// 淡入
@keyframes fadeIn {
  from { opacity: 0; }
  to { opacity: 1; }
}

// 上滑淡入
@keyframes slideUp {
  from { opacity: 0; transform: translateY(20rpx); }
  to { opacity: 1; transform: translateY(0); }
}

// 缩放淡入
@keyframes scaleIn {
  from { opacity: 0; transform: scale(0.95); }
  to { opacity: 1; transform: scale(1); }
}

// 闪烁
@keyframes shimmer {
  0% { background-position: -200% 0; }
  100% { background-position: 200% 0; }
}

// 脉冲
@keyframes pulse {
  0%, 100% { opacity: 1; }
  50% { opacity: 0.6; }
}

// 浮动
@keyframes float {
  0%, 100% { transform: translateY(0); }
  50% { transform: translateY(-8rpx); }
}
```

---

## 八、组件规范

### 8.1 导航栏 (Navigation Bar)

```scss
.navbar {
  position: fixed;
  top: 0;
  left: 0;
  right: 0;
  height: 100rpx;
  background: linear-gradient(180deg, rgba(0,0,0,0.9) 0%, rgba(0,0,0,0) 100%);
  backdrop-filter: blur(20px) saturate(180%);
  -webkit-backdrop-filter: blur(20px) saturate(180%);
  z-index: 100;
  padding: 0 $space-6;
  padding-top: env(safe-area-inset-top);
}

.navbar-blur {
  position: absolute;
  inset: 0;
  background: rgba(10, 10, 15, 0.7);
  backdrop-filter: blur(30px);
  mask-image: linear-gradient(180deg, black 0%, transparent 100%);
}

.navbar-title {
  font: $font-h3;
  font-weight: $font-bold;
  color: $text-primary;
  letter-spacing: 0.5rpx;
}
```

### 8.2 视频卡片 (Video Card)

```scss
.video-card {
  position: relative;
  width: 220rpx;
  border-radius: $radius-lg;
  overflow: hidden;
  background: $bg-elevated;
  transition: transform $duration-normal $ease-out,
              box-shadow $duration-normal $ease-out;
  
  &:active {
    transform: scale(0.97);
  }
}

.video-card-poster {
  width: 100%;
  height: 308rpx;
  object-fit: cover;
  transition: transform $duration-slow $ease-out;
}

.video-card:hover .video-card-poster {
  transform: scale(1.05);
}

.video-card-overlay {
  position: absolute;
  inset: 0;
  background: linear-gradient(180deg, 
    transparent 0%, 
    transparent 50%, 
    rgba(0,0,0,0.9) 100%
  );
  opacity: 0;
  transition: opacity $duration-normal $ease-out;
}

.video-card:hover .video-card-overlay {
  opacity: 1;
}

.video-card-rating {
  position: absolute;
  top: $space-3;
  right: $space-3;
  padding: $space-1 $space-3;
  background: $gradient-gold;
  border-radius: $radius-sm;
  font-size: $font-caption;
  font-weight: $font-bold;
  color: $bg-base;
  box-shadow: $shadow-gold;
}

.video-card-title {
  padding: $space-4;
  font: $font-body-sm;
  font-weight: $font-medium;
  color: $text-primary;
  white-space: nowrap;
  overflow: hidden;
  text-overflow: ellipsis;
}
```

### 8.3 按钮 (Button)

```scss
.btn {
  display: inline-flex;
  align-items: center;
  justify-content: center;
  gap: $space-2;
  padding: $space-4 $space-6;
  border-radius: $radius-md;
  font: $font-body;
  font-weight: $font-semibold;
  transition: all $duration-fast $ease-out;
  cursor: pointer;
  user-select: none;
  
  &:active {
    transform: scale(0.97);
  }
}

.btn-primary {
  background: $gradient-primary;
  color: white;
  box-shadow: $shadow-primary;
  
  &:active {
    box-shadow: 0 4rpx 16rpx rgba(225, 29, 72, 0.4);
  }
}

.btn-secondary {
  background: rgba(255, 255, 255, 0.08);
  color: $text-primary;
  border: 1rpx solid rgba(255, 255, 255, 0.1);
  backdrop-filter: blur(10px);
  
  &:active {
    background: rgba(255, 255, 255, 0.15);
  }
}

.btn-ghost {
  background: transparent;
  color: $text-secondary;
  
  &:active {
    background: rgba(255, 255, 255, 0.05);
  }
}

.btn-lg {
  padding: $space-5 $space-8;
  font-size: 32rpx;
  border-radius: $radius-lg;
}

.btn-sm {
  padding: $space-2 $space-4;
  font-size: 24rpx;
  border-radius: $radius-sm;
}

.btn-icon {
  width: 88rpx;
  height: 88rpx;
  padding: 0;
  border-radius: $radius-full;
}
```

### 8.4 输入框 (Input)

```scss
.input-wrapper {
  position: relative;
}

.input {
  width: 100%;
  height: 96rpx;
  padding: 0 $space-6;
  background: rgba(255, 255, 255, 0.05);
  border: 2rpx solid transparent;
  border-radius: $radius-lg;
  font: $font-body;
  color: $text-primary;
  transition: all $duration-fast $ease-out;
  
  &::placeholder {
    color: $text-muted;
  }
  
  &:focus {
    background: rgba(255, 255, 255, 0.08);
    border-color: $brand-primary;
    box-shadow: 0 0 0 6rpx rgba(225, 29, 72, 0.15);
  }
}

.input-icon {
  position: absolute;
  left: $space-5;
  top: 50%;
  transform: translateY(-50%);
  width: 40rpx;
  height: 40rpx;
  color: $text-muted;
}
```

### 8.5 卡片容器 (Card)

```scss
.card {
  background: $bg-elevated;
  border-radius: $radius-xl;
  padding: $space-6;
  border: 1rpx solid rgba(255, 255, 255, 0.05);
  
  // 玻璃态变体
  &.card-glass {
    background: rgba(255, 255, 255, 0.03);
    backdrop-filter: blur(20px) saturate(150%);
    -webkit-backdrop-filter: blur(20px) saturate(150%);
  }
  
  // 渐变边框变体
  &.card-gradient-border {
    position: relative;
    background: $bg-elevated;
    
    &::before {
      content: '';
      position: absolute;
      inset: 0;
      padding: 1rpx;
      border-radius: $radius-xl;
      background: linear-gradient(135deg, rgba(225,29,72,0.5) 0%, transparent 50%);
      mask: linear-gradient(#fff 0 0) content-box, linear-gradient(#fff 0 0);
      mask-composite: exclude;
      -webkit-mask-composite: xor;
      pointer-events: none;
    }
  }
}
```

### 8.6 标签 (Tag)

```scss
.tag {
  display: inline-flex;
  align-items: center;
  padding: $space-1 $space-3;
  border-radius: $radius-sm;
  font-size: $font-caption;
  font-weight: $font-medium;
  
  &.tag-primary {
    background: rgba(225, 29, 72, 0.15);
    color: $brand-primary-light;
  }
  
  &.tag-gold {
    background: $gradient-gold;
    color: $bg-base;
  }
  
  &.tag-cyan {
    background: rgba(34, 211, 238, 0.15);
    color: $accent-cyan;
  }
}
```

### 8.7 底部导航 (Tab Bar)

```scss
.tabbar {
  position: fixed;
  bottom: 0;
  left: 0;
  right: 0;
  height: 120rpx;
  padding-bottom: env(safe-area-inset-bottom);
  background: rgba(10, 10, 15, 0.95);
  backdrop-filter: blur(30px) saturate(180%);
  -webkit-backdrop-filter: blur(30px) saturate(180%);
  border-top: 1rpx solid rgba(255, 255, 255, 0.05);
  display: flex;
  align-items: center;
  justify-content: space-around;
  z-index: 100;
}

.tabbar-item {
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: $space-1;
  padding: $space-3 $space-5;
  transition: all $duration-fast $ease-out;
  
  &.active {
    .tabbar-icon {
      color: $brand-primary;
      transform: scale(1.1);
    }
    .tabbar-label {
      color: $brand-primary;
    }
  }
}

.tabbar-icon {
  width: 44rpx;
  height: 44rpx;
  color: $text-muted;
  transition: all $duration-fast $ease-spring;
}

.tabbar-label {
  font-size: $font-caption;
  color: $text-muted;
  transition: color $duration-fast $ease-out;
}
```

### 8.8 Hero Banner

```scss
.hero-banner {
  position: relative;
  width: 100%;
  height: 520rpx;
  overflow: hidden;
}

.hero-backdrop {
  width: 100%;
  height: 100%;
  object-fit: cover;
  transition: transform 8s linear;
}

.hero-gradient {
  position: absolute;
  inset: 0;
  background: linear-gradient(180deg, 
    rgba(0,0,0,0.3) 0%,
    transparent 20%,
    transparent 50%,
    rgba(0,0,0,0.8) 80%,
    $bg-base 100%
  );
}

.hero-content {
  position: absolute;
  left: $space-6;
  right: $space-6;
  bottom: $space-8;
}

.hero-title {
  font: $font-hero;
  font-weight: $font-bold;
  color: $text-primary;
  text-shadow: 0 4rpx 16rpx rgba(0,0,0,0.5);
  max-width: 80%;
}

.hero-meta {
  display: flex;
  align-items: center;
  gap: $space-4;
  margin-top: $space-4;
}

.hero-rating {
  display: inline-flex;
  align-items: center;
  gap: $space-1;
  padding: $space-1 $space-3;
  background: $gradient-gold;
  border-radius: $radius-sm;
  font-size: $font-caption;
  font-weight: $font-bold;
  color: $bg-base;
}

.hero-actions {
  display: flex;
  gap: $space-4;
  margin-top: $space-6;
}

.hero-btn-play {
  padding: $space-4 $space-8;
  background: $gradient-primary;
  border-radius: $radius-full;
  color: white;
  font-weight: $font-semibold;
  box-shadow: $shadow-primary;
  
  &:active {
    transform: scale(0.96);
  }
}

.hero-btn-info {
  padding: $space-4 $space-6;
  background: rgba(255, 255, 255, 0.15);
  backdrop-filter: blur(10px);
  border-radius: $radius-full;
  color: white;
  font-weight: $font-medium;
  border: 1rpx solid rgba(255, 255, 255, 0.1);
}
```

---

## 九、页面布局规范

### 9.1 页面容器

```scss
.page {
  min-height: 100vh;
  background: $bg-base;
  padding-top: 100rpx;
  padding-top: calc(100rpx + env(safe-area-inset-top));
  padding-bottom: 120rpx;
  padding-bottom: calc(120rpx + env(safe-area-inset-bottom));
}
```

### 9.2 内容区域

```scss
.content {
  padding: $space-6;
}

.section {
  margin-bottom: $space-10;
}

.section-header {
  display: flex;
  align-items: center;
  justify-content: space-between;
  margin-bottom: $space-5;
}

.section-title {
  font: $font-h3;
  font-weight: $font-bold;
  color: $text-primary;
}

.section-more {
  display: flex;
  align-items: center;
  gap: $space-1;
  font-size: $font-body-sm;
  color: $text-muted;
  padding: $space-2 $space-3;
  border-radius: $radius-md;
  transition: all $duration-fast $ease-out;
  
  &:active {
    background: rgba(255, 255, 255, 0.05);
  }
}
```

---

## 十、特殊效果

### 10.1 玻璃态效果

```scss
.glass {
  background: rgba(255, 255, 255, 0.03);
  backdrop-filter: blur(20px) saturate(150%);
  -webkit-backdrop-filter: blur(20px) saturate(150%);
  border: 1rpx solid rgba(255, 255, 255, 0.05);
}

.glass-strong {
  background: rgba(10, 10, 15, 0.8);
  backdrop-filter: blur(30px) saturate(180%);
  -webkit-backdrop-filter: blur(30px) saturate(180%);
}
```

### 10.2 光泽扫过效果

```scss
.shine {
  position: relative;
  overflow: hidden;
  
  &::after {
    content: '';
    position: absolute;
    top: 0;
    left: -100%;
    width: 100%;
    height: 100%;
    background: linear-gradient(90deg, 
      transparent, 
      rgba(255,255,255,0.1), 
      transparent
    );
    animation: shine 2s infinite;
  }
}

@keyframes shine {
  0% { left: -100%; }
  50%, 100% { left: 100%; }
}
```

### 10.3 骨架屏

```scss
.skeleton {
  background: linear-gradient(90deg,
    $bg-elevated 0%,
    rgba(255,255,255,0.05) 50%,
    $bg-elevated 100%
  );
  background-size: 200% 100%;
  animation: shimmer 1.5s infinite;
  border-radius: $radius-md;
}
```

---

## 十一、响应式断点

```scss
$breakpoint-xs: 320px;
$breakpoint-sm: 375px;
$breakpoint-md: 414px;
$breakpoint-lg: 428px;
$breakpoint-xl: 768px;
$breakpoint-2xl: 1024px;

@mixin respond-to($breakpoint) {
  @media screen and (min-width: $breakpoint) {
    @content;
  }
}
```

---

## 十二、无障碍支持

```scss
// 减少动画偏好
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

// 焦点可见
:focus-visible {
  outline: 2rpx solid $brand-primary;
  outline-offset: 2rpx;
}

// 高对比度模式
@media (prefers-contrast: high) {
  .video-card {
    border: 2rpx solid $text-secondary;
  }
}
```

---

## 十三、实施优先级

| 优先级 | 组件 | 说明 |
|--------|------|------|
| P0 | 色彩系统 | 全局变量重构 |
| P0 | 字体系统 | 引入新字体 |
| P0 | 导航栏 | 玻璃态 + 渐变 |
| P0 | Hero Banner | 沉浸式设计 |
| P0 | 视频卡片 | 悬停效果 + 阴影 |
| P1 | 按钮 | 渐变 + 阴影 |
| P1 | 输入框 | 焦点状态优化 |
| P1 | 底部导航 | 玻璃态 + 图标动画 |
| P2 | 列表项 | 微交互优化 |
| P2 | 空状态 | 插图 + 动画 |
| P2 | 加载状态 | 骨架屏优化 |
