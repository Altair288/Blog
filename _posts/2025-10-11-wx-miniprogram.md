---
layout: post
title: 微信小程序学习笔记-3
date: 2025-10-11
Author: 
categories: 
tags: [miniprogram]
comments: false
toc: true
---


# 微信小程序数据绑定学习笔记

> 目标：弄清楚“页面 data ↔ WXML 视图”之间是如何联动的，能熟练使用插值、属性绑定、事件更新等。

---

## 一、数据绑定的基本概念

- 小程序采用 **数据驱动视图**：
  - 页面 `data` 中的字段 → 自动渲染到 `wxml`
  - 通过 `this.setData()` 修改数据 → 视图自动更新
- 核心思想：**不要直接操作 DOM，只操作数据**

---

## 二、最基础的插值绑定（Mustache 语法）

### 2.1 文本插值

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

> 注意：`{{ }}` 内只能写 **表达式**，不能写语句（如 `if`、`for`）。

### 2.2 支持的简单表达式例子

```wxml
<view>{{count + 1}}</view>
<view>{{user.name}}</view>
<view>{{flag ? '开启' : '关闭'}}</view>
```

```js
Page({
  data: {
    count: 1,
    user: { name: 'Tom' },
    flag: true
  }
})
```

---

## 三、属性绑定（标签属性里的数据绑定）

除了内容可以使用 `{{ }}`，**标签属性**也可以使用：

```wxml
<image src="{{imgUrl}}" mode="{{imgMode}}" />
<view class="box {{isActive ? 'active' : ''}}">内容</view>
<button disabled="{{isDisabled}}">按钮</button>
```

```js
Page({
  data: {
    imgUrl: 'https://example.com/a.png',
    imgMode: 'aspectFit',
    isActive: true,
    isDisabled: false
  }
})
```

常见场景：

- 控制样式类名：`class="xxx {{condition ? 'class-a' : 'class-b'}}"`
- 控制组件属性：`disabled`、`autoplay`、`indicator-dots` 等
- 控制图片 / 音频 / 视频资源地址

---

## 四、列表数据绑定（wx:for）

### 4.1 基本用法

```wxml
<view wx:for="{{list}}" wx:key="id">
  {{index}} - {{item.name}}
</view>
```

```js
Page({
  data: {
    list: [
      { id: 1, name: '苹果' },
      { id: 2, name: '香蕉' },
      { id: 3, name: '梨' }
    ]
  }
})
```

- `wx:for`：要遍历的数组（支持基本类型和对象）
- 默认变量名：
  - `item`：当前项
  - `index`：当前下标
- `wx:key`：**必须写**（官方强烈建议），用于提高性能和避免警告

### 4.2 自定义变量名

```wxml
<view
  wx:for="{{list}}"
  wx:for-item="fruit"
  wx:for-index="i"
  wx:key="id"
>
  {{i}} - {{fruit.name}}
</view>
```

---

## 五、条件渲染（wx:if / wx:elif / wx:else）

```wxml
<view wx:if="{{status === 'loading'}}">加载中...</view>
<view wx:elif="{{status === 'success'}}">加载成功</view>
<view wx:else>加载失败</view>
```

```js
Page({
  data: {
    status: 'loading' // loading / success / error
  }
})
```

### 5.1 wx:if 与 hidden 的区别（常考点）

- `wx:if`：真正地 **创建 / 销毁 DOM 节点**
- `hidden`：仅仅是 **控制显示或隐藏**（节点还在）

使用建议：

- 切换不频繁 → 用 `wx:if`
- 频繁显示 / 隐藏 → 用 `hidden="{{!isShow}}"`

---

## 六、事件配合数据绑定：交互更新视图

数据绑定只决定“显示什么”，要让用户操作改变数据，需要事件。

### 6.1 点击按钮增加数字

```wxml
<view>当前计数：{{count}}</view>
<button bindtap="onAdd">+1</button>
```

```js
Page({
  data: {
    count: 0
  },

  onAdd() {
    this.setData({
      count: this.data.count + 1
    })
  }
})
```

---

## 七、`setData` 的用法与注意点

### 7.1 基本用法

```js
this.setData({
  message: '更新后的文本',
  count: this.data.count + 1
})
```

- 调用 `setData` → 框架把变更同步到视图
- **只能更新 data 中已有的字段**（动态新增字段要小心）

### 7.2 更新对象 / 数组的子属性（使用“点语法”）

