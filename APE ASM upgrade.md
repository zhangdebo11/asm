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
  --ca citadel    # --ca mesh_ca



# 切换istio-ingressgateway到新版本
kubectl label namespace istio-system \
    istio-injection- istio.io/rev=asm-1172-1 \
    --overwrite
kubectl -n istio-system rollout restart deploy istio-ingressgateway

gcr.io/gke-release/asm/proxyv2:1.17.2-asm.1

# 业务应用切换到新版本ASM，需要重建pod重新注入sidecar
kubectl label namespace spirytus \
    istio-injection- istio.io/rev=asm-1172-1 \
    --overwrite



# 删除旧版本的istio
kubectl -n istio-system get cm istio-asm-1124-1 -oyaml > istio-asm-1124-1.cm.yaml
kubectl -n istio-system get cm istio-sidecar-injector-asm-1124-1 -oyaml > istio-sidecar-injector-asm-1124-1.cm.yaml
kubectl get mutatingwebhookconfigurations istio-sidecar-injector-asm-1124-1 -oyaml > istio-sidecar-injector-asm-1124-1.mutatingwebhookconfigurations.yaml
kubectl get validatingwebhookconfigurations istio-validator-asm-1124-1-istio-system -oyaml > istio-validator-asm-1124-1-istio-system.validatingwebhookconfigurations.yaml
kubectl -n istio-system get svc istiod-asm-1124-1 -oyaml > istiod-asm-1124-1.svc.yaml
kubectl -n istio-system get deploy istiod-asm-1124-1 -oyaml > istiod-asm-1124-1.deploy.yaml
kubectl -n istio-system get hpa istiod-asm-1124-1 -oyaml > istiod-asm-1124-1.hpa.yaml
kubectl -n istio-system get pdb istiod-asm-1124-1 -oyaml > istiod-asm-1124-1.pdb.yaml
kubectl -n istio-system get IstioOperator installed-state-asm-1124-1 -oyaml > installed-state-asm-1124-1.IstioOperator.yaml



kubectl delete -f istio-asm-1124-1.cm.yaml
kubectl delete -f  istio-sidecar-injector-asm-1124-1.cm.yaml
kubectl  delete -f  istio-sidecar-injector-asm-1124-1.mutatingwebhookconfigurations.yaml
kubectl  delete -f  istio-validator-asm-1124-1-istio-system.validatingwebhookconfigurations.yaml
kubectl  delete -f  istiod-asm-1124-1.svc.yaml
kubectl  delete -f  istiod-asm-1124-1.deploy.yaml
kubectl  delete -f  istiod-asm-1124-1.hpa.yaml
kubectl  delete -f  istiod-asm-1124-1.pdb.yaml
kubectl  delete -f  installed-state-asm-1124-1.IstioOperator.yaml


```


kubectl -n istio-system get IstioOperator installed-state-asm-1124-1




I am going to upgrade ASM for cluster sct-spirytus-cluster and spirytus-cluster in project RMP APE Staging, maybe 10-30 min downtime.
If any problem, please ping me.