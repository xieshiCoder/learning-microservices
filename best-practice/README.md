这几篇文章就主要来自于自身所在项目的一些总结，更加注重实战性，尝试从侧面的角度分析具体技术栈的What&Why&How，以便于在今后项目中能够更好地被单独引入，而不局限于当前应用场景。

**可能的， 大致内容：**

- [security]=>模拟业务场景 eg: 不同品牌的咖啡馆，买单和取咖啡
- 架构设计 RESTful与遗留依赖分析 eg: SOAP交互, What's really REST
- Domain Model与processor eg: provider, Orika mapping
- 技术框架 SpringBoot 以及 Spring DI/MVC eg: AutoConfig Properties yaml etc
- 构建工具 Gradle以及插件 eg: custom task, checkstyle, cobertura, offline mode etc
- API设计与展示 eg: JSON format, Swagger, Jackson, documentation
- TDD/xx测试 eg: JUnit, JUnitParams, Mockito, BBDMockito
- Contract Testing eg: pact-jvm, postman
- 稳定性/mentoring eg: gatling as test, log management(splunk, metrics) springboot-admin
- Health Check/Alerts: 提醒，以及图表分析
- Thread pool eg: Hytrix & Dashboard, configurable downtime
- 持续集成/DevOps eg: Jenkins, Devops & Ansbile, 环境的区分
- Java语法/工具包 eg: Guava, Bean, 设计模式 etc
- Git工作流 团队协作 eg: 环境搭建, Stash ssh问题, proxy

------

接下来的计划：

- Development
    + API Gateway
    + AWS RDS
    + AWS ElasticCache
- Deployment
    + different environments
    + AWS instances / Ploaris
    + Container tech
- production support
    + Monitoring
    + Security
    + Logging management
- Front-end
    + Angular
