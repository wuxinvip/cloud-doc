# 分布式网关中心

# 应用场景介绍

*   在很多场景下我们的服务实例是不直接暴露在外网中的、而是通过代理方式
*   比如：反向代理nginx、通过upstream和location完成服务转发、还可以进行过滤
*   那么我们使用java实现的网关优点呢、可以结合注册中心代理实例、直接从注册中心获取实例信息、还可以在网关进行统一的过滤验证、统计等等。


## 配置

```yaml
#启动类
@EnableZuulServer
【前置处理器Pre filters、返回处理器Post filters、错误拦截处理器SendErrorFilter】

#properties
zuul
  routes:
    api: /api/**
#url转发
#zuul.routes.api-a-url.path=/api-a/**
#zuul.routes.api-a-url.url=http://127.0.0.1:9000/

#结合eureka 进行服务转发
zuul.routes.agent.path=/project/**
zuul.routes.agent.serviceId=project-center

zuul.routes.user-center.path=/uc/**
zuul.routes.user-center.serviceId=user-center

zuul.routes.eureka=/eureka/**
zuul.routes.eureka.serviceId=eureka-server

#此外结合eureka 
eureka.client.service-url.defaultZone=http://eureka.wuxinvip.com/


#以及结合config配置【以下配置需要放到bootstrap.properties中】

#可以通过eureka注册中心获取配置中心
spring.application.name=api-gateway
#eureka.client.service-url.defaultZone=http://eureka.wuxinvip.com/
#spring.cloud.config.discovery.enabled=true
#spring.cloud.config.discovery.service-id=CONFIG-SERVER

#也可以采用此种配置获取uri地址
spring.application.name=api-gateway
spring.cloud.config.uri=http://config.wuxinvip.com

#熔断配置
hystrix:
  command:
    myusers-service:
      execution:
        isolation:
          thread:
            timeoutInMilliseconds: ...
#service超时设置
myusers-service:
  ribbon:
    NIWSServerListClassName: com.netflix.loadbalancer.ConfigurationBasedServerList
    listOfServers: https://example1.com,http://example2.com
    ConnectTimeout: 1000
    ReadTimeout: 3000
    MaxTotalHttpConnections: 500
    MaxConnectionsPerHost: 100

ribbon:
  eureka:
    enabled: false

users:
  ribbon:
    listOfServers: example.com,google.com
    retryableStatusCodes: 404,502
    
zuul:
  forceOriginalQueryStringEncoding: true
  decodeUrl: false

```

*   自定义拦截器等

```java



/**
 * @title:
 * @description:
 * @author: 无心
 * @date:2019/4/17 13:03
 * @location com.wuxinvip.apigateway.route.DemoFilter.class
 */
public class DemoFilter extends ZuulFilter {

    private static Logger logger = LoggerFactory.getLogger(DemoFilter.class);
    
    //前置过滤器
    @Override
    public String filterType() {
        return null;
    }
    //优先级，数字越大，优先级越低
    @Override
    public int filterOrder() {
        return 0;
    }
    //是否执行该过滤器，true代表需要过滤
    @Override
    public boolean shouldFilter() {
        return false;
    }

    @Override
    public Object run() throws ZuulException {
        RequestContext ctx = RequestContext.getCurrentContext();
        HttpServletRequest request = ctx.getRequest();

        logger.info("send {} request to {}", request.getMethod(), request.getRequestURL().toString());

        //获取传来的参数accessToken
        Object accessToken = request.getParameter("token");
        
        //TODO
        //toke验证 可以选择连接redis、也可以选择连接权限组件、
        
        //这里return的值没有意义，zuul框架没有使用该返回值
        return null;
    }
}

```

*   指定服务实例特殊处理
```

/**
 * @title:
 * @description:
 * @author: wuxin
 * @date:2019/4/17 13:27
 * @location com.wuxinvip.zuul.fallback.DemoFallBack.class
 */
public class DemoFallBack extends FallBack implements FallbackProvider {

    private static Logger logger = LoggerFactory.getLogger(DemoFallBack.class);

    //指定要处理的 service。
    @Override
    public String getRoute() {
        return "project-center";
    }

    //返回响应
    @Override
    public ClientHttpResponse fallbackResponse(String route, Throwable cause) {
        if (cause != null && cause.getCause() != null) {
            String reason = cause.getCause().getMessage();
            logger.info("Excption {}",reason);
        }
        return fallbackResponse();
    }
}

```

## 流程说明


#   缺点

*   对文件上传支持并不是很好


