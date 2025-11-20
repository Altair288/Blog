---
layout: post
title: 微信小程序学习笔记-2
date: 2025-10-10
Author: 
categories: 
tags: [miniprogram2]
comments: false
toc: true
---


# 微信小程序轮播图学习笔记

> 目标：掌握在微信小程序中使用 `swiper` 组件实现基础轮播图，并能做常见配置和简单封装。

---

## 一、轮播图实现思路概览

在小程序中实现轮播图，主要依靠官方提供的组件：

- 容器组件：`<swiper />`
- 子项组件：`<swiper-item />`

核心步骤：

1. 在 `wxml` 中使用 `swiper` + `swiper-item` 结构
2. 在 `js` 中准备轮播图数据（比如图片数组）
3. 在 `wxss` 中设置容器高度 / 宽度等样式
4. 可选：监听滑动事件、配置自动切换、指示点等

---

## 二、最简单的轮播图示例

### 2.1 WXML 结构

```wxml
<!-- pages/banner/banner.wxml -->
<view class="banner-container">
  <swiper
    class="banner-swiper"
    indicator-dots="true"
    autoplay="true"
    interval="3000"
    duration="500"
    circular="true"
  >
    <block wx:for="{{imgList}}" wx:key="url">
      <swiper-item>
        <image class="banner-img" src="{{item.url}}" mode="aspectFill" />
      </swiper-item>
    </block>
  </swiper>
</view>
```

### 2.2 JS 数据

```js
// pages/banner/banner.js
Page({
  data: {
    imgList: [
      { url: 'https://example.com/banner1.jpg' },
      { url: 'https://example.com/banner2.jpg' },
      { url: 'https://example.com/banner3.jpg' }
    ]
  }
})
```

### 2.3 WXSS 样式

```css
/* pages/banner/banner.wxss */
.banner-container {
  width: 100%;
}

.banner-swiper {
  width: 100%;
  height: 300rpx; /* 轮播图高度可根据设计调整 */
}

.banner-img {
  width: 100%;
  height: 100%;
}
```

---

## 三、swiper 常用属性总结

```wxml
<swiper
  indicator-dots="{{indicatorDots}}"  <!-- 是否显示面板指示点 -->
  autoplay="{{autoplay}}"             <!-- 是否自动切换 -->
  interval="{{interval}}"             <!-- 自动切换时间间隔，ms -->
  duration="{{duration}}"             <!-- 滑动动画时长，ms -->
  circular="{{circular}}"             <!-- 是否衔接滑动（循环） -->
  vertical="{{vertical}}"             <!-- 是否纵向滑动 -->
  current="{{current}}"               <!-- 当前所在滑块的 index -->
  bindchange="onSwiperChange"         <!-- 当前页改变事件 -->
  bindanimationfinish="onAnimationFinish"
>
</swiper>
```

常用配置说明：

- `indicator-dots`：轮播底部的小点点（true/false）
- `autoplay`：是否自动播放
- `interval`：自动切换时间，常用 3000（3 秒）
- `duration`：每次滑动动画时间 300–500 比较常见
- `circular`：打开后可以从最后一张滑到第一张
- `vertical`：一般轮播图用横向，不勾选
- `current`：可以用来“手动控制”当前显示第几张

---

## 四、监听滑动事件与当前索引

有时需要知道当前滑到了第几张，比如同步显示标题。

### 4.1 监听 change 事件

```wxml
<swiper
  indicator-dots="true"
  autoplay="true"
  circular="true"
  bindchange="onSwiperChange"
>
  <!-- ... -->
</swiper>
```

```js
Page({
  data: {
    current: 0
  },

  onSwiperChange(e) {
    const current = e.detail.current
    this.setData({ current })
    console.log('当前轮播索引：', current)
  }
})
```

---

## 五、实现轮播图点击跳转

常见需求：轮播图点击跳转到详情页或外部链接。

### 5.1 给每个图片绑定点击事件

```wxml
<swiper
  indicator-dots="true"
  autoplay="true"
  circular="true"
>
  <block wx:for="{{imgList}}" wx:key="id">
    <swiper-item>
      <image
        class="banner-img"
        src="{{item.url}}"
        mode="aspectFill"
        data-id="{{item.id}}"
        bindtap="onBannerTap"
      />
    </swiper-item>
  </block>
</swiper>
```

