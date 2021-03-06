# XSS和CSRF攻击

### 一. 什么是XSS攻击

XSS（Cross Site Scripting）为了区分开CSS，故简称XSS。XSS是指黑客往HTML文件中或者DOM中注入恶意脚本，从而在用户浏览器页面时利用注入的恶意脚本对用户实施攻击的一种手段。

往HTML文件中注入恶意代码的方式有很多，那么恶意脚本可以做哪些事情？

* 窃取Cookie信息
* 监听用户行为
* 修改DOM
* 在页面内生成浮窗广告
* ...等

#### 1.1 恶意脚本是怎么注入的

主要有**存储型XSS攻击**、**反射型XSS攻击**和**基于DOM的XSS攻击**三种方式来注入恶意脚本。

#### 1.2 存储型XSS攻击

其大致经过如下步骤：

* 黑客利用站点漏洞将一段恶意 JavaScript 代码提交到网站的数据库中；
* 后用户向网站请求包含了恶意 JavaScript 脚本的页面；
* 当用户浏览该页面的时候，恶意脚本就会将用户的 Cookie 信息等数据上传到服务器

#### 1.3 反射型XSS攻击

在一个反射型 XSS 攻击过程中，恶意 JavaScript 脚本属于用户发送给网站请求中的一部分，随后网站又把恶意 JavaScript 脚本返回给用户。当恶意 JavaScript 脚本在用户页面中被执行时，黑客就可以利用该脚本做一些恶意操作。

需要注意的是，**Web 服务器不会存储反射型 XSS 攻击的恶意脚本，这是和存储型XSS 攻击不同的地方**。

#### 1.4 基于DOM的XSS攻击

基于 DOM 的 XSS 攻击是不牵涉到页面 Web 服务器的。

具体来讲，黑客通过各种手段将恶意脚本注入用户的页面中，比如通过网络劫持在页面传输过程中修改 HTML 页面的内容，这种劫持类型很多，有通过 WiFi 路由器劫持的，有通过本地恶意软件来劫持的，它们的共同点是在 Web 资源传输过程或者在用户使用页面的过程中修改 Web 页面的数据。

#### 1.5 如何阻止XSS攻击

存储型 XSS 攻击和反射型 XSS 攻击都是需要经过 Web 服务器来处理的，因此可以认为这两种类型的漏洞是服务端的安全漏洞。

基于 DOM 的 XSS 攻击全部都是在浏览器端完成的，因此基于 DOM 的 XSS 攻击是属于前端的安全漏洞。



但无论何种XSS攻击，他们都是通过往浏览器中注入恶意脚本，再通过恶意脚本将用户信息发送至黑客的服务器上的。

所以要阻止XSS攻击，就可以通过阻止恶意JavaScript脚本的注入和恶意消息的发送来实现。

#### 1.5.1 服务器对输入脚本进行过滤或转码

不管是反射型还是存储型XSS攻击，都可以在服务器端将一些关键的字符进行转码，就可以过滤掉<script>标签。这样在HTML文本中识别不出来恶意的<script>脚本插入，就无法执行恶意代码。

#### 1.5.2 充分利用CSP

CSP 的实质就是白名单制度，开发者明确告诉客户端，哪些外部资源可以加载和执行，等同于提供白名单。

**CSP的使用**

* 在HTTP header中使用（首选）

  * ```javascript
    "Content-Security-Policy:" 策略
    "Content-Security-Policy-Report-Only:" 策略
    ```

* 在HTML上使用

  * ```javascript
    <meta http-equiv="content-security-policy" content="策略">
    <meta http-equiv="content-security-policy-report-only" content="策略">
    ```

CSP有如下几个功能：

* 限制加载其他域下的资源文件，这样即使黑客插入一个JavaScript文件，这个JavaScript也是无法被加载的
* 禁止向第三方域提交数据，这样用户数据也不会外泄
* 禁止执行内联脚本和未授权的脚本
* 还提供了上报机制，这样可以帮助我们尽快发现有哪些XSS攻击

#### 1.5.3 使用HttpOnly属性

很多XSS攻击都是来盗Cookie的，因此可以通过HttpOnly属性来保护Cookie的安全。

服务器可以将某些Cookie设置为HttpOnly标志，HttpOnly是服务器通过HTTP响应头来设置的。

使用HttpOnly标记的Cookie只能使用在HTTP请求过程中，无法通过JavaScript来读取这段Cookie。

### 二. 什么是CSRF攻击

（陌生连接不要点）

CSRF（Cross-site request forgery，跨站请求伪造）简单来讲，**CSRF 攻击就是黑客利用了用户的登录状态，并通过第三方的站点来做一些坏事**。

#### 2.1 黑客有三种方式去实施CSRF攻击：

* **自动发起get请求**。比如，黑客页面上有一个`<img src="https://xxxxx.xx?user=xxx&number=xxx">`代码。那么当页面被加载时，浏览器就会自动发起img资源请求。如果服务器没有对请求做判断的话，服务器就会认为这是一个正常的http请求。
* **自动发起post请求**。比如黑客页面有一个隐藏的表单，那么写入`<script> document.getElementById('btn').submit();</script>`，页面就会自动发起post请求。
* **引诱用户点击链接**。

#### 2.2 CSRF攻击的三个必要条件

* 目标站点一点要有CSRF漏洞
* 用户要登陆过目标站点，并且在浏览器上保持有该站点的登陆状态
* 需要用户打开一个第三方站点，可以是黑客的站点，也可以是一些论坛

#### 2.3 如何防护CSRF攻击

主要的CSRF防护手段是提升服务器的安全性。

* **充分利用好Cookie的SameSite属性**。为了防止CSRF攻击，最好能实现从第三方站点发送请求时禁止Cookie的发送。

  > 浏览器通过不同来源发送HTTP请求时，要区别：
  >
  > * 如果从第三方站点发起的请求没那么需要浏览器禁止发送某些关键Cookie数据到服务器
  > * 如果是同一个站点发起的请求，就需要保证Cookie数据正常发送
  >
  > ----
  >
  > **SameSite**正是为了解决这个问题的。在HTTP响应头中，通过set-cookie字段设置cookie时，可以带上SameSite选项。
  >
  > SameSite通常有三个值
  >
  > * Strict：浏览器会完全禁止第三方的Cookie。
  > * Lax：第三方站点的连接打开和提交Get方式会携带Cookie，但在第三方站点使用Post、通过img、iframe等加载的url都不会携带cookie。
  > * None：任何情况下都会发送Cookie

* **验证请求的来源站点**

  > 那么如何判断请求是否来自第三方站点呢？
  >
  > 看！HTTP请求头中的**Referer**和**Origin**属性。
  >
  > * Referer在HTTP请求头中记录了该HTTP请求的来源地址（可选可不选）。但在服务器验证请求头Referer不太可靠，于是有了Origin属性。
  > * 一些比如通过XMLHttpRequest、Fetch发起跨站请求或通过Post方式发送请求时，都会带上Origin属性。Origin属性只包含了域名信息，没有包含具体的URL路径。这是Origin和Referer的一个主要区别。Origin之所以不包含详细的路径信息是出于安全考虑。
  >
  > 因此，服务器的策略是优先判断Origin，如果没有Origin，再判断是否使用Referer。

* **CSRF Token**

  > 用户打开页面的时候，服务器需要给这个用户生成一个Token，Token其实就是服务器生成的字符串，然后将该字符串植入到返回的页面中。之后页面发起的请求都会带上Token，然后服务器会验证该Token是否合法。第三方网站是无法获取到的。

