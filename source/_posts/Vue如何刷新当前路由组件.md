---
title: Vue如何刷新当前路由组件
date: 2019-03-15 10:44:29
tags: [Vue]
---

## Vue刷新当前页面，provide/inject组合方式实现vue页面刷新

* 项目遇到的问题：点击按钮需要清空页面数据（相当于刷新页面）为了用户体验，页面刷新时不能有flash闪烁

<!-- more -->

弹出dialog对话框的时候点击确定按钮使表格重新加载

```c++
<el-dialog>
    <el-button @click="resetForm()">继续新增</el-button>
</el-dialog>
```

当dialog弹框出现，点击继续新增按钮，刷新该路由当前展示的表单数据

### 最简单暴力的方法是刷新当前页面

```Typescript
methods: {
    resetForm(){
            location.reload();
        }
    }
```

问题是刷新页面会使得整个页面出现空白，用户体验差

### provide / inject 组合

* 首先要修改 App.vue 组件 添加如下属性

```javascript
<template>
  <div id="app">
    <router-view v-if="isRouterAlive"></router-view>
  </div>
</template>

<script>
export default {
  name: 'app',
  provide(){
    return{
      reload:this.reload
    }
  },
  data(){
    return{
      isRouterAlive:true
    }
  },
  methods:{
    reload(){
      this.isRouterAlive = false;
      this.$nextTick(function(){
        this.isRouterAlive = true;
      })
    }
  }
}
</script>
```

* 在需要当前页面刷新的页面中注入App.vue组件提供（provide）的 reload 依赖，然后直接用this.reload来调用就行

```javascript
export default {
    inject:['reload'],
    resetForm(){
        this.reload();
    },
}
```

### 详情（写在最后(´ο｀*)）

感兴趣的朋友想要深入理解可以查询 ** [Vue官方解释](https://cn.vuejs.org/v2/api/#provide-inject) **




