要记录请求header内容，输出到日志中

# 方法一

## 设置日志格式

可选步骤，有默认格式

kubectl -n istio-system edit cm istio-asm-managed

```yaml
apiVersion: v1
data:
  mesh: |2-

    # This section can be updated with user configuration settings from https://istio.io/latest/docs/reference/config/istio.mesh.v1alpha1/
    # Some options required for ASM to not be modified will be ignored
    accessLogFormat: "[%START_TIME%] \"%REQ()%\" \n"
kind: ConfigMap
metadata:
  name: istio-asm-managed
  namespace: istio-system

```

## 为特定服务开启访问日志

```yaml
apiVersion: telemetry.istio.io/v1alpha1
kind: Telemetry
metadata:
  name: test-asmlog
  namespace: default
spec:
  selector:
    matchLabels:
      app.kubernetes.io/name: monaka  # pod选择器
  accessLogging:
    - providers:
      - name: envoy
```



# 方法二

使用lua脚本

```yaml
apiVersion: networking.istio.io/v1alpha3
kind: EnvoyFilter
metadata:
  name: access-log-filter
  namespace: istio-system
spec:
  configPatches:
    - applyTo: HTTP_FILTER
      match:
        context: SIDECAR_INBOUND
        listener:
          filterChain:
            filter:
              name: envoy.filters.network.http_connection_manager
              subFilter:
                name: envoy.filters.http.router
      patch:
        operation: INSERT_BEFORE
        value:
          name: envoy.lua
          typed_config:
            '@type': type.googleapis.com/envoy.extensions.filters.http.lua.v3.Lua
            inlineCode: |
              function envoy_on_request(request_handle)
                  local h = ""
                  for key, value in pairs(request_handle:headers()) do
                      h = h .. ", \"" .. key .. "\": \"" .. value .. "\""
                  end
                  h = string.gsub(h, ", " , "" , 1)
                  h = "{" .. h .. "}"
                  request_handle:logCritical(h)
              end
  workloadSelector:
    labels:
      app.kubernetes.io/name: macaron  # pod选择器

```

# 参考资料

- https://istio.io/latest/docs/tasks/observability/logs/access-log/
- https://www.envoyproxy.io/docs/envoy/latest/configuration/observability/access_log/usage#
- https://dev.to/aws-builders/understanding-istio-access-logs-2k5o
- https://github.com/envoyproxy/envoy/issues/11592
