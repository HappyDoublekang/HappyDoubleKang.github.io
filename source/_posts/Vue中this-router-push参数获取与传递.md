---
title: Vue中this.$router.push参数获取与传递
date: 2019-03-18 10:49:21
tags: [Vue]
---

> 传递参数的方法：

* Params

<!-- more -->

由于动态路由也是传递params的，所以在 this.$router.push() 方法中path不能和params一起使用，否则params将无效。需要用name来指定页面。

即通过路由配置的name属性访问

`在路由配置文件中定义参数：`

```javascript
<el-button @click='update(scope.row.id,scope.row.code)'>点击</el-button>
```

```javascript
methods:{
    update(id,code){
        this.$router.push({
            'name':'changeToNewRouter',
            params:{
                id:'id',
                code:'code'
            }
        })
    }
}
```
name对应指定的路由页面（router文件夹下的index.js配置好的路由选项）

```javascript
const changeToNewRouter = resolve => require(['@/pages/.....文件路径'],resolve)
```

```javascript
{
    path:'change_to_new_router',
    name:'changeToNewRouter',
    components:{
        default:changeToNewRouter
    }
}
```

这里的path是显示在url上的地址，name和default则对应上方定义的路由页面

`在目标页面通过this.$route.params获取参数：`

```javascript
export default {
    components:{ElRow},
    name:'update',
    data() {
        return {
            id: this.$router.params.id,
            code: this.$router.params.code
        }
    }
}
```

这样就可以通过$.router.params将页面的参数带入目标页面使用了

* Query

页面通过path和query传递参数，该实例中row为某行表格数据

```javascript
this.$router.push({
    path:'change_to_new_router',
    query:{
        index:index,
        row:row,
        flag:'2'
    }
})
```

```javascript
this.$router.push({
    name:'changeToNewRouter',
    query:{
        index:index,
        row:row,
        flag:'2'
    }
})
```

`在目标页面通过this.$route.query获取参数`

```javascript
index: this.$route.query.index
row: this.$route.query.row
flag: this.$route.query.flag
```

直白的来说query相当于get请求，页面跳转的时候，可以在地址栏看到请求参数，而params相当于post请求，参数不会再地址栏中显示





