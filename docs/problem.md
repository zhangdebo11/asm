# istio sidecar 启动完成之前，manju、monaka 等容器会连接数据库失败

解决方案 https://istio.io/latest/docs/ops/common-problems/injection/#pod-or-containers-start-with-network-issues-if-istio-proxy-is-not-ready

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

# melonpan显示的客户端IP全都是127.0.0.6
melonpan 无法获取到客户端的 IP

解决办法：
要求是【standard集群】，修改istio-system命名空间下的cm istio-asm-managed，添加配置interceptionMode: TPROXY
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


# 在 melonpan 页面访问 monaka 文档失败

原因是，monaka会判断请求的referer的值。melonpan发送给monaka的referer值是 "xxx.com"格式，缺少协议信息，被istio认为无效并且删除了。
修改办法就是melonpan发送referer的值改为"http://xxx.com"格式
