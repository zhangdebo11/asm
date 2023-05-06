# 卸载ASM

## 取消sidecar注入

```sh
kubectl label namespace YOUR_NAMESPACE istio.io/rev-
kubectl label namespace YOUR_NAMESPACE istio-injection-
```
重启pod后sidecar消失

## 移除hook

kubectl delete validatingwebhookconfiguration istiod-istio-system-mcp
kubectl delete mutatingwebhookconfiguration RELEASE_CHANNEL

## 删除相关命名空间
kubectl delete namespace istio-system asm-system --ignore-not-found=true

## 删除相关CRD
kubectl get crd  | grep istio

## 取消fleet注册
gcloud container fleet memberships unregister staging-manju-melonpan-cluster \
   --project=smartcart-stagingization \
   --gke-cluster=asia-northeast1/staging-manju-melonpan-cluster
