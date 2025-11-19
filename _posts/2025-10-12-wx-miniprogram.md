---
layout: post
title: 微信小程序学习笔记-4
date: 2025-10-12
Author: 
categories: 
tags: [miniprogram]
comments: false
toc: true
---


# 微信小程序「页面与路由」学习笔记

> 目标：理解小程序“页面”与“路由栈”的概念，熟练使用各种跳转方式（navigateTo / redirectTo / switchTab / reLaunch / navigateBack），并能在项目中设计合理的页面结构和跳转流程。

---

## 一、核心概念：页面 & 路由栈

### 1.1 页面（page）

- 每一个 `pages/xxx/xxx` 下的 `xxx.js + xxx.wxml + xxx.wxss + xxx.json` 就是一个页面
- 在 `app.json` 的 `pages` 数组中注册：
  ```json
  {
    "pages": [
      "pages/index/index",
      "pages/logs/logs"
    ]
  }
  ```
- `pages` 数组的第一项，就是小程序的首页

### 1.2 路由栈（页面栈）

- 小程序维护一个页面栈（类似浏览器的历史记录）
- 新开页面 → 入栈  
  返回上一页 → 出栈
- 栈顶页面 = 当前正在展示的页面

可使用 `getCurrentPages()` 查看当前页面栈：

```js
const pages = getCurrentPages()
console.log(pages)
```

> 注意：`getCurrentPages()` 只能在页面/组件逻辑中使用，不能在 app.js 的 `onLaunch` 里使用。

---

## 二、常用的 5 种页面跳转方式

### 2.1 `wx.navigateTo` —— 普通跳转（入栈）

```js
wx.navigateTo({
  url: '/pages/detail/detail?id=123'
})
```

特点：

- 打开非 `tabBar` 页面
- **保留当前页面**，新页面压入栈顶
- 支持传递参数（通过 url query）

适用场景：

- 列表 → 详情
- 首页 → 二级页面

---

### 2.2 `wx.redirectTo` —— 替换当前页面

```js
wx.redirectTo({
  url: '/pages/login/login'
})
```

特点：

- 打开非 `tabBar` 页面
- **关闭当前页面**，用新页面替换当前栈顶
- 页面栈长度不增加

适用场景：

- 不需要返回上一页的跳转（比如登录成功后不需要再回登录页）
- 流程跳转中“不可返回”的节点

---

### 2.3 `wx.switchTab` —— 切换 tabBar 页面

```js
wx.switchTab({
  url: '/pages/index/index'
})
```

特点：

- 只能跳转到 `app.json` 中 `tabBar.list` 声明的页面
- 切换后会 **清空其它非 tabBar 页面的栈记录**
- 不支持携带 query 参数（即 url 后面不能拼 `?a=1`）

> 如需在切换 tab 时传参，一般方案是：
> - 使用全局变量 / 全局状态管理
> - 使用本地存储 `wx.setStorageSync` / `wx.setStorage`

---

### 2.4 `wx.reLaunch` —— 关闭所有页面并打开新页面

```js
wx.reLaunch({
  url: '/pages/index/index'
})
```

特点：

- 关闭所有页面（清空页面栈）
- 打开指定的页面
- 可以是 tabBar 页，也可以是普通页

适用场景：

- 登录成功后，直接回到首页，避免用户通过返回按钮回到登录页
- 从错误页面重进主流程

---

### 2.5 `wx.navigateBack` —— 返回上一页 / 多级返回

```js
// 返回上一页
wx.navigateBack({
  delta: 1
})

// 返回上两级
wx.navigateBack({
  delta: 2
})
```

特点：

- `delta` 默认是 1
- 如果 `delta` 超过页面栈长度，会回到栈底页面

---

## 三、参数传递与接收

### 3.1 通过 URL 传参

**跳转端：**

```js
wx.navigateTo({
  url: `/pages/detail/detail?id=${item.id}&name=${item.name}`
})
```

**接收端（`pages/detail/detail.js`）：**

```js
Page({
  onLoad(options) {
    console.log(options.id, options.name)
  }
})
```

- `onLoad(options)` 中即可拿到所有 query 参数（字符串类型）

### 3.2 参数类型处理

```js
Page({
  onLoad(options) {
    const id = Number(options.id)  // 字符串转数字
    const isVip = options.isVip === 'true'
    console.log(id, isVip)
  }
})
```

> 要注意：路由参数一律为字符串，需要自己进行类型转换。

---

## 四、页面生命周期与路由关系

### 4.1 单个页面生命周期回顾

```js
Page({
  onLoad(options) {
    // 页面加载，获得路由参数
  },

  onShow() {
    // 页面显示（每次进入 / 回到页面都会触发）
  },

  onReady() {
    // 首次渲染完成
  },

  onHide() {
    // 页面隐藏：被 navigateTo 其他页面或切到后台
  },

  onUnload() {
    // 页面卸载：navigateBack / redirectTo / reLaunch 导致当前页面被销毁
  }
})
```

