---
title: 一文给你道破Eureka服务发现慢的原因，深度剖析Eureka客户端服务发现原理以及 Eureka服务端服务剔除原理
date: 2023-07-03 15:41:08
tags: java
---

## 先直接给出配置让你尝鲜
> EurekaServer端配置
```
eureka:
    server:
        #Eureka Server会定时(间隔值是eureka.server.eviction-interval-timer-in-ms，默认60s)进行检查,如果发现实例在在一定时间
        #(此值由客户端设置的eureka.instance.lease-expiration-duration-in-seconds定义，默认值为90s)内没有收到心跳，则会注销此实例。
        #我们这里配置每秒钟去检测一次，驱除失效的实例
        eviction-interval-timer-in-ms: 1000
        #关闭一级缓存，让客户端直接从二级缓存去读取，省去各缓存之间的同步的时间
        use-read-only-response-cache: false
```


> EurekaClient端(应用端)配置
```
eureka:
    client:
        service-url:
            defaultZone: http://localhost:8761/eureka/
        # EurekaClient每隔多久从EurekaServer拉取一次服务列表，默认30秒，这里修改为2秒钟从注册中心拉取一次
        registry-fetch-interval-seconds: 2
        #租约期限以及续约期限的配置
    instance:
        #租约到期，服务失效时间，默认值90秒，服务超过90秒没有发生心跳，EurekaServer会将服务从列表移除
        #这里修改为6秒
        lease-expiration-duration-in-seconds: 6
        #租约续约间隔时间，默认30秒，这里修改为3秒钟
        lease-renewal-interval-in-seconds: 3
```


> 这里是Ribbon缓存实例列表的刷新间隔，默认30秒钟，这里修改为每秒钟刷新一次实例信息
```
ribbon:
    ServerListRefreshInterval: 1000

```


## 下面给你剖析原理
> Eureka服务端详解
* 服务端缓存
如图所示

![服务端缓存](..\imgs\eureka.png)

服务注册到注册中心后，服务实例信息是存储在注册表中的，也就是内存中。但Eureka为了提高响应速度，在内部做了优化，加入了两层的缓存结构，将Client需要的实例信息，直接缓存起来，获取的时候 直接从缓存中拿数据然后响应给 Client。

第一层缓存是readOnlyCacheMap，readOnlyCacheMap是采用ConcurrentHashMap来存储数据的，主要负责定时与readWriteCacheMap进行数据同步，默认同 步时间为 30 秒一次。

第二层缓存是readWriteCacheMap，readWriteCacheMap采用Guava来实现缓存。缓存过期时间默认为180秒，当服务下线、过期、注册、状态变更等操作都会清除此缓存中的数据。

第三层是数据存储层。

Client获取服务实例数据时，会先从一级缓存中获取，如果一级缓存中不存在，再从二级缓存中获取，如果二级缓存也不存在，会触发缓存的加载，从存储层拉取数据到缓存中，然后再返回给 Client。

Eureka 之所以设计二级缓存机制，也是为了提高 Eureka Server 的响应速度，缺点是缓存会导致 Client 获取不到最新的服务实例信息，然后导致无法快速发现新的服务和已下线的服务。

了解了服务端的实现后，想要解决这个问题就变得很简单了，我们可以缩短只读缓存的更新时间 (eureka.server.response-cache-update-interval-ms)让服务发现变得更加及时，或者直接将只读缓 存关闭(eureka.server.use-read-only-response-cache=false)，多级缓存也导致Client层面(数据一致性)很薄弱。

* 客户端缓存

客户端缓存主要分为两块内容，一块是 Eureka Client 缓存，一块是 Ribbon 缓存。

**Eureka Client缓存** ，EurekaClient负责跟EurekaServer进行交互，在EurekaClient中的 com.netflix.discovery.DiscoveryClient.initScheduledTasks() 方法中，初始化了一个 CacheRefreshThread 定时任务专⻔用来拉取 Eureka Server 的实例信息到本地。所以我们需要缩短这个定时拉取服务信息的时间间隔(此值在客户端配置eureka.client.registryFetchIntervalSeconds) 来快速发现新的服务

**Ribbon缓存**，Ribbon会从EurekaClient中获取服务信息，ServerListUpdater是Ribbon中负责服务实例 更新的组件，默认的实现是PollingServerListUpdater，通过线程定时去更新实例信息。定时刷新的时 间间隔默认是30秒，当服务停止或者上线后，这边最快也需要30秒才能将实例信息更新成最新的。我们可以将这个时间调短一点，比如 3 秒。

刷新间隔的参数是通过 getRefreshIntervalMs 方法来获取的，方法中的逻辑也是从 Ribbon的配置中进行取值的。所以我们需要缩短这个更新间隔（此值在客户端配置ribbon.ServerListRefreshInterval)来快速的更新Ribbon缓存实例列表

将这些服务端缓存和客户端缓存的时间全部缩短后，跟默认的配置时间相比，快了很多。我们通过调整 参数的方式来尽量加快服务发现的速度，但是还是不能完全解决报错的问题，间隔时间设置为3秒，也还是会有间隔。所以我们一般都会开启重试功能，当路由的服务出现问题时，可以重试到另一个服务来 保证这次请求的成功。

### 服务端缓存部分源码如下：

```
/**
 *The class that is responsible for caching registry information that will be
 *queried by the clients.
 */
public class ResponseCacheImpl implements ResponseCache {
    //一级缓存
    private final ConcurrentMap<Key, Value> readOnlyCacheMap = new ConcurrentHashMap<Key, Value>();
    //二级缓存（Guava实现）
    private final LoadingCache<Key, Value> readWriteCacheMap;
    //数据存储层
    private final AbstractInstanceRegistry registry;
}
```
### 客户端缓存更新部分源码如下:

Eureka Client缓存刷新部分源码

    //Eureka Client缓存刷新部分源码
    /**
     * Initializes all scheduled tasks.
     * 在实例化com.netflix.discovery.DiscoveryClient被调用
     */
    private void initScheduledTasks() {
        if (clientConfig.shouldFetchRegistry()) {
            // registry cache refresh timer
            int registryFetchIntervalSeconds = clientConfig.getRegistryFetchIntervalSeconds();
            int expBackOffBound = clientConfig.getCacheRefreshExecutorExponentialBackOffBound();
            scheduler.schedule(
                    new TimedSupervisorTask(
                            "cacheRefresh",
                            scheduler,
                            cacheRefreshExecutor,
                                //从哪个Eureka客户端拉取实例列表的间隔时间
                                        //通过eureka.client.registryFetchIntervalSeconds可配置
                            registryFetchIntervalSeconds,
                            TimeUnit.SECONDS,
                            expBackOffBound,
                                //执行刷新的Runable定时任务
                            new CacheRefreshThread()
                    ),
                        //从哪个Eureka客户端拉取实例列表的间隔时间
                        //通过eureka.client.registryFetchIntervalSeconds可配置
                    registryFetchIntervalSeconds, TimeUnit.SECONDS);
        }

        if (clientConfig.shouldRegisterWithEureka()) {
            int renewalIntervalInSecs = instanceInfo.getLeaseInfo().getRenewalIntervalInSecs();
            int expBackOffBound = clientConfig.getHeartbeatExecutorExponentialBackOffBound();
            logger.info("Starting heartbeat executor: " + "renew interval is: {}", renewalIntervalInSecs);

            // Heartbeat timer
            scheduler.schedule(
                    new TimedSupervisorTask(
                            "heartbeat",
                            scheduler,
                            heartbeatExecutor,
                            renewalIntervalInSecs,
                            TimeUnit.SECONDS,
                            expBackOffBound,
                            new HeartbeatThread()
                    ),
                    renewalIntervalInSecs, TimeUnit.SECONDS);

            // InstanceInfo replicator
            instanceInfoReplicator = new InstanceInfoReplicator(
                    this,
                    instanceInfo,
                    clientConfig.getInstanceInfoReplicationIntervalSeconds(),
                    2); // burstSize

            statusChangeListener = new ApplicationInfoManager.StatusChangeListener() {
                @Override
                public String getId() {
                    return "statusChangeListener";
                }

                @Override
                public void notify(StatusChangeEvent statusChangeEvent) {
                    if (InstanceStatus.DOWN == statusChangeEvent.getStatus() ||
                            InstanceStatus.DOWN == statusChangeEvent.getPreviousStatus()) {
                        // log at warn level if DOWN was involved
                        logger.warn("Saw local status change event {}", statusChangeEvent);
                    } else {
                        logger.info("Saw local status change event {}", statusChangeEvent);
                    }
                    instanceInfoReplicator.onDemandUpdate();
                }
            };

            if (clientConfig.shouldOnDemandUpdateStatusChange()) {
                applicationInfoManager.registerStatusChangeListener(statusChangeListener);
            }

            instanceInfoReplicator.start(clientConfig.getInitialInstanceInfoReplicationIntervalSeconds());
        } else {
            logger.info("Not registering with Eureka server per configuration");
        }
    }   
### Ribbon缓存刷新部分源码

```
//PollingServerListUpdater,Ribbon缓存刷新部分源码
   @Override
    public synchronized void start(final UpdateAction updateAction) {
        if (isActive.compareAndSet(false, true)) {
            final Runnable wrapperRunnable = new Runnable() {
                @Override
                public void run() {
                    if (!isActive.get()) {
                        if (scheduledFuture != null) {
                            scheduledFuture.cancel(true);
                        }
                        return;
                    }
                    try {
                        updateAction.doUpdate();
                        lastUpdated = System.currentTimeMillis();
                    } catch (Exception e) {
                        logger.warn("Failed one update cycle", e);
                    }
                }
            };

            scheduledFuture = getRefreshExecutor().scheduleWithFixedDelay(
                    wrapperRunnable,
                    initialDelayMs,
                    refreshIntervalMs,//默认30秒，通过ribbon.ServerListRefreshInterval来配置更小的值来快速更新Ribbon实例缓存
                    TimeUnit.MILLISECONDS
            );
        } else {
            logger.info("Already active, no-op");
        }
    }
```

