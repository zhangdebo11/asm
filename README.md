# 为集群开启ASM

集群要开启 VPC-native traffic routing，然后集群的feature打开“Anthos service mesh”

# 修改ASM配置

kubectl -n istio-system edit cm istio-asm-managed

```yaml
apiVersion: v1
data:
  mesh: |2-

    # This section can be updated with user configuration settings from https://istio.io/latest/docs/reference/config/istio.mesh.v1alpha1/
    # Some options required for ASM to not be modified will be ignored
    defaultConfig:
      interceptionMode: TPROXY
      holdApplicationUntilProxyStarts: true
kind: ConfigMap
metadata:
  name: istio-asm-managed
  namespace: istio-system
```

# 为应用开启ASM

## 为一个namespace开启

为namespace增加label `istio-injection=enabled` ，则该namespace下的pod在创建时会自动注入一个容器 `istio-proxy` 。

## 为一个应用开启/关闭

如果namespace开启了注入，但是namespace下的某个应用需要关闭注入，可以为pod设置label  `sidecar.istio.io/inject="false"` 。

相反，如果namespace关闭了注入，但是namespace下的某个应用需要开启注入，可以为pod设置label  `sidecar.istio.io/inject="true"` 。

# 其他

- [为什么要修改ASM配置](./docs/problem.md)
- [为集群卸载ASM](./docs/uninstall.md)
- [ASM记录访问日志](./docs/access-log.md)
- [APE集群升级ASM](./docs/APE-ASM-upgrade.md)
- [限流](./docs/rate-limit.md)
- [Ingress Gateway](./docs/ingressgateway.md)
