# 1. 本地限流与全局限流

## 1.1 本地限流

![image](../image/local-rate-limit.png)

## 1.2 全局限流

![image](../image/global-rate-limit.png)

## 1.3 两种限流的区别


|  | 本地限流 | 全局限流 |
|--|---------|----------|
| 架构 | 不依赖rate-limit-server服务，istio-sidecar即可实现 | 依赖rate-limit-server服务 |
| 算法 | 令牌桶算法 | 固定窗口限流，即单位时间内允许n个请求 |
| 功能 | descriptors的value不能为空，因此，它不能对远程IP不确定、header值不确定的请求进行限流 | descriptors的value可以为空。当value为空时，如果两个请求的value值不同，则各自单独限流，互不挤占流量 |

# 2. Demo

请参考目录 demo-rate-limit

# 3. 现行方案

采用全局限流。

## 3.1 限流服务

限流服务由一个 rateLimitServer 服务和 redis 服务组成，redis可以使用我们自己部署的redis集群，rateLimitServer则需要连接redis记录访问次数。

![image](../image/ratelimit-redis.png)

rateLimitServer 使用argoCD管理，helm模板保存在`manju-helm`工程中。rateLimitServer连接redis的信息保存在其环境变量中。

## 3.2 限流规则管理

限流规则管理分成两部分：

- 第一部分是rateLimitServer的配置文件，它以configmap的形式挂载在pod中。为了便于管理，我们把限流规则分组成不同的domain，比如，manju的规则都在`raicart-manju`这个domain中，gulab的规则都在`raicart-gulab`这个domain中。
- 第二部分是EnvoyFilter资源，每个应用对应一个EnvoyFilter资源，EnvoyFilter需要指定对应的工作负载以及对应的domain。

注意：`/`，`/ping`，`/metrics` 接口在armor层做限流，不在ASM中。

# 4. 参考资料

- https://istio.io/latest/docs/tasks/policy-enforcement/rate-limit/
- https://www.aboutwayfair.com/tech-innovation/understanding-envoy-rate-limits
- https://discuss.istio.io/t/envoy-local-rate-limit-by-ip-address-using-remote-address-not-working/12718
- https://github.com/envoyproxy/ratelimit
