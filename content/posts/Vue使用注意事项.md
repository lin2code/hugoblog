---
title: Vue使用注意事项
date: 2020-12-01T20:11:34+08:00
lastmod: 2020-12-01T20:11:34+08:00
author: 林二狗
# authorlink: https://author.site
cover: https://blog-lin2code-1252382346.file.myqcloud.com/img/cover10.jpg
categories:
  - 前端
tags:
  - vue
# showcase: true
draft: false
---

## 组件的属性参数传递

```javascript
limitTypes=".jpg" //直接赋值得到字符串
:onlyUseButton="false"//单向表达式绑定v-bind的缩写 传递的是表达式的值
:successCall="uploadDone"//单向表达式绑定 可传递方法 达到回调目的
:limitCount.sync="limCount" //.sync后缀双向绑定

//v-on事件绑定 用于接收组件中$.emit()的事件  
//区别于使用属性绑定方法的是 v-on中绑定事件可以接收自定义的参数忽略emit的值  
//而属性绑定的方法得到的参数是由内部回调决定
v-on:callback="cb"
@cancelCall="cancelDone"//v-on的缩写

v-on:keyup.enter="login"//keyup修饰符 监听按键
v-on:keyup.enter.native="login"//el-input中需要加上native后缀才能生效
```

---

## props的使用和双向绑定的更新

props是单向绑定更新，如果要双向使用`:xxx.sync="xxx"`形式绑定  
同时在组件内部需要修改对应prop的值时使用`this.$emit('update:xxx', val)`进行同步更新  
update:xxxx相当于默认事件触发update事件更新.sync绑定的属性

**组件内部尽量不要直接绑定或修改props中的属性，props中的属性应该只用来初始化**  
**如果是双向绑定的prop更新值时应通过emit方式更新**

---

## 自定义实现v-model

需要定义一个prop，其实就是.sync的语法糖

假设组件内部有一个事件

```html
<div @click="itemClick(op.value)">xxx</div>
```

当触发该事件时更新v-model的值

```javascript
{
    model:{
        prop: 'selected',//说明v-model对应内部那个prop
        event: 'change'//对应那个emit事件 将事件的value更新到外部v-model和内部prop上
    },
    data() { return {} },
    props: ['selected'],
    mounted() {
    },
    methods: {
        itemClick(val){
            //发出事件 更新v-model
            this.$emit('change', val);
        }
    }
}

```

---

## slot总结

组件中使用`<slot></slot>`标签定义插槽的位置，使用组件时标签内部的内容就会渲染到插槽位置

**定义多个插槽则使用具名插槽**  
定义：

```html
<slot name="xxx"></slot>
```

使用：

```html
<!--通过template标签和v-slot来使用-->
<!--旧版vue使用 slot="xxx"-->
<template v-slot:xxx>
    <h1>Here might be a page title</h1>
</template>
```

**通过slot使用组件内部数据**  
定义：

```html
<!--slot标签上使用v-bind传递组件数据-->
<span>
  <slot name="xxx" v-bind:user="user">
    {{ user.lastName }}
  </slot>
</span>
```

使用：

```html
<!--通过v-slot:xxx="xxx"对应具名slot并访问其数据-->
<!--旧版vue 通过slot="xxx"对应具名slot 通过slot-scope="xxx"访问对应slot的数据-->
<current-user>
  <template v-slot:default="slotProps">
    {{ slotProps.user.firstName }}
  </template>
  <template v-slot:other="otherSlotProps">
    ...
  </template>
</current-user>
```

---

## computed属性不生效 或 组件不同步更新 问题

需要computed的属性一定要在初始化时声明，特别是对象的属性，否则vue不会监听到改属性的变更，也就无法触发computed属性的更新

组件绑定的属性也要初始化，如果没有初始值，vue无法给其绑定自动更新事件也就会造成  
初始化没问题，但数据更新时无法更新组件

---

## 基于vue原型的全局变量赋值 & Vuex

定义 `Vue.prototype.$xxx = xxxx;`  
页面中使用直接 `this.$xxx`，因为页面中的this都继承Vue  
修改值需要使用 `Vue.prototype.$xxx = xxxx`

**应尽量使用 Vuex 来保存需要绑定界面的全局变量**  
Vuex的属性直接修改是不行的 报错没有setter  
Vuex的对象可以直接修改对象内的属性 是不会报错的 且全局同步  
最好还是使用map的mutations的方法来修改值

---

## 表达式绑定class属性更新慢问题

class的表达式中 `判断条件对应的变量`  
不能是 `数组的值或数组中对象的属性`，否则更新class速度将非常慢。  

普通标签的class属性绑定应直接使用data中定义的变量

循环标签中class `绑定的变量` 需要是 `动态变量` 时  
可以把`数组转换成对象`，循环对象生成元素  
然后使用 `对象的属性的属性` 作为标签class的 `判断条件中的变量`  
如 `:class="[val == attrValDic[name].sv ? 'on': '']"`

---

## vue cli

* 组件定义

一个页面就是一个组件  
使用时import该页面并在components中声明  
声明时的变量名称使用UpDown形式，使用时的标签名称使用up-down形式（因为html标签不支持大写）

* 配置环境变量

项目根目录添加两个文件 `.env.development` 和 `.env.production`  
分别对应启动命令 `vue-cli-service serve` 和 `vue-cli-service build`

默认两个变量 `NODE_ENV` 和 `BASE_URL` 可以直接访问  
自定义变量必须 `VUE_APP_` 开头才能访问，如：VUE_APP_TITLE  

使用`process.env.VUE_APP_BaseUrl`访问

* 使用静态资源

因为会webpack打包，所以  
js中需要使用 `require('../../xxx/xx.jpg')` 方法和项目中的相对路径来获取路径

---

## Vue.use 和 Vue 构造对象

全局组件需要使用 Vue.use 注册，原理是执行组件的 install 方法注册 Vue 的 components  
一般在 router 和 store 的配置代码中写 Vue.use

use之后在 main.js 中 import  
然后在构造对象中添加属性 `xxx: store 或 xxx: router`  
一般同名缩写为 `store, router`  
效果是自动在vue原型上添加全局属性，可以在后续环境中使用 `this.$router`

如果组件没有提供 install 方法就不能 Vue.use，也不能从构造对象传入构造方法，如axios  
此时需要手动定义全局属性 `Vue.prototype.$axios = axios`