```js
Page({
  data: {
    imgList: [
      { id: 1, url: 'https://example.com/banner1.jpg', target: '/pages/detail/detail?id=1' },
      { id: 2, url: 'https://example.com/banner2.jpg', target: '/pages/detail/detail?id=2' }
    ]
  },

  onBannerTap(e) {
    const id = e.currentTarget.dataset.id
    console.log('点击的 banner id:', id)
    // 示例：根据 id 跳转
    wx.navigateTo({
      url: `/pages/detail/detail?id=${id}`
    })
  }
})
```

---

## 六、从接口动态加载轮播图数据

实际项目中，图片地址通常来自后台接口。

```js
Page({
  data: {
    imgList: []
  },

  onLoad() {
    this.fetchBanner()
  },

  fetchBanner() {
    wx.request({
      url: 'https://example.com/api/banner', // 示例地址
      method: 'GET',
      success: (res) => {
        // 假设返回数据为 { code: 0, data: [ { url: '' }, ... ] }
        if (res.data.code === 0) {
          this.setData({
            imgList: res.data.data
          })
        }
      },
      fail: (err) => {
        console.error('获取轮播图失败', err)
      }
    })
  }
})
```

---

## 七、封装一个通用轮播图组件（进阶）

### 7.1 目录结构示例

```text
components/banner-swiper/
  ├── banner-swiper.wxml
  ├── banner-swiper.wxss
  ├── banner-swiper.js
  └── banner-swiper.json
```

### 7.2 组件配置与逻辑

`banner-swiper.json`

```json
{
  "component": true
}
```

`banner-swiper.wxml`

```wxml
<view class="banner-container">
  <swiper
    class="banner-swiper"
    indicator-dots="{{indicatorDots}}"
    autoplay="{{autoplay}}"
    interval="{{interval}}"
    duration="{{duration}}"
    circular="{{circular}}"
    bindchange="onSwiperChange"
  >
    <block wx:for="{{list}}" wx:key="id">
      <swiper-item>
        <image
          class="banner-img"
          src="{{item.url}}"
          mode="aspectFill"
          data-item="{{item}}"
          bindtap="onTap"
        />
      </swiper-item>
    </block>
  </swiper>
</view>
```

`banner-swiper.js`

```js
Component({
  properties: {
    list: {
      type: Array,
      value: []
    },
    indicatorDots: {
      type: Boolean,
      value: true
    },
    autoplay: {
      type: Boolean,
      value: true
    },
    interval: {
      type: Number,
      value: 3000
    },
    duration: {
      type: Number,
      value: 500
    },
    circular: {
      type: Boolean,
      value: true
    }
  },

  data: {
    current: 0
  },

  methods: {
    onSwiperChange(e) {
      const current = e.detail.current
      this.setData({ current })
      // 向外通知当前索引
      this.triggerEvent('change', { current })
    },

    onTap(e) {
      const item = e.currentTarget.dataset.item
      // 向外抛出点击事件和数据
      this.triggerEvent('tap', { item })
    }
  }
})
```

`banner-swiper.wxss`

```css
.banner-container {
  width: 100%;
}

.banner-swiper {
  width: 100%;
  height: 300rpx;
}

.banner-img {
  width: 100%;
  height: 100%;
}
```

### 7.3 在页面中使用组件

页面 json：

```json
{
  "usingComponents": {
    "banner-swiper": "/components/banner-swiper/banner-swiper"
  }
}
```

页面 wxml：

```wxml
<banner-swiper
  list="{{imgList}}"
  bind:tap="handleBannerTap"
  bind:change="handleBannerChange"
/>
```

页面 js：

```js
Page({
  data: {
    imgList: [
      { id: 1, url: 'https://example.com/banner1.jpg', title: 'Banner 1' },
      { id: 2, url: 'https://example.com/banner2.jpg', title: 'Banner 2' }
    ]
  },

  handleBannerTap(e) {
    const item = e.detail.item
    console.log('点击的轮播项：', item)
    // TODO: 自定义跳转逻辑
  },

  handleBannerChange(e) {
    const current = e.detail.current
    console.log('轮播切换到：', current)
  }
})
```

---

## 八、常见问题与排查思路

1. **轮播图高度为 0 或不显示**
   - 检查 `swiper` / `image` 是否设置了 `height`
   - 检查父级容器是否被 `overflow: hidden` 或其他样式影响

2. **图片拉伸变形**
   - 调整 `image` 的 `mode` 属性，例如 `aspectFill` / `widthFix`

3. **图片闪烁 / 加载慢**
   - 可考虑图片压缩、CDN
   - 预加载：先显示占位图，加载完成再替换

4. **指示点位置不满足需求**
   - 小程序原生指示点支持有限，可以自己写一套指示器（在 `swiper` 外面根据 `current` 手写小圆点）

---