```js
Page({
  data: {
    user: {
      name: 'Tom',
      age: 18
    },
    list: [1, 2, 3]
  },

  updateData() {
    this.setData({
      'user.name': 'Jerry',
      'list[1]': 20
    })
  }
})
```

> 关键点：`'user.name'`、`'list[1]'` 要写成 **字符串键名**。

### 7.3 性能注意事项

- 尽量一次 `setData` 修改多个字段，而不是多次调用
- 只传 **必要字段**，不要每次把整个大对象 / 大数组都传回去
- 复杂页面中，频繁大数据 `setData` 会有卡顿问题

---

## 八、表单输入与双向“效果”

小程序没有真正的“v-model”，但可以通过事件做到类似双向更新。

### 8.1 input 输入同步到 data

```wxml
<input
  value="{{username}}"
  placeholder="请输入用户名"
  bindinput="onInputChange"
/>

<view>你输入的是：{{username}}</view>
```

```js
Page({
  data: {
    username: ''
  },

  onInputChange(e) {
    this.setData({
      username: e.detail.value
    })
  }
})
```

- `e.detail.value`：当前输入框的值
- `data.username` 更新后，视图显示跟着变

### 8.2 多个输入框的通用处理（用 data-*）

```wxml
<input
  data-field="username"
  value="{{username}}"
  bindinput="onInputChange"
/>
<input
  data-field="email"
  value="{{email}}"
  bindinput="onInputChange"
/>
```

```js
Page({
  data: {
    username: '',
    email: ''
  },

  onInputChange(e) {
    const field = e.currentTarget.dataset.field
    this.setData({
      [field]: e.detail.value
    })
  }
})
```

> 这里用到 ES6 计算属性名：`[field]`。

---

## 九、复杂结构的绑定示例（综合）

### 9.1 数据结构

```js
Page({
  data: {
    banners: [
      { id: 1, img: 'https://xx.com/b1.png', title: '图一' },
      { id: 2, img: 'https://xx.com/b2.png', title: '图二' }
    ],
    products: [
      { id: 101, name: '商品 A', price: 99, hot: true },
      { id: 102, name: '商品 B', price: 199, hot: false }
    ],
    currentBanner: 0
  }
})
```

### 9.2 WXML 绑定展示

```wxml
<!-- 轮播图 -->
<swiper
  indicator-dots="true"
  autoplay="true"
  circular="true"
  current="{{currentBanner}}"
  bindchange="onBannerChange"
>
  <block wx:for="{{banners}}" wx:key="id">
    <swiper-item>
      <image src="{{item.img}}" mode="aspectFill" />
      <view class="banner-title">{{item.title}}</view>
    </swiper-item>
  </block>
</swiper>

<!-- 商品列表 -->
<view
  class="product-item {{item.hot ? 'hot' : ''}}"
  wx:for="{{products}}"
  wx:key="id"
>
  <view>{{item.name}}</view>
  <view>￥{{item.price}}</view>
</view>
```

```js
Page({
  data: {
    // 同上
  },

  onBannerChange(e) {
    this.setData({
      currentBanner: e.detail.current
    })
  }
})
```

---

## 十、常见错误与排查

1. **`{{xxx}}` 不显示 / 显示为 `undefined`**
   - 检查 `data` 里是否有这个字段
   - 检查拼写：字段名、层级、大小写

2. **更新数据了，但界面没变化**
   - 是否通过 `this.setData()` 更新？直接修改 `this.data.xxx = ...` 不会触发视图更新
   - `setData` 写法是否正确，尤其是更新子属性时的字符串键名

3. **`wx:for` 报警告 / 渲染异常**
   - 是否漏写 `wx:key`
   - `wx:key` 的值是否是数组项中独一无二的字段（推荐用 `id`）

4. **频繁 `setData` 导致卡顿**
   - 合并多次修改为一次 `setData`
   - 精简要传递的数据，只更新真正变化的字段

---

## 十一、练习建议

- [ ] 写一个“计数器”页面：显示数字、增加 / 减少 / 重置
- [ ] 写一个“商品列表”页面：从数据数组渲染列表，点击商品改变选中状态
- [ ] 写一个“表单页”：输入昵称 + 年龄，提交后在下方实时展示
- [ ] 尝试更新嵌套对象和数组中的某一项（用 `'obj.key'`、`'list[0]'` 写法）

---
