要记录请求header内容，输出到日志中

# 方法一

为特定服务开启访问日志

```yaml
apiVersion: telemetry.istio.io/v1alpha1
kind: Telemetry
metadata:
  name: test-asmlog
  namespace: default
spec:
  selector:
    matchLabels:
      app.kubernetes.io/name: monaka
  accessLogging:
    - providers:
      - name: envoy
```

设置日志格式

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
        # context: GATEWAY
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
      app.kubernetes.io/name: macaron
    #   app: test-nginx1

```

这个方法存在问题：logCritical日志级别太高，改成其他级别则无法输出，没有找到修改日志级别的方法
