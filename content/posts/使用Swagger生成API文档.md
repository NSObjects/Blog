---
title: "使用Swagger生成API文档"
date: 2018-03-18T21:04:40+08:00
draft: true
---

swagger是一个款强大的API文档生成工具集，他分为swagger-ui，swagger-codegen，swagger-editor。通过这三个工具我们可以编写非常完善Restful API文档。

####swagger-ui
swagger-ui顾名思义这个工具就是用来显示编写好的api文档，这个工具通过解析编写好的json文档显示一个web页面。
![](http://7u2kla.com1.z0.glb.clouddn.com/15213791666907.jpg)


通过docker安装

```
docker pull swaggerapi/swagger-ui
docker run -p 80:8080 swaggerapi/swagger-ui
```

###swagger-editor
swagger-editor是一个编辑器，可以实时预览修改结果,不过这编辑器对中文支持非常不友好，建议在其他编辑器里写好后再粘贴进来查看语法是否错误
![](http://7u2kla.com1.z0.glb.clouddn.com/15213793329758.jpg)

同样通过docker安装

```
docker pull swaggerapi/swagger-editor
docker run -d -p 80:8080 swaggerapi/swagger-editor
```

###swagger-codegen
swagger-codegen工具是可以通过编辑好的api(yaml文件)文档直接生成各种后端的代码。现在支持的语言有:

```
 C# (ASP.NET Core, NancyFx), C++ (Pistache, 
 Restbed), Erlang, Go, Haskell (Servant), Java 
 (MSF4J, Spring, Undertow, JAX-RS: CDI, CXF, 
 Inflector, RestEasy, Play Framework, PKMST), 
 Kotlin, PHP (Lumen, Slim, Silex, Symfony, Zend 
 Expressive), Python (Flask), NodeJS, Ruby 
 (Sinatra, Rails5), Rust (rust-server), Scala 
 (Finch, Lagom, Scalatra)
```

###本文将通过openapi3规范编写api文档:
####OpenAPI规范




