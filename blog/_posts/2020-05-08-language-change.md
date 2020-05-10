---
layout: post
title: '使用vue添加切换语言功能'
subtitle: '如何添加多种语言切换'
date: 2020-05-08
categories: 技术
tags: vue
image: /assets/img/blog/lang.jpg
---

在写项目的时候 如果这个项目面向国际化 我们就需要给项目添加多语言选项
在vue中使用 ``VueI18n``模块来实现

下载``VueI18n``

```dos
npm install vue-i18n --save
```

在``main.js``中导入使用``VueI18n``

```js
// 导入
import VueI18n from 'vue-i18n'
//注册
Vue.use(VueI18n);
```

之后在当前目录下创建``lang``文件夹内含有zh.js / en.js 文件

zh.js:

```js
// 双语规范的变量(中文)
// 在页面进行渲染 使用 m.index 输出指定值
export const m ={
    'index':'美多商城',
    'home':'主页',
    'shop':'商铺',
    'pages':'页面',
    'contact':'联系',
    'homepage':'主页',
    'catalog':'目录',
    'item detail':'详情',
    'cart':'购物车',
    'login':'登录',
    'register':'注册',
    'logout':'退出',
    'update':'修改',
    'chinese':'中',
    'english':'English',
    'welcome':'欢迎您'
}
```

en.js:

```js
// 双语规范的变量(英文)
// 在页面进行渲染 使用 m.index 输出指定值
export const m ={
    'index':'Mei Duo Shop',
    'home':'Home',
    'shop':'Shop',
    'pages':'Pages',
    'contact':'Contact',
    'homepage':'HomePage',
    'catalog':'Catalog',
    'item detail':'Item Detail',
    'cart':'Cart',
    'login':'Login',
    'register':'Register',
    'logout':'Logout',
    'update':'Update',
    'chinese':'Chinese',
    'english':'English',
    'welcome':'Welcome'
}
```

在mian.js中导入所定义的语言包

```js
// 导入语言包
const i18n = new VueI18n({
  //当前默认语言
  locale:'zh',
  //语言包
  messages:{
    'zh': require('./lang/zh'),
    'en':require('./lang/en'),
  }
})
```

如何使用

```js
// 当main.js中定义了语言种类 就使用了默认语言内的m变量
// 使用$t('m.key') 进行输出
{ {$t('m.chinese')} }
```

使用 heyui 设置一个开关

```html
<!-- 开关标签 -->
<h-switch v-model="lang" @change="lang_change">中/英</h-switch>
```

v-model 绑定一个初始值 lang : 0

这样在点击开关时候就会更改lang的值 返回true/false

思路如下：

当lang是0时语言为zh  否则语言为en

防止每次刷新会还原语言 所以将语言参数存入application

在首次加载页面设置为默认zh 切换语言时更改application值

```js
//判断语言
if_lang(){
    // 获取存储的语言
    var lang_locale = localStorage.getItem('lang')
    // 如果语言存在
    if(lang_locale){
        // 语言赋值
        this.$i18n.locale = lang_locale
        if(lang_locale=='zh'){
            //lang状态为zh
            this.lang=0
        }else{
            this.lang=1
        }
    }else{
        //设置默认语言
        this.$i18n.locale = 'zh'
    }
},
```

```js
//使用开关切换语言
lang_change(){
    if(this.lang==false){
        this.$i18n.locale = 'zh'
        // 更改参数
        localStorage.setItem('lang','zh')
    }else{
        this.$i18n.locale = 'en'
        localStorage.setItem('lang','en')
    }
},
```

