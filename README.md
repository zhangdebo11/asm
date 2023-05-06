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

关于为什么要修改配置，请参考 [problem.md](problem.md)

# 为应用开启ASM

# 如何为集群卸载ASM

[uninstall.md](uninstall.md)