### 4.2 路由操作与生命周期触发对照

| 跳转方式              | 当前页面生命周期       | 目标页面生命周期           |
|-----------------------|------------------------|----------------------------|
| `navigateTo`          | 当前页触发 `onHide`    | 目标页触发 `onLoad`→`onShow`→`onReady` |
| `redirectTo`          | 当前页触发 `onUnload`  | 目标页触发 `onLoad`→`onShow`→`onReady` |
| `switchTab`           | 当前页 `onHide` / `onUnload`（视栈情况） | 目标 tab 页 `onLoad`(首次) + `onShow` |
| `reLaunch`            | 所有页触发 `onUnload`  | 新页触发 `onLoad`→`onShow`→`onReady` |
| `navigateBack`        | 当前页触发 `onUnload`  | 上一页触发 `onShow`        |

> 常见实战点：  
> - 需要在“回到页面时刷新数据” → 放在 `onShow`  
> - 仅首次进入页面请求数据 → 放在 `onLoad`

---

## 五、实践：列表页 → 详情页 → 返回

### 5.1 列表页（`pages/list/list.wxml`）

```wxml
<view wx:for="{{list}}" wx:key="id" class="item"
      data-id="{{item.id}}"
      bindtap="goDetail">
  {{item.title}}
</view>
```

`pages/list/list.js`：

```js
Page({
  data: {
    list: [
      { id: 1, title: '文章一' },
      { id: 2, title: '文章二' }
    ]
  },

  goDetail(e) {
    const id = e.currentTarget.dataset.id
    wx.navigateTo({
      url: `/pages/detail/detail?id=${id}`
    })
  }
})
```

### 5.2 详情页（`pages/detail/detail.js`）

```js
Page({
  data: {
    id: null,
    detail: null
  },

  onLoad(options) {
    const id = Number(options.id)
    this.setData({ id })
    this.fetchDetail(id)
  },

  fetchDetail(id) {
    // 模拟请求
    const detail = {
      id,
      title: `文章 ${id}`,
      content: `这是文章 ${id} 的内容`
    }
    this.setData({ detail })
  },

  back() {
    wx.navigateBack()
  }
})
```

`pages/detail/detail.wxml`：

```wxml
<view>详情：{{detail.title}}</view>
<view>{{detail.content}}</view>
<button bindtap="back">返回列表</button>
```

---

## 六、路由跳转与 tabBar 配合示例

假设定义了 tabBar：

```json
{
  "tabBar": {
    "list": [
      {
        "pagePath": "pages/index/index",
        "text": "首页"
      },
      {
        "pagePath": "pages/profile/profile",
        "text": "我的"
      }
    ]
  }
}
```

### 6.1 非 tab 页面跳转到 tab 页面

```js
// 比如在某个设置页点击“回首页”
wx.switchTab({
  url: '/pages/index/index'
})
```

### 6.2 登录成功后，清空栈并回首页

```js
// 登录成功后的回调
wx.reLaunch({
  url: '/pages/index/index'
})
```

> 小技巧：避免用户使用“返回”按钮回到登录页或其他流程页。

---

## 七、设计路由结构的实践经验

1. **首页 + tabBar = 导航骨架**
   - 典型结构：`首页 / 分类 / 购物车 / 我的`
   - tabBar 页面尽量作为一级入口，不要做复杂流程

2. **深度页面使用 `navigateTo`**
   - 列表 → 详情 → 子详情 → ...  
   - 用户可以逐级返回

3. **不应该返回的页面使用 `redirectTo` 或 `reLaunch`**
   - 如：完成支付成功页 → `redirectTo` 到“订单详情页”
   - 引导页 / 新手教程完成后 → `reLaunch` 到首页

4. **参数设计**
   - 尽量使用简短、易理解的字段，如 `id`、`type`、`from`
   - 复杂数据用 ID 传递，并在目标页重新请求详细数据

---

## 八、常见问题与易踩坑

1. **在页面 B 调 `navigateBack`，却没有回到预期页面**
   - 可能：页面栈层级与预期不一致
   - 建议：在关键流程中打印 `getCurrentPages()` 检查栈结构

2. **跳转到 tabBar 页面失败**
   - 检查：目标页面是否已经配置在 `app.json` 的 `tabBar.list` 中
   - 使用方式：必须用 `wx.switchTab`，而不是 `navigateTo`

3. **切换 tab 时想要传参**
   - 不能在 `url` 后加 query
   - 替代方法：
     ```js
     // 设置全局变量
     getApp().globalData.fromPage = 'xxx'

     wx.switchTab({ url: '/pages/index/index' })

     // 在 index 页 onShow 中读取
     const app = getApp()
     const fromPage = app.globalData.fromPage
     ```

4. **`onLoad` 中通过 `getCurrentPages()` 获取不到当前页**
   - 文档说明：`getCurrentPages()` 中的页面实例与当前页面文件中的 `Page` 对象**不是同一个引用**
   - 一般调试用即可，不要做复杂逻辑依赖

---
