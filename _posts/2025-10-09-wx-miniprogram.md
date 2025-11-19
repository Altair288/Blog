---
layout: post
title: 微信小程序学习笔记-1
date: 2025-10-09
Author: 
categories: 
tags: [miniprogram]
comments: false
toc: true
---


# 微信小程序学习笔记

> 适合入门和回顾使用的个人笔记草稿，可按需增删。

---

## 一、基础概念

### 1.1 什么是微信小程序

- 运行在微信客户端里的“轻应用”
- 无需下载安装，扫码或搜索即可使用
- 以“页面 + 逻辑 + 样式 + 配置”的形式组织
- 使用自有技术栈：`WXML + WXSS + JS + JSON`

### 1.2 与传统 Web / App 的区别（简要）

- 运行环境：基于微信的 WebView + JSCore，不等于完整浏览器
- 能力：可以调用部分原生能力（相机、位置、扫一扫等）
- 体积限制：代码包大小有限制（主包 + 分包）
- 生命周期、路由管理方式不同

---

## 二、项目结构与文件类型

### 2.1 典型目录结构

```text
project-root/
  ├── app.js          # 全局 JS 逻辑
  ├── app.json        # 全局配置（页面路由、窗口样式等）
  ├── app.wxss        # 全局样式
  ├── project.config.json  # 微信开发者工具项目配置
  ├── sitemap.json    # 页面收录配置
  ├── pages/          # 各业务页面目录
  │   └── index/
  │       ├── index.wxml
  │       ├── index.wxss
  │       ├── index.js
  │       └── index.json
  ├── utils/          # 工具模块
  └── ...             # 组件、自定义 tabBar 等
```

### 2.2 四大基础文件

1. **WXML**  
   - 类似 HTML，用于描述页面结构  
   - 支持数据绑定、条件渲染、列表渲染等

2. **WXSS**  
   - 类似 CSS，增加了 `rpx` 自适应单位  
   - 支持全局样式 + 局部样式

3. **JS**  
   - 页面逻辑、事件处理、网络请求、数据处理  
   - 使用微信提供的 API，如 `wx.request`、`wx.showToast` 等

4. **JSON**  
   - 项目和页面配置  
   - `app.json`：全局配置  
   - 页面同名 json：页面个性化配置

---

## 三、配置文件详解

### 3.1 app.json（全局配置）

常见字段：

```json
{
  "pages": [
    "pages/index/index",
    "pages/logs/logs"
  ],
  "window": {
    "navigationBarBackgroundColor": "#ffffff",
    "navigationBarTitleText": "Demo",
    "navigationBarTextStyle": "black",
    "backgroundTextStyle": "light"
  },
  "tabBar": {
    "list": [
      {
        "pagePath": "pages/index/index",
        "text": "首页",
        "iconPath": "assets/tab/home.png",
        "selectedIconPath": "assets/tab/home-selected.png"
      }
    ]
  },
  "sitemapLocation": "sitemap.json",
  "style": "v2",
  "usingComponents": {}
}
```

### 3.2 页面配置（如 `pages/index/index.json`）

```json
{
  "navigationBarTitleText": "首页",
  "enablePullDownRefresh": true,
  "usingComponents": {
    "my-card": "/components/my-card/my-card"
  }
}
```

---

## 四、WXML 基础语法

### 4.1 数据绑定

```wxml
<view>{{message}}</view>
```

```js
Page({
  data: {
    message: 'Hello 小程序'
  }
})
```

### 4.2 条件渲染

```wxml
<view wx:if="{{isLogin}}">已登录</view>
<view wx:else>未登录</view>
```

### 4.3 列表渲染

```wxml
<view wx:for="{{list}}" wx:key="id">
  {{index}} - {{item.name}}
</view>
```

### 4.4 事件绑定

```wxml
<button bindtap="handleTap" data-id="{{item.id}}">点击</button>
```

```js
Page({
  handleTap(e) {
    const id = e.currentTarget.dataset.id
    console.log('点击 id：', id)
  }
})
```

---

## 五、WXSS 与自适应布局

### 5.1 rpx 单位

