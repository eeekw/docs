# CSRF

## 简介

跨站点请求伪造：通过盗用用户的身份信息，以你的名义向第三方网站发起恶意请求。解决思路，校验发起请求不为第三方。

在一个域中访问第三方域的资源时或发起第三方域的请求时，会发送第三方域的cookie

## 防御

* 验证码
* 验证HTTP Referer字段：Referer字段是浏览器添加的指向当前HTTP请示发送所在页面的URL。Referer可伪造
* Anti CSRF Token: 请求中增加token参数，token是随机的，不可预测。注意token的保密性和随机性，需保证不存在XSS漏洞。
* 设置Set-Cookie: key=value; SameSite=Strict

## 参与链接

[解释samesite cookie](https://web.dev/samesite-cookies-explained/)
[使用samesite cookie](https://web.dev/samesite-cookie-recipes/)