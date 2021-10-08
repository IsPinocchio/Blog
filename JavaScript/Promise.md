# Promise

Promise是一种异步解决方案，比传统的解决方案——回调函数，更合理和更强大。

Promise简单来说是一个容器，里面保存着未来才会结束的事件（通常是一个异步操作）的结果。

### 基本原理

> * Promise是一个类，在创建实例的时候，会传入一个`执行器`，这个执行器会立即执行
> * Promise有`三种状态`
>   * Pending 等待
>   * Fulfilled 完成
>   * Rejected 失败
> * 状态只能由`Pending->Fulfilled`或者`Pending->Rejected`，状态一旦改变便不可二次更改
> * Promise中使用resolve和reject两个函数来`改变状态`
> * then方法内部做的事情就是`根据状态`的改变来调用相应的回调函数

### 逐步解决哪些细致问题

* then方法的异步回调
* then方法的多次调用
* then方法的链式调用
* then方法返回值不能是promise本身
* then可以不传任何参数
* 将then方法的回调函数变为微任务

### 简版Promise

```javascript
class MyPromise {
  constructor(executor) {
    this.status = "pending";
    this.value = undefined;
    this.reason = undefined;
    this.onResolveCallbacks = [];
    this.onRejectedCallbacks = [];
    const resolve = (value) => {
      if (this.status === "pending") {
        this.status = "fulfilled";
        this.value = value;
        while (this.onResolveCallbacks.length) {
          this.onResolveCallbacks.shift()(this.value);
        }
      }
    };
    const reject = (reason) => {
      if (this.status === "pending") {
        this.status = "rejected";
        this.reason = reason;
        while (this.onRejectedCallbacks.length) {
          this.onRejectedCallbacks.shift()(this.reason);
        }
      }
    };

    executor(resolve, reject);
  }

  then(onFulfilled, onRejected) {
    if (this.status === "fulfilled") {
      onFulfilled(this.value);
    } else if (this.status === "rejected") {
      onRejected(this.reason);
    } else if (this.status === "pending") {
      this.onResolveCallbacks.push(onFulfilled);
      this.onRejectedCallbacks.push(onRejected);
    }
  }
}
```

以上实现了then的多次调用，但并没有实现then的链式调用

### then的链式调用

then方法要实现链式调用就需要then返回一个Promise对象。

then方法里面return一个返回值作为下一个then方法的参数，如果return的是一个promise对象，就需要判断这个promise的状态。

```javascript
class MyPromise {
  constructor(executor) {
    this.status = "pending";
    this.value = undefined;
    this.reason = undefined;
    this.onResolveCallbacks = [];
    this.onRejectedCallbacks = [];
    const resolve = (value) => {
      if (this.status === "pending") {
        this.status = "fulfilled";
        this.value = value;
        while (this.onResolveCallbacks.length) {
          this.onResolveCallbacks.shift()();
        }
      }
    };
    const reject = (reason) => {
      if (this.status === "pending") {
        this.status = "rejected";
        this.reason = reason;
        while (this.onRejectedCallbacks.length) {
          this.onRejectedCallbacks.shift()();
        }
      }
    };
    executor(resolve, reject);
  }
  
  then(onFulfilled, onRejected) {
    const p2 = new MyPromise((res, rej) => {
      if (this.status === "fulfilled") {
        queueMicrotask(() => {
          const x = onFulfilled(this.value);
          resolvePromise(p2, x, res, rej);
        });
      } else if (this.status === "rejected") {
        queueMicrotask(() => {
          const x = onRejected(this.reason);
          resolvePromise(p2, x, res, rej);
        })
      } else if (this.status === "pending") {
        this.onResolveCallbacks.push(() => {
          queueMicrotask(() => {
            const x = onFulfilled(this.value);
            resolvePromise(p2, x, res, rej);
          });
        });
        this.onRejectedCallbacks.push(() => {
          queueMicrotask(() => {
            const x = onRejected(this.reason);
            resolvePromise(p2, x, res, rej);
          });
        });
      }
    });
    return p2;
  }
}

function resolvePromise(promise2, x, res, rej) {
  //如果相等了，说明return的是自己，抛出类型错误并返回
  if (promise2 === x) {
    return rej(new TypeError("不能循环调用自己"));
  }
  if (x instanceof MyPromise) {
    x.then(
      (value) => res(value),
      (reason) => rej(reason)
    );
  } else {
    res(x);
  }
}
```

> 注意上方用到了**queueMicrotask**方法
> **目的一：**为了resolvePromise方法可以接收到p2参数
> **目的二：**为了将回调任务变为微任务执行（如果不这么做，那么当Promise同步执行resolve时，then方法调用会当成当前宏任务执行）
>
> 比如：
>
> ```javascript
> new Promise(res => {
> 	console.log(1)
>   res()
> }).then(res => {
>   console.log(3)
> }).then(res => {
>   console.log(4)
> })
> console.log(2)
> ```
>
> 正常来讲应该输出：1 2 3 4
>
> 而如果不用queueMicrotask包装的话，那就是单纯的不断同步调用，最终会输出1 3 4 2

### 捕获错误

**执行器时捕获错误**

```javascript
constructor(executor) {
	try {
    executor(resolve, reject)
  } catch(err) {
		reject(err)
  }
}
```

**then执行时捕获错误**

```javascript
then(onFulfilled, onRejected) {
  const p2 = new MyPromise((resolve, reject) => {
    ...
    queueMicrotask(() => {
      try{
        const x = onFulfilled(this.value);
        resolvePromise(p2, x, resolve, reject)
      } catch(err) {
        reject(err)
      }
    })
  })
}
```

### then的参数变为可选

```javascript
then(onFulfilled, onRejected) {
  onFulfilled = typeof onFulfilled === 'function' ? onFulfilled : value  => value;
  onRejected = typeof onRejected === 'function' ? onRejected : reason => reason;
  
  const p2 = new MyPromise((resolve, reject) => {
    ...
  })
  return p2;
}
```

### 实现resolve和reject静态方法

```javascript
class MyPromise {
  ...
  
  static resolve(value) {
    if (value instanceof MyPromise) {
      return value;
    }
    return new MyPromise((resolve) => {
      resolve(value);
    });
  }

  static reject(reason) {
    return new MyPromise((resolve, reject) => {
      reject(reason);
    });
  }
}
```

### 其他方法

**Promise.all**

```javascript
class MyPromise {
  ...
  
  static all(promises) {
		const result = [];
    let count = 0;
    return new MyPromise((resolve, reject) => {
      function addData(index, data) {
				result[index] = data;
        if(++count === promises.length) resolve(result);
      }
      promises.forEach((promise, index) => {
        if(promise instanceof MyPromise) {
          promise.then(res => {
            addData(index, res);
          }, err => reject(err))
        } else {
          addData(index, promise);
        }
      })
    })
  }
}
```

**Promise.race**

```javascript
class MyPromise {
  ...
  
  static race(promises) {
    return new Promise((resolve, reject) => {
      promises.forEach((promise) => {
        if(promise instanceof MyPromise) {
          promise.then(res => {
            resolve(res);
          }, err => reject(err))
        } else {
          resolve(promise)
        }
      });
    });
  }
}
```

