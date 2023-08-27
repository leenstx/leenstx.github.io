---
title: RESTful API 实践
date: 2023-07-12
categories: ['技术']
alias: restful_docs
draft: false
---

### 一、 RESTFul API 风格

广泛使用的 API 风格分别为 RESTful 和 GraphQL，像 [Github](https://docs.github.com/en) 就分别实现了一套。二者各有优劣，不过相对于 GraphQL，RESTful 定义的 API 职责更加清晰，后期也容易维护。

### 1、命名风格

- 必须-全部小写
- 必须-多单词使用 `-` 连接
- 应该-使用单词名词或过去分词属性
- 必须-使用单词单数形式

例：

```
https://api.com/api/v1/user

https://api.com/api/v1/user/123456/disabled

https://api.com/api/v1/user/123456/class-name
```

### 2、HTTP 方法

### GET

表示从服务器获取资源，包括资源详情或资源列表信息

获取资源详情：/:resource/:id

```
GET https://api.com/api/v1/user/123456
```

获取资源列表：/:resource

```
GET https://api.com/api/v1/user
```

### POST

表示从创建资源

创建资源：/:resource

```
POST https://api.com/api/v1/user
```

### PUT

方法表示从修改资源信息
修改资源全部信息：/:resource/:id

```
PUT https://api.com/api/v1/user/123456
```

修改资源部分信息：/:resource/:id/part

```
PUT https://api.com/api/v1/user/123456/part
```

执行一个 action：/:resource/:id/:action

```
# 禁用 id 为 123456 的用户
PUT https://api.com/api/v1/user/123456/disabled
```

### DELETE

方法表示从修改资源信息
删除资源：/:resource/:id

```
DELETE https://api.com/api/v1/user/123456
```

### 3、API 版本

如果 API 需要支持版本，通常会在路径上标识，也可以在 域名上标识

```
https://api.com/api/v1/user/123456

https://v1.api.com/api/user/123456
```

无论有没有版本的需求建议直接加上，一般选择第一种即可

### 二、 通用项说明

### 1、请求头

| 字段 | 是否必传 | 说明 |
| --- | --- | --- |
| Content-Type | 是 | 请求的内容格式，一般来说都是 application/json |
| Authorization | 否 | 认证字段，除白名单接口外必传，值的格式为 Bearer <token> |

### 2、响应头

| 字段 | 是否必传 | 说明 |
| --- | --- | --- |
| Content-Type | 是 | 响应的内容格式，一般来说都是 application/json |
| Location | 否 | 新建资源的访问路径 |

### 3、时间

双方传递时间为标准的 UTC 时间
日期格式为 yyyy-MM-dd 即 2023-05-06
时间格式为 HH:mm:ss 即 18:00:00
时间日期格式 yyyy-MM-ddTHH:mm:ss 即 2023-05-06T18:00:00Z

### 4、分页

字段说明
请求传参：

| 字段 | 必传 | 类型 | 说明 |
| --- | --- | --- | --- |
| page | 否 默认 1 | number | 当前页数 |
| size | 否 默认 20 | number | 数据条数 |

如：

```
GET https://api.com/api/v1/user?page=1&size=10
```

返回字段

| 字段 | 类型 | 说明 |
| --- | --- | --- |
| page | number | 当前页数 |
| size | number | 数据条数 |
| list | array | 返回的实体数组 |
| total | number | 总数据数量 |
| pages | number | 总页数 |

如：

```
{
    "pages": 5,
    "page": 1,
    "size": 20,
    "total": 100,
    "list": []
}

```

### 5、筛选

参考: [Guide](https://github.com/Microsoft/api-guidelines/blob/vNext/Guidelines.md#972-operator-examples)
使用 Query 参数 关键字 filter
操作符

| 操作符 | 描述 |
| --- | --- |
| gt | 大于 |
| ge | 大于等于 |
| lt | 小于 |
| le | 小于等于 |
| eq | 等于 |
| ne | 不等于 |
| and | 并且 |
| or | 或者 |
| in | 属于 |
| lk (like) | 包含 |
| nl (not like) | 不包含 |

**注意: 为了简化接口这里我们规定 `filter` 中的各字段关系要么全 `and` 要么全是 `or`***复杂的 filter 如 and or 混杂，括号又层级嵌套，就非常不利于前后端解析*

例:
查询姓名包含小明的用户

```
GET /api/v1/user?filter=name:lk:小明

```

查询姓名包含小明并且年龄在18到30之间的用户

```
GET /api/v1/user?filter=name:lk:小明 and age:gt:18 and age:lt:30

```

### 6、排序

排序使用 sort 字段，多个字段使用 `,` 连接
升序: asc 默认，可不传
降序: desc
例：

```
GET /api/v1/user?filter=name:lk:小明 and age:gt:18 and age:lt:30&sort=id asc,name desc

```

### 三、 返回说明

### 1、返回要求

- HTTP Status 不可以全部返回 200，该接口应按照 HTTP 标准协议中的 Status 定义来返回
- 除了特殊接口外（如下载）一律返回 JSON 数据格式
- 返回的应该是 JSON 对象，而不是 JSON 数组，数组不容易扩展

例:

```
# 请求列表
HTTP/1.1 200
Content-Type: application/json
Date: Wed, 24 May 2023 08:14:00 GMT

{
  "list": [
    {
      "id": 24,
      "fileId": "1661191122577076224",
      "title": "新建模版",
      "url": "https://api.com/api/v1/file/1661191122577076224.docx",
      "fileType": ".docx"
    }
  ]
}

# 请求详情
HTTP/1.1 200
Content-Type: application/json
Date: Wed, 24 May 2023 08:14:00 GMT

{
    "id": 24,
    "fileId": "1661191122577076224",
    "title": "新建模版",
    "url": "https://api.com/api/v1/file/1661191122577076224.docx",
    "fileType": ".docx"
}

# 请求错误 Token 无效
HTTP/1.1 401
Content-Type: application/json;charset=UTF-8
Date: Thu, 18 May 2023 01:35:13 GMT

{
  "code": "Auth.InvalidToken",
  "message": "令牌无效"
}

```

### 2、错误码

错误返回建议采用以下格式

```
HTTP/1.1 400
Content-Type: application/json;charset=UTF-8
Date: Thu, 18 May 2023 01:35:13 GMT

{
  "code": "Arg.BadArgument",
  "message": "The parameter does not meet the validation rules",
  "target": "Root",
  "details": [
    {
      "code": "Arg.Size",
      "message": "The length must be between 1 and 50",
      "target": "username"
    },
    {
      "code": "Arg.NotNull",
      "message": "Must not be null",
      "target": "password"
    }
  ]
}

```

推荐公共错误码的定义

| 错误码 | HTTP状态码 | 中文 | 英文 |
| --- | --- | --- | --- |
| Arg.BadArgument | 400 | 传递的参数不满足校验规则 | The parameter does not meet the validation rules |
| Arg.InvalidJson | 400 | 不是合法的 JSON 格式，无法解析 | The JSON you provided was not well-formed |
| Arg.Max | 400 | 最大值为{1} | Max value is {1} |
| Arg.Min | 400 | 最小值{1} | Min value {1} |
| Arg.NotBlank | 400 | 不能为空 | Must not be null |
| Arg.NotEmpty | 400 | 不能为空 | Must not be null |
| Arg.NotNull | 400 | 不能为空 | Must not be null |
| Arg.Size | 400 | 字段长度必须在{2}到{1}之间 | The length must be between {2} and {1} |
| Auth.AccessDenied | 401 | 权限不足 | Access denied |
| Auth.InvalidToken | 401 | 令牌无效 | The token is invalid |
| Auth.LoginElsewhere | 401 | 您已在其他地方登录 | You have logged in elsewhere, the token has become invalid |
| Auth.NotAccessToken | 401 | 该令牌无资源访问权限 | The token does not have access permissions |
| Auth.NotBearerToken | 401 | 当前系统仅支持 ‘Bearer’ 类型的令牌认证 | The current system only supports Bearer token authentication |
| Auth.UserDisabled | 401 | 您的账户已到期或已被禁用 | User has been disabled |
| Filter.BadFilter | 400 | filter 字段不满足规则 | The filter parameter does not comply with the rules |
| Filter.OperatorNotSupport | 400 | 暂不支持{0}操作符 | The filter parameter does not support {0} operator |
| Filter.OperatorNotSupportApi | 400 | 此接口暂不支持{0}操作符 | The filter parameter does not support {0} operator in this API |
| Inter.Error | 500 | 服务器内部异常，请稍后再试或联系开发人员 | We encountered an internal error Please try again |

业务上的错误可以自定义错误码

### 四、API 接口文档

接口文档把接口解释清楚即可，以用户登录接口文档说明作为参考

---

**用户登录**

> 登录成功后会返回 accessToken 但是有过期机制。当任何请求的响应头包含 X-Refresh-Token-Required 且值为 true 的时候，客户端需要请求刷新 Token 的接口。该接口失败表明用户该重新登录了
> 

`POST` `/api/v1/login`

**请求：**

- 请求结构

```
POST /api/v1/login HTTP/1.1
Content-Type: application/json; charset=utf-8

{
    "username": "",
    "password": ""
}

```

- Header 参数

除了公共请求头外，无特殊参数

- Path 参数

无

- Query 参数

无

- Body 参数

| 字段 | 类型 | 必传 | 说明 |
| --- | --- | --- | --- |
| username | string | 是 | 用户名，登录名不是昵称 |
| password | string | 是 | 密码 |
- 响应头

除公共响应头外，无特殊参数

- 响应体

| 字段 | 类型 | 说明 |
| --- | --- | --- |
| accessToken | string | 用于后续请求认证 |
- 错误码

| 错误码 | HTTP状态码 | 中文 | 英文 |
| --- | --- | --- | --- |
| User.Disabled | 401 | 您的账户已到期或已被禁用 | User has been disabled |
| User.UsernameOrPasswordIncorrect | 401 | 用户名或密码错误 | The username or the password is incorrect |

**请求示例**

```
POST /api/v1/login HTTP/1.1
Content-Type: application/json; charset=utf-8

{
    "username": "admin",
    "password": "123456"
}

```

**响应示例**

```
HTTP/1.1 201
Content-Type: application/json
Date: Fri, 05 May 2023 08:03:40 GMT

{
    "accessToken": ""
}

```

---