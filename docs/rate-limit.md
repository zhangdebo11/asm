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

# 3. 其他

## 3.1 限制入口流量

以下实现了，当外部流量访问容器的 80 端口时，会被限流

```yaml
apiVersion: networking.istio.io/v1alpha3
kind: EnvoyFilter
metadata:
  name: test-rate-limit
  namespace: default
spec:
  workloadSelector:
    labels:
      app: test-nginx1
  configPatches:
    - applyTo: HTTP_FILTER  # HTTP
      match:
        context: SIDECAR_INBOUND    # 入口流量
        listener:
          portNumber: 80   # 要限制的容器端口
          filterChain:
            filter:
              name: "envoy.filters.network.http_connection_manager"     # HTTP
      patch:
        operation: INSERT_BEFORE
        value:
          name: envoy.filters.http.local_ratelimit
          typed_config:
            "@type": type.googleapis.com/udpa.type.v1.TypedStruct
            type_url: type.googleapis.com/envoy.extensions.filters.http.local_ratelimit.v3.LocalRateLimit
            value:
              stat_prefix: http_local_rate_limiter
              token_bucket:
                max_tokens: 1
                tokens_per_fill: 1
                fill_interval: 60s
              filter_enabled:
                runtime_key: local_rate_limit_enabled
                default_value:
                  numerator: 100
                  denominator: HUNDRED
              filter_enforced:
                runtime_key: local_rate_limit_enforced
                default_value:
                  numerator: 100
                  denominator: HUNDRED

```

另一个办法，基于TCP

```yaml
apiVersion: networking.istio.io/v1alpha3
kind: EnvoyFilter
metadata:
  name: test-rate-limit
  namespace: default
spec:
  workloadSelector:
    labels:
      app: test-nginx1
  configPatches:
    - applyTo: NETWORK_FILTER
      match:
        context: SIDECAR_INBOUND
      patch:
        operation: INSERT_BEFORE
        value:
          name: envoy.filters.network.local_ratelimit
          typed_config:
            "@type": type.googleapis.com/udpa.type.v1.TypedStruct
            type_url: type.googleapis.com/envoy.extensions.filters.network.local_ratelimit.v3.LocalRateLimit
            value:
              stat_prefix: rate_limited
              token_bucket:
                max_tokens: 1
                tokens_per_fill: 1
                fill_interval: 60s
              filter_enabled:
                runtime_key: local_rate_limit_enabled
                default_value:
                  numerator: 100
                  denominator: HUNDRED
              filter_enforced:
                runtime_key: local_rate_limit_enforced
                default_value:
                  numerator: 100
                  denominator: HUNDRED

```


## 3.2 限制出口流量

以下实现了，当容器访问外部的 1111 端口时，会被限流

```yaml
apiVersion: networking.istio.io/v1alpha3
kind: EnvoyFilter
metadata:
  name: test-rate-limit
  namespace: default
spec:
  workloadSelector:
    labels:
      app: test-nginx3
  configPatches:
    - applyTo: HTTP_FILTER
      match:
        context: SIDECAR_OUTBOUND
        listener:
          portNumber: 1111
          filterChain:
            filter:
              name: "envoy.filters.network.http_connection_manager"
      patch:
        operation: INSERT_BEFORE
        value:
          name: envoy.filters.http.local_ratelimit
          typed_config:
            "@type": type.googleapis.com/udpa.type.v1.TypedStruct
            type_url: type.googleapis.com/envoy.extensions.filters.http.local_ratelimit.v3.LocalRateLimit
            value:
              stat_prefix: http_local_rate_limiter
              token_bucket:
                max_tokens: 1
                tokens_per_fill: 1
                fill_interval: 60s
              filter_enabled:
                runtime_key: local_rate_limit_enabled
                default_value:
                  numerator: 100
                  denominator: HUNDRED
              filter_enforced:
                runtime_key: local_rate_limit_enforced
                default_value:
                  numerator: 100
                  denominator: HUNDRED
```

另一个办法，基于TCP

```yaml
apiVersion: networking.istio.io/v1alpha3
kind: EnvoyFilter
metadata:
  name: test-rate-limit
  namespace: default
spec:
  workloadSelector:
    labels:
      app: test-nginx3
  configPatches:
    - applyTo: NETWORK_FILTER
      match:
        context: SIDECAR_OUTBOUND
        listener:
          portNumber: 1111
      patch:
        operation: INSERT_BEFORE
        value:
          name: envoy.filters.network.local_ratelimit
          typed_config:
            "@type": type.googleapis.com/udpa.type.v1.TypedStruct
            type_url: type.googleapis.com/envoy.extensions.filters.network.local_ratelimit.v3.LocalRateLimit
            value:
              stat_prefix: rate_limited
              token_bucket:
                max_tokens: 1
                tokens_per_fill: 1
                fill_interval: 60s
              filter_enabled:
                runtime_key: local_rate_limit_enabled
                default_value:
                  numerator: 100
                  denominator: HUNDRED
              filter_enforced:
                runtime_key: local_rate_limit_enforced
                default_value:
                  numerator: 100
                  denominator: HUNDRED

```

# 4. 参考资料

- https://istio.io/latest/docs/tasks/policy-enforcement/rate-limit/
- https://www.aboutwayfair.com/tech-innovation/understanding-envoy-rate-limits
- https://discuss.istio.io/t/envoy-local-rate-limit-by-ip-address-using-remote-address-not-working/12718
