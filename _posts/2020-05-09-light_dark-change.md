---
layout: post
title: '实现暗夜模式切换'
description: '如何根据时间切换暗夜模式'
categories: [Vue]
image: /assets/img/blog/dark.png
related_posts:
    - _posts/2020-04-22-vue-use.md

---
- Table of Contents
{:toc .large-only}

## Dark Mode
在当前 暗黑模式无非是一些软件所必备的功能  
一些网站也具有切换暗黑模式的功能  
我们就可以在自己的网站/博客上实现暗黑模式   

那么如何来改变网页的样式呢 并且是可控的

## CSS设置
使用css 内部定义参数 格式如下

```css
:root{
  --bg-color:#fff;
  --a-color:black;
  --text-color:#d3d3d3;
  --menu-text: #bbb;
}
```

在css样式内使用``var(变量名)``来绑定参数 如：``var(--bg-color)``

```css
body {
  font-size: 16px;
  background-color: var(--bg-color);
  color: var(--a-color);
  line-height: 1.7;
  font-weight: 400;
}

a {
  color:var(--a-color); }

```

这样就实现参数唯一  

当:root 内的参数发生变化 那么所有绑定该参数的样式都会变化  实现了可控性

## 触发按钮
使用icon标签 定义一个按钮

```html
<button @click="style_button">
    <i class="adjust icon"></i>
</button>

```

## 思路：

​		定义一个 light_dark 默认为1 1就是白天

​		当点击按钮的时候就改变light_dark 的值

​		然后根据 light_dark 来判断当前的样式

##### 页面渲染时 会检测当前的样式 如果为1 就是白天 否则 黑夜

## 触发逻辑
```js
//判断背景
change_back(){
    // 获取页面属性
    var styles = getComputedStyle(document.documentElement)
    if(this.light_dark==0){
        //0 是黑夜模式
        document.documentElement.style.setProperty('--bg-color','#292a2d')
        document.documentElement.style.setProperty('--a-color','white')
        document.documentElement.style.setProperty('--text-color','#d3d3d3')
    }else{
        //1 白天模式
        document.documentElement.style.setProperty('--bg-color','#ffff')
        document.documentElement.style.setProperty('--a-color','black')
        document.documentElement.style.setProperty('--text-color','#444342')
    }
},
```

##### 当点击该按钮时候改变指定的light_dark  并且重新检测样式

```js
//切换背景颜色
style_button(){
    //获取样式表
    var styles = getComputedStyle(document.documentElement)
    if(this.light_dark==1){
        //0 是黑夜模式
        this.light_dark=0
        // 调用判断样式函数
        this.change_back()
    }else{
        //1 白天模式
        this.light_dark=1
        this.change_back()
    }
    console.log(this.light_dark)
},
```

这样点击按钮的时候就会切换网页的样式  

但如果更要人性化呢  判断当前时间来指定样式

```js
//判断当前时间
now_back(){
    // 获取时间
    var myDate = new Date();
    var hours=myDate.getHours();       //获取当前小时数(0-23)
    console.log(hours)
    //判断当前时间小于18 大于7
    if(hours<18 & hours>7){
        //白天
        this.light_dark=1
    }else{
        this.light_dark=0
    }
},
```



