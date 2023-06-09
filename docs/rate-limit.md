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

请参考目录 rate-limit-demo

# 3. 参考资料

- https://istio.io/latest/docs/tasks/policy-enforcement/rate-limit/
- https://www.aboutwayfair.com/tech-innovation/understanding-envoy-rate-limits
- https://discuss.istio.io/t/envoy-local-rate-limit-by-ip-address-using-remote-address-not-working/12718
