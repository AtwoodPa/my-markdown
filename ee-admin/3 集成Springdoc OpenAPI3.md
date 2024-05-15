# 集成Springdoc OpenAPI3

## 前言
OpenApi是一个业界的API文档标准，目前该规范有两大实现，分别是：
- SpringFox
- SpringDoc

Springdoc-openapi 可以帮助Spring Boot项目自动化生成API文档。

Springdoc主要借助注解扫描来实现文档的自动化生成，springdoc 会扫描你的 Spring Boot 应用程序中的控制器和其他组件，特别是使用了 OpenAPI 相关的注解的类和方法。

并且springdoc提供友好的UI界面以供开发者使用，生成的 OpenAPI 文档可以通过一个预定义的端点（如 /v3/api-docs）获取，也可以配合一些 UI 组件（如 Swagger UI 或 Redoc）展示成易于阅读和交互的格式，方便开发人员查看和测试 API。

官方文档：https://openapi.apifox.cn/

SpringDoc支持：
- OpenAPI 3.0
- Spring Boot 全版本
- JSR-303注解：@NotNull, @Min, @Max, @Pattern, @Size, @ApiParam, @ApiModelProperty
- Swagger UI

## Quick Start

springdoc+javadoc 实现无注解零入侵生成规范的openapi结构体