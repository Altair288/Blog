---
layout: post
title: 微信小程序学习笔记-7
date: 2025-10-15
Author: 
categories: 
tags: [miniprogram]
comments: false
toc: true
---


# 微信小程序「登录与授权」学习笔记

> 目标：理解小程序登录整体流程，区分“登录”和“获取用户信息/授权”的关系，能在项目中实现一个可用的登录体系。

---

## 一、几个容易混淆的概念

1. **小程序登录（后端会话）**
   - 核心是：用 `wx.login` 拿到 `code` → 发给你自己的服务器 → 换取 `openid`、`session_key`，建立你自己的登录态（token）。
   - 这是“服务端鉴权”的基础，与 UI 授权弹窗没有直接关系。

2. **用户信息 / 授权（头像、昵称、手机号、地理位置等）**
   - 需要用户同意授权（按钮触发），后台才可以拿到用户信息。
   - 常见 API：`wx.getUserProfile`（旧规范）、手机号获取、地理位置授权等。

3. **微信侧登录 vs 业务侧登录**
   - 微信侧：微信已经登录（用户在用微信），小程序天然拥有用户在微信的身份（openid）。
   - 业务侧：你自己的应用系统需要知道“这是哪个用户”，并给他分配账号、权限、数据（由你自己的 token  / userId 维护）。

> 记忆：**微信登录是“Who”，授权是“Can I read your X”**（头像、手机号、位置等）。

---

## 二、小程序登录整体流程（推荐做法）

一个常见的“服务端登录”流程：

1. 小程序调用 `wx.login()` 拿到 `code`
2. 把 `code` 发送给你自己的服务器
3. 服务器调用微信的 `jscode2session` 接口，得到 `openid`、`session_key`（和可选 `unionid`）
4. 服务器根据 `openid` 在数据库中查找 / 创建用户，生成自己的 `token`（例如 JWT）
5. 服务器把 `token` 和业务用户信息返回给小程序
6. 小程序保存 `token` 到：
   - 内存（`App.globalData`）
   - 本地缓存（`wx.setStorageSync`）用于下次自动登录

---

## 三、前端：`wx.login` 基本使用

```js
// pages/login/login.js
Page({
  data: {
    loading: false
  },

  onLoginTap() {
    this.setData({ loading: true })

    wx.login({
      success: (res) => {
        if (res.code) {
          // 把 code 发给后台
          this.requestLogin(res.code)
        } else {
          console.error('登录失败！' + res.errMsg)
          this.setData({ loading: false })
        }
      },
      fail: (err) => {
        console.error('wx.login 调用失败', err)
        this.setData({ loading: false })
      }
    })
  },

  requestLogin(code) {
    wx.request({
      url: 'https://your-domain.com/api/login',
      method: 'POST',
      data: { code },
      success: (res) => {
        // 假设返回 { code: 0, data: { token, userInfo } }
        if (res.data.code === 0) {
          const { token, userInfo } = res.data.data
          const app = getApp()

          app.globalData.token = token
          app.globalData.userInfo = userInfo

          wx.setStorageSync('TOKEN', token)
          wx.setStorageSync('USER_INFO', userInfo)

          // 登录成功，跳转到首页或个人中心
          wx.reLaunch({
            url: '/pages/index/index'
          })
        } else {
          wx.showToast({
            title: res.data.msg || '登录失败',
            icon: 'none'
          })
        }
      },
      fail: (err) => {
        console.error('请求登录接口失败', err)
      },
      complete: () => {
        this.setData({ loading: false })
      }
    })
  }
})
```

---

## 四、后端（简要思路，语言自定）

> 这里只记“思路”，实现看你用什么语言 / 框架。

1. 接收小程序传来的 `code`
2. 调用微信接口：

   ```text
   https://api.weixin.qq.com/sns/jscode2session?appid=APPID&secret=SECRET&js_code=JSCODE&grant_type=authorization_code
   ```

   得到类似数据：

   ```json
   {
     "openid": "OPENID",
     "session_key": "SESSIONKEY",
     "unionid": "UNIONID"
   }
   ```

3. 根据 `openid` 在数据库中查找用户：
   - 有 → 读取该用户
   - 无 → 创建新用户（可初始化一些信息）
4. 生成业务 `token`（例如 JWT or 自定义 sessionId）
5. 将 `token + 用户基本信息` 返回给前端

关键点：

- `session_key` 是解密敏感数据（如手机号）的关键，不要直接暴露给前端。
- 后端统一处理 token 的验证，前端每次请求接口都带上 token。

---

## 五、自动登录（启动时恢复登录态）

### 5.1 app.js 中恢复缓存

