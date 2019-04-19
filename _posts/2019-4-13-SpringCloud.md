---
layout:     post
title:      谈谈微服务那些事
subtitle:   Spring Cloud 
date:       2019-4-10
author:     Chen
header-img: img/post-bg-ios9-web.jpg
catalog: 	 true
tags:
    - 微服务
---

Spring Cloud 是一系列框架的有序集合。它利用 Spring Boot 的开发便利性巧妙地简化了分布式系统基础设施的开发，如服务发现注册、配置中心、消息总线、负载均衡、断路器、数据监控等，都可以用 Spring Boot 的开发风格做到一键启动和部署。Spring 并没有重复制造轮子，它只是将目前各家公司开发的比较成熟、经得起实际考验的服务框架组合起来，通过 Spring Boot 风格进行再封装屏蔽掉了复杂的配置和实现原理，最终给开发者留出了一套简单易懂、易部署和易维护的分布式系统开发工具包。

1. Spring Cloud基于Springboot的，利用了Spring boot 的特性
1. Spring Cloud组合了很多优秀的服务框架
# 什么是微服务
要了解 Spring Cloud 就绕不开微服务这个概念。因为 Spring Cloud 是 Spring 为微服务架构思想做的一个一站式实现。从某种程度是可以简单的理解为，微服务是一个概念、一个项目开发的架构思想。Spring Cloud 是微服务架构的一种 Java 实现。

