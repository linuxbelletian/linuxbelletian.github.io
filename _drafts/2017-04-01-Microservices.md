---
layout: post
title: Python
date:   2017-03-31 14:01:00 +0800
categories: Pattern
---

see[Microservices](http://microservices.io/patterns/microservices.html)

# Pattern: Microservice Architecture

## Context

你正在开发一个服务端企业级应用. 它必须支持多种不同的客户端其中包括了桌面浏览器,移动浏览器，本地化的移动应用。这个应用可能也需要暴露一个API给第三方来使用。它也可能需要与其他的应用比如一个web services 或者 一个message broker 进行集成。这个应用通知执行商业逻辑来处理请求(HTTP请求以及消息);访问一个数据库;与其他系统交换消息;返回一个HTML/JSON／XML响应。会有一个逻辑组件对应与应用的不同功能区域。

## Problem
什么事该应用的部署结构

## Forces(影响)
- 又一个开发团队在开发维护这个应用
- 新的团队成员必须快速地投入生产
- 应用必须简单到容易理解且容器修改
- 你想要实践连续部署应用
- 你必须运行该应用的拷贝在多台机器上以便满足伸缩性以及可用性要求
- 你想要利用新兴的技术(emeriging technologies)(frameworks,programming languages, etc)

## Solution
定义一个结构(architecture) 该结构以一组松散的相互合作的服务来构造这个应用。... 。每一个服务实现一组弱小的，相关的功能。例如， 一个应用可能由这样一组服务器组成:
订单管理服务，消费者管理服务器等等。

服务间通信可以使用同步协议 例如 HTTP/REST 或者异步协议例如AMQP. 服务可以被独立的开发并部署。每一个服务有自己的数据库以便与其他服务器解耦。两个服务器之间的数据一致性则通过使用 event-driven-architecture 来维护。


# Examples

## 虚假的电子商务应用

让我们来想象一下你正在构建一个电子商务应用，从消费者那里拿到订单，验证存货以及信用可可用性，接着开始送货。该应用由几个组件组成包括了StoreFrontUI(商店前端UI)，
它实现了用户界面，以及跟随而来的后端服务器用于检查信用，维护库存以及运送订单。应用由一组服务组成。

![mage](http://microservices.io/i/Microservice_Architecture.png)


## Show me the code
请访问[Example Host](http://eventuate.io/exampleapps.html)
[Example on github](https://github.com/eventuate-examples/eventuate-examples-java-customers-and-orders)
这些在github.com 上的例子 列举了微服务结构的多个方面的问题


# Resulting context

## 益处
这中解决方案有以下的好处:
- 每一个微结构相对而言都很小
    - 更加容易开发者理解
    - 集成开发环境更快速开发者更加容易产出
    - 应用启动更加快速，使得开发者更具生产能力，加速部署速度
- 每一个服务可以被独立于其它服务部署-更加简单的频繁的部署新版本服务
- 更加加单的伸缩性开发。它能够使你去组织开发工作分发给多个团队。每一个团队拥有并且负责一个或者更多的单一服务器。每一个团队可以开发部署伸缩她们自己的服务 独立于其它所有的团队.
- 改善缺点隔离。例如，如果某个服务器存在内存泄漏那么只有该服务收到影响。其他服务器将继续处理请求。相对而言，在一个宏系统中一个故障的组件可能代理整个系统的停摆.
- 每一个服务都可以被独立的开发以及部署
- 排除任何对某个技术栈的长期维护需求。当开发一个新服务，你可以选择新的技术栈，相似的当你对已经存在的服务实行重要改变时，你可以使用新技术重写该服务


## Drawbacks 缺点
该方案有数个缺点:
- 开发者在创建分布式系统时处理额外的复杂性
    - 开发者工具或者IDE都是面向构建宏应用的并部提供明确的为分布式应用提供支持
    - 测试变得困难
    - 开发者必须实现服务间通信机制
    - 实现横跨多个服务的用例而没有使用分布式事务是非常困难的
    - 实现横跨多个服务的用例要求非常小心的协调各个团队
- 部署变得复杂。在生产中，同样也有部署和管理一个由大量不同类型服务器构成的系统的的操作的复杂性
- 增加内存消耗，为微服务结构代替了N宏内核应用实例而变成了N*M服务器实例。每一个服务运行在自己的JVM中，这通常是独立化实例所必要的，那么这里就有
M倍数的JVM运行时损耗。更多的，如果每一个服务器运行在自己的VM中，那么损耗会更加高.

# Issues
有许多问题你必须处理.

## 什么时候用microservie architecture
使用这个方式的一个挑战就是决定什么事用它是最好的。当开发一个应用的第一版时，你经常没有用遇到这个方式解决的问题。更进一步的，使用一个精细的，分布式的结构将会是的开发速度很慢。对于初创企业来说 开发速度是主要问题，其最大的挑战往往是如何快速的发展业务模式以及随附的应用程序.

使用Y轴分离可能使得重复迭代非常困难。接着 ，不仅如此，当挑战是如何伸缩并且你需要将功能分解时，纠缠的依赖关系可能使得分解你的宏应用为一组服务变得困难。

## 如何分解应用为一组服务器
另外一个挑战是决定如何将系统划分为微服务。这是一个非常的艺术性问题，但是这里有大量的策略可以帮助我们:
- 通过业务能力分解，并且根据业务能力来定义相应的服务。
- 通过领域驱动设计子领域分解
- 通过动词或者用例并且定义负责该特定动作的服务，例如 一个Shipping Service 负责运送完成的订单。
- 通过使用名词或者资源，通缩定一个服务使得该服务负责所有的该实体或者该资源类型的操作来分解，例如
一个Account Service 负责管理用户账户

理想情况下，每一个服务应该只有一小部分责任集合。
(Uncle)Bob Martin 谈论了关于使用单一责任原则（Single Responsibility Principle SRP）来设计类。SRP 定义了当一个类由于某种理由发生改变时的责任以及申明了一个类职能有一个理由才改变。这个同样适用于将SRP应用到service design。

另外一个类似的可以帮助面向服务设计的是Unix工具集的设计思路。Unix 提供了一个庞大的工具集合 例如：
grep，cat ，find etc。每一个工具只做一个事情，并且做的特别好，然后可以跟其他工具使用shell脚本结合来执行更加复杂的任务。

## 如何维护数据一致性
为了确保松散的联系，每一个服务器都拥有自己的数据库，维护两个服务之间的数据一致性会成为一个挑战，因为两步完成的 提交/分发 事物 并不是对许多应用是一个可选项。一个应用必须转而使用Event-driven architecture。一个服务发布一个事件当它的数据改变发生。其他服务消费该事件并且更新它们自己的数据。这里有几种可靠的更新数据以及发布事件的方式
包括[Event Sourcing](http://microservices.io/patterns/data/event-sourcing.html) 以及 [Transaction Log Tailing](http://microservices.io/patterns/data/transaction-log-tailing.html).

## 如何实现查询
另外一个挑战是查询被多个服务拥有的数据如何实现。一个通用的解决方案是使用[Command Query Responsibility Segregation](http://microservices.io/patterns/data/cqrs.html) 并且维护一个或者更多的呈现视图 这些视图保持更新通过订阅数据发生变化的的事件流

# 相关模式
在microservices 模式中，有许多模式被使用或者相关，[Monolithic architecture](http://microservices.io/patterns/monolithic.html)对于微服务来说是一个可代替方案。其他的模式定位了你在应用微服务结构时会遇到的问题.

![pattern](http://microservices.io/i/PatternsRelatedToMicroservices.jpg)

- 分解模式
    - [Decompose by business capability](http://microservices.io/patterns/decomposition/decompose-by-business-capability.html)
    - [Decompose by subdomain]
    - 