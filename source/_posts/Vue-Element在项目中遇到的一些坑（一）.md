---
title: Vue-Element在项目中遇到的一些坑（一）
date: 2019-07-30 09:49:43
tags: [Vue, element]
---

## 目前在用Vue框架结合element做后台管理系统，遇到了一些坑和大家分享一下。

> 先简单介绍一下这个坑产生的公司业务场景

<!-- more -->

公司系统需要在任何一个页面任何一次http请求，增加对token过期的判断，如果token已过期，需要跳转至登录页面。如果要在每个页面中的http请求操作中添加一次判断，那么会是一个非常大的修改工作量。

a few moments later...

### 经过谷歌上百度一下发现（当然是用vue拦截器啦），这个大伙肯定都知道，这有啥坑的呢，照搬一套，撸码到自己页面上

下边代码添加在main.js中

```javascript
/*vue-resource请求拦截器*/
Vue.http.interceptors.push((request, next) => {
    if (request.url) {
        let ticket = '';
        let auth_ticket = ''; // AUTH_TICKET
        if (isAdmin) {
            ticket = sessionStorage.getItem('adminticket'); //超级管理员
        } else {
            ticket = sessionStorage.getItem('partnerticket');  //合作伙伴
        }
        if (ticket != undefined && ticket != null && ticket != "null" && ticket != "") {
            if (request.url.indexOf("?") != -1) {
                request.url = request.url  + "&domain=" + (isAdmin ? 'admin' : 'partner') + "&isAjax=true" + "&ticket=" + ticket
            } else {
                request.url = request.url  + "?&domain=" + (isAdmin ? 'admin' : 'partner') + "&isAjax=true" + "?ticket=" + ticket
            }
        } else {
            // alert("没有获取到合管存储SessionStorage的值：ticket");
        }
    } else {
        // alert("没有获取到请求接口的URL");
    }
    next((response) => {
    });
})
```

### 是的没错，我们的token转换成了ticket存在session里面，不要在意这些细节...

一开始业务让我们把ticket也就是token拼接在所有页面的接口请求后面，开心的一腿，这还不简单，疯狂拼接一坨之后...

发现好了，没事了继续划水，然后特么的，业务的嘴，骗人的鬼，果然那个吊毛又不满意了，让我把ticket拿下来，放到请求头里去(此时心中一万匹草泥马跑过)

拿就拿吧，此时还没发现问题即将出现，新的风暴即将来临，拦截器把token放请求头的方法如下(重复代码就不贴了)：

```javascript
    request.headers.set('ticket', ticket)   
    document.cookie = `AUTH_TICKET=${ticket};path=/`
    ...
    if (request.url.indexOf("?") != -1) {
        request.url = request.url  + "&domain=" + (isAdmin ? 'admin' : 'partner') + "&isAjax=true";
    } else {
        request.url = request.url  + "?&domain=" + (isAdmin ? 'admin' : 'partner') + "&isAjax=true";
    }
```

弄个`request.headers.set('ticket', ticket)`方法即可

### 突然发现有点问题，因为拦截器有一个坑，无法拦截下载excel等文件的请求，此时内心是崩溃的，这可咋整...

好了不多说，直接上解决办法

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
|   ├── extend              
|   |   ├── api.js          // 接口统一管理
|   |   ├── utils.js        // 公共方法统一调用
│   ├── router              // 路由
│   │   ├── index.js 
│   ├── App.vue
│   ├── main.js             // 入口文件
├── static
├── README.md
├── index.html
├── package.json
```

> 设置公共方法untils

```javascript
import Vue from 'vue'

let utils = {
    /**
     * 导出（post/get）
     * @param methods
     * @param params（obj）
     * @param api
     */
    exportIt(methods, params, api) {
        let ticket =  sessionStorage.getItem('adminticket') || sessionStorage.getItem('partnerticket');
        /*post*/
        if (methods == 'post') {
            var url = api;
            var xhr = new XMLHttpRequest();
            xhr.open('POST', url, true); // 也可以使用POST方式，根据接口
            xhr.setRequestHeader('Content-Type', 'application/json');
            xhr.setRequestHeader('ticket', ticket);
            xhr.responseType = "blob"; // 返回类型blob
            // 定义请求完成的处理函数，请求前也可以增加加载框/禁用下载按钮逻辑
            xhr.onload = function() {
                // 请求完成
                if (this.status === 200) {
                    // 获取文件名称
                    let nameArr = xhr.getResponseHeader('content-disposition');
                    let filename;
                    if (nameArr) {
                        filename = nameArr.split('filename=')[1]
                        filename = decodeURIComponent(filename)
                    } else {
                        filename = '未命名';
                    }
                    // 返回200
                    var blob = this.response;
                    // var name = this.response.getHearder;
                    var reader = new FileReader();
                    reader.readAsDataURL(blob); // 转换为base64，可以直接放入a表情href
                    reader.onload = function(e) {
                        // 转换完成，创建一个a标签用于下载
                        var a = document.createElement('a');
                        a.download = filename;
                        a.href = e.target.result;
                        document.body.append(a); // 修复firefox中无法触发click
                        a.click();
                        a.remove();
                    }
                }
            };
            // 发送ajax请求
            xhr.send(JSON.stringify(params))
            return
        }else if(methods == 'get'){
            let getApi = api + '?' + this.urlEncode(params);
            let nowDate = this.getNowDatePlus();
            let execlName = '渠道信息列表' + nowDate + '.xlsx';
            let token = '';
            if(sessionStorage.getItem('adminticket')){
                token = sessionStorage.getItem('adminticket');
            }else{
                token = sessionStorage.getItem('partnerticket');
            }
            Vue.http.get(getApi, {
                headers:{
                    "ticket":token
                },
                responseType: 'blob', //二进制流
            }).then(function (res) {
                if(!res) return
                let blob = new Blob([res.data], {type: 'application/vnd.ms-excel;charset=utf-8'})
                let getApi = window.URL.createObjectURL(blob);
                let aLink = document.createElement("a");
                aLink.style.display = "none";
                aLink.href = getApi;
                aLink.setAttribute("download", execlName);
                document.body.appendChild(aLink);
                aLink.click();
                document.body.removeChild(aLink);
                window.URL.revokeObjectURL(getApi);
            }).catch(function (error) {
                console.log(error)
            });
        }
        /*get*/
        // window.open(api + '?' + this.urlEncode(params));
    }
}
export default utils;
```

> 在需要用到下载的页面调用公共方法即可，支持get和post请求

```javascript
let params = {
    'xxx': this.xxx,
}
this.utils.exportIt('get', params, this.api.xxx)
```

