```js
// app.js
App({
  onLaunch() {
    const token = wx.getStorageSync('TOKEN')
    const userInfo = wx.getStorageSync('USER_INFO')

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

### 5.2 页面中检查登录态

```js
// pages/profile/profile.js
Page({
  onShow() {
    const app = getApp()

    if (!app.globalData.token) {
      // 未登录，引导去登录页
      wx.navigateTo({
        url: '/pages/login/login'
      })
    } else {
      // 已登录，正常显示页面
      this.setData({
        userInfo: app.globalData.userInfo
      })
    }
  }
})
```

---

## 六、用户信息授权（头像昵称）

> 新政策下推荐按“必要即申请”的原则，不要随意弹授权框。

### 6.1 老的 `button open-type="getUserInfo"` 已不推荐

现在更多是：使用 `wx.getUserProfile` 或相关接口（注意看最新官方文档）。

示意写法（逻辑思路）：

```wxml
<button type="primary" bindtap="onGetUserProfile">获取头像昵称</button>
```

```js
Page({
  onGetUserProfile() {
    wx.getUserProfile({
      desc: '用于完善会员资料',
      success: (res) => {
        // res.userInfo 中有 avatarUrl, nickName 等
        const app = getApp()
        const userInfo = res.userInfo

        // 更新全局和本地缓存
        app.globalData.userInfo = {
          ...app.globalData.userInfo,
          ...userInfo
        }

        wx.setStorageSync('USER_INFO', app.globalData.userInfo)

        // 可以把用户信息同步给自己的后台
        this.updateUserProfileToServer(userInfo)
      },
      fail: () => {
        wx.showToast({
          title: '你拒绝了头像昵称授权',
          icon: 'none'
        })
      }
    })
  },

  updateUserProfileToServer(userInfo) {
    // 调后台更新资料
  }
})
```

> 实际开发中需要根据最新隐私政策选用正确的接口，尊重用户授权。

---

## 七、获取手机号授权（常见需求）

**必须**使用微信提供的获取手机号能力，一般流程：

1. 在页面放一个按钮，`open-type="getPhoneNumber"`（由用户点击触发）
2. 在回调中获取 `encryptedData` 和 `iv`
3. 把这两个字段连同 `session_key` 或 `code` 发送给你的服务器
4. 服务器用微信提供的解密算法解密出真实手机号，并绑定到用户

前端示例：

```wxml
<button
  type="primary"
  open-type="getPhoneNumber"
  bindgetphonenumber="onGetPhoneNumber"
>
  获取手机号
</button>
```

```js
Page({
  onGetPhoneNumber(e) {
    if (e.detail.errMsg === 'getPhoneNumber:ok') {
      const { encryptedData, iv } = e.detail
      // 建议后台使用 session_key 解密
      this.sendPhoneToServer(encryptedData, iv)
    } else {
      wx.showToast({
        title: '你拒绝了手机号授权',
        icon: 'none'
      })
    }
  },

  sendPhoneToServer(encryptedData, iv) {
    wx.request({
      url: 'https://your-domain.com/api/bindPhone',
      method: 'POST',
      data: {
        encryptedData,
        iv
      },
      success: (res) => {
        // 后台解密后返回手机号等
      }
    })
  }
})
```

> 解密逻辑必须在服务器侧完成，避免泄露 `session_key` 和解密逻辑。

---

## 八、登录态校验与过期处理

### 8.1 前端请求时统一带上 token

封装 request（简化示意）：

```js
// utils/request.js
const BASE_URL = 'https://your-domain.com'

function request(options) {
  const app = getApp()
  const token = app.globalData.token || wx.getStorageSync('TOKEN')

  return new Promise((resolve, reject) => {
    wx.request({
      url: BASE_URL + options.url,
      method: options.method || 'GET',
      data: options.data || {},
      header: {
        Authorization: token ? `Bearer ${token}` : '',
        ...(options.header || {})
      },
      success(res) {
        // 举例：后端约定 code = 401 表示未登录 / 登录过期
        if (res.data.code === 401) {
          handleNotLogin()
          reject(res.data)
        } else {
          resolve(res.data)
        }
      },
      fail(err) {
        reject(err)
      }
    })
  })
}

function handleNotLogin() {
  const app = getApp()
  app.globalData.token = ''
  app.globalData.userInfo = null
  wx.removeStorageSync('TOKEN')
  wx.removeStorageSync('USER_INFO')

  wx.navigateTo({
    url: '/pages/login/login'
  })
}

module.exports = {
  request
}
```

### 8.2 后端统一校验 token

- 拦截所有需要登录的接口
- 没有 token 或 token 失效 → 返回统一错误码（如 `401`）
- 前端根据这个错误码做“清除本地登录信息 + 跳到登录页”

---

## 九、注销 / 退出登录

前端逻辑一般：

```js
// pages/profile/profile.js
Page({
  onLogout() {
    wx.showModal({
      title: '提示',
      content: '确定要退出登录吗？',
      success: (res) => {
        if (res.confirm) {
          const app = getApp()
          app.globalData.token = ''
          app.globalData.userInfo = null
          wx.removeStorageSync('TOKEN')
          wx.removeStorageSync('USER_INFO')

          // 可选：通知服务器注销（清 token）
          // wx.request({ url: '/api/logout', method: 'POST' })

          wx.reLaunch({
            url: '/pages/index/index'
          })
        }
      }
    })
  }
})
```

---

## 十、隐私 & 合规简要提示

- 在用户第一次打开小程序时，建议展示隐私弹窗（参考微信最新规范）
- 获取用户信息、手机号、位置等，必须说明用途，并由用户主动触发
- 不要在未授权的情况下偷偷上报敏感信息
- 对用户可识别的信息（如手机号）要在服务端做好安全存储和访问控制

---
