# 判断集群ASM用的是mesh_ca还是istio_ca

找到一个注入了sidecar的pod，查看其明细

```sh
kubectl -n spirytus get pod spirytus-sct-7f84d7997f-h2xwj -oyaml | grep CA_ADDR -A 1
```

如果是 mesh-CA， CA_ADDR的值是类似于这样的

```
meshca.googleapis.com:443
```

如果是 istio-CA， CA_ADDR的值是类似于这样的

```
istiod-asm-1172-1.istio-system.svc:15012
```

# ASM升级

```sh

# 准备：要拥有修改IAM policy的权限


# 下载工具
curl https://storage.googleapis.com/csm-artifacts/asm/asmcli_1.17 > asmcli
chmod  +x asmcli

# 开始安装ASM
./asmcli create-mesh \
    ssc-ape-staging \
    ssc-ape-staging/asia-northeast1-a/sct-spirytus-cluster
./asmcli install \
  --project_id ssc-ape-staging \
  --cluster_name sct-spirytus-cluster \
  --cluster_location asia-northeast1-a \
  --fleet_id ssc-ape-staging \
  --output_dir output_dir_APE_staging_sct-spirytus-cluster \
  --enable_all \
  --ca citadel    # 值为 citadel 表示 istio_CA ; 值为 mesh_ca 表示 mesh_CA



# 切换istio-ingressgateway到新版本
kubectl label namespace istio-system \
    istio-injection- istio.io/rev=asm-1172-1 \
    --overwrite
# 修改相关版本号
kubectl -n istio-system edit deploy istio-ingressgateway 


# 业务应用切换到新版本ASM，需要重建pod重新注入sidecar
kubectl label namespace spirytus \
    istio-injection- istio.io/rev=asm-1172-1 \
    --overwrite

kubectl -n spirytus rollout restart deploy xxx


# 删除旧版本的istio
kubectl -n istio-system delete cm istio-asm-1124-1 
kubectl -n istio-system delete cm istio-sidecar-injector-asm-1124-1 
kubectl delete mutatingwebhookconfigurations istio-sidecar-injector-asm-1124-1 
kubectl delete validatingwebhookconfigurations istio-validator-asm-1124-1-istio-system 
kubectl -n istio-system delete svc istiod-asm-1124-1 
kubectl -n istio-system delete deploy istiod-asm-1124-1 
kubectl -n istio-system delete hpa istiod-asm-1124-1 
kubectl -n istio-system delete pdb istiod-asm-1124-1 
kubectl -n istio-system delete IstioOperator installed-state-asm-1124-1 
kubectl -n istio-system delete sa istiod-asm-1124-1
kubectl -n istio-system delete role istiod-asm-1124-1
kubectl -n istio-system delete rolebinding istiod-asm-1124-1

```

# 参考资料

- https://cloud.google.com/service-mesh/docs/unified-install/upgrade?hl=zh-cn
