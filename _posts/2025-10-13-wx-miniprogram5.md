---
layout: post
title: 微信小程序学习笔记-5
date: 2025-10-13
Author: 
categories: 
tags: [miniprogram5]
comments: false
toc: true
---


# 微信小程序「全局数据 & 本地缓存」学习笔记

> 目标：掌握在小程序中如何存放全局共享数据（`App.globalData`）以及如何使用本地缓存（`wx.setStorage` / `wx.getStorage` 等），并理解各自适用场景。

---

## 一、为什么需要全局数据和缓存？

常见需求：

- 用户信息、多页面共享的配置（主题、语言、登录状态等）
- 登录态、token、用户偏好（夜间模式、上次选择的城市）
- 部分数据希望 **进程内共享**（只在本次使用内有效）
- 部分数据希望 **下次打开小程序仍然存在**（例如登录信息、历史记录）

对比：

- **全局数据（`globalData`）**：只在“这次打开小程序”的生命周期内存在，关闭后清空。
- **本地缓存（storage）**：写进本地，关闭重开仍然存在，除非手动清除或被系统清理。

---

## 二、全局数据：`App.globalData`

### 2.1 定义全局数据

`app.js`：

```js
App({
  onLaunch() {
    // 小程序初始化时可以进行一些全局设置
    console.log('App Launch')
  },

  onShow() {
    console.log('App Show')
  },

  globalData: {
    userInfo: null,      // 用户信息
    token: '',           // 登录 token
    theme: 'light',      // 主题：light/dark
    language: 'zh-CN'    // 当前语言
  }
})
```

- `globalData`：一个普通对象，存放全局共享数据
- 所有页面和组件都可以通过 `getApp()` 访问

### 2.2 在页面中读取 / 修改全局数据

```js
Page({
  data: {
    userInfo: null
  },

  onLoad() {
    const app = getApp()
    this.setData({
      userInfo: app.globalData.userInfo
    })
  },

  // 更新全局主题
  switchTheme() {
    const app = getApp()
    const newTheme = app.globalData.theme === 'light' ? 'dark' : 'light'
    app.globalData.theme = newTheme
    this.setData({ theme: newTheme })
  }
})
```

要点：

- `getApp()` 返回当前小程序的 **App 实例**
- 不建议在 `App` 的 `onLaunch` 中立即调用 `getCurrentPages()`（这在文档中有说明）

---

## 三、全局数据的典型使用场景

1. **用户登录信息（不必持久化的部分）**
   - 如当前登录用户基础信息
   - token 通常会同时放入缓存，便于下次自动登录

2. **在多个页面需要共享的临时状态**
   - 当前选择的城市 / tab / 主题
   - 某些操作的临时结果（如一个复杂表单分步保存的数据）

3. **全局配置 & 常量**
   - API 基础地址（虽然更推荐单独封装 config 文件）
   - 业务层常量（如某些枚举）

> 注意：**全局数据不是持久存储**，小程序被完全关闭（进程被杀）后会消失。

---

## 四、本地缓存（Storage）基础

微信提供了两套 API：

- **同步接口**：`wx.setStorageSync` / `wx.getStorageSync` / `wx.removeStorageSync` / `wx.clearStorageSync`
- **异步接口**：`wx.setStorage` / `wx.getStorage` / `wx.removeStorage` / `wx.clearStorage`

### 4.1 同步接口（简洁，适合小量数据）

```js
// 写入
wx.setStorageSync('token', 'abc123')

// 读取
const token = wx.getStorageSync('token')

// 删除某项
wx.removeStorageSync('token')

// 清空所有缓存
wx.clearStorageSync()
```

特点：

- 调用后立即返回，逻辑简单
- 在同步调用很多、数据较大时会阻塞 js 线程（注意性能）

### 4.2 异步接口（建议在大多数业务中使用）

```js
// 写入
wx.setStorage({
  key: 'userInfo',
  data: { name: 'Tom', age: 18 },
  success() {
    console.log('保存成功')
  },
  fail(err) {
    console.error('保存失败', err)
  }
})

// 读取
wx.getStorage({
  key: 'userInfo',
  success(res) {
    console.log('读取到的数据：', res.data)
  },
  fail() {
    console.log('没有这个 key 或读取失败')
  }
})

// 删除
wx.removeStorage({
  key: 'userInfo',
  success() {
    console.log('删除成功')
  }
})

// 清空所有缓存
wx.clearStorage()
```

---

## 五、缓存中可以存什么数据？

- `data` 支持：字符串、数字、布尔、对象、数组等（会被序列化）
- 建议存储：
  - token、用户 ID、简单用户信息
  - 用户偏好设置（语言、主题、开关状态）
  - 非敏感的业务数据缓存（如最近浏览记录）

> 安全注意：不要在本地缓存中存 **密码** 这类敏感信息。

---

## 六、全局数据 + 缓存：常见登录流程示例

### 6.1 登录成功后保存数据

