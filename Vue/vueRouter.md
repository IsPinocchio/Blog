# VueRouter

### 一. 后端路由简介

路由这个概念最先由后端出现。简单来说路由就是用来跟后端服务器进行交互的一种方式，通过不同的路径来请求不同的资源。

### 二. 前端路由

随着ajax的流行，在不刷新浏览器的情况下异步请求数据应用越来越广泛。而异步交互体验的更高级版本就是`SPA`。单页面应用不仅仅是页面交互无刷新，连页面跳转都是无刷新的。

类似于服务器路由，前端路由实现起来也是通过匹配不同的url路径，来动态渲染某区域的html内容。

但主要需要解决的问题是，如果在url变化的时候不让页面刷新。

#### 1. hash模式

`http://www.xxx.com/#/login`

这种#后面拼接的是hash部分，hash值的变化不会导致浏览器向服务器发出请求，也就不会发生刷新。hash值的变化还会触发`haschange`这个事件，可以通过这个事件得知hash值的变化，从而更新部分页面内容。

#### 2. history模式

HTML5标准发布，多了两个API——`pushState`和`replaceState`，通过这两个API可以改变url地址并且不会发起请求，同时还有popstate事件。但是要注意在页面点击刷新时会发送请求导致请求路径不存在情况。

### 三. 手动实现VueRouter

#### 3.1 全局注册vueRouter

```javascript
// myVueRouter.js
let _vue = null;
export default class myVueRouter {
  static install(Vue) {
    //判断是否注册了myVueRouter，避免重复注册
    if(myVueRouter.install.installed) return;
    myVueRouter.install.installed = true;
    _Vue = Vue;
    _Vue.mixin({
      beforeCreate() {
				if(this.$options.router) {
          //将router挂到_Vue原型上
          _Vue.prototype.$router = this.$options.router
        }
      }
    })
  }
}
```

#### 3.2 实现myVueRouter构造方法

`options`保存传入的规则

`routerMap`确定地址和组件的关系

`current`表示当前的地址是响应式的之后渲染组件和它相关

```javascript
export default class myVueRouter {
  constructor(options) {
    this.options = options;
    this.routerMap = {};
    this.data = _Vue.observable({
      current: '/'
    })
  }
}
```

