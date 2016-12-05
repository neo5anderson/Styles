# RESTful API 设计规范指南

## 目录
## RESTful

一种软件架构风格，提供了一组设计原则和约束条件。它主要用于客户端和服务器交互类的软件。基于这个风格设计的软件可以更简洁，更有层次，更易于实现缓存等机制。

### 原则条件

REST 是一组架构约束条件和原则。满足约束条件和原则的应用程序或设计就是 RESTful。

Web 应用程序最重要的 REST 原则是，客户端和服务器之间的交互在请求之间是无状态。从客户端到服务器的每个请求都必须包含理解请求所必须的信息。如果服务器在请求之间的任何时间点重启，客户端不会得到通知。此外，无状态请求可以由任何可用服务器应答，十分适合云计算之类的环境。客户端可以缓存数据已改进性能。

在服务端，应用程序状态和功能可分为各种资源。每个资源都使用 URL 得到一个唯一的地址。所有资源都共享统一的接口，以便在客户端和服务器之间传输状态。使用的是标准的 HTTP 方法，比如 `GET`、`PUT`、`POST` 和 `DELETE`。

### 分层系统

这是另外一个重要的原则，表示着组件无法了解与之交互的中间层以外的组件。通过将系统知识限制在单个层次内，限制整个系统的复杂性，促进底层的独立性。

当 REST 框架的约束条件作为一个整体应用时，将生成一个可以扩展到大量客户端的应用程序。它还降低了客户端和服务器之间的交互延迟。统一界面简化了整个系统架构，改进了子系统之间交互的可见性。

在 RPC 样式的架构中，关注点在于方法；而 REST 关注点在于资源，使用标准方法检索并操作信息片段。

+ Web 浏览器客户端作为 GUI 的前端，表示层生成 HTML
+ 业务路基层充当表示层和数据访问层之间的数据交换的中间层
+ 数据访问层与数据存储层交互，支持 DAO 设计模式或对象关系映射解决方案
+ 数据存储层包括数据库系统、LDAP 服务器、文件系统和信息系统

## URI

URI 表示资源，资源一般对应服务器端领域模型中的实体类。

### URI 规范

+ 不用大写；
+ 用中杠 `-` 不用下杠 `_`；
+ 参数列表要 `encode`；
+ URI 中的名词表示资源集合，使用复数形式。

### 资源集合 vs 单个资源

URI 表示资源的两种方式：资源集合、单个资源。

资源集合：

```
// 所有动物园
/zoos

// id 为 1 的动物园中的所有动物
/zoos/1/animals
```

单个资源：

```
// id 为 1 的动物园
/zoos/1

// id 为 1、2、3 的动物园
/zoos/1;2;3
```

### 避免层级过深的 URI

`/` 在 URL 中表达层级，用于 **按实体关联关系进行对象导航**，一般根据 id 导航。

过深的导航容易导致 URL 膨胀，不易维护，如 `GET /zoos/1/areas/3/animals/4` ，尽量使用查询参数代替路径中的实体导航，如 `GET /animals?zoo=1&area=3`；

### 对 Composite 资源的访问

服务器端的组合实体必须在 URI 中通过父实体的 id 导航访问。

组合实体不是 First-Class 的实体，它的生命周期完全依赖父实体，无法独立存在，在实现上通常是对数据库表中某些列的抽象，不直接对应表，也无 id。一个常见的例子是 User — Address，Address 是对 User 表中 zipCode、country、city 三个字段的简单抽象，无法独立于 User 存在。必须通过 User 索引到 Address：`GET /user/1/addresses`

## Request

### HTTP 方法

通过标准 HTTP 方法对资源 CRUD：

GET：查询

```
GET /zoos
GET /zoos/1
GET /zoos/1/employees
```

POST：创建单个资源。 **POST一般向“资源集合”型 URI 发起**

```
// 新增动物
POST /animals

// 为 id 为 1 的动物园雇佣员工
POST /zoos/1/employees
```

PUT：更新单个资源（全量），客户端提供完整的更新资源。与之对应的是 PATCH，PATCH 负责部分更新，客户端提供要更新的那些字段。 **PUT/PATCH 一般向“单个资源”型 URI 发起**

```
PUT /animals/1
PUT /zoos/1
```

DELETE：删除

```
DELETE /zoos/1/employees/2
DELETE /zoos/1/employees/2;4;5
// 删除 id 为 1 的动物园内的所有动物
DELETE /zoos/1/animals
```

### 安全性和幂等性

1. 安全性：不会改变资源状态，可以理解为只读的；
2. 幂等性：执行 1 次和执行 N 次，对资源状态改变的效果是等价的。

. | 安全性 | 幂等性
--|--------|------
GET | √ | √
POST | × | ×
PUT | × | √
DELETE | × | √

安全性和幂等性均不保证反复请求能拿到相同的 `response`。以 `DELETE` 为例，第一次 `DELETE` 返回 200 表示删除成功，第二次返回 404 提示资源不存在，这是允许的。

### 复杂查询

查询可以捎带以下参数：

. | 示例 | 备注
--|--|--
过滤条件 | `?type=1&age=16` | 允许冗余，如 `/zoos/1` 与 `/zoos?id=1` |
排序 | `?sort=age,desc` | -
投影 | `?whitelist=id,name,email` | -
分页 | `?limit=10&offset=3` | -

### 书签

经常使用的、复杂的查询标签化，降低维护成本。

```
GET /trades?status=closed&sort=created,desc
```

快捷方式：

```
GET /trades#recently-closed
```

