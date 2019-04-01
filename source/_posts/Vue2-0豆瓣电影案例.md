---
title: Vue 2.0 豆瓣电影项目
date: 2019-04-01 14:10:25
tags: [Vue,element,axios]
---

## vue-movie 

> 借鉴于简书上的 vue 2.0 豆瓣电影项目并在基础上做了改进，项目包含三个功能模块（首页列表、搜索列表、详情页） ，适合刚入坑 vue 的同学

<!-- more -->

### 所需组件

我们会用到vue的核心组件：vue-router、axios、vue-cli、element

1. 使用vue-cli脚手架创建项目（使用webpack模板）
2. 使用vue-router实现单页面应用
3. 使用axios请求豆瓣API的数据
4. 使用bootstrap样式库
5. 使用element分页

### 安装

> PS：请先安装node和npm，这个就不做多介绍。嫌npm下载太慢，可以安装一个cnpm淘宝npm镜像，安装方式：npm install -g cnpm --registry=https://registry.npm.taobao.org

第一步安装vue-cli

```
cnpm i -g vue-cli
```

然后使用vue-cli脚手架创建一个webpack项目，根据提示一步步来，在是否安装vue-router的时候选择y，省的后面再安装

```
vue init webpack vue-movie
```

![image](https://github.com/HappyDoublekang/image/blob/master/vue-movie.png?raw=true)

安装成功后根据提示进入项目中

```
cd vue-movie
```

安装依赖模块

```
cnpm i
```

启动服务, 它会去package.json中找到script对象下的start，进行热加载部署

```
cnpm run dev
```

在浏览器中输入：localhost:8080进入下面界面就代表我们第一步环境搭建已经完成。给大家提供一个网址，里面详细介绍了vue-cli的webpack模板项目配置文件分析：http://blog.csdn.net/hongchh/article/details/55113751
![image](https://github.com/HappyDoublekang/image/blob/master/vue.png?raw=true)

### 效果图
先感受这三个页面的效果图
![image](https://github.com/HappyDoublekang/image/blob/master/movieList.png?raw=true)
![image](https://github.com/HappyDoublekang/image/blob/master/movieSearch.png?raw=true)
![image](https://github.com/HappyDoublekang/image/blob/master/moveDetailes.png?raw=true)
* 图片加载不出来是因为豆瓣api做了限制

### 正式开工

首先，我们把剩下需要的组件下下来：

```
cnpm i bootstrap axios element-ui -S
```

### 项目结构
```javascript
.
├── build                   // 构建后路径
│   ├── build.js
│   ├── check-versions.js
│   ├── dev-client.js
│   ├── dev-server.js
│   ├── utils.js
│   ├── vue-loader.conf.js
│   ├── webpack.base.conf.js
│   ├── webpack.dev.conf.js
│   └── webpack.prod.conf.js
├── config                  // 配置文件
│   ├── dev.env.js
│   ├── index.js            // api的转发拦截
│   └── prod.env.js
├── src
│   ├── assets
│   ├── components
│   │   ├── movieList.vue   // 电影列表组件
│   │   └── header.vue      // 头部导航栏组件
|   ├── extend              
|   |   ├── api.js          // 接口统一管理
│   ├── router              // 路由
│   │   ├── index.js
│   └── views               // 视图
│       ├── index.vue       // 首页-显示正在上映电影
│       └── search.vue      // 搜索页
│       └── detail.vue      // 详情页
│   ├── App.vue
│   ├── main.js             // 入口文件
├── static
├── README.md
├── index.html
├── package.json
```
电影项目展示主要包含三个模块：首页、搜索页、详情页，根据这三个页面我们在views下新建三个.vue文件。其中头部是固定的，首页和搜索页都要展示影片列表，所以我们抽出来2个共用组件movieList.vue和header.vue。

###  初始化 main.js

```javascript
// The Vue build version to load with the `import` command
// (runtime-only or standalone) has been set in webpack.base.conf with an alias.
import Vue from 'vue'
import App from './App'
import router from './router'
import axios from 'axios'
import ElementUI from 'element-ui'

import 'bootstrap/dist/css/bootstrap.css'
import 'element-ui/lib/theme-chalk/index.css'

import Api from '@/extend/api.js'

// 设置为 false 以阻止 vue 在启动时生成生产提示
Vue.config.productionTip = false
Vue.use(router);
Vue.use(ElementUI);

// 请求绑定Vue属性上面，方便使用
Vue.prototype.$http = axios;
Vue.prototype.api = Api;

/* eslint-disable no-new */
new Vue({
  el: '#app',
  router,
  components: { App },
  template: '<App/>'
})
```

引入组件、路由、样式库、数据请求，并初始化Vue实例，这里我们引入了一个入口App组件以及一个路由配置文件，接下来我们就先开始实行他们

### 创建App组件

```javascript
// src/App.vue
<template>
  <div id="app">
    <v-header></v-header>
    <div class="container">
      <!-- keep-alive来缓存页面的组件 -->
      <keep-alive>
        <router-view></router-view>
      </keep-alive>
    </div>
  </div>
</template>

<script>
import vHeader from './components/Header.vue'

export default {
  name: 'App',
  data() {
    return {
    }
  },
  components:{
    vHeader
  }
}
</script>
```
这里使用了keep-alive来缓存页面的组件，减少性能消耗。  还引入了头部组件，下面就开始写头部代码。

### 头部组件

```javascript
// src/components/header.vue
<template>
    <nav class="navbar navbar-default">
        <div class="container-fluid">
            <!-- Brand and toggle get grouped for better mobile display -->
            <div class="navbar-brand">
                <router-link to="/">豆瓣电影</router-link>
            </div>
            <form class="navbar-form navbar-left">
                <div class="form-group">
                    <input type="text" class="form-control" placeholder="输入电影名称" v-model="searchKey">
                </div>
                <button type="submit" class="btn btn-default" @click.prevent="submit()">搜索</button>
            </form>
        </div><!-- /.container-fluid -->
    </nav> 
</template>

<script>
export default {
    name: 'header',
    data() {
        return {
            searchKey: '',
        }
    },
    methods: {
        submit(){
            if (this.searchKey) {
                // 搜索页面跳转
                //  this.$router.push({
                //     path: '/search/' + this.searchKey,
                // })
                this.$router.push({name:'Search',params:{searchKey:this.searchKey}})
                this.searchKey = "";
            }else{
                alert('请输入搜索内容');
                return;
            }
        }
    },
}
</script>
```
这里包含了搜索功能，输入关键字进行搜索， 下面开始配置各个页面之间的路由。

## 路由配置

```javascript
// src/router/index.js
import Vue from 'vue'
import Router from 'vue-router'
// 路由懒加载
const Index = resolve => require(['@/views/index.vue'],resolve)
const Search = resolve => require(['@/views/search.vue'],resolve)
const Detail = resolve => require(['@/views/detail.vue'],resolve)
const movieList = resolve => require(['@/components/movieList.vue'],resolve)


Vue.use(Router)

export default new Router({
  routes: [
    {
      path: '/',
      name: 'Index',
      components: {
          default: Index
      }
    },
    {
      path: '/search/:searchKey',
      name: 'Search',
      components: {
          default: Search
      }
    },
    {
      path:'/detail/:id',
      name: 'Detail',
      components: {
          default: Detail
      }
    },
    {
      path:'/movieList/:searchKey',
      name: 'movieList',
      components: {
          default: movieList
      }
    }
  ]
})
```

### 豆瓣API
该应用使用了下面3个api：
* `/v2/movie/search?q=关键字` 电影搜索api
* `/v2/movie/in_theaters` 正在上映的电影
* `/v2/movie/subject/:id` 单个电影条目信息

> PS:更多关于豆瓣的api可以前往[豆瓣api官网](https://developers.douban.com/wiki/?title=api_v2)查看。

```javascript
/*
 * @Author: doubleKang 
 * @Date: 2019-03-21 15:10:29 
 * @Last Modified by: doubleKang
 * @Last Modified time: 2019-03-27 16:01:49
 */
//window.location.origin = window.location.protocol + "//" + window.location.hostname + (window.location.port ? ':' + window.location.port: '');

let path = location.hostname === 'localhost'
    ? window.location.protocol + "//" + window.location.hostname + (window.location.port ? ':' + window.location.port : '') + '/'
    : window.location.protocol + "//" + window.location.hostname + (window.location.port ? ':' + window.location.port : '') + '/';

let api = {

    /** 豆瓣电影API */
    inTheaters:'movie/in_theaters', // 获取正在热映电影列表 post
    search:'movie/search', // 查询电影 post
    subject:'movie/subject', // 电影详情 get

};

// for (let key in api) {
//     api[key] = path + api[key] + '?ticket=5f592f53-3db7-4206-9f9c-57ef097a4fad&domain=admin&isAjax=true';
// }

for (let key in api) {
    api[key] = path + api[key];
}


export default api;
```

### API 跨域处理

```javascript
// config/index.js
// see http://vuejs-templates.github.io/webpack for documentation.
var path = require('path')

let proxy_service ='http://api.douban.com/v2';

module.exports = {
    build: {

        env: require('./prod.env'),
        index: path.resolve(__dirname, '../dist/index.html'),
        assetsRoot: path.resolve(__dirname, '../dist'),
        assetsSubDirectory: 'static',
        assetsPublicPath: './',
        productionSourceMap: true,
        // Gzip off by default as many popular static hosts such as
        // Surge or Netlify already gzip all static assets for you.
        // Before setting to `true`, make sure to:
        // npm install --save-dev compression-webpack-plugin
        productionGzip: false,
        productionGzipExtensions: ['js', 'css'],
        // Run the build command with an extra argument to
        // View the bundle analyzer report after build finishes:
        // `npm run build --report`
        // Set to `true` or `false` to always turn it on or off
        bundleAnalyzerReport: process.env.npm_config_report
    },
    dev: {
        env: require('./dev.env'),
        host: 'localhost',
        port: 8080,
        autoOpenBrowser: true,
        assetsSubDirectory: 'static',
        assetsPublicPath: '/',
        proxyTable: {
            '/movie/in_theaters': {
                target: proxy_service,
                changeOrigin: true
            },
            '/movie/search': {
                target: proxy_service,
                changeOrigin: true
            },
            '/movie/subject': {
                target: proxy_service,
                changeOrigin: true
            },
        },
        // CSS Sourcemaps off by default because relative paths are "buggy"
        // with this option, according to the CSS-Loader README
        // (https://github.com/webpack/css-loader#sourcemaps)
        // In our experience, they generally work as expected,
        // just be aware of this issue when enabling this option.
        cssSourceMap: false
    }
}
```

### 电影列表

首页展示的是正在热映的电影(index.vue)，搜索页展示的是搜索后的结果集(search.vue)，他们都是展示电影列表，所以我们共用了一个movieList.vue组件，先上代码：

```javascript
// src/views/index.vue
<template>
    <div>
        <movie-list></movie-list>
    </div>
</template>

<script>
import movieList from '@/components/movieList.vue'

export default {
    name: 'index',
    data() {
        return {
        }
    },
    components:{
        movieList
    }
}
</script>
```

```javascript
// src/views/search.vue
<template>
  <div>
    <!-- <movie-list :movieType="this.$route.params.searchKey" ref="updateList"></movie-list> -->
    <!-- 直接把this.$route.params.searchKey传给子组件用props接收可以用watch监听到变化 -->
    <movie-list :movieType="searchKey" ref="updateList"></movie-list>
    <!-- watch只能监听产生变化的数据，这种写法data只初始化一次不刷新传给子组件watch监听不到 -->
  </div>
</template>

<script>
import movieList from '@/components/movieList.vue'

export default {
    name: 'search',
    data () {
        return{
            searchKey: this.$route.params.searchKey,
            // watch只能监听产生变化的数据，这种写法data只初始化一次不刷新传给子组件watch监听不到
        }
    },
    components:{
        movieList
    },
    watch: {
        // 监听路由，搜索页重复搜索的时候改变路由状态，页面重新加载，不监听的话组件实例会被复用
        '$route' (to, from) {
            // const toDepth = to.path.split('/')
            // const fromDepth = from.path.split('/')
            // this.searchKey = toDepth[2]
            // 防止返回重复调用
            if (to.path.indexOf('/search/') == 0) {
                // 调用子组件方法
                this.$refs.updateList.loadMovieList();
            }
        }
    }
}
</script>
```

```javascript
// src/components/movieList.vue
<template>
  <div class="container">
    <div class="canvas" v-show="loading">
      <div class="spinner"></div>
    </div>
    <h2>{{title}}</h2>
    <div class="row">
      <div class="col-md-2 text-center" v-for="item in list" :key="item.id">
        <router-link :to="{path:'/detail/'+item.id}">
          <img :src="item.images.medium">
          <div class="title">{{item.title}}</div>
        </router-link>
      </div>
    </div>

    <!-- 分页用的是element-ui -->
    <el-pagination
      style="text-align: right;margin-bottom: 20px;"
      @size-change="handleSizeChange"
      @current-change="handleCurrentChange"
      :current-page="currentPage"
      :page-sizes="[18,36,54,72]"
      :page-size="10"
      layout="total,sizes, prev, pager, next, jumper"
      :total="totalItem">
    </el-pagination>
  </div>
</template>

<script>
  export default {
    props: ['movieType'],// 接收父组件传过来的值
    data () {
      return {
        loading: true,
        title: '',
        list: [],
        commonApi:'',
        totalItem: 0,
        currentPage: 1,
        postDate:{
          q: '',
          count: '18',
          start: '1'
        }
      }
    },
    watch: { //watch不起作用因为props传过来的值永远不变始终为父组件的data里面第一次接收到的值
      movieType(newV,oldV){
        console.log(newV)
        if(newV){
          this.movieType = newV
        }
      }
    },
    created(){
      this.loadMovieList();
    },
    methods: {
      loadMovieList(){
        this.loading = true;
        let api;
        console.log(this.movieType)
        // props的data初始化了一次所以this.movieType始终为第一次传的值
        if(this.movieType){
          api = this.api.search;
          this.postDate.q = this.$route.params.searchKey
          // this.postDate.q = this.movieType
        }else{
          api = this.api.inTheaters;
        }
        this.$http.post(api, this.postDate).then((res) => {
          this.list = res.data.subjects;
          this.title = res.data.title;
          this.loading = false;
          this.totalItem = res.data.total;
        })
      },
      /** 修改页码 */
      handleCurrentChange(val) {
          this.postData.start = val;
          this.loadMovieList();
      },
      /** 单页显示数量 */
      handleSizeChange(val) {
          this.postData.count = val;
          this.loadMovieList();
      },
    }
  }
</script>
```

> 组件实例的作用域是孤立的。这意味着不能 (也不应该) 在子组件的模板内直接引用父组件的数据。父组件的数据需要通过 prop 才能下发到子组件中。 所以我们在header.vue 和 search.vue中的movie-list我们定义了一个Prop属性movieType, 下发给子组件movieList.vue,告知当前类型是什么，并根据类型请求相应类型的数据。
```TypeScript
// 父级
<movie-list :movieType="searchKey" ref="updateList"></movie-list>
```

```TypeScript
// 子级
export default {
  ...
  props: ['movieType'],// 接收父组件传过来的值
  ...
}
```

> 在重复搜索的时候，我们会发现页面根本不会重新请求数据，并渲染页面。这是因为：当使用路由参数时，例如从 /search/a 导航到 search/b，原来的组件实例会被复用。因为两个路由都渲染同个组件，比起销毁再创建，复用则显得更加高效。不过，这也意味着组件的生命周期钩子不会再被调用。复用组件时，想对路由参数的变化作出响应的话，可以简单地 watch（监测变化） $route 对象  ,所以在Search.vue中加入如下代码

```TypeScript
watch: {
    // 监听路由，搜索页重复搜索的时候改变路由状态，页面重新加载，不监听的话组件实例会被复用
    '$route' (to, from) {
        // const toDepth = to.path.split('/')
        // const fromDepth = from.path.split('/')
        // this.searchKey = toDepth[2]
        // 防止返回重复调用
        if (to.path.indexOf('/search/') == 0) {
            // 调用子组件方法
            this.$refs.updateList.loadMovieList();
        }
    }
}
```

### 详情

```TypeScript
// src/views/details.vue
<template>
  <div class="container">
    <div class="canvas" v-show="loading">
      <div class="spinner"></div>
    </div>
    <h2>{{movie.title}}({{movie.year}})</h2>
    <div class="box">
      <div class="left">
        <img :src="movie.images.large">
      </div>
      <div class="main">
        <div>导演：{{getDirectors}}</div>
        <div>主演：{{getCasts}}</div>
        <div>评分：{{movie.rating.average}}</div>
        <div>
          <h3>{{movie.title}}剧情介绍</h3>
          {{movie.summary}}
        </div>
      </div>
    </div>
  </div>
</template>

<script>
  export default {
    data () {
      return {
        loading: true,
        movie: {
          images: {
            large: ''
          },
          rating: {
            average: 0
          }
        }
      }
    },
    activated(){
      // 在APP.vue中使用了构建组件keep-alive,所有可用activated属性来代替watch中的$route监听
      // 获取电影详情
      this.getMovieDetail();
    },
    methods: {
      // 获取电影详情
      getMovieDetail(){
         this.$http.get(this.api.subject + '/' + this.$route.params.id).then((res) => {
          console.log(res.data);
          this.movie = res.data;

          this.loading = false;
        })
      },
      // 过滤数据
      getFilterData(obj){
        let arr = [];
        if (!obj || obj.length == 0)return '';

        for (let i = 0; i < obj.length; i++) {
          arr.push(obj[i].name)
        }
        return arr.join('/');
      },
    },
    computed: {
      // 获取导演
      getDirectors(){
        return this.getFilterData(this.movie.directors);
      },
      // 获取主演
      getCasts(){
        return this.getFilterData(this.movie.casts);
      },
    }
  }
</script>
```

> 这个也会碰到跟重复搜索一样的问题，进入详情后返回在进入另一个详情页面不变，原理一样。但是还有另一种实现方式，由于我们在App.vue中使用了构建组件keep-alive,所以可用钩子函数activated属性来代替watch中的$route监听。可多了解下vue的生命周期

### 总结（遇到的问题）

* 对于search和index共用一个moviList渲染页面，所以一开始的想法就是从header里v-model数据push到search再props给movieList，问题在于header的数据push给了search

```javascript
//search
<movie-list :movieType="searchKey" ref="updateList"></movie-list>

data () {
    return{
        searchKey: this.$route.params.searchKey,   // 第一次传参数 1111111 第二次 2222222
    }
```

这种写法data只初始化一次所以search传给movieList的data不刷新movieList子组件watch监听不到参数变化，这样悲剧就发了

```javascript
//movieList
<script>
  export default {
    props: ['movieType'],   // 第二次搜索参数应该为 2222222 但此时却显示为 1111111
    watch: {                //watch不起作用因为props传过来的值永远不变始终为父组件的data里面第一次接收到的值
      movieType(newV,oldV){
        if(newV){
          this.movieType = newV
        }
      }
    },
    methods: {
      loadMovieList(){
        let api;
        console.log(this.movieType)
        // props的data初始化了一次所以this.movieType始终为第一次传的值 1111111
        if(this.movieType){
          api = this.api.search;
          this.postDate.q = this.movieType
        // 这里的q的值永远为 1111111 所以搜索功能失效
        }else{
          api = this.api.inTheaters;
        }
        this.$http.post(api, this.postDate).then((res) => {
         ...
        })
      },
    }
```
捣鼓了一下午，利用watch的深拷贝，data的初始化发现都没什么卵用（此时接近崩溃）

最终解决办法是直接 `this.postDate.q = this.$route.params.searchKey` 不通过data定义


> [源码地址：Vue2.0 豆瓣电影项目](https://github.com/HappyDoublekang/DouBan)