- `rpx`：根据屏幕宽度自适应，设计稿一般按 750 宽
- 公式：`750rpx == 屏幕宽度`
- 例：iPhone 375 宽 => `1rpx = 0.5px`

### 5.2 常用写法示例

```css
.container {
  padding: 20rpx;
}

.title {
  font-size: 32rpx;
  font-weight: bold;
}
```

---

## 六、页面与生命周期

### 6.1 小程序整体生命周期（app.js）

```js
App({
  onLaunch(options) {
    // 小程序初始化
  },
  onShow(options) {
    // 从后台进入前台
  },
  onHide() {
    // 进入后台
  },
  onError(err) {
    console.error(err)
  }
})
```

### 6.2 页面生命周期（page.js）

```js
Page({
  data: {},

  onLoad(options) {
    // 页面加载（只触发一次）
  },

  onShow() {
    // 页面显示
  },

  onReady() {
    // 页面初次渲染完成
  },

  onHide() {
    // 页面隐藏
  },

  onUnload() {
    // 页面卸载
  },

  onPullDownRefresh() {
    // 监听用户下拉动作
  },

  onReachBottom() {
    // 触底事件
  },

  onShareAppMessage() {
    // 用户点击右上角分享
  }
})
```

---

## 七、路由与页面跳转

### 7.1 常用跳转 API

- `wx.navigateTo`：保留当前页面，跳转到非 tabBar 页面
- `wx.redirectTo`：关闭当前页面，跳转
- `wx.switchTab`：切换到 tabBar 页面
- `wx.reLaunch`：关闭所有页面，打开到某个页面
- `wx.navigateBack`：返回上一级或多级页面

```js
wx.navigateTo({
  url: '/pages/detail/detail?id=123'
})
```

接收参数：

```js
Page({
  onLoad(options) {
    console.log(options.id) // 123
  }
})
```

---

## 八、网络请求与本地存储

### 8.1 网络请求

```js
wx.request({
  url: 'https://example.com/api/list',
  method: 'GET',
  data: { page: 1 },
  success(res) {
    console.log(res.data)
  },
  fail(err) {
    console.error(err)
  }
})
```

### 8.2 本地存储

```js
// 同步
wx.setStorageSync('token', 'abc123')
const token = wx.getStorageSync('token')

// 异步
wx.setStorage({
  key: 'userInfo',
  data: { name: 'Tom' },
  success() {}
})
```

---

## 九、组件与模块化

### 9.1 自定义组件基本结构

```text
components/my-card/
  ├── my-card.wxml
  ├── my-card.wxss
  ├── my-card.js
  └── my-card.json
```

`my-card.json`：

```json
{
  "component": true
}
```

`my-card.js`：

```js
Component({
  properties: {
    title: {
      type: String,
      value: ''
    }
  },
  data: {},
  methods: {
    handleTap() {
      this.triggerEvent('tapCard')
    }
  }
})
```

使用组件（页面 json 与 wxml）：

```json
{
  "usingComponents": {
    "my-card": "/components/my-card/my-card"
  }
}
```

```wxml
<my-card title="测试卡片" bind:tapCard="onTapCard" />
```

```js
Page({
  onTapCard() {
    console.log('card tapped')
  }
})
```

---

## 十、常用能力与 API 速记

- UI 提示  
  - `wx.showToast`、`wx.showModal`、`wx.showLoading`、`wx.hideLoading`
- 媒体相关  
  - `wx.chooseImage`、`wx.previewImage`、`wx.saveImageToPhotosAlbum`
- 位置与地图  
  - `wx.getLocation`、`wx.openLocation`、`<map />` 组件
- 授权与登录  
  - `wx.login`、`wx.getUserProfile`（注意隐私合规）
- 文件上传下载  
  - `wx.uploadFile`、`wx.downloadFile`

---

## 十一、调试与发布流程

1. 安装并打开 **微信开发者工具**
2. 使用 AppID 创建项目（个人练习可使用测试号）
3. 开发阶段：
   - 使用“预览”在手机端测试
   - 使用“真机调试”排查问题
4. 上传代码到微信后台
5. 在微信公众平台配置：
   - 服务器域名
   - 业务域名
   - 体验版 / 审核 / 正式发布