或者

```
GET /trades/recently-closed
```

### 格式化

只用以下 3 种 body format：

Content-Type: application/json

```
POST /v1/animal HTTP/1.1
Host: api.example.org
Accept: application/json
Content-Type: application/json
Content-Length: 24

{   
  "name": "Gir",
  "animalType": "12"
}
```

Content-Type: application/x-www-form-urlencoded

```
POST /login HTTP/1.1
Host: example.com
Content-Length: 31
Accept: text/html
Content-Type: application/x-www-form-urlencoded

username=root&password=Zion0101
```

Content-Type: multipart/form-data; boundary=—-RANDOM_jDMUxq4Ot5

### Content Negotiation

资源可以有多种表示方式，如 json、xml、pdf、excel 等等，客户端可以指定自己期望的格式，通常有两种方式：

1. http-header Accept ：

```
Accept:application/xml;q=0.6,application/atom+xml;q=1.0
```

q为各项格式的偏好程度

2. URL 中加文件后缀：`/zoo/1.json`

## Response

1. 不要包装，response 的 body 直接就是数据，不要做多余的包装。错误示例：

```
{
    "success":true,
    "data":{"id":1,"name":"xiaotuan"},
}
```

2. HTTP 方法成功处理后的数据格式：

· | response 格式
--|--|--
GET | 单个对象、集合
POST | 新增成功的对象
PUT/PATCH | 更新成功的对象
DELETE | 空

3. json 格式的约定：

  + 时间用长整形毫秒时间戳，客户端自己按需解析
  + 不传 `null` 字段

### 分页 response

```
{
    "paging":{"limit":10,"offset":0,"total":729},
    "data":[{},{},{}...]
}
```

## 错误处理

1. 不要发生了错误但给 2xx 响应，客户端可能会缓存成功的 http 请求；
2. 正确设置 http 状态码，不要自定义；
3. Response body 提供错误代码（日志/问题追查）和描述文本（展示给用户）。

对第三点的实现稍微多说一点：

Java 服务器端一般用异常表示 RESTful API 的错误。API 可能抛出两类异常：业务异常和非业务异常。 

+ 业务异常，由自己的业务代码抛出，表示一个用例的前置条件不满足、业务规则冲突等，比如参数校验不通过、权限校验失败。
+ 非业务类异常，表示不在预期内的问题，通常由类库、框架抛出，或由于自己的代码逻辑错误导致，比如数据库连接失败、空指针异常、除 0 错误等等。

业务类异常必须提供 2 种信息：

1. 如果抛出该类异常，HTTP 响应状态码应该设成什么；
2. 异常的文本描述；

在 Controller 层使用统一的异常拦截器：

1. 设置 HTTP 响应状态码：对业务类异常，用它指定的 HTTP code；对非业务类异常，统一500；
2. Response Body 的错误码：异常类名
3. Response Body 的错误描述：对业务类异常，用它指定的错误文本；对非业务类异常，线上可以统一文案如“服务器端错误，请稍后再试”，开发或测试环境中用异常的 stacktrace，服务器端提供该行为的开关。

常用的http状态码及使用场景：

状态码 | 使用场景
--|--
400 bad request | 常用在参数校验
401 unauthorized | 未经验证的用户，常见于未登录。如果经过验证后依然没权限，应该 403（即 authentication 和 authorization 的区别）。
403 forbidden | 无权限
404 not found | 资源不存在
500 internal server error | 非业务类异常
503 service unavaliable | 由容器抛出，自己的代码不要抛这个异常

## 服务型资源

除了资源简单的 CRUD，服务器端经常还会提供其他服务，这些服务无法直接用上面提到的 URI 映射。如：

1. 按关键字搜索；
2. 计算地球上两点间的距离；
3. 批量向用户推送消息

可以把这些服务看成资源，计算的结果是资源的presentation，按服务属性选择合适的HTTP方法。

例：

```
// 搜索
GET /search?q=filter?category=file
GET /distance-calc?lats=47.480&lngs=-122.389&late=37.108&lnge=-122.448
POST /batch-publish-msg
[{"from":0,"to":1,"text":"abc"},{},{}...]
```

## 异步任务

对耗时的异步任务，服务器端接受客户端传递的参数后，应返回创建成功的任务资源，其中包含了任务的执行状态。客户端可以轮训该任务获得最新的执行进度。

```
// 提交任务：
POST /batch-publish-msg
[{"from":0,"to":1,"text":"abc"},{},{}...]

// 返回：
{"taskId":3,"createBy":"Anonymous","status":"running"}

GET /task/3
{"taskId":3,"createBy":"Anonymous","status":"success"}
```

如果任务的执行状态包括较多信息，可以把“执行状态”抽象成 组合资源 ，客户端查询该状态资源了解任务的执行情况。

```
// 提交任务：
POST /batch-publish-msg
[{"from":0,"to":1,"text":"abc"},{},{}...]

// 返回：
{"taskId":3,"createBy":"Anonymous"}

GET /task/3/status
{"progress":"50%","total":18,"success":8,"fail":1}
```

## API 的演进

### 版本

常见的三种方式：

1. 在uri中放版本信息： `GET /v1/users/1`
2. Accept Header： `Accept: application/json+v1`
3. 自定义 Header： `X-Api-Version: 1`

用第一种，虽然没有那么优雅，但最明显最方便。

### URI 失效

随着系统发展，总有一些 API 失效或者迁移，对失效的 API，返回 404 not found 或 410 gone；对迁移的 API，返回 301 重定向。

