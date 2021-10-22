# Vuex

**状态管理：**开发应用程序过程中，有些数据需要保存在某一位置，对于这些数据的管理称之为**状态管理**。

### 一. 核心原理

* Vuex本质是一个对象，这个对象有两个属性，一个是install方法，一个是Store这个类
* install方法的作用是将Store这个实例挂载到所有组件上，注意是同一个Store实例‘
* Store这个类拥有commit、dispatch这些方法
* Store类里将用户传入的state包装成data，作为new Vue的参数，从而实现了state值的响应式

### 二. 剖析Vuex本质

**我们需要解决什么问题？**

* 如何安装到Vue实例上
* 如何让所有组件都可以拥有store实例
* 如何让store数据响应式化

#### 2.1 如何注册到Vue实例上

利用`vue.use()`方法，调用对象的install方法来注册

```javascript
class Store {
  
}
let install = function(){
  
}
let vuex = {
  Store,
  install
}
export default vuex
```

#### 2.2 如何让所有组件都可以拥有store实例

我们注意到，在`store/index.js`里写vuex的配置时，是调用了`new Vuex.Store`方法构造一个Store实例，然后导出这个`store`实例。在根组件实例里，通过`new Vue({store})`来先让根组件拥有`$store`属性。

前面提到通过Vue.use(Vuex)使得每个组件都可以拥有store实例,是通过`vue.mixin`全局注入`beforeCreate: vuexInit`来实现的。

```javascript
let install = function(Vue) {
  Vue.mixin({
    beforeCreate() {
      if(this.$options && this.$options.store) {  //这里判断出是根组件
				this.$store = this.$options.store;
      } else {    //否则是子组件
        this.$store = this.$parent && this.$parent.$store;
      }
    }
  })
}
```

#### 2.3 如何让store数据响应式化

们知道，我们new Vue（）的时候，传入的data是响应式的，那我们是不是可以new 一个Vue，然后把state当作data传入呢？ 没有错，就是这样。

```javascript
class Store{
    constructor(options) {
        this.vm = new Vue({
            data:{
                state:options.state
            }
        })
    }

}
```

现在虽然实现了响应式，但似乎只能通过`this.$store.vm.state`来获得，而实际上使用时我们是直接通过`this.$store.state`来获取数据的。

我们可以给Store类的state设定一个属性getter

```javascript
class Store{
    constructor(options) {
        this.vm = new Vue({
            data:{
                state:options.state
            }
        })
    }
    //新增代码
    get state(){
        return this.vm.state
    }
}
```

