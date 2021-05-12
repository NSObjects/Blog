---
title: "OpenApi2.0"
date: 2018-03-27T21:45:57+08:00
draft: false
toc: true
---

## 什么是OpenApi规范？
OpenApi规范是一套用语描述REST API的描述格式，分为以下几
```language  go
func main() {
 var a = 2
}
 
```
* 描述每个路径上的操作 GET POST DELETE
* 请求参数和返回值
* 认证方法
* 联系信息，许可证，使用条款和其他信息

API规范可以用YAML或JSON编写。该格式对人和机器都很容易学习和阅读。

OpenApi当前版本是3.0，本文将描述OpenApi2.0规范
可以在这里查看完整规范https://github.com/OAI/OpenAPI-Specification/blob/master/versions/2.0.md

本文将用一个YAML格式的例子向你描述如何编写OpenApi2.0文档



### 文档信息描述

```
swagger: "2.0"
info:
  title: Sample API
  description: API description in Markdown.
  version: 1.0.0

host: api.example.com
basePath: /v1
schemes:
  - https
```

`swagger: "2.0"` 说明当前使用的OpenApi版本，（Openapi 2.0是用swagger2.0标准演化而来）

`info`对象将描述本文的信息，下表是info对象的所有可用字段


| 字段 | 类型 | 描述 |
| --- | --- | --- |
| title | String | 本文标题 |
| description  | String | 本文的描述信息 |
| termsOfService | String |API的服务条款|
| contact | Object | 联系方式对象 |
| license | Object | 许可证对象 |
| version | String | 版本 |


`contact` 对象用于描述本文作者联系方式

| 字段 | 类型 | 描述 |
| --- | --- | --- |
| name | String | 本文作者名称 |
| url | String | 本文作者的网站 |
| email | String | 本文作者的邮件 |

`license` 对象用于描述本文遵循的协议


| 字段 | 类型 | 描述 |
| --- | --- | --- |
| name | String | 协议名称 |
| url | String | 协议URL |


`host` 用于描述api的服务器地址，在host中不用填写路径，但是要包括服务所用到的端口号

`basePath` 用于描述api的基础路径，例如/v1/api

`schemes` 用于描述使用的传输协议例如`http/htttps`

`paths`定义了api中每个路径以及这些路径下的的操作方法（GET,POST),如果我们想在users路径下定义一个GET方法，我们可以像下面这样。

```
paths:
  /users:
    get:
      tags:
        - pets
      summary: Returns a list of users.
      description: Optional extended description in Markdown.
      produces:
        - application/json
      responses:
        200:
          description: OK
```

`tags` 关键词可以将多个路径组成一个群组显示，我们可以在全局定义`tags`关键词

```
tags:
  - name: pets
    description: Everything about your Pets
    externalDocs:
      url: http://docs.my-api.com/pet-operations.htm
  - name: store
    description: Access to Petstore orders
    externalDocs:
      url: http://docs.my-api.com/store-orders.htm
```

### 请求参数

向服务器发起请求时我们以不同的方式提交数据给服务器，

* 查询参数: /users?role=admin
* 路径参数: /users/{id}
* 请求头参数: X-MyHeader: Value
* Request Body 在POST或者PUT请求时上传一段JSON或者XML类型的参数
* form 参数：当Content-Type为application/x-www-form-urlencoded 或者 multipart/form-data

##### 路径参数

```
paths:
  /users/{userId}:
    get:
      summary: Returns a user by ID.
      parameters:
        - in: path
          name: userId
          required: true
          type: integer
          minimum: 1
          description: Parameter description in Markdown.
      responses:
        200:
          description: OK
```

##### 查询参数

```
paths:
  /users/{userId}:
    get:
        parameters:
            - in: query
            name: offset
            type: integer
            description: The number of items to skip before starting to collect the result set.
            - in: query
            name: limit
            type: integer
            description: The numbers of items to return.    
            
```

##### Request Body 
我们可以根据`Consumes, Produces`关键词描述将要传输的数据格式。例如:

```
consumes:
  - application/json
  - application/xml
produces:
  - application/json
  - application/xml
```

```
paths:
  /users:
    post:
      summary: Creates a new user.
      consumes:
        - application/json
      parameters:
        - in: body
          name: user
          description: The user to create.
          schema:
            type: object
            required:
              - userName
            properties:
              userName:
                type: string
              firstName:
                type: string
              lastName:
                type: string
      responses:
        201:
          description: Created
```


##### 请求头参数

