## NexFrame

> 快速构建可单体可微服务的运营级系统

NexFrame是一款基于 Go 语言免费开源的，快速、简单的企业级应用开发框架，旨在为开发者提供高效、可靠、可扩展的应用开发框架。通过集成serviceweaver服务能力，实现单体开发，微服务运行的效果。

包含清晰简单的API定义，简单的微服务添加。

![架构图](img/nexframe.png)

## NexFrame开发背景

Sagoo团队在2022年基于GoFrame框架开发了SagooIoT企业级物联网系统，到2024年近两年时间SagooIoT移定后，虽然支持分布式部署来解决大规模设备运行，但是云环境盛行的时代下，传统的分布式还是有一定的局限，经过一些时间的对比决定开发新框架，解决SagooIoT产品基础框架，从运营级出发，重新构建。

**技术栈决定采用：**

- HTTP路由：**gorilla/mux**
- 微服务能力：**serviceweaver**
- 数据库ORM：**GORM**

## NexFrame框架设计思考

我们希望提供一个能开箱即用的、稳定的、支持高并发的运营级的基础框架。核心业务无关的组件服务化，通用功能服务化。

- 稳定运行，第一原则
- 高并发
- 业务开发友好

## 主要特性

* **微服务架构：** 通过 **serviceweaver** 提供强大的微服务能力，支持服务注册、发现、负载均衡等功能，帮助开发者构建灵活、可伸缩的分布式系统。
* **ORM 支持：** 集成 **GORM** ORM 框架，简化数据库操作，提供丰富的功能，如结构映射、查询构建、关联关系等，提升开发效率。
* **高性能路由：** 完全兼容net/http，使用 **gorilla/mux** 构建高性能的 HTTP 路由系统，支持 URL 参数、正则表达式匹配等功能，灵活配置路由规则。
* **API 自动绑定：** API 定义与控制器自动绑定，API参数自动校验，简化 API 开发流程，提高开发效率。
* **OpenAPI 文档自动生成：** 支持自动生成 OpenAPI 规范文档，方便 API 的调试和集成。
* **模块化设计：** 采用模块化设计，各模块之间耦合度低，易于扩展和维护。
* **丰富的中间件：** 提供丰富的中间件，如日志记录、错误处理、权限验证等，方便开发者定制化开发。
* **强大的工具集：** 提供一系列开发工具，如代码生成、测试框架等，提升开发效率。

## **优势**

* **高性能：** 基于 Go 语言的高并发特性，以及 **serviceweaver**、**gorilla/mux** 等高性能库，保证了框架的高性能。
* **易用性：** 提供简洁易用的 API，降低开发门槛，提高开发效率。
* **可扩展性：** 采用模块化设计，易于扩展和定制。
* **可靠性：** 经过大量测试和验证，保证了框架的稳定性和可靠性。

## **适用场景**

* **微服务架构：** 适用于构建大型、复杂的微服务系统。
* **企业级应用：** 适用于开发企业级的后台管理系统、业务系统等。

## 版权申明

在发布本资料时，请严格遵守开放出版许可协议 1.0 或其后续版本的规定。未经版权所有者的明确书面授权，不得擅自发布本文档的修改版本。此外，除非事先获得版权所有者的特别许可，否则禁止将此作品或其衍生作品以标准纸质书籍的形式进行发行。

若您有意再发行或再版本手册的全部或部分内容，无论是否经过修改，或有任何相关疑问，请及时与版权所有者联系，邮箱地址为：nexframe@sagoo.cn。我们期待与您共同维护和尊重知识产权，确保作品的合法合规传播。
