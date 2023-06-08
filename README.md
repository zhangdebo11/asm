# 为集群开启ASM

集群的feature打开“Anthos service mesh”

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

更多配置 https://istio.io/latest/docs/reference/config/istio.mesh.v1alpha1/

# 为应用开启ASM

## 为一个namespace开启

为namespace增加label `istio-injection=enabled` ，则该namespace下的pod在创建时会自动注入一个容器 `istio-proxy` 。

## 为一个应用开启/关闭

如果需要为某个应用单独设置，可以为pod设置label  `sidecar.istio.io/inject` ，值可以是 `true` 或者 `false` 。

# 其他

- [为什么要修改ASM配置](./docs/problem.md)
- [为集群卸载ASM](./docs/uninstall.md)
- [ASM记录访问日志](./docs/access-log.md)
- [APE集群升级ASM](./docs/APE-ASM-upgrade.md)
- [限流](./docs/rate-limit.md)
