---
layout: post
title: 微信小程序学习笔记-6
date: 2025-10-14
Author: 
categories: 
tags: [miniprogram]
comments: false
toc: true
---


# 微信小程序「模块化页面」学习笔记

> 目标：学会把“一个大页面”拆成多个可复用的模块（组件 / 公共函数 / 配置），提升代码可维护性与复用性。

---

## 一、为什么要做“模块化页面”？

常见痛点：

- 单个页面文件越来越长：`wxml`、`js` 逻辑堆在一起，难以维护
- 多个页面重复写类似的 UI 区块：例如商品卡片、标题栏、操作按钮区
- 相似的业务逻辑在多个页面拷贝粘贴：例如分页加载、列表刷新

模块化的核心目标：

1. **UI 模块化**：把重复使用的 UI 模块拆成组件
2. **逻辑模块化**：把通用逻辑封装成工具函数 / 行为（behavior）
3. **配置模块化**：把常量和接口配置抽离到独立文件

---

## 二、组件化：把重复的 UI 区块抽成组件

### 2.1 什么时候考虑做组件？

- 多个页面出现相同结构的 UI：
  - 商品卡片、用户信息卡片、空状态提示、标题区、底部操作栏
- 单个页面内部同一块 UI 重复多次
- 某块 UI 逻辑较复杂（内部有状态和交互）

### 2.2 组件的基本结构回顾

```text
components/product-card/
  ├── product-card.wxml
  ├── product-card.wxss
  ├── product-card.js
  └── product-card.json
```

`product-card.json`：

```json
{
  "component": true
}
```

`product-card.wxml`（示例）：

```wxml
<view class="product-card" bindtap="onTap">
  <image class="cover" src="{{item.cover}}" mode="aspectFill" />
  <view class="info">
    <view class="title">{{item.title}}</view>
    <view class="price">￥{{item.price}}</view>
  </view>
</view>
```

`product-card.js`（接收数据 + 发事件）：

```js
Component({
  properties: {
    item: {
      type: Object,
      value: {}
    }
  },

  methods: {
    onTap() {
      this.triggerEvent('tap', { item: this.data.item })
    }
  }
})
```

在页面中使用：

页面 json：

```json
{
  "usingComponents": {
    "product-card": "/components/product-card/product-card"
  }
}
```

页面 wxml：

```wxml
<product-card
  wx:for="{{list}}"
  wx:key="id"
  item="{{item}}"
  bind:tap="onProductTap"
/>
```

页面 js：

```js
Page({
  data: {
    list: []
  },

  onProductTap(e) {
    const item = e.detail.item
    // TODO: 跳转详情页等
  }
})
```

---

## 三、逻辑模块化：抽离公共函数 / 请求

### 3.1 Utils 工具函数

项目结构示例：

```text
utils/
  ├── request.js   # 封装网络请求
  ├── format.js    # 封装格式化相关函数
  └── auth.js      # 登录、权限等
```

`utils/request.js` 示例：

```js
const BASE_URL = 'https://example.com/api'

function request({ url, method = 'GET', data = {} }) {
  return new Promise((resolve, reject) => {
    wx.request({
      url: BASE_URL + url,
      method,
      data,
      header: {
        // 示例：统一加 token
        Authorization: wx.getStorageSync('token') || ''
      },
      success(res) {
        if (res.statusCode === 200) {
          resolve(res.data)
        } else {
          reject(res)
        }
      },
      fail(err) {
        reject(err)
      }
    })
  })
}

module.exports = {
  request
}
```

页面中使用：

```js
const { request } = require('../../utils/request')

Page({
  async fetchList() {
    const res = await request({ url: '/product/list', data: { page: 1 } })
    this.setData({ list: res.data })
  }
})
```

> 这样每个页面就不用重复写 `wx.request`，页面逻辑更干净。

### 3.2 行为（Behavior）：抽离通用 Page 逻辑（进阶）

适合场景：

- 多个页面都有相似的数据结构和方法
  - 如“列表页”：分页、下拉刷新、上拉加载更多

示例：封装一个列表行为 `behaviors/list.js`

```js
// behaviors/list.js
module.exports = Behavior({
  data: {
    list: [],
    page: 1,
    pageSize: 10,
    hasMore: true,
    loading: false
  },

  methods: {
    async loadMore() {
      if (!this.data.hasMore || this.data.loading) return
      this.setData({ loading: true })
      const nextPage = this.data.page + 1
      const list = await this.fetchPage(nextPage) // 要求页面实现此方法
      this.setData({
        page: nextPage,
        list: this.data.list.concat(list),
        hasMore: list.length === this.data.pageSize,
        loading: false
      })
    }
  }
})
```

在页面中使用：

```js
const listBehavior = require('../../behaviors/list')
const { request } = require('../../utils/request')

Page({
  behaviors: [listBehavior],

  async fetchPage(page) {
    const res = await request({ url: '/product/list', data: { page } })
    return res.data
  },

  onReachBottom() {
    this.loadMore()
  }
})
```

> 行为可以理解为“可混入的公共逻辑模块”，适合多个页面共用复杂逻辑。

---

## 四、配置模块化：常量 / 接口统一管理

