# RESTful API 概述
## 基本概念
REST（Representational State Transfer）直译为表现层状态转移，首次是由 Roy Thomas Fielding 在他 2000 年的博士论文中提出。

REST 是一种描述网络中 client 和 server 之间的资源交互方式。

而 RESTful API 就是完全遵循 REST 方式的一套 API 设计规范，简单来说，通过 API 来描述资源的访问方式：

- 通过HTTP URL 描述访问什么资源
- 通过HTTP METHOD 描述对资源的交互方式
- 通过HTTP CODE 描述资源的交互结果

# 幂等性
幂等性(Idempotence)本身是一个数学概念，在 HTTP/1.1 规范中幂等性是指

> Methods can also have the property of “idempotence” in that (aside from error or expiration issues) the side-effects of N > 0 identical requests is the same as for a single request.
> 
> 如果某个方法调用一次或多次产生的副作用是相同的，那么这个方法具有幂等性。

比如在 HTTP 中使用 GET 获取某个资源，无论调用多少次，产生的额外效果都是从服务器获取资源，所以 GET 方式具有幂等性。

而 POST 方法用于在服务器上创建一个资源，由于最终创建的结果每次都是不同的，所以 POST 不具有幂等性。

但是 PUT 方法却是幂等的，因为每次调用产生的效果都是对资源进行更新。

## 安全方法
安全方法是指不修改资源的 HTTP 方法。譬如，当使用 GET 或者 HEAD 作为资源 URL，都必须不去改变资源的表现形式。

注意：安全方法并不是指服务器上的资源完全不变，而是指资源的表现形式。

比如 GET 方法导致数据上报，关联的一些数据记录等，他实际上是改变了服务器上的某些附加资源的，但是这并不会改变资源的表现形式。

## HTTP 方法特性概览
![](https://img-blog.csdnimg.cn/20191104232811959.png)

# RESTful API设计规则
## HTTP URL
HTTP URL 只用于描述访问的资源，而不应该包含对资源的交互方式。HTTP URL 的最佳实践：

- 将API部署在专用的域名上，如 `api.example.com`
- 不使用任何大写字符
- 不使用下划线 `_`，可使用中划线 `-` 代替来分割单词
- 参数列表要进行 URL 编码
- 不应该出现描述资源交互方式的动词，应尽量使用描述资源的名词
- URI 中的名词表示资源集合，应该使用复数形式
- 应该避免资源层级太深，可以适当使用参数减少层级
- 使用斜杆 `/` 表示资源之间的层级关系
- 在URI的末尾避免使用 `/`
- 避免在 URI 中使用文件扩展名
- 如果有必要，需要在 URI 中添加版本号标识

相应的示例如下：
```
// 下面列出几个错误案例以及改进方法
/userPost        // 不应使用大写字母
-> /user-posts

/user/post       // 描述资源集合应该用复数 
-> /users/posts

/users/addPost   // 应避免使用动词
-> POST /users/posts

/users/posts/    // 应移除末尾的 /
-> /users/posts

/users/posts/picture.gif  // 应移除文件扩展名 .gif
-> /users/posts/picture

/users/recent_posts       // 使用 - 代替 _
-> /users/recent-posts

/users/posts?title=up up  // 参数使用 URL 编码
-> /users/posts?name=up%20up

/users/posts/games/picture/today  // 应避免资源层级太深
-> /users/posts/picture?type=game&time=today 

/users/learning-posts     // 应添加版本标识
-> /v1/users/learning-posts
```

补充说明：

- /users/search 像这个 URI，虽然包含动词 search，但是 search 并没有描述对资源的交互方式，所以这种 URI 也是可以的
- /users/posts, /user-posts 这样的两个 URI，第一个描述了层级关系，第二个没有，但是两个 URI 仍然能够很明确的表示资源，并没有谁最好的说法，可以结合所在团队的命名习惯等来选取

## HTTP METHOD
严格使用下面的 HTTP 方法来描述资源 CURD 操作方式，应该尽量避免在 POST 方式中删除资源等违背直觉的操作：

- GET: 查询资源
- POST: 创建资源
- PUT: 更新资源
- DELETE: 删除资源

对于一些 GET 中一次性获取大量资源的情况下，比如获取一万个指定资源的情况，可能会出现 HTTP URL 超过长度限制，这个时候可以分批获取。

虽然新版的 HTTP 标准支持在 GET 请求中传送 body，但是一些网络服务器并不能很好的支持，应该慎重使用。

另外对于复杂的资源获取，应该提供通用的资源筛选、排序、分页、字段选择等功能支持，并统一参数规范。例如：

```
// 过滤:
GET /cars?color=red // 返回红色汽车列表
GET /cars?seats<=2  // 返回最多 2 个座位的汽车列表

// 排序：
GET /cars?sort=-manufactorer,+model

// 字段选择：
GET /cars?fields=manufacturer,model,id,color

// 分页：
GET /cars?offset=10&limit=5
```

## 重写 HTTP 方法

一些 HTTP 客户端只支持 GET 和 POST 请求。为了能够加强这些客户端的访问能力，API 需要能够覆盖 HTTP 方法。

尽管这里没有任何强制的标准，但流行的做法是 API 会接收一个请求头 X-HTTP-Method-Override，它的值可以是 PUT、PATCH 或者 DELETE 三者之一。

注意，用来重写 HTTP 方法的 header 只能在 POST 请求中被接受。GET 请求永远不能修改服务器上的数据。

## HTTP RESPONSE
关于HTTP请求的返回值，有以下几个最佳实践:

- 采用格式化返回数据，推荐 json
- 充分利用 HTTP 状态码来描述错误类型
- 规范返回数据的统一格式，以及细分错误类型和提示
- 应该尽量返回有用的错误信息和提示，唯一的错误码

常用的 HTTP 状态码：

- 200 OK – 对成功的 GET、PUT、PATCH 或 DELETE 操作进行响应，也可以被用在不创建新资源的 POST 操作上
- 201 Created – 对创建新资源的 POST 操作进行响应。应该带着指向新资源地址的 Location header
- 204 No Content – 对不会返回响应体的成功请求进行响应（比如DELETE请求）
- 304 Not Modified – HTTP 缓存 header 生效的时候用
- 400 Bad Request – 请求异常，比如请求中的 body 无法解析
- 401 Unauthorized – 没有进行认证或者认证非法。当 API 通过浏览器访问的时候，可以用来弹出一个认证对话框
- 403 Forbidden – 当认证成功，但是认证过的用户没有访问资源的权限
- 404 Not Found – 当一个不存在的资源被请求
- 405 Method Not Allowed – 所请求的 HTTP 方法不允许当前认证用户访问
- 410 Gone – 表示当前请求的资源不再可用。当调用老版本 API 的时候很有用
- 415 Unsupported Media Type – 如果请求中的内容类型是错误的
- 422 Unprocessable Entity – 用来表示校验错误
- 429 Too Many Requests – 由于请求频次达到上限而被拒绝访问