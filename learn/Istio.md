# Istio架构中每个部分的作用:
## 1.Sidecar(默认为Envoy)
Envoy是一个高性能代理,用于调解服务网络中所有服务的入站和出站流量,和对应的应用服务部署在同一个Kubernetes的Pod中
Envoy调解所有出入应用服务的流量.所有经过Envoy的流量行为都会调用Mixer,而Mixer提供一组描述请求和请求周围环境的Attribute
根据Envoy的配置和Attribute,Mixer会调用各种后台的基础设施资源
而Attribute又可以在Mixer中用于决策使用何种策略,并发送给监控系统,以提供整个网格行为的信息

## 2.Pilot
Pilot为Sidecar提供"服务发现"功能,并管理高级路由(如A/B测试和金丝雀部署)和故障处理(超时.重试.熔断)的流量
Pilot将这些"高级"的流量行为转换为详尽的Sidecar配置项,并在运行时将他们配置到Sidecar中
Pilot将服务发现机制提炼为供数据面使用的API,即任何Sidecar都可以使用的标准格式
这种松耦合的设计模式使Istio能在多种环境下运行,同时保持用于流量管理操作的相同

## 3.Mixer
Mixer是一个独立于平台的组件,通过从Sidecar和一些其他服务处收集数据,进而在整个Service Mesh上控制访问和执行策略
Sidecar请求级别的Attribute被发送到Mixer进行评估
Mixer中还包括一个灵活的插件,使其能接入各种主机环境和基础设施的后段,并得到Sidecar代理和Istio所管理的服务

#### Mixer的设计特点:
1.无状态:Mixer本身是无状态的,它没有持久化存储的管理功能
2.高可用:Mixer被设计成高度可用的组件,任何单独的Mixer实例实现->99.999%的正常运行时间
3.缓存和缓冲: Mixer能够积累大量短暂的瞬间状态


## 4.Citadel
Citadel通过内置身份和凭证管理提供"服务间"和"最终用户"身份验证,Citadel可用于升级服务网络中未加密的流量,并能够为运维人员提供基于服务表示而不是网络层的强制执行策略

#### Sidecar的服务发现及传输规则机制,主要遵循以下流程:
1. 平台适配器从Kubernetes API server中获取Pod的注册信息(Istio本身不剧本服务注册的功能,它需要通过平台适配器和特定的平台结合才能具有完整的服务发现的功能)
2. 用户通过Pilot的Rules API对所有被发现的服务的Sidecar进行各种高级特性(包括路由规则,HTTP层的流量管理等)的配置
3. 用户配置的这些规则被抽象模型翻译成低级设置
4. Sidecar API(Envoy API)将这些翻译好的低级配置通过discovery API分发到每个微服务的Sidecar实例中


#### Istio的安全体系的组成:
* Citadel:秘钥与证书的管理系统
* Sidecar与边界网关:为客户端与服务端接口提供安全服务
* Pilot:用于向网关分发安全认证策略与安全命名信息(Secure Naming Information)
* Mixer:用于对链路请求进行鉴权与审计

Citadel提供的是安全性非常强的服务到服务以及用户到用户的认证(Authentication)功能,内置角色与密钥管理,支持基本角色的权限控制


Istio提供一种健全认证.策略服务,TLS传输安全机制来提供基础的安全平台,同时提供鉴权.认证.审计功能接口,帮助用户保护自己的服务及传输链路

在请求开始的时候,双方通过它们的身份交换密钥以实现双向认证
客户端通过验证服务端的身份来确保不会路由到欺诈服务,这种验证方式被称为安全目录


Envoy不仅是Istio提供的服务网络中的数据承载,也是安全机制的最终执行者

通过yaml文件定义好权限策略,然后Istio通过Pilot下发给各个Envoy
同时Pilot还会监听配置的变化,当出现变动时及时通知部署在服务旁边的Envoy代理
在Envoy里,每个到达的请求都会经过一个鉴权模块,并根据之前配置的策略对请求进行鉴权处理,当前不符合鉴权时返回拒收信息





##### 服务运行流程:
1.每个微服务的Pod通过Kubernetes提供的服务注册机制进行注册
2.已注册的Service A的Envoy通过Pilot中汇总的Envoy信息发现欲访问的Service B
3.Service A的Envoy根据设置好的负载均衡或流量管理规则,对Service B的Envoy发起请求
4.Service B的Envoy根据设置好的规则确认是否接受Service A发出的请求

#### Attribute是Istio策略和遥测功能中的基本概念
Attribute是一小段数据,用于描述服务请求的一系列属性(列如特定请求的大小,操作相应代码.请求来自的IP);通过Attribute,我们就可以东西服务的方方面面
Mixer提供一组描述请求及其周围环境的Attribute,基于Envoy的配置和相应的Attribute,Mixer会调用各种基础设施后端


#### 遥测功能的信息汇总流程(所有微服务都是通过Envoy进行通信的):
1.Envoy通过调用将Attribute传递给Mixer
2.Mixer得到Envoy的Attribute,并根据Attribute调用各种后端资源
3.同时,Mixer可以汇总所有Envoy的Attribute,用于集群范围的监控以及运维人员提供可观测性


MOSN(Modular Observable Smart Network) 模块化可观察的只能网络
Service Mesh的未来,是将服务间通讯的能力下沉到基础设施
Service Mesh和Spring Cloud的本质差异,不仅仅在于将服务间通讯从应用程序剥离出来,更在于一路下沉到基础设施层
并充分利用底层基础设施的能力

Istio的CRD由Galley读取,而K8s的原生资源(Service/Deployment/Pod等)暂时还是由Pilot读取

Service Mesh的技术关注点在于服务间通讯,其目标是剥离客户端SDK,为应用减负,提供的能力主要包括安全性,路由,策略执行,流量管理等
Serverless技术的关注点在于服务运维,目标是客户无需关注服务运维,提供服务实例的自动伸缩,以及按照实际使用付费

云原生的代表技术: 容器,服务网络,微服务,不可变基础设施和声明式API


#### 完成一次RPC调用的步骤:
1.底层的网络通信协议处理
2.解决寻址问题
3.请求/响应过程中参数的序列化和反序列化工作

#### 服务治理的三个要素:
服务的动态注册与发现
服务的扩容评估
服务的升/降级处理

#### 分布式调用跟踪系统的四个关键设计目标:
1.服务性能低损耗
2.业务代码低侵入
3.监控界面可视化
4.数据分析准实时

#### 埋点需要收集的信息:
TraceId
SpanId
ParentSpanId
RpcContext中包含的数据信息
服务执行异常时的堆栈信息
Trace中各个Span过程的开始时间和结束时间