### 4.1 常量配置

```text
config/
  ├── api.js        # 接口路径
  └── consts.js     # 常量枚举
```

`config/api.js`：

```js
const API = {
  PRODUCT_LIST: '/product/list',
  PRODUCT_DETAIL: '/product/detail',
  USER_INFO: '/user/info'
}

module.exports = API
```

使用：

```js
const API = require('../../config/api')
const { request } = require('../../utils/request')

Page({
  async fetchList() {
    const res = await request({ url: API.PRODUCT_LIST })
    this.setData({ list: res.data })
  }
})
```

> 这样改接口路径时，只需要改配置文件，不用到处查找字符串。

---

## 五、页面内部的“区域模块化”实践

即使不抽成组件，也可以在一个页面内部做逻辑分区，让结构更清晰。

### 5.1 JS 中按功能块组织代码

```js
Page({
  data: {
    // UI 数据
    loading: false,
    activeTab: 0,

    // 业务数据
    productList: [],
    cartList: []
  },

  // ========== 生命周期 ==========
  onLoad() {
    this.init()
  },

  // ========== 初始化逻辑 ==========
  async init() {
    this.setData({ loading: true })
    await Promise.all([this.fetchProductList(), this.fetchCartList()])
    this.setData({ loading: false })
  },

  // ========== 数据请求 ==========
  async fetchProductList() {
    // ...
  },

  async fetchCartList() {
    // ...
  },

  // ========== 事件处理 ==========
  onTabChange(e) {
    this.setData({ activeTab: e.detail.index })
  },

  onProductTap(e) {
    const item = e.currentTarget.dataset.item
    // ...
  }
})
```

> 按“生命周期 / 初始化 / 请求 / 事件处理”等分段，让页面 JS 更可读。

### 5.2 WXML 中按区域分块注释

```wxml
<!-- 顶部 banner 区 -->
<swiper>...</swiper>

<!-- 分类 tab 区 -->
<view class="tabs">...</view>

<!-- 商品列表区 -->
<view class="product-list">...</view>

<!-- 底部操作栏 -->
<view class="bottom-bar">...</view>
```

> 即使暂时不抽组件，清晰的区域划分也是“模块化”的重要一步。

---

## 六、小项目实践示例：把“首页”做成模块化

假设首页包括：

- Banner 轮播图
- 分类入口区
- 推荐商品列表

我们可以拆为：

- `components/banner-swiper`：轮播图组件
- `components/category-grid`：分类九宫格组件
- `components/product-card`：商品卡片组件
- `utils/request.js`：请求封装
- `config/api.js`：接口地址

首页 wxml：

```wxml
<view class="page-home">
  <banner-swiper list="{{bannerList}}" bind:tap="onBannerTap" />

  <category-grid list="{{categoryList}}" bind:tap="onCategoryTap" />

  <view class="section-title">推荐商品</view>
  <product-card
    wx:for="{{productList}}"
    wx:key="id"
    item="{{item}}"
    bind:tap="onProductTap"
  />
</view>
```

首页 js：

```js
const API = require('../../config/api')
const { request } = require('../../utils/request')

Page({
  data: {
    bannerList: [],
    categoryList: [],
    productList: []
  },

  onLoad() {
    this.init()
  },

  async init() {
    const [bannerRes, categoryRes, productRes] = await Promise.all([
      request({ url: API.BANNER_LIST }),
      request({ url: API.CATEGORY_LIST }),
      request({ url: API.PRODUCT_RECOMMEND })
    ])

    this.setData({
      bannerList: bannerRes.data,
      categoryList: categoryRes.data,
      productList: productRes.data
    })
  },

  onBannerTap(e) { /* ... */ },
  onCategoryTap(e) { /* ... */ },
  onProductTap(e) { /* ... */ }
})
```

> 页面变得更像“拼装模块”，而非堆积代码。

---

## 七、常见问题与建议

1. **什么时候抽组件？什么时候不抽？**
   - 明显重复出现、后续会复用 → 抽组件
   - 只在单一页面使用、逻辑简单 → 先不抽，后面多处用了再抽

2. **组件之间通信过于复杂**
   - 父 → 子：尽量用 `properties`
   - 子 → 父：`triggerEvent`
   - 复杂跨层级通信：考虑把状态上移到更高层或使用全局状态

3. **utils 里写了太多与页面高度耦合的逻辑**
   - 工具函数应该是“通用、无业务或弱业务”的
   - 具体页面逻辑仍然应在页面 / 组件内部

4. **文件过多导致结构混乱**
   - 使用清晰的目录：`components/`、`utils/`、`config/`、`behaviors/`
   - 文件名做到“见名知意”，如 `user-card`、`list-behavior`、`api.js`

---

## 八、练习任务（可按自己项目实际改动）

- [ ] 找一个你当前项目中“最长的页面”，尝试从中抽出 1–2 个 UI 组件
- [ ] 把重复使用的 `wx.request` 调用改为单一 `request` 工具函数
- [ ] 尝试写一个行为（behavior），例如封装列表分页逻辑，并在两个页面使用
- [ ] 建立 `config/api.js`，统一管理所有接口路径字符串

---
