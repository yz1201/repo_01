# taxi ol-04

### 今日导读

~~~
1，CAP
2，三级缓存
3，自我保护机制
4，服务测算
5，Eureka Server端源码
~~~

###  CAP源码定位

~~~html
启动注册类里边
1，从其他peer拉取注册表
2，启动定时剔除任务
3，自我保护模式   
// Copy registry from neighboring eureka node
int registryCount = this.registry.syncUp();
从其他peer（对等eureka节点）拉取注册表。看源码，并不满足C一致性条件，因为一个server只有启动时才会去其他节点拉取节点的注册信息，所以后边注册的是拉不到的。
3，P:分区容错性，网络情况不好的情况下，还是可以拉取到注册表中节点信息进行调用的。？这是分区容错？这不是高可用？--所谓分区容错，应该指的集群间各个节点的状态不是强相关，一个server节点死掉了不影响其他server节点的服务。
~~~

### 自我保护剔除

~~~
1，开关 激活条件，15分钟内心跳失败比例低于**，最后1分钟内心跳大于阈值
2，阈值 count * 2 * 设置的阈值
server源码：
	剔除（本质也是下线），长时间无心跳的服务，eureka server将它从注册表中evict
	注册 ApplicationResource addInstance
	续约 InstanceResource renew
	下线 InstanceResource cancel
	集群间同步
		PeerAwareInstanceRegistryImpl
 	replicateToPeers(Action.Register, info.getAppName(), info.getId(), info, null, isReplication);
 		1,注册，第一个节点注册进来，null（二进宫true）
 		new InstanceReplicationTask(targetHost, Action.Register, info, null, true)
 		2,续约：一直同步，所有集群。
 		        ReplicationTask replicationTask = new InstanceReplicationTask(targetHost, Action.Heartbeat, info, overriddenStatus, false) {
		3，取消下线一样，都会传递
		4，剔除，没有同步这项业务。
	拉取注册表 
		全量拉取 ALL_APPS
		增量拉取 ALL_APPS_DELTA
	
	Lease<InstanceInfo> 租约 InstanceInfo 服务实例
~~~

### 服务测算

~~~
20个微服务，每个服务部署5个实例。100个服务实例
注册之后，默认每分钟有心跳200次。一天288000的心跳访问。
每个client还会拉取注册列表，拉取间隔默认30s，一天288000访问。故一天能有57w对注册中心的访问
Eureka 用的数据结构ConcurrentHashMap 每秒差不多能处理100000的存取。算上其他代码对性能的影响，也远远顶得住几百个服务的注册，拉取注册表的并发访问。
~~~

### EurekaServer

~~~
配置文件中的区域配置。
	地区，可用区。
	同机房高可用，同城双活+异地多活。
	作用：减少网络延迟。不跨网络，效率高。
	region： 

单体：高可用，多区域
源码剖析
	注册
	拉取
	下线
	续约
	剔除
	心跳
	集群同步
~~~





