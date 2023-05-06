# 部署入口网关

准备示例代码
```sh
git clone https://github.com/zhangdebo11/asm-test.git
cd asm-test
```

创建namespace
```sh
kubectl apply -f samples/istio-ingressgateway/namespace.yaml
```

注意这个namespace带有label `istio.io/rev: asm-managed`
```yaml
apiVersion: v1
kind: Namespace
metadata:
  labels:
    istio.io/rev: asm-managed
  name: istio-gateway
```

部署ingressgateway。这个网关通过ingress对外暴露，并配置了GCP托管的证书。证书所签发的域名应包含此集群内所有需要对外暴露的服务域名，所有服务的访问流量均通过此网关。
```sh
kubectl apply -f samples/istio-ingressgateway/
```


# 部署示例应用

首先创建namespace

```sh
kubectl apply -f samples/online-boutique/kubernetes-manifests/namespaces
```

注意，为namespace增加label  `istio-injection: enabled` ，那么在这个namespace下的pod会自动注入sidecar

```sh
kubectl label ns xxx istio-injection=enabled

```

创建应用资源

```sh
kubectl apply -f samples/online-boutique/kubernetes-manifests/deployments
kubectl apply -f samples/online-boutique/kubernetes-manifests/services
```

# 通过ingressgateway访问示例应用

访问 https://test-asm-boutique.raicart.io
