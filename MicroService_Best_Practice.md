# MicroService Best Practice

> 就是当前项目的经验书面化。

## Introduction/Idea

- 工作总结
- 各种坑
    + Java语法坑
    + 开源库坑
- 业务简介
- 模拟业务 -> REST实战（Restbucks）

**可能的， 大致内容：**

- [security]=>模拟业务场景 eg: 不同品牌的咖啡馆，买单和取咖啡
- 架构设计 RESTful与遗留依赖分析 eg: SOAP交互, What's really REST
- Domain Model与processor eg: provider, Orika mapping
- 技术框架 SpringBoot 以及 Spring DI/MVC eg: AutoConfig Properties etc
- 构建工具 Gradle以及插件 eg: custom task, checkstyle, cobertura, offline mode etc
- API设计与展示 eg: JSON format, Swagger, Jackson, documentation
- TDD/xx测试 eg: JUnit, JUnitParams, Mockito, BBDMockito
- Contract Testing eg: pact-jvm, postman
- 稳定性/mentoring eg: gatling, log management(splunk, metrics) springboot-admin
- Health Check/Thread pool eg: Hytrix & Dashboard, configurable downtime
- 持续集成/DevOps eg: Jenkins, Devops & Ansbile, 环境的区分
- Java语法/工具包 eg: Guava, Bean, 设计模式 etc
- Git工作流 团队协作 eg: 环境搭建, Stash ssh问题, proxy

------

正文

## 模拟业务场景：RestBucks

## Gradle

[Boosting the performance for Gradle in your Android projects](https://medium.com/@erikhellman/boosting-the-performance-for-gradle-in-your-android-projects-6d5f9e4580b6#.vf2v3kb25)
