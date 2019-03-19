---
title: Vue组件之间的数据传递
date: 2019-03-19 09:52:00
tags: [Vue]
---

## Vue 的组件作用域都是孤立的，不允许在子组件的模板内直接引用父组件的数据。必须使用特定的方法才能实现组件之间的数据传递。

* 首先用 vue-cli 创建一个项目，其中 App.vue 是父组件，components 文件夹下都是子组件。

<!-- more -->

```javascript
├── src
│   ├── assets
│   ├── components
|   |   ├── login.vue
|   |   ├── header.vue
│   |   └── footer.vue
│   ├── css
│   ├── img
|   ├── App.vue
│   ├── main.js
|
```

### 父组件向子组件传递数据

* 可以使用 props 向子组件传递数据。

** 子组件部分： **

```javascript
<template>
    <header class='header'>
        <div id='logo'>{{logo}}</div>
        <ul>
            <li v-for='nav in navs'>{{nav.li}}</li>
        </ul>
    </header>
</template>
```

如果需要从父组件获取 logo 的值，就需要使用 props: ['logo']

```javascript
<script>
    export default{
        name: 'headerDiv',
        props: ['logo'],
        data () {
            return {
                navs:[
                    {li: '主页'},
                    {li: '日志'},
                    {li: '说说'},
                    {li: '留言'},
                    {li: '相册'}
                ]
            }
        }
    }
</script>
```

在 props 中添加了元素之后，就不需要在 data 中再添加变量了

** 父组件部分： **

```Typescript
<template>
    <div id='app'>
        <HeaderDiv :logo='logoMsg'></HeaderDiv>
    <div>
</template>
```

在调用组件的时候，使用 v-bind 将 logo 的值绑定为 App.vue 中定义的变量 logoMsg

```javascript
<script>
    import HeaderDiv from './components/header.vue',

    export default{
        name: 'app',
        data () {
            return {
                logoMsg: 'test'
            }
        },
        components: {
            HeaderDiv
        }
    }
</script>
```

然后就能将App.vue中 logoMsg 的值传给 header.vue 了：

`test 主页 日志 说说 留言 相册`

<!-- ![图片上传失败...](https://github.com/HappyDoublekang/image/blob/master/wisewrong.png?raw=true) -->

### 子组件向父组件传递数据

* 子组件主要通过事件传递数据给父组件

** 子组件部分： **

```Typescript
<template>
    <section>
        <div class='login'>
            <label>
                <span>用户名：</span>
                <input v-model='userName' @change='setUser'>
            </label>
        </div>
    </section>
</template>
```

这是 login.vue 的 HTML 部分，当 input 的值发生变化的时候，将 username 传递给 App.vue

首先声明一个了方法 setUser，用 change 事件来调用 setUser

```javascript
<script>
    export default {
        name: 'login',
        data () {
            return {
                userName: ''
            }
        },
        methods: {
            setUser () {
                this.$emit('transferUser',this.userName);
            }
        }
    }
</script>
```

在 setUser 中，使用了 $emit 来遍历 transferUser 事件，并返回 this.userName

其中 transferUser 是一个自定义的事件，功能类似于一个中转，this.userName 将通过这个事件传递给父组件 

** 父组件部分： **

```Typescript
<template>
    <div id='app'>
        <LoginDiv @transferUser='getUser'></LoginDiv>
        <p>用户名为：{{user}}</p>
    <div>
</template>
```

在父组件 App.vue 中，声明了一个方法 getUser，用 transferUser 事件调用 getUser 方法，获取到从子组件传递过来的参数 userName

```javascript
<script>
import LoginDiv from './components/login.vue'

export default {
    name: 'app',
    data () {
        return {
            user: ''
        }
    },
    methods: {
        getUser (msg) {
            this.user = msg
        }
    },
    components: {
        LoginDiv
    }
}
</script>
```

getUser 方法中的参数 msg 就是从子组件传递过来的参数 userName

`用户名：test 用户名为：test`

<!-- ![图片上传失败...](https://github.com/HappyDoublekang/image/blob/master/wisewrong.gif?raw=true) -->

### 子组件向子组件传递数据

Vue 没有直接子对子传参的方法，建议将需要传递数据的子组件，都合并为一个组件。如果一定需要子对子传参，可以先从传到父组件，再传到子组件。

为了便于开发，Vue 推出了一个状态管理工具 Vuex，可以很方便实现组件之间的参数传递






