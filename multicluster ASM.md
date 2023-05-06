export PROJECT_1=smartcart-stagingization
export LOCATION_1=asia-northeast1-a
export CLUSTER_1=staging-asm-test
export CTX_1="gke_${PROJECT_1}_${LOCATION_1}_${CLUSTER_1}"

export PROJECT_2=smartcart-stagingization
export LOCATION_2=asia-northeast1
export CLUSTER_2=staging-asm-test-2
export CTX_2="gke_${PROJECT_2}_${LOCATION_2}_${CLUSTER_2}"


curl https://storage.googleapis.com/csm-artifacts/asm/asmcli_1.15 > asmcli

./asmcli create-mesh \
    smartcart-stagingization \
    ${PROJECT_1}/${LOCATION_1}/${CLUSTER_1} \
    ${PROJECT_2}/${LOCATION_2}/${CLUSTER_2}

报错
```
asmcli: Setting up necessary files...
asmcli: Creating temp directory...
asmcli:
asmcli: *****************************
asmcli: No output folder was specified with --output_dir|-D, so configuration and
asmcli: binaries will be stored in the following directory.
asmcli: /tmp/tmp.DQ2XWXGh2K
asmcli: *****************************
asmcli:
asmcli: Using  as the kubeconfig...
asmcli: Checking installation tool dependencies...
asmcli: Downloading kpt..
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
  0     0    0     0    0     0      0      0 --:--:-- --:--:-- --:--:--     0
100 11.8M  100 11.8M    0     0  7580k      0  0:00:01  0:00:01 --:--:-- 18.0M
asmcli: Downloading ASM..
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100 24.3M  100 24.3M    0     0  9882k      0  0:00:02  0:00:02 --:--:-- 9878k
asmcli: Downloading ASM kpt package...
fetching package "/asm" from "https://github.com/GoogleCloudPlatform/anthos-service-mesh-packages" to "asm"
fetching package "/samples" from "https://github.com/GoogleCloudPlatform/anthos-service-mesh-packages" to "samples"
asmcli: Checking for project smartcart-stagingization...
asmcli: Fetching/writing GCP credentials to /tmp/tmp.wFqPUNcFsV...
asmcli: Fetching/writing GCP credentials to /tmp/tmp.rbNvXs6lzg...
asmcli: Verifying cluster registration.
asmcli: [ERROR]: Cluster has memberships.hub.gke.io CRD but no identity provider specified.
Please ensure that the registered cluster has fleet workload identity enabled:
https://cloud.google.com/anthos/multicluster-management/fleets/workload-identity
```

经检查发现，标准集群的 CR Membership 比起autopilot集群缺少 identity_provider 和 workload_identity_pool 两个属性。

autopilot集群中的membership
```
apiVersion: hub.gke.io/v1
kind: Membership
metadata:
  labels:
    hub.gke.io/system: "true"
  name: membership
spec:
  identity_provider: https://container.googleapis.com/v1/projects/smartcart-stagingization/locations/asia-northeast1/clusters/staging-asm-test-2
  owner:
    id: //gkehub.googleapis.com/projects/495370126123/locations/global/memberships/staging-asm-test-2-1
  workload_identity_pool: smartcart-stagingization.svc.id.goog
```

标准集群中的membership
```
apiVersion: hub.gke.io/v1
kind: Membership
metadata:
  labels:
    hub.gke.io/system: "true"
  name: membership
spec:
  owner:
    id: //gkehub.googleapis.com/projects/495370126123/locations/global/memberships/staging-asm-test-1
```

手动修改标准集群中的membership，补充信息后再重试。

然后在两个集群中分别部署test服务、vs等资源。

不注入sidecar也能实现跨集群负载均衡。。。


集群A中创建namespace1，部署服务1，创建namespace2，部署服务2，
集群B中创建namespace1，部署服务1，创建namespace2，部署服务2，
四个服务全部注入sidecar，服务1访问服务2，有负载均衡效果。
服务1注入sidecar，服务2不注入，服务1访问服务2，有负载均衡效果。
服务1不注入sidecar，服务2注入，服务1访问服务2，没有负载均衡效果。


