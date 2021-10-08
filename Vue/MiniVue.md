# Mini-Vue

简洁版的Vue包括三大模块：

* 渲染系统模块
* 响应式系统模块
* 应用程序入口模块

## 一. 渲染系统实现

主要包含三个功能：

* h函数，用于返回一个VNode对象
* mount函数，用于将VNode挂载到DOM上
* patch函数，用于对两个VNode进行对比，决定如何处理新的VNode

### 1.1 h函数实现

**h函数主要用来创建vnode**

h函数主要有三个参数，tag标签，props标签的属性key/value，以及children（可以是textContent内容，也可以是子节点）

由于vnode本质就是JavaScript对象，所以可以直接将三个参数包裹成对象返回成为VNode。

`注意：children传入子节点时，传入的也是h函数返回结果，这样就保证了在直接return包装后的对象是嵌套的。`

```javascript
const h = (tag, props, children) => {
  //vnode -> javascript对象
  return {
    tag,
    props,
    children,
  }
}

//使用
const vnode = h('div', {class: "lyk"}, [
  h('h2', null, "当前计数：100"),
  h('button',{onClick: function(){console.log('点击btn')}}, "点击")
])
```

### 1.2 mount函数实现

mount函数用于将vnode变成NODE节点并挂载到DOM上

```javascript
const mount = (vnode, container) => {
  //vnode -> element

  //1.创建出真实的dom，并在vnode上做保留
  const el = vnode.el = document.createElement(vnode.tag);

  //2. 处理props
  if(vnode.props) {
    for(const key in vnode.props) {
      const value = vnode.props[key];
      if(key.startsWith("on")) {    //用于处理监听事件
        el.addEventListener(key.slice(2).toLowerCase(), value);
      } else {
        el.setAttribute(key, value);
      }
    }
  }

  //3. 处理children
  if(vnode.children) {
    if(typeof vnode.children === 'string') {   //判断children是本文节点还是嵌套的node节点
      el.textContent = vnode.children;
    } else {
      vnode.children.forEach(item => {
        mount(item, el);            //递归调用便可生成带有嵌套层级的DOM树结构
      })
    }
  }

  //4. 将el挂载到container上
  container.appendChild(el);
}

//使用
mount(vnode, document.getElementById('app'));
```

### 1.3 patch函数实现

patch函数主要用于对比前后两个vnode的改变，以更新DOM上的节点

```javascript
const patch = (vnode1, vnode2) => {
  if(vnode1.tag != vnode2.tag) {
    const v1Parent = vnode1.el.parentElement;
    v1Parent.removeChild(vnode1.el);
    mount(vnode2, v1Parent);
  } else {
    // 1.取出element对象，并且在vnode2中进行保存
    const el = vnode2.el = vnode1.el;

    // 2.1 处理props
    const oldProps = vnode1.props || {};
    const newProps = vnode2.props || {};
    for(const key in newProps) {
      const oldValue = oldProps[key];
      const newValue = newProps[key];
      if( newValue != oldValue ) {
        if(key.startsWith("on")) {
          const value = oldProps[keys];
          el.addEventListener(key.slice(2).toLowerCase(), newValue);
        } else {
          el.setAttribute(key, newValue);
        }
      }
    }

    // 2.2 删除旧的props
    for(const key in oldProps) {
      if(!(key in newProps)) {
        if(key.startsWith('on')) {
          const value = oldProps[key];
          el.removeEventListener(key.slice(2).toLowerCase, value);
        }else {
          el.removeAttribute(key);
        }
      }
    }

    // 3. 处理children
    const oldChildren = vnode1.children || [];
    const newChildren = vnode2.children || [];
    if(typeof newChildren == 'string') {
      el.innerHTML = newChildren;
    } else {
      if( typeof oldChildren == 'string') {
        el.innerHTML = '';
        newChildren.forEach(item => {
          mount(item, el);
        })
      } else {
        const commonLength = Math.min(oldChildren.length, newChildren.length);
        for(let i = 0; i < commonLength; i++) {
          console.log(oldChildren[i], newChildren[i]);
          patch(oldChildren[i], newChildren[i]);
        }

        if(oldChildren.length < newChildren.length) {
          newChildren.slice(commonLength).forEach(item => {
            mount(item, el);
          })
        }

        if(oldChildren.length > newChildren.length) {
          oldChildren.slice(commonLength).forEach(item => {
            el.removeChild(item.el);
          })
        }
      }
    }
  }
}
```

## 二. 响应式系统实现

```javascript
class Dep {
  constructor() {
    this.subs = new Set();
  }
  
  addSub(target) {
    this.subs.add(target);
  }
  
  notify() {
    this.subs.forEach(target => {
      target()
    })
  }
}

const targetMap = new WeakMap();
function getDep(target, key) {
  let depsMap = targetMap.get(target);
  if(!depsMap) {
    depsMap = new Map();
    targetMap.set(target, depsMap);
  }

  let dep = depsMap.get(key);
  if(!dep) {
    dep = new Dep();
    depsMap.set(key, dep);
  }
}

// vue2实现
function reactive(raw) {
  Object.keys(raw).forEach(key => {
    const dep = getDep(raw, key);
    const value = rwa[key];
    Object.defineProperty(raw, key, {
      get() {
        dep.addSub();
        return value;
      },
      set(newValue) {
        value = newValue;
        dep.notify();
      }
    })
  })
  return raw;
}

//vue3实现
function reactive(raw) {
  return new Proxy(raw, {
    get(target, key) {
      const dep = getDep(target, key);
      dep.addSub();
      return target[key];
    },
    set(target, key, newValue) {
      const dep = getDep(target, key);
      target[key] = newValue;
      dep.notify();
    }
  });
}
```

