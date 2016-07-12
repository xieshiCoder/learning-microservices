# 微服务架构设计与实现

## RestBucks 系统依赖

接口设计及其依赖：

- 用户信息 b2b-customer-service
    + customer-api 提供RESTful API的用户信息接口
    + product-api 提供用户在Restbucks所充值的会员卡号码
    + self-loyalty-service 提供用户的所有会员卡具体信息，公用 Domain，以及共有逻辑
- 会员卡信息 b2b-loyalty-service
    + gcis 提供用户的个人身份信息
    + self-loyalty-service 提供不同品牌会员卡的统一接口
        * wmb 提供每张会员卡的具体信息，部分品牌
        * group protect 提供每个会员卡的具体信息，但支持的品牌不同
    + order centre 提供不同种类咖啡的可购买次数
- 订单信息 b2b-order-service
    + order centre 提供咖啡交易的具体信息
    + 分为 getOrder 和 getOrderList 两个接口
    + 并含有相关服务的集成，比如：
        * 下订单 placeAnOrder
        * 获取门店照片 storePhotos
- 电商网站 my-order-manager webapp 前后端分离
    + 前端 my-order-manager-webapp
    + 后端 my-order-manager-services

## 遗留系统问题分析

- 一些依赖的旧服务采用 SOAP，只能用 XML 进行交互，并且根据 WSDL 生成相关代码，每当一更新就有可能破坏接口，当字段发生改变就有可能因为无法匹配而失败。
    + 在 REST service 底下一层抽象出 connectors 的概念，封装成为一个 library
    + 这个 connectors 有两种方式获取数据
        * getOrder 使用 Apache CXF 来生成 WDSLToJava 代码
            - 包含 Domain 和对应的接口
        * getOrderList 和 getStorePhotos 则使用常规 SOAP Request 来处理旧 API
            - commons-client

