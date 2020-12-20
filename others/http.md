### 状态码

通常状态码满足`status >= 200 && status < 300 || status == 304`返回数据，反之报出异常。
* 200 OK
* 201 Created
* 202 Accepted
* 204 No Content
* 301 Moved Permanently
* 302 Moved Temporarily
* 304 Not Modified
* 400 Bad Request
* 401 Unauthorized
* 403 Forbidden
* 404 Not Found
* 500 Internal Server Error
* 501 Not Implemented
* 502 Bad Gateway
* 503 Service Unavailable

## HTTP/2

主要目标是通过开启完整的请求和响应多路复用来减少延迟，通过有效压缩HTTP标头字段来最小化协议开销，并增加对请求优先级和服务器推送的支持