微服务的概念源于 Martin Fowler 于 2014 年 3 月 25 日写的一篇文章 [Microservices](https://martinfowler.com/articles/microservices.html)

微服务架构模式（Microservices Architecture Pattern）的目的是将大型的、复杂的、长期运行的应用程序构建为一组相互配合的服务，每个服务都可以很容易得局部改良。 Micro 这个词意味着每个服务都应该足够小，但是，这里的小不能用代码量来比较，而应该是从业务逻辑上比较——符合 SRP 原则的才叫微服务。

关于微服务的一个形象表达：

![](/img/微服务.jpg)

X 轴：水平扩展，即在负载均衡服务器后增加多个运行实例

Z 轴：数据库的扩展，即分库分表

Y 轴：功能分解，即将不同职能的模块分成不同的服务

# 微服务架构优势
### 复杂度可控:
在将应用分解的同时，规避了原本复杂度无止境的积累。每一个微服务专注于单一功能，并通过定义良好的接口清晰表述服务边界。由于体积小、复杂度低，每个微服务可由一个小规模开发团队完全掌控，易于保持高可维护性和开发效率。

### 独立部署：
由于微服务具备独立的运行进程，所以每个微服务也可以独立部署。当某个微服务发生变更时无需编译、部署整个应用。由微服务组成的应用相当于具备一系列可并行的发布流程，使得发布更加高效，同时降低对生产环境所造成的风险，最终缩短应用交付周期。
### 技术选型灵活：
微服务架构下，技术选型是去中心化的。每个团队可以根据自身服务的需求和行业发展的现状，自由选择最适合的技术栈。由于每个微服务相对简单，故需要对技术栈进行升级时所面临的风险就较低，甚至完全重构一个微服务也是可行的。
### 容错（fault isolation）：
当某一组建发生故障时，在单一进程的传统架构下，故障很有可能在进程内扩散，形成应用全局性的不可用。在微服务架构下，故障会被隔离在单个服务中。若设计良好，其他服务可通过重试、平稳退化等机制实现应用层面的容错。
### 扩展：
每个服务可以各自进行 x 扩展和 z 扩展，而且，每个服务可以根据自己的需要部署到合适的硬件服务器上。当应用的不同组件在扩展需求上存在差异时，微服务架构便体现出其灵活性，因为每个服务可以根据实际需求独立进行扩展。
# 为什么微服务架构需要 Spring Cloud

简单来说，服务化的核心就是将传统的一站式应用根据业务拆分成一个一个的服务，而微服务在这个基础上要更彻底地去耦合（不再共享 DB、KV，去掉重量级 ESB），并且强调 DevOps 和快速演化。这就要求我们必须采用与一站式时代、泛 SOA 时代不同的技术栈，而 Spring Cloud 就是其中的佼佼者。

DevOps 是英文 Development 和 Operations 的合体，他要求开发、测试、运维进行一体化的合作，进行更小、更频繁、更自动化的应用发布，以及围绕应用架构来构建基础设施的架构。这就要求应用充分的内聚，也方便运维和管理。这个理念与微服务理念不谋而合。

接下来我们从服务化架构演进的角度来看看为什么 Spring Cloud 更适应微服务架构。
### 从使用 nginx 说起
最初的服务化解决方案是给提供相同服务提供一个统一的域名，然后服务调用者向这个域名发送 HTTP 请求，由 Nginx 负责请求的分发和跳转。

![](/img/006tNc79ly1fqc316du2nj30eq0dgdfx.jpg)

使用 Dubbo 构建的微服务，已经可以比较好地解决上面提到的问题：

- 调用中间层变成了可选组件，消费者可以直接访问服务提供者。
- 服务信息被集中到 Registry 中，形成了服务治理的中心组件。
- 通过 Monitor 监控系统，可以直观地展示服务调用的统计信息。
- Consumer 可以进行负载均衡、服务降级的选择。

但是对于微服务架构而言，Dubbo 也并不是十全十美的：

- Registry 严重依赖第三方组件（zookeeper 或者 redis），当这些组件出现问题时，服务调用很快就会中断。
- Dubbo 只支持 RPC 调用。使得服务提供方与调用方在代码上产生了强依赖，服务提供者需要不断将包含公共代码的 jar 包打包出来供消费者使用。一旦打包出现问题，就会导致服务调用出错。

### 新的选择——Spring Cloud

作为新一代的服务框架，Spring Cloud 提出的口号是开发 “面向云环境的应用程序”，它为微服务架构提供了更加全面的技术支持。

根据微服务架构在各方面的要素，我们把 Spring Cloud 与 Dubbo 进行一番对比。

其实把 Spring Cloud 和 Dubbo 放一起对比有点不公平，Dubbo 只是实现了服务治理，而 Spring Cloud 下面有 23 个子项目（截止到 2018.4）分别覆盖了微服务架构下的方方面面，服务治理只是其中的一个方面，一定程度来说，Dubbo 只是 Spring Cloud Netflix 中的一个子集。但是在选择框架上，方案完整度恰恰是一个需要重点关注的内容。

   
&nbsp;|Dubbo|Spring Cloud
:-:|:-:|:-:
服务注册中心|Zookeeper|Spring Cloud Netflix Eureka
服务调用方式|RPC|REST API
服务监控|Dubbo-monitor|Spring Boot Admin
断路器|不完善|Spring Cloud Netflix Hystrix
服务网关|无|Spring Cloud Netflix Zuul
分布式配置|无|Spring Cloud Config
服务跟踪|无|Spring Cloud Sleuth
消息总线|无|Spring Cloud Bus
数据流|无|Spring Cloud Stream
批量任务|无|Spring Cloud Task
……|……|……

服务调用方式：Spring Cloud 抛弃了 Dubbo 的 RPC 通信，采用的是基于 HTTP 的 REST 方式。严格来说，这两种方式各有优劣。虽然从一定程度上来说，后者牺牲了服务调用的性能，但也避免了上面提到的原生 RPC 带来的问题。而且 REST 相比 RPC 更为灵活，服务提供方和调用方的依赖只依靠一纸契约，不存在代码级别的强依赖，这在强调快速演化的微服务环境下，显得更加合适。

服务注册和发现：Eureka 相比于 zookeeper，更加适合于服务发现的场景，这点会在下一篇会详细展开。

很明显，Dubbo 的功能只是 Spring Cloud 体系的一部分。Spring Cloud 的功能更加强大，涵盖面更广，而且作为 Spring 的拳头项目，它也能够与 Spring Framework、Spring Boot、Spring Data、Spring Batch 等其他 Spring 项目完美融合，这些对于微服务而言是至关重要的。前面提到，微服务背后一个重要的理念就是持续集成、快速交付，而在服务内部使用一个统一的技术框架，显然比把分散的技术组合到一起更有效率。更重要的是，Spring Cloud 是 Spring Source 的产物，Spring 社区的强大背书可以说是 Java 企业界最有影响力的组织了，除了 Spring Source 之外，还有 Pivotal 和 Netfix 是其强大的后盾与技术输出。另外，从目前 Spring Cloud 的被关注度和活跃度上来看，很有可能将来会成为微服务架构的标准框架
#### Spring Cloud 是微服务架构的最佳落地方案

有关这两个框架更详细的的对比可以看知乎和这两篇文章：
- [微服务架构的基础框架选择：Spring Cloud 还是 Dubbo？](http://blog.didispace.com/microservice-framework/)
- [阿里 Dubbo 疯狂更新，关 Spring Cloud 什么事？](http://www.ityouknow.com/springcloud/2017/11/20/dubbo-update-again.html)

# Spring Cloud 技术概览

下图展示了 Spring Cloud 的完整技术组成：

![](/img/SpringCloud.jpg)

- 服务治理：这是 Spring Cloud 的核心。目前 Spring Cloud 主要通过整合 Netflix 的相关产品来实现这方面的功能（Spring Cloud Netflix），包括用于服务注册和发现的 Eureka，调用断路器 Hystrix，调用端负载均衡 Ribbon，Rest 客户端 Feign，智能服务路由 Zuul，用于监控数据收集和展示的 Spectator、Servo、Atlas，用于配置读取的 Archaius 和提供 Controller 层 Reactive 封装的 RxJava。除此之外，针对于服务的注册和发现，除了 Eureka，Spring Cloud 也整合了 Consul 和 Zookeeper 作为备选，但是因为这两个方案在 CAP 理论上都遵循 CP 而不是 AP（下一篇会详细介绍这点），所以官方并没有推荐使用。

  Feign 和 RxJava 并不是 Netiflix 的产品，但是被整合到了 Spring Cloud Netflix 中。

- 分布式链路监控：Spring Cloud Sleuth 提供了全自动、可配置的数据埋点，以收集微服务调用链路上的性能数据，并发送给 Zipkin 进行存储、统计和展示。
- 消息组件：Spring Cloud Stream 对于分布式消息的各种需求进行了抽象，包括发布订阅、分组消费、消息分片等功能，实现了微服务之间的异步通信。Spring Cloud Stream 也集成了第三方的 RabbitMQ 和 Apache Kafka 作为消息队列的实现。而 Spring Cloud Bus 基于 Spring Cloud Stream，主要提供了服务间的事件通信（比如刷新配置）。
- 配置中心：基于 Spring Cloud Netflix 和 Spring Cloud Bus，Spring 又提供了 Spring Cloud Config，实现了配置集中管理、动态刷新的配置中心概念。配置通过 Git 或者简单文件来存储，支持加解密。
- 安全控制：Spring Cloud Security 基于 OAuth2 这个开放网络的安全标准，提供了微服务环境下的单点登录、资源授权、令牌管理等功能。
- 命令行工具：Spring Cloud Cli 提供了以命令行和脚本的方式来管理微服务及 Spring Cloud 组件的方式。
- 集群工具：Spring Cloud Cluster 提供了集群选主、分布式锁（暂未实现）、一次性令牌（暂未实现）等分布式集群需要的技术组件。

### Eureka

Eureka 是 Netflix 开源的一款提供服务注册和发现的产品，它提供了完整的 Service Registry 和 Service Discovery 实现。也是 Spring Cloud 体系中最重要最核心的组件之一。

用大白话讲，Eureka 就是一个服务中心，将所有的可以提供的服务都注册到它这里来管理，其它各调用者需要的时候去注册中心获取，然后再进行调用，避免了服务之间的直接调用，方便后续的水平扩展、故障转移等。



当然服务中心这么重要的组件一但挂掉将会影响全部服务，因此需要搭建 Eureka 集群来保持高可用性，生产中建议最少两台。随着系统的流量不断增加，需要根据情况来扩展某个服务，Eureka 内部已经提供均衡负载的功能，只需要增加相应的服务端实例既可。那么在系统的运行期间某个实例挂了怎么办？Eureka 内容有一个心跳检测机制，如果某个实例在规定的时间内没有进行通讯则会自动被剔除掉，避免了某个实例挂掉而影响服务。

因此使用了 Eureka 就自动具有了注册中心、负载均衡、故障转移的功能。

### Hystrix
在微服务架构中通常会有多个服务层调用，基础服务的故障可能会导致级联故障，进而造成整个系统不可用的情况，这种现象被称为服务雪崩效应。服务雪崩效应是一种因 “服务提供者” 的不可用导致 “服务消费者” 的不可用, 并将不可用逐渐放大的过程。

如下图所示：A 作为服务提供者，B 为 A 的服务消费者，C 和 D 是 B 的服务消费者。A 不可用引起了 B 的不可用，并将不可用像滚雪球一样放大到 C 和 D 时，雪崩效应就形成了。

![](https://ws2.sinaimg.cn/large/006tNc79ly1fqc5fkiehej30g60e30st.jpg)

在这种情况下就需要整个服务机构具有故障隔离的功能，避免某一个服务挂掉影响全局。在 Spring Cloud 中 Hystrix 组件就扮演这个角色。

Hystrix 会在某个服务连续调用 N 次不响应的情况下，立即通知调用端调用失败，避免调用端持续等待而影响了整体服务。Hystrix 间隔时间会再次检查此服务，如果服务恢复将继续提供服务。

### Hystrix Dashboard 和 Turbine

当熔断发生的时候需要迅速的响应来解决问题，避免故障进一步扩散，那么对熔断的监控就变得非常重要。熔断的监控现在有两款工具：Hystrix-dashboard 和 Turbine

Hystrix-dashboard 是一款针对 Hystrix 进行实时监控的工具，通过 Hystrix Dashboard 我们可以直观地看到各 Hystrix Command 的请求响应时间, 请求成功率等数据。但是只使用 Hystrix Dashboard 的话, 你只能看到单个应用内的服务信息, 这明显不够. 我们需要一个工具能让我们汇总系统内多个服务的数据并显示到 Hystrix Dashboard 上, 这个工具就是 Turbine.
监控的效果图如下：

![](https://ws1.sinaimg.cn/large/006tNc79ly1fqc5gk1wivj30gn0am0th.jpg)

### Spring Cloud Config

随着微服务不断的增多，每个微服务都有自己对应的配置文件。在研发过程中有测试环境、UAT 环境、生产环境，因此每个微服务又对应至少三个不同环境的配置文件。这么多的配置文件，如果需要修改某个公共服务的配置信息，如：缓存、数据库等，难免会产生混乱，这个时候就需要引入 Spring Cloud 另外一个组件：Spring Cloud Config。

Spring Cloud Config 是一个解决分布式系统的配置管理方案。它包含了 Client 和 Server 两个部分，Server 提供配置文件的存储、以接口的形式将配置文件的内容提供出去，Client 通过接口获取数据、并依据此数据初始化自己的应用。

其实就是 Server 端将所有的配置文件服务化，需要配置文件的服务实例去 Config Server 获取对应的数据。将所有的配置文件统一整理，避免了配置文件碎片化。配置中心 git 实例参考：配置中心 git 示例；

如果服务运行期间改变配置文件，服务是不会得到最新的配置信息，需要解决这个问题就需要引入 Refresh。可以在服务的运行期间重新加载配置文件，具体可以参考这篇文章：配置中心 svn 示例和 refresh

当所有的配置文件都存储在配置中心的时候，配置中心就成为了一个非常重要的组件。如果配置中心出现问题将会导致灾难性的后果，因此在生产中建议对配置中心做集群，来支持配置中心高可用性。

### Spring Cloud Bus
上面的 Refresh 方案虽然可以解决单个微服务运行期间重载配置信息的问题，但是在真正的实践生产中，可能会有 N 多的服务需要更新配置，如果每次依靠手动 Refresh 将是一个巨大的工作量，这时候 Spring Cloud 提出了另外一个解决方案：Spring Cloud Bus

Spring Cloud Bus 通过轻量消息代理连接各个分布的节点。这会用在广播状态的变化（例如配置变化）或者其它的消息指令中。Spring Cloud Bus 的一个核心思想是通过分布式的启动器对 Spring Boot 应用进行扩展，也可以用来建立一个或多个应用之间的通信频道。目前唯一实现的方式是用 AMQP 消息代理作为通道。

Spring Cloud Bus 是轻量级的通讯组件，也可以用在其它类似的场景中。有了 Spring Cloud Bus 之后，当我们改变配置文件提交到版本库中时，会自动的触发对应实例的 Refresh，具体的工作流程如下：
![](https://ws1.sinaimg.cn/large/006tNc79ly1fqc5hzoj1qj30hw0bqq38.jpg)

### Spring Cloud Zuul

在微服务架构模式下，后端服务的实例数一般是动态的，对于客户端而言很难发现动态改变的服务实例的访问地址信息。因此在基于微服务的项目中为了简化前端的调用逻辑，通常会引入 API Gateway 作为轻量级网关，同时 API Gateway 中也会实现相关的认证逻辑从而简化内部服务之间相互调用的复杂度。
![](https://ws3.sinaimg.cn/large/006tNc79ly1fqc5ihuft2j30pi0e6mxw.jpg)

Spring Cloud 体系中支持 API Gateway 落地的技术就是 Zuul。Spring Cloud Zuul 路由是微服务架构中不可或缺的一部分，提供动态路由，监控，弹性，安全等的边缘服务。Zuul 是 Netflix 出品的一个基于 JVM 路由和服务端的负载均衡器。

它的具体作用就是服务转发，接收并转发所有内外部的客户端调用。使用 Zuul 可以作为资源的统一访问入口，同时也可以在网关做一些权限校验等类似的功能。
### 链路跟踪
随着服务的越来越多，对调用链的分析会越来越复杂，如服务之间的调用关系、某个请求对应的调用链、调用之间消费的时间等，对这些信息进行监控就成为一个问题。在实际的使用中我们需要监控服务和服务之间通讯的各项指标，这些数据将是我们改进系统架构的主要依据。因此分布式的链路跟踪就变的非常重要，Spring Cloud 也给出了具体的解决方案：Spring Cloud Sleuth 和 Zipkin
![](https://ws1.sinaimg.cn/large/006tNc79ly1fqc5j15b71j30d60cj75i.jpg)

Spring Cloud Sleuth 为服务之间调用提供链路追踪。通过 Sleuth 可以很清楚的了解到一个服务请求经过了哪些服务，每个服务处理花费了多长时间。从而让我们可以很方便的理清各微服务间的调用关系。

Zipkin 是 Twitter 的一个开源项目，允许开发者收集 Twitter 各个服务上的监控数据，并提供查询接口

分布式链路跟踪需要 Sleuth+Zipkin 结合来实现

## 总结

我们从整体上来看一下 Spring Cloud 各个组件如何来配套使用：
![](/img/springcloud-组件.jpg)

从上图可以看出 Spring Cloud 各个组件相互配合，合作支持了一套完整的微服务架构。

- 其中 Eureka 负责服务的注册与发现，很好将各服务连接起来
- Hystrix 负责监控服务之间的调用情况，连续多次失败进行熔断保护
- Hystrix dashboard，Turbine 负责监控 Hystrix 的熔断情况，并给予图形化的展示
- Spring Cloud Config 提供了统一的配置中心服务
- 当配置文件发生变化的时候，Spring Cloud Bus 负责通知各服务去获取最新的配置信息
- 所有对外的请求和服务，我们都通过 Zuul 来进行转发，起到 API 网关的作用
- 最后我们使用 Sleuth+Zipkin 将所有的请求数据记录下来，方便我们进行后续分析
- 后续章节我们会详细介绍我们实际使用到的 Spring Cloud 组件。


最后附上原文地址感谢作者Yibo   [原文](https://windmt.com/2018/04/14/spring-cloud-0-microservices/)
