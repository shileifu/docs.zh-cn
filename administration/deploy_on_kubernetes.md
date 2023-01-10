# Deploy StarRocks in Kubernetes
本页面展示如何在kubernetes部署一个starrocks集群服务。在kubernetes之上部署StarRocks可从以下方式选择其中一种完成。
* [Manual StarRocks on Kubernetesg](#operator)
* [Helm StarRocks on Kubernetes](#helm)

## 开始之前
### Kuernetes相关术语： 

| Feature                                                                               | Description                                                    |
|:--------------------------------------------------------------------------------------|:---------------------------------------------------------------|  
| [node](https://kubernetes.io/docs/concepts/architecture/nodes/)                       | 是 K8s 集群中资源的实际供给方，是调度 Pod 运行的场所。在生产环境中，一个 K8s 集群通常由多个 Node 组成。 |  
| [pod](https://kubernetes.io/docs/concepts/workloads/pods/)                            | 是可以在 K8s 中创建、管理、可部署的最小计算单元，是一组共享存储网络资源的容器。                     |
| [StatefulSet](https://kubernetes.io/docs/concepts/workloads/controllers/statefulset/) | 是一组Pod作为有状态应用单元的组合。                                            |
| [StorageClass](https://kubernetes.io/docs/concepts/storage/storage-classes/)                                                                      | 是存储pv的模板抽象。                                                    |  
### Limitations
operator 1.2 版本之前适应kubernetes的版本1.23-1.25

## 1. Start Kubernetes
你可以使用自管的[Google Kubernetes Engine (GKE)](#gke) 服务或者自管的[Amazon Elastic Kubernetes Service (EKS)](#eks)快速部署kubernetes集群。

### Hosted GKE
<div id="gke"> </div>

1. GKE创建之前，完成在[Google Kubernetes Engine Quickstart](https://cloud.google.com/kubernetes-engine/docs/deploy-app-cluster) 描述的所有前置操作。  
   这其中包括gcloud用户创建和删除kubernetes集群，还有kubectl在工作的机器上管理你kubernetes集群。
2. 选择合适的region创建kubernetes集群
    ```gcloud container clusters create cockroachdb --machine-type n2-standard-4 --region {region-name} --num-nodes 1```

<div id="eks"> </div>

### Hosted EKS
1. EKS创建之前，完成在[EKS Getting Started](https://docs.aws.amazon.com/eks/latest/userguide/getting-started-eksctl.html) 描述的所有前置操作。
2. 使用eksctl创建集群，例如：  
   ``` eksctl create cluster --name cockroachdb --nodegroup-name standard-workers --node-type m5.xlarge --nodes 3 --nodes-min 1 --nodes-max 4 --node-ami auto ```

<div id="operator"> </div>

## 2. 部署StarRocks
### [使用配置的方式](https://github.com/StarRocks/starrocks-kubernetes-operator)
1. 为operator部署[custom resource definition (CRD)](https://kubernetes.io/docs/concepts/extend-kubernetes/api-extension/custom-resources/#customresourcedefinitions)
```shell
kubectl apply -f https://raw.githubusercontent.com/StarRocks/starrocks-kubernetes-operator/main/deploy/starrocks.com_starrocksclusters.yaml
```
2. 默认情况下Operator会被部署到`starrocks`默认的namespace中，operator会管理所有namespace下的StarRocks集群服务。
   如果想改变默认的一些配置：  
    a. 下载operator的manifest  
       ```curl -O https://raw.githubusercontent.com/StarRocks/starrocks-kubernetes-operator/main/deploy/operator.yaml```  
    b. 使用其他namespace，编辑manifest中所有`namespace: cockroach-operator-system`部分。  
    c. 如果更改了默认配置，使用如下命令来部署operator:  
       ```kubectl apply -f operator.yaml```  
    d. 使用默认的命令：  
       ```kubectl apply -f https://raw.githubusercontent.com/StarRocks/starrocks-kubernetes-operator/main/deploy/operator.yaml```
3. 检测operator是否正在运行:  
   ```kubectl -n starrocks get pods```  

4. 可参考[operator](https://github.com/StarRocks/starrocks-kubernetes-operator/tree/main/examples/starrocks) 仓库中使用用例配置和相关[api](https://github.com/StarRocks/starrocks-kubernetes-operator/blob/main/doc/api.md) 解释说明根据自己需要配置部署StarRocks集群。默认使用如下：
   ```kubectl apply -f https://raw.githubusercontent.com/StarRocks/starrocks-kubernetes-operator/blob/main/examples/starrocks/starrocks-fe-and-be.yaml```

<div id="helm"> </div>

### [使用Helm的方式](https://github.com/StarRocks/helm-charts)
1. 添加StarRocks的helm仓库。
```helm repo add starrocks-community https://starrocks.github.io/helm-charts ```
2. 更新仓库确保你使用的是最新的[starrocks chart](https://github.com/StarRocks/helm-charts) 
3. operator和集群的配置信息都在helm chart [values.yaml](https://github.com/StarRocks/helm-charts/blob/main/charts/kube-starrocks/values.yaml)中。
  a. 创建一个本地的YAML文件(例如：my-values.yaml), 可以被用来覆盖在values.yaml中默认的配置。
  b. 如果全都使用默认配置不用创建。
4. 安装StarRocks Helm Chart
   a. ```helm install -f {custom-values}.yaml starrocks starrocks-community/kube-starrocks```
   b. ``` helm install starrocks starrocks-community/kube-starrocks```

### 确认集群初始化成功
``` kubectl -n {custom-namespace} get pods ```
custom-namespace是在使用配置方式或者helm方式部署过程中指定的namespace，默认是`starrocks`.