#### 服务下线
* 当服务正常关闭操作时，会发送服务下线的REST请求给EurekaServer。
* 服务中心接受到请求后，将该服务置为下线状态
#### 失效剔除
Eureka Server 中会有定时任务去检测失效的服务，将服务实例信息从注册表中移除，也可以将这个失效检测的时间缩短，这样服务下线后就能够及时从注册表中清除。

* Eureka Server会定时(间隔值是eureka.server.eviction-interval-timer-in-ms，默认60s)进行检查，
如果发现实例在在一定时间
(此值由客户端设置的eureka.instance.lease-expiration-duration-in-seconds定义，默认值为90s)内没有收到心跳，
则会注销此实例。

>  Eureka客户端

### 服务提供者(也是Eureka客户端)
* 服务提供者(也是Eureka客户端)要向EurekaServer注册服务，并完成服务续约等工作
* 服务注册详解(服务提供者)
  * 当我们导入了eureka-client依赖坐标，配置Eureka服务注册中心地址
  * 服务在启动时会向注册中心发起注册请求，携带服务元数据信息
  * Eureka注册中心会把服务的信息保存在Map中。
* 服务续约详解(服务提供者)
  * 服务每隔30秒会向注册中心续约(心跳)一次(也称为报活)，如果没有续约，租约在90秒后到期，然后服务会被失效。
  * 每隔30秒的续约操作我们称之为心跳检测往往不需要我们调整这两个配置

```
#向Eureka服务中心集群注册服务 
eureka:
  #租约期限以及续约期限的配置
  instance:
    #租约到期，服务失效时间，默认值90秒，服务超过90秒没有发生心跳，EurekaServer会将服务从列表移除
    #这里修改为6秒
    lease-expiration-duration-in-seconds: 6
    #租约续约间隔时间，默认30秒，这里修改为3秒钟
    lease-renewal-interval-in-seconds: 3
```

### 服务消费者(也是Eureka客户端)
* 服务消费者每隔30秒服务会从注册中心中拉取一份服务列表，这个时间可以通过配置修改。往往不需要我们调整
  * 服务消费者启动时，从 EurekaServer服务列表获取只读备份，缓存到本地;
  * 默认每隔30秒，会重新获取并更新数据；
  * 每隔30秒的时间可以通过配置eureka.client.registry-fetch-interval-seconds修改，如下。

```
#向Eureka服务中心集群注册服务 
eureka:
    client:
      # EurekaClient每隔多久从EurekaServer拉取一次服务列表，默认30秒，这里修改为2秒钟从注册中心拉取一次
        registry-fetch-interval-seconds: 2
```


## 客户端缓存见Eureka服务端详情章节
> 至此您应该明白Eureka的服务发现机制了吧

### 最后再说说Eureka自我保护机制
> 服务提供者 —> 注册中心

* 定期的续约(服务提供者和注册中心通信)，假如服务提供者和注册中心之间的网络有点问题，
不代表 服务提供者不可用，不代表服务消费者无法访问服务提供者，所以有自我保护的机制
* 如果在15分钟内超过85%的客户端节点都没有正常的心跳，那么Eureka就认为客户端与注册中心出现了网络故障，
Eureka Server自动进入自我保护机制。
* 为什么会有自我保护机制?
  * 默认情况下，如果Eureka Server在一定时间内(默认90秒)没有接收到某个微服务实例的心跳， Eureka Server将会移除该实例。
  但是当网络分区故障发生时，微服务与Eureka Server之间无法正常通信，而微服务本身是正常运行的，此时不应该移除这个微服务，
  所以引入了自我保护机制。
  服务中心⻚面会显示如下提示信息
  * 紧急情况！Eureka可能错误地声称实例已启动，而事实并非如此，续约低于阈值，因此实例不会为了安全而过期

### 当处于自我保护模式时
* 不会剔除任何服务实例(可能是服务提供者和EurekaServer之间网络问题)，保证了大多数服务依 然可用
* Eureka Server仍然能够接受新服务的注册和查询请求，但是不会被同步到其它节点上，保证当前节点依然可用，
当网络稳定时，当前Eureka Server新的注册信息会被同步到其它节点中。
* 在Eureka Server工程中通过eureka.server.enable-self-preservation配置可用关停自我保护，默认值是打开
```
eureka:
  server:
    enable-self-preservation: false # 关闭自我保护模式(缺省为打开)
```

如果有大批量的集群且存在网络分区，强烈建议开启自我保护机制