```js
// 假设在 login 页面
loginSuccessHandler(loginResult) {
  const app = getApp()
  const { token, userInfo } = loginResult

  // 1. 更新全局数据
  app.globalData.token = token
  app.globalData.userInfo = userInfo

  // 2. 写入缓存（方便下次打开自动登录）
  wx.setStorageSync('token', token)
  wx.setStorageSync('userInfo', userInfo)

  // 3. 跳转到首页
  wx.reLaunch({
    url: '/pages/index/index'
  })
}
```

### 6.2 小程序启动时尝试从缓存恢复登录态

`app.js`：

```js
App({
  onLaunch() {
    // 从本地缓存中恢复用户信息
    const token = wx.getStorageSync('token')
    const userInfo = wx.getStorageSync('userInfo')

    if (token) {
      this.globalData.token = token
      this.globalData.userInfo = userInfo
    }
  },

  globalData: {
    token: '',
    userInfo: null
  }
})
```

页面中使用时：

```js
Page({
  onShow() {
    const app = getApp()
    if (!app.globalData.token) {
      // 未登录，跳转到登录页
      wx.navigateTo({
        url: '/pages/login/login'
      })
    }
  }
})
```

---

## 七、全局数据与缓存的对比与搭配使用

| 项目             | 全局数据 (`globalData`)           | 本地缓存 (`storage`)                               |
|------------------|----------------------------------|---------------------------------------------------|
| 生命周期         | 当前小程序运行期间               | 持久存在（直到主动清除或被系统回收）              |
| 访问方式         | `getApp().globalData.xxx`        | `wx.getStorage*` / `wx.setStorage*`               |
| 典型用途         | 运行时共享状态                   | 登录态、配置、偏好、历史等                        |
| 是否持久化       | 否                               | 是                                                |
| 应用关闭后是否保留 | 否                               | 是                                                |
| 是否影响性能     | 一般较小                         | 大量/频繁读写可能有性能影响                       |

典型搭配：

- 登录成功后：  
  - **全局数据**：立即供各页面使用  
  - **缓存**：用于下次启动时恢复状态
- 设置项（例如夜间模式）：  
  - 页面操作时：更新全局 + 缓存  
  - 启动时：从缓存读出，写入全局

---

## 八、封装一个 Storage 工具模块（推荐）

### 8.1 创建工具文件

```javascript

name=utils/storage.js

const TOKEN_KEY = 'TOKEN'
const USER_INFO_KEY = 'USER_INFO'
const SETTINGS_KEY = 'SETTINGS'

const storage = {
  // token
  setToken(token) {
    wx.setStorageSync(TOKEN_KEY, token)
  },
  getToken() {
    return wx.getStorageSync(TOKEN_KEY) || ''
  },
  clearToken() {
    wx.removeStorageSync(TOKEN_KEY)
  },

  // 用户信息
  setUserInfo(userInfo) {
    wx.setStorageSync(USER_INFO_KEY, userInfo)
  },
  getUserInfo() {
    return wx.getStorageSync(USER_INFO_KEY) || null
  },
  clearUserInfo() {
    wx.removeStorageSync(USER_INFO_KEY)
  },

  // 设置（比如主题、语言等）
  setSettings(settings) {
    wx.setStorageSync(SETTINGS_KEY, settings)
  },
  getSettings() {
    return wx.getStorageSync(SETTINGS_KEY) || {}
  },

  clearAll() {
    wx.clearStorageSync()
  }
}

module.exports = storage
````

### 8.2 在页面或 app 中使用

````javascript

// app.js
const storage = require('./utils/storage')

App({
  onLaunch() {
    const token = storage.getToken()
    const userInfo = storage.getUserInfo()
    const settings = storage.getSettings()

    this.globalData.token = token
    this.globalData.userInfo = userInfo
    this.globalData.settings = settings
  },

  globalData: {
    token: '',
    userInfo: null,
    settings: {}
  }
})
````

页面中:

````javascript

const storage = require('../../utils/storage')

Page({
  onLoad() {
    const settings = storage.getSettings()
    this.setData({ settings })
  },

  logout() {
    const app = getApp()
    app.globalData.token = ''
    app.globalData.userInfo = null
    storage.clearToken()
    storage.clearUserInfo()
  }
})
````

## 九、注意事项与常见坑

1. **缓存读不到 / 报 key 不存在**
   - `wx.getStorageSync('xxx')` 如果没有对应 key，返回 `""`（空字符串）或 `undefined`，要做好防御判断。
   - 异步的 `wx.getStorage` 在 `fail` 回调里处理“没有 key”的情况。

2. **频繁大数据读写**
   - 不要在频繁触发的事件（如滚动、滑动）里大量读写 storage，容易卡顿。
   - 可在内存（data / globalData）中维护，合适时机再写回缓存。

3. **不同小程序间不共享缓存**
   - 每个小程序都有自己独立的 storage 空间，不会互相干扰。

4. **key 命名冲突**
   - 建议封装 storage 工具模块，统一管理 key，避免手写字符串时写错或冲突。

5. **globalData 不是响应式**
   - 修改 `getApp().globalData.xxx = 1` 并不会自动更新页面显示，仍需配合 `this.setData()`。
   - 常见写法：页面读全局数据 + setData 到本页面 data 中。

---
