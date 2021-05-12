---
title: "OpenApi3.0"
date: 2018-03-27T21:46:00+08:00
draft: true
---
OpenAPI3.0是OpenApi规范的最新版本,[openapi3完整语法参考](https://github.com/OAI/OpenAPI-Specification/blob/master/versions/3.0.0.md),现在大多数应用只支持2.0规范，可以在这里查看2.0


### 文档结构
整个Openapi文档由下列字段组成，openapi,info,paths对象想都是必选，其他字段都是可选。

| 字段名 | 类型 | 描述 |
| --- | :-: | :-: |
| openapi| `string` |  描述openapi使用的版本 |
| info| Info Object |  文档信息 |
| servers | Server Object |  用于描述接口地址 |
| paths | Paths Object |  api的路径描述 |
| components | Components Object | 定义模型，对象重用  |
| security | Security Requirement] |   |
| tags |  Tag Object] | 定义标签，可以将不同路径下同一标签的api集合显示  |
| externalDocs | External Documentation Object |   外部文档|

### 定义文档信息

```
openapi: 3.0.0
info:
  description: 文档描述支持部分markdown语法，例如
   ![百度](http://www.baidu.com) 
  version: "1.0.0"
  title: OpenApi 3.0
  termsOfService: 'http://swagger.io/terms/'
  contact:
    email: apiteam@swagger.io
  license:
    name: Apache 2.0
    url: 'http://www.apache.org/licenses/LICENSE-2.0.html

```

####OpenAPI规范的版本号
在2.0我们使用`swagger: "2.0"`描述API文档所使用的OpenApi版本，在3.0之后将改为`openapi: 3.0.0`

####API描述信息
`info`对象用于描述文档当信息。

字段名 | 类型 | 描述
---|:---:|---
title | `string` | 文档标题，必填字段
description | `string` | 文档描述，这个字段支持CommonMark语法
termsOfService | `string` | API的服务条款的网址。 必须采用URL格式。
contact | Contact Object | 文档作者的联系方式
license | License Object | API的许可证信息。
version | `string` |  文档版本，必填字段

`Contact`

字段名 | 类型 | 描述
---|:---:|---
name | `string` | 联系人名
url | `string` | 联系人网址
email | `string` | 联系人的邮件地址

`license`

字段名 | 类型 | 描述
---|:---:|---
name | `string` | 必填字段，许可名称
url | `string` | 用于该API的许可证的URL。 必须采用URL格式

