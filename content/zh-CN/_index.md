---
date: 2018-10-08T21:07:13+01:00
title: Hiboot云原生应用框架
type: index
tags: ["hiboot", “cloud-native” "framework"]
weight: 1
---

## 关于 

{{<badges>}}

Hiboot 是一款用Go语言实现的高性能网络及命令行应用框架。Hiboot提供了Web MVC框架，支持控制反转（依赖注入），拦截器，依赖包自动配置（类似Spring Boot的Starter）。

Hiboot并不打算重复造轮子，故使用了自动配置的机制，很轻松的集成任何第三方依赖包，并使得其更方便使用，例如数据库连接，不需要在业务代码中配置及创建数据库连接，你只要依赖[hiboot-data/starter/gorm](https://hidevops.io/hiboot-data)即可注入数据库连接实例，直接实现数据库查询。

## 总览

* MVC架构 (Model-View-Controller).
* 自动配置, 事先配置好的依赖包可以被注入到任何你指定的构造函数参数中或者结构体变量
* 依赖注入， 使用标签 \`inject:""\` 或者构造函数来实现.

## 功能列表

* Apps - 应用
    * cli - 命令行应用
    * web - 网络应用
* Starters - 自动配置
    * actuator - 健康检测
    * locale - 国际化
    * logging - 日志
    * jaeger - 分布式链路追踪
    * jwt - JWT 令牌
    * grpc - 服务间通讯gRPC
    * websocket - websocket 服务端与客户端实时通信
* Tags - 标签
    * inject - 注入依赖
    * default - 注入默认值
    * value - 注入常量，环境变量或引用值
* Utils - 工具
    * cmap - 支持并发的map
    * copier - 复制struct工具
    * crypto - aes, base64, md5, and rsa 加／解密工具
    * gotest - go test 工具
    * idgen - twitter snowflake 唯一 id 生成器
    * io - 文件操作工具
    * mapstruct - 转换map 到 struct
    * replacer - 替换struct中的值为引用或环境变量
    * sort - 切片排序
    * str - 字符串处理工具
    * validator - 参数校验