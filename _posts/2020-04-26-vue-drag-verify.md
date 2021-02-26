---
layout: post
title: 'Vue滑动模块使用'
description: '如何使用Vue滑动模块'
categories: [Vue]
image: /assets/img/blog/vue-drag-verify.gif
related_posts:
  - _posts/2020-04-22-vue-use.md
---

- Table of Contents
{:toc .large-only}


## 安装
```powershell
npm install vue-drag-verify --save
```

## 滑动模块的使用

在vue页面使用时需要导包

```js
import dragVerify from 'vue-drag-verify'
```

## 注册组件

在`components`函数中注册组件 才能使用该标签
```js
components: {
    dragVerify: dragVerify,
  },
```

## 使用标签
```vue
<!-- 滑动模块 -->
<dragVerify :width='300' :height='40' :text='请滑动' ref='Verify'></dragVerify>
<!--ref='Verify' 标签用于判断是否滑动完成-->
```

## 判断是否完成验证
```js

// 如果滑动完成返回true
console.log(this.$refs.Verify.isPassing)

```