```
paths:
  /ping:
    get:
      summary: Checks if the server is alive.
      parameters:
        - in: header
          name: X-Request-ID
          type: string
          required: true
```

##### Form参数

```
paths:
  /survey:
    post:
      summary: A sample survey.
      consumes:
        - application/x-www-form-urlencoded
      parameters:
        - in: formData
          name: name
          type: string
          description: A person's name.
        - in: formData
          name: fav_number
          type: number
          description: A person's favorite number.
      responses:
        200:
          description: OK
```

在同一个路径下定义不同的方法

```
paths:
  /user/{id}:
    parameters:
      - in: path
        name: id
        type: integer
        required: true
        description: The user ID.
    get:
      summary: Gets a user by ID.
    patch:
      summary: Updates an existing user with the specified ID.
    delete:
      summary: Deletes the user with the specified ID.

```


在定义参数时我们可以将某个参数定义为是否可选通过`required: true`,参数的默认值`default: 0`,我们还可以定义参数值的范围 `minimum: 1 maximum: 100`

### 响应
`responses`对象用于定义响应的内容，响应包括Response Body，HTTP Status Codes，Response Headers,每一个API都必须有一个responses，最基础的定义如下:

```
paths:
  /ping:
    get:
      produces:
        - application/json
      responses:
        200:
          description: OK
```



##### HTTP Status Codes

```

responses:
        200:
          description: OK
        400:
          description: Bad request. User ID must be an integer and bigger than 0.
        401:
          description: Authorization information is missing or invalid.
        404:
          description: A user with the specified ID was not found.
          
```

##### Response Body

```
responses:
        200:
          description: A User object
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

##### Response Header

```

paths:
  /ping:
    get:
      summary: Checks if the server is alive.
      responses:
        200:
          description: OK
          headers:
            X-RateLimit-Limit:
              type: integer
              description: Request limit per hour.
            X-RateLimit-Remaining:
              type: integer
              description: The number of requests left for the time window.
            X-RateLimit-Reset:
              type: string
              format: date-time
              description: The UTC date/time at which the current rate limit window resets.
              
```

### 对象重用
我们可以通过全局关键词`definitions` 定义一个可重用的对象，然后使用`$ref`引用这个对象,例如:

```
paths:
  /users:
    post:
      summary: Creates a new user.
      consumes:
        - application/json
      parameters:
        - in: body
          name: user
          description: The user to create.
          schema:
            $ref: "#/definitions/User"     # <----------
     responses:
         200:
           description: OK
definitions:
  User:           # <----------
    type: object
    required:
      - userName
    properties:
      userName:
        type: string
      firstName:
        type: string
      lastName:
        type: string
```

我们也可以在响应中这样使用

```
paths:
  /users:
    get:
      summary: Gets a list of users.
      response:
        200:
          description: OK
          schema:
            $ref: "#/definitions/ArrayOfUsers"
        401:
          $ref: "#/responses/Unauthorized"   # <-----
  /users/{id}:
    get:
      summary: Gets a user by ID.
      response:
        200:
          description: OK
          schema:
            $ref: "#/definitions/User"
        401:
          $ref: "#/responses/Unauthorized"   # <-----
        404:
          $ref: "#/responses/NotFound"       # <-----
# Descriptions of common responses
responses:
  NotFound:
    description: The specified resource was not found
    schema:
      $ref: "#/definitions/Error"
  Unauthorized:
    description: Unauthorized
    schema:
      $ref: "#/definitions/Error"
definitions:
  # Schema for error response body
  Error:
    type: object
    properties:
      code:
        type: string
      message:
        type: string
    required:
      - code
      - message
```

在定义对象时我们可以对属性添加一个示例值

对象

```
definitions:
  CatalogItem:
    id:
      type: integer
      example: 38
    title:
      type: string
      example: T-shirt
    image:
      type: object
      properties:
        url:
          type: string
        width:
          type: integer
        height:
          type: integer
      required:
        - url
      example:   # <-----
        url: images/38.png
        width: 100
        height: 100
    required:
      - id
      - title
```

数组

```
definitions:
  ArrayOfStrings:
    type: array
    items:
      type: string
    example:
      - foo
      - bar
      - baz
```

数组中嵌套的对象可以直接引用另一个已经定义好的对象

```
definitions:
  ArrayOfCatalogItems:
    type: array
    items:
      $ref: "#/definitions/CatalogItem"
    example:
      - id: 38
        title: T-shirt
      - id: 114
        title: Phone
```


