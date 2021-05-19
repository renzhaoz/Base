# cookie 客户端缓存

## 简介

最初用于储存登录信息、个人偏好等其它数据.Web应用程序提供者都需要有办法把它们保存在客户端. 现在只是在客户端存储数据的一个选项.

HTTP cookie 通常也叫作 cookie,最初用于在客户端存储会话信息。这个规范要求服务器在响应.例如,下面是包含这个头部的一个 HTTP
HTTP 请求时,通过发送 Set-Cookie HTTP 头部包含会话信息:

```
HTTP/1.1 200 OK
Content-type: text/html
Set-Cookie: name=value
Other-header: other-header-value
```

等下次浏览器和服务器通信时将会cookie通过以下数据返回：

```
GET /index.jsl HTTP/1.1
Cookie: name=value
Other-header: other-header-value
```

## 起步API

- document.cookie 访问域名中所有的cookie以分好隔开,cookie的name属性和值是必须的

- expires 超时时间(Wdy, DD-Mon-YYYY HH:MM:SS GMT)toGMTString()获取

- domain 域名限制

- path 域名路径限制(/时所有页面显示)

- secure 只能在SSL下使用

- httpOnly 只能在http中传递  不能通过js访问

## 特性

- cookie名不区分大小写
```Set-Cookie: name=value;```

- cookie必须经过URL编码

- cookie会过期 过期时间设置为过去的时间会立即执行删除  不设置关闭浏览器时会清除
```Set-Cookie: name=value; expires=Mon, 22-Jan-07 07:10:24 GMT;```

- cookie与域名绑定, 只能在限制的域名内交换使用cookie数据,不会被其它域访问.
只能在.wrox.com域名下及子域(p2p.wrox.com)使用:
```Set-Cookie: name=value; expires=Mon, 22-Jan-07 07:10:24 GMT; domain=.wrox.com```

- cookie可以设置安全标志 设置之后只能在SSL安全连接的情况下才会发送cookie
如下代码将会在wrox.com及子域下的所有页面生效(path属性), 必须在https协议下(secure)
```Set-Cookie: name=value; domain=.wrox.com; path=/; secure```

- httpOnly 设置该字段后只能在http中传递？不能通过js访问？

- cookie 不能占用太多空间,各个浏览器都有自己的限制
以下限制可以满足大多数浏览器:
 1 不超过 300 个 cookie;
 2 每个 cookie 不超过 4096 字节;
 3 每个域不超过 20 个 cookie;
 4 每个域不超过 81 920 字节。
 5 每个域能设置的 cookie 总数也是受限的,但不同浏览器的限制不同。例如:
 6 最新版 IE 和 Edge 限制每个域不超过 50 个 cookie;
 7 最新版 Firefox 限制每个域不超过 150 个 cookie;
 8 最新版 Opera 限制每个域不超过 180 个 cookie;
 9 Safari 和 Chrome 对每个域的 cookie 数没有硬性限制

- cookie 个数超限 浏览器会清除旧的cookie
ie & opera 清理使用少的
firefox 随机

- cookie大小超限 则会被静默删除 无任何提示