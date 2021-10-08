# Ajax与Fetch

Ajax全称Asynchronnous JavaScript and XML，即异步javas与XML。JavaScript基于**XMLHttpRequest**对象与服务端进行交互。Ajax拥有**异步**特质，可以在页面不刷新的情况下，通过交互数据在页面上进行局部更新。

### 一. Ajax几种请求方式

常用：post、get、delete

不常用copy、head、link

#### 1.1 区别

* 代码上
  * get通过url传递参数
  * post设置请求头，规定请求数据类型
* 使用上
  * post比get安全，因为post参数是在请求体上，get参数在url上
  * get传输速度比post快
  * post传输大文件理论上没有限制，get传输文件大小大概在7-8k左右
  * get获取数据，post上传数据

### 二. Ajax使用步骤

```javascript
//创建XMLHttpRequest对象
var xhr = new XMLHttpRequest();
//规定请求类型、url、是否异步处理请求、user、password
xhr.open('get', 'http://127.0.0.1:3000');
//发送信息至服务器时内容类型，服务端根据指定 Content-type进行请求主体的解析
xhr.setRequestHeader("Content-type", "text/html")
//接受服务器响应数据
xhr.onreadystatechange = function () {
  if (xhr.readyState === 4 && xhr.status === 200) {
    //当返回成功需要做什么
    console.log(xhr.responseText)
    ...
  } 
};
//发送请求
xhr.send()
```

### 三. XMLHttpRequest属性解读

**readyState**

只读属性，readyState属性记录了ajax调用过程中所有可能的状态

| **readState**   | **对应常量**         | **描述**                                                     |
| --------------- | -------------------- | ------------------------------------------------------------ |
| 0（未初始化）   | xhr.UNSENT           | 请求已建立，但未初始化（此时未调用open方法）                 |
| 1（初始化）     | xhr.OPENED           | 请求已建立，但未发送（已调用open方法，但未调用send方法）     |
| 2（发送数据）   | xhr.HEADERS_RECEIVED | 请求已发送（send方法已调用，已收到响应头）                   |
| 3（数据传送中） | xhr.LOADING          | 请求处理中，因响应内容不全，这时通过responseText等获取数据会出现错误 |
| 4（完成）       | xhr.DONE             | 数据接收完毕，此时可以通过responseText等获取完整数据         |

### 四. Fetch

`fetch`API是基于Promise进行设计的，相比于原生xhr写法上更加的方便和简单，更符合关注点分离的原则，不会将所有配置和状态混淆在一个对象里。

```javascript
// 写法一：
fetch(url)
.then(response => {
  if(response.ok) {
		return res.json();
  }
})
.then(data => {...})
.catch(err => {...})

//写法二：
const fetchSend = async (url) => {
  try {
		const response = await fetch(url);
    if(response.ok) {
			return response.json()
    } 
  } catch(e) {

  }
}
```