#### 定义服务器地址
`servers`关键词是3.0新增特性，通过servers关键词我们可以定义多个服务器地址方便切换测试环境和生成环境[更多参考](https://swagger.io/docs/specification/api-host-and-base-path/)

![](media/15222430425205.jpg)


```
servers:
  - url: http://api.example.com/v1
    description: Optional server description, e.g. Main (production) server
  - url: http://staging-api.example.com
    description: Optional server description, e.g. Internal staging server for testing
        
```

### 定义API路径

Field Name | Type | Description
---|:---:|---
$ref | `string` | 
summary| `string` | 
description | `string` | 
get | Operation Object | 
put | Operation Object | 
post | Operation Object | 
delete | Operation Object | 
options | Operation Object |
head | Operation Object | 
patch | Operation Object |
trace | Operation Object | 
servers | Server Object | 
parameters | Parameter Object| 

Field Name | Type | Description
---|:---:|---
tags | [`string`] | 
summary | `string` | 
description | `string` | 
externalDocs | [External Documentation Object] | 
operationId | `string` | 
parameters | Parameter Object | 
requestBody | Request Body Object| .
responses | Responses Object | **REQUIRED**. callbacks | Map| 
deprecated | `boolean` | 
security | Security Requirement Object | 
servers | Server Object |

paths对象用于定义api中的各个路径，一份openap文档必须有一个paths对象，最简单的例子如下:

```
paths:
  /users:
    get:
      tags:
        - user
      summary: get user list
      operationId: getUsers
      parameters: 
        - name: limit
          in: query
          description: limit
          required: true
          schema:
            type: string
        - name: offset
          in: query
          description: offset 
          required: true
          schema:
            type: string
      responses:
        '200':
          description: successful operation
          content:
            application/json:
              schema:
                type: array
                items:
                  $ref: '#/components/schemas/User'
```

`paths`中每个路径以/开头，路径必须有一`responses`属性。在同一路径下不同操作定义如下:

```
 /users:
    get:
      tags:
        - user
      responses:
        '200':
    post: 
      tags:
        - user
      responses:
        '200':
    delete: 
       tags:
        - user
      responses:
        '200':
    put: 
      tags:
        - user  
      responses:
        '200':
```


在2.0我们可以用`Consumes, Produces`来定义请求或者响应的数据格式，在3.0中我们统一使用关键字`content`定义数据格式

```
/user:
    post:
      tags:
        - user
      summary: Create user
      description:
      operationId: createUser
      requestBody:
        content:
          application/json:
            schema:
              $ref: '#/components/schemas/User'
        description: Created user object
        required: true
      responses:
        '200':
          description: successful operation
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/User'
      
```

`components`对象替换掉2.0中的`definitions`，并规范了对象的重用。


####路径
* paths定义了API中的各个路径,以及这些路径的HTTP方法。 例如，GET /user 可以描述为：

```
paths:
  /users:
    get:
  /user/{id}:
    put:
```

* 在同一路径下不同的操作

**正确方式**

```
paths:
  /users:
    get:
    put:
    post:
```

**错误方式**

```
paths:
  /users:
    get:
  /users:
    post:
```

####请求参数
请求参数可以通过拼接路径/users/{userId}，查询字符串(/users?role=admin),将参数放到请求头X-CustomHeader: Value，你可以定义参数数据类型，格式。

* 拼接路径参数 `GET /users/2`

```
paths:
  /user/{userId}:
    get:
      summary: Returns a user by ID.
      parameters:
        - name: userId
          in: path
          required: true
          description: Parameter description in CommonMark or HTML.
          schema:
            type : integer
            format: int64
            minimum: 1
```

* 查询参数 `GET /users?role=value`

```
paths:
  /users:
    get:
      parameters:
        - in: query
          name: role
          schema:
            type: string
            enum: [user, poweruser, admin]
          required: true
```

* 将请求参数放到请求头

```
paths:
  /user:
    get:
      summary: Checks if the server is alive
      parameters:
        - in: header
          name: X-Request-ID
          schema:
            type: string
            format: uuid
          required: true
```



* 将请求参数放到request body，可以通过关键字requestBody来描述body的内容,在content关键词下定义Media Type

```
application/json
application/xml
application/x-www-form-urlencoded
multipart/form-data
text/plain; charset=utf-8
text/html
application/pdf
image/png
```

```
/user:
  post:
    summary: 创建一个用户
      requestBody:
        required: true
        content:
          application/json:
            schema:
              type: object
              properties:
                username:
                  type: string
```

* 文件上传

```
/file:
  post:
      summary: 文件上传.
      requestBody:
        content:
          multipart/form-data:
            schema:
              properties:
                fileName:
                  type: integer
                file:
                  type: string
                  format: binary
```


####返回值

API规范需要指定所有API操作的Responses。 每个操作必须至少定义一个Responses，通常是成功的Responses。 Responses由其HTTP状态代码和Responses正文和/或标题中返回的数据定义。
这是一个最简单的例子:

```
paths:
  /ping:
    get:
      responses:
        '200':
          description: OK
          content:
            text/plain:
              schema:
                type: string
                example: pong
```

API可以用各种数据类型(json/xml)进行响应。 JSON是数据交换中最常用的格式，但不是唯一可能的格式。

要指定Response的数据类型，在content关键字下说明。

```
paths:
  /users:
    get:
      summary: Get all users
      responses:
        '200':
          description: A list of users
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/ArrayOfUsers'
            application/xml:
              schema:
                $ref: '#/components/schemas/ArrayOfUsers'
            text/plain:
              schema:
                type: string

  # This operation returns image
  /logo:
    get:
      summary: Get the logo image
      responses:
        '200':
          description: Logo image in PNG format
          content:
            image/png:
              schema:
                type: string
                format: binary
```

####HTTP Status Codes
在Response下，每个Response定义以状态码（例如200或404）开始。操作通常会返回一个成功的状态码和一个或多个错误状态。 要定义一系列Response代码，您可以使用以下范围定义：1XX，2XX，3XX，4XX和5XX。 如果使用显式代码定义响应范围，则显式代码定义优先于该代码的范围定义。

```
responses:
        '200':
          description: OK
        '400':
          description: Bad request. User ID must be an integer and larger than 0.
        '401':
          description: Authorization information is missing or invalid.
        '404':
          description: A user with the specified ID was not found.
        '5XX':
          description: Unexpected error.
```


####Response Headers

来自API的响应可以包含自定义标题，以提供有关API调用结果的附加信息。 例如，限速API可以通过响应标头提供速率限制状态，如下所示：

```
HTTP 1/1 200 OK
X-RateLimit-Limit: 100
X-RateLimit-Remaining: 99
X-RateLimit-Reset: 2016-10-12T11:00:00Z

{ ... }
```

你可以像这样定义响应headers

```
paths:
  /ping:
    get:
      summary: Checks if the server is alive.
      responses:
        '200':
          description: OK
          headers:
            X-RateLimit-Limit:
              schema:
                type: integer
              description: Request limit per hour.
            X-RateLimit-Remaining:
              schema:
                type: integer
              description: The number of requests left for the time window.
            X-RateLimit-Reset:
              schema:
                type: string
                format: date-time
              description: The UTC date/time at which the current rate limit window resets.
```

####Response Body
schema关键字用于描述Response Body，schema可以这样定义：

* 用一个 object或者 array ,通常用于JSON或者XML
* 原始数据类型，如数字或字符串 - 用于纯文本响应，
* 一个文件

```
responses:
        '200':
          description: A User object
          content:
            application/json:
              schema:
                type: object
                properties:
                  id:
                    type: integer
                    description: The user ID.
                  username:
                    type: string
                    description: The user name. 
```

####重用
如果多个操作返回相同的响应（状态码和数据），则可以在全局components对象的响应部分中定义它，然后通过$ref在操作级别引用该定义。 这对于具有相同状态代码和响应主体的错误响应非常有用

```
responses:
        '200':
          description: A User object
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/user'
components:
  schemas:
    user:
      type: object
      properties:
        id:
          type: integer
          format: int64
        password:
          type: string
          description: 密码
        name:
          type: string
          description: 用户
        avator:
          type: string
          description: 头像
          default: xxxx
```

* 在请求时我们也可以使用定义好的schema:

```
post:
      summary: 创建用户.
      tags:
        - 用户
      requestBody:
        content:
          application/json:
            schema:
              $ref: '#/components/schemas/user'
      responses:
        '200':
          description: 操作结果
          content:
            application/json:
              schema:
                type: object
                properties:
                  code:
                    type: integer
                  msg:
                    type: string
```

