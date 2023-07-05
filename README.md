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

# 开启后

## 维护mesh_id到terraform模板

集群开启ASM后，集群会增加一个label `mesh_id`，把这个label增加到terraform模板中

```yaml
resource "google_container_cluster" "yakiimo" {
  resource_labels = {
    "mesh_id" = "proj-973347293934"
  }
}
```

## 监控数据问题

ASM开启后，发现yakiimo的API监控数据为空了，解决办法是把chart的filter从 "app.kubernetes.io/name=yakiimo" 改为 "pod_name=~yakiimo.*"。

问题原因可能是ASM的开启导致了监控数据的变化。

综上，开启ASM后请注意查看监控数据是否正常。

# 其他

- [为什么要修改ASM配置](./docs/problem.md)
- [为集群卸载ASM](./docs/uninstall.md)
- [ASM记录访问日志](./docs/access-log.md)
- [APE集群升级ASM](./docs/APE-ASM-upgrade.md)
- [限流](./docs/rate-limit.md)
