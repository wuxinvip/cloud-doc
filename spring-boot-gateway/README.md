# 分布式网关中心

### [官方文档](https://cloud.spring.io/spring-cloud-static/spring-cloud-gateway/2.1.0.RELEASE/single/spring-cloud-gateway.html)

# 应用场景介绍

*   在很多场景下我们的服务实例是不直接暴露在外网中的、而是通过代理方式
*   比如：反向代理nginx、通过upstream和location完成服务转发、还可以进行过滤
*   那么我们使用java实现的网关优点呢、可以结合注册中心代理实例、直接从注册中心获取实例信息、还可以在网关进行统一的过滤验证、统计等等。

*   gateway以spring webflux 的HandlerMapping 作为底层路由设计
*   Spring Cloud Gateway包含许多内置的Route Predicate工厂 、 所有route动作都和可以匹配不同的请求


## 配置

```yaml
#开关
spring.cloud.gateway.enabled=false
spring:
  cloud:
    gateway:
      routes:
      - id: after_route
        uri: http://example.org
        #期望代理 该时间以后所有服务
        predicates:
        - After=2017-01-20T17:42:47.789-07:00[America/Denver]
        - Before=2017-01-20T17:42:47.789-07:00[America/Denver]
        - Between=2017-01-20T17:42:47.789-07:00[America/Denver], 2017-01-21T17:42:47.789-07:00[America/Denver]
        - Cookie=chocolate, ch.p
        - Header=X-Request-Id, \d+
        - Host=**.somehost.org,**.anotherhost.org
        - Method=GET
        - Path=/foo/{segment},/bar/{segment}
        - Query=foo, ba.
        - RemoteAddr=192.168.1.1/24
        filters：
        -  AddRequestHeader = X-Request-Foo,Bar
        - PrefixPath=/mypath
        - PreserveHostHeader
        #redis 令牌桶限流
        - name: RequestRateLimiter
          args:
            # 数值可以配置到配置中心获取 实时控制令牌数量
            redis-rate-limiter.replenishRate: 10
            redis-rate-limiter.burstCapacity: 20
        #网关重试
        - name: Retry
          args:
            retries: 3
            statuses: BAD_GATEWAY
        #请求大小限制
        - name: RequestSize
          args:
            maxSize: 5000000
        # 重定向
        -  RedirectTo = 302 ， http ：//acme.org
        #保存session
        - SaveSession
      httpclient:
        ssl:
          useInsecureTrustManager: true
          trustedX509Certificates:
          - cert1.pem
          - cert2.pem
server:
  ssl:
    enabled: true
    key-alias: scg
    key-store-password: scg1234
    key-store: classpath:scg-keystore.p12
    key-store-type: PKCS12
```
*   java-config
```
/**
     * @title: 全局filter
     * @description: 
     * @author: lianyanjin
     * @date: 2019/5/30 14:21
     * @param 
     * @return 
     */
    @Bean
    @Order(-1)
    public GlobalFilter a() {
        return (exchange, chain) -> {
            logger.info("first pre filter");
            return chain.filter(exchange).then(Mono.fromRunnable(() -> {
                logger.info("third post filter");
            }));
        };
    }

    @Bean
    @Order(0)
    public GlobalFilter b() {
        return (exchange, chain) -> {
            logger.info("second pre filter");
            return chain.filter(exchange).then(Mono.fromRunnable(() -> {
                logger.info("second post filter");
            }));
        };
    }

    @Bean
    @Order(1)
    public GlobalFilter c() {
        return (exchange, chain) -> {
            logger.info("third pre filter");
            return chain.filter(exchange).then(Mono.fromRunnable(() -> {
                logger.info("first post filter");
            }));
        };
    }
```

## 流程说明

*   gateway设计原理

[handller流处理过滤](image/gateway-设计原理.png)


# 介绍
    
*   gateway 基于spring webflux流处理设计网关、匹配原则进行请求过滤


# 设计原理