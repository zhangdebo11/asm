# istio sidecar 启动完成之前，manju、monaka 等容器会连接数据库失败


kubectl -n istio-system edit cm istio-asm-managed

```yaml
apiVersion: v1
data:
  mesh: |2-

    # This section can be updated with user configuration settings from https://istio.io/latest/docs/reference/config/istio.mesh.v1alpha1/
    # Some options required for ASM to not be modified will be ignored
    defaultConfig:
      holdApplicationUntilProxyStarts: true
kind: ConfigMap
metadata:
  name: istio-asm-managed
  namespace: istio-system
```

cm修改完成后，重建业务pod重新注入sidecar。

参考资料 https://istio.io/latest/docs/ops/common-problems/injection/#pod-or-containers-start-with-network-issues-if-istio-proxy-is-not-ready


# melonpan显示的客户端IP全都是127.0.0.6

melonpan 无法获取到客户端的 IP

kubectl -n istio-system edit cm istio-asm-managed

```yaml
apiVersion: v1
data:
  mesh: |2-

    # This section can be updated with user configuration settings from https://istio.io/latest/docs/reference/config/istio.mesh.v1alpha1/
    # Some options required for ASM to not be modified will be ignored
    defaultConfig:
      interceptionMode: TPROXY
kind: ConfigMap
metadata:
  name: istio-asm-managed
  namespace: istio-system
```

cm修改完成后，重建业务pod重新注入sidecar。

参考资料 https://www.cnblogs.com/tencent-cloud-native/p/16355019.html

# 在 melonpan 页面访问 monaka 文档失败

原因是，monaka会判断请求的referer的值。melonpan发送给monaka的referer值是 "xxx.com"格式，缺少协议信息，被istio认为无效并且删除了。

修改办法就是melonpan发送referer的值改为"http://xxx.com"格式

参考资料 https://www.envoyproxy.io/docs/envoy/latest/configuration/http/http_conn_man/headers#config-http-conn-man-headers-referer
