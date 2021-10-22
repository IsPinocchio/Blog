### 1. instanceof的模拟实现

用来判断对象obj是不是构造函数ctor的实例对象

**原理**：通过obj的原型链查找obj的**proto**属性是否存在ctor的prototype

```javascript
function myinstanceof(obj, ctor) {
	if(typeof obj != 'object' && obj == null) return false;
  let proto = Object.getPrototypeof(obj);
  let pt = ctor.prototype;
  while(true) {
    if(proto == pt) return true;
    if(proto == null) return false;
    proto = Object.getPrototypeof(proto);
  }
}
```

### 2. new的模拟实现

遵循：

* 可访问`构造函数`里的属性
* 可访问`构造函数.prototype`里的属性
* 如果构造函数有返回值，就返回这个对象，否则就返回内部创建的对象

```javascript
function mynew(ctor, ...args) {
	let obj = new Object();
  obj.__proto__ = ctor.prototype;
  let rec = ctor.apply(obj, args);
  return typeof rec === 'object' ? rec : obj;
}
```

### 3. call、apply、bind的模拟实现

call和apply主要都遵循三个步骤：

* 给目标对象绑定这个方法
* 目标对象调用这个方法
* 目标对象删除这个方法

**call**

```javascript
Function.prototype.mycall = function(context, ...args) {
  context = context || window;
  context.fn = this;
  let res = context.fn(...args);
  delete context.fn;
  return res;
}
```

**apply**

```javascript
Function.prototype.myapply = function(context, args) {
	context = context || window;
  context.fn = this;
  let res;
  if(!args) { res = context.fn();}
  else { res = context.fn(...args);}
  delete context.fn;
  return res;
}
```

**bind**

bind遵循：

* 他返回的是一个绑定目标对象后的函数
* 返回后的函数可以作为构造函数创建实例，并且该实例对象的构造函数是原函数
* 返回的函数和在创建实例对象时传递的参数可以承接

```javascript
Function.prototype.mybind = function(context, ...args) {
  if(typeof this != 'function') throw new TypeError(this, 'is not a function');
  let _this = this;
  return function func(...a) {
		return this instanceof func ? new _this(...a) : _this.apply(context, [...args, ...a]);
  }
}
```

### 4. 浅拷贝和深拷贝的模拟实现

**浅拷贝**

```javascript
function shallowClone(target) {
  if(typeof target == 'object' && typeof target != null) {
    let res = Array.isArray(target) ? [] : {};
    for(let prop in target) {
      if(target.hasOwnProperty(prop)) {
				res[prop] = target[prop];
      }
    }
    return res;
  } else {
    return target;
  }
}
```

**浅拷贝**

```javascript
const isobject = (target) => (typeof target === 'object' || typeof target === 'function') && target != null;

function deepClone(target, map = new Map()) {
  if(map.get(target)) return target;
  if(isobject(target)) {
    map.set(target, true);
    const res = Array.isArray() ? [] : {};
    for(let prop in target) {
			if(target.hasOwnProperty(prop)) {
        res[prop] = deepClone(target[prop], map);
      }
    }
    return res;
  } else {
    return target;
  }
}

// 
function deepClone(target) {
  if(typeof target !== 'object') return;
  const res = Array.isArray(target) ? [] : {};
  for(let prop in target) {
		if(target.hasOwnProperty(prop)) {
      res[prop] = typeof target[prop] === 'object' ? deepClone(target[prop]) : target[prop];
    }
  }
  return res;
}
```

### 5. Array.prototype.filter

过滤出满足条件的元素，并返回一个满足条件的新数组

```javascript
Array.prototype.myfilter = function (cb) {
  let newArr = [];
  for (let i = 0; i < this.length; i++) {
    if (cb(this[i])) {
      newArr.push(this[i]);
    }
  }
  return newArr
}
```

### 6. Array.prototype.map

返回一个新数组，数组中的元素为原始数组元素调用函数处理后的值。

```javascript
Array.prototype.mymap = function (cb) {
  let newArr = [];
  for (let i = 0; i < this.length; i++) {
    newArr.push(cb(this[i]));
  }
  return newArr;
}
```

### 7. Array.prototype.reduce

接收一个函数作为累加器，数组中的每个值（从左到右）开始迭代，最终计算为一个值。

```javascript
Array.prototype.myreduce = function (cb, initVal) {
  for (let i = 0; i < this.length; i++) {
    initVal = cb(initVal, this[i]);
  }
  return initVal;
}
```

