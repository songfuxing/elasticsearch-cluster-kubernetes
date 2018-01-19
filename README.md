# Elasticsearch 集群在k8s中的高可用部署，以及数据持久化方案
## elasticsearch集群部署方案
elasticsearch使用单一角色进行部署，使用单一角色的好处是：增强es集群的稳定性、可用性，以及便于维护。
```
master节点：3个
data节点：3个
client节点：2个
```
## es的docker镜像
es镜像基于5.6.4版本，并打入了结巴分词工具。详情查看[`es-images`](es-images)

## kubernetes的数据持久化方案
在k8s中，为了增强数据可用性，master节点和data节点的部署使用有状态的[`StatefulSet`](https://kubernetes.io/docs/concepts/abstractions/controllers/statefulsets/)，而不是无状态的deployment。stateful状态的pod如果失败，将会自动救活和重启，而不是重新部署pod。这样就可以保证数据在pod重启之后不会丢失。在数据层面，使用网络存储的方式保存数据，把k8s中的数据存储到远程的glusterfs中。创建步骤如下：
- 创建[`StorageClass`](http://blog.kubernetes.io/2016/10/dynamic-provisioning-and-storage-in-kubernetes.html)，自动为statefulset中的pod申请pvc. 同时向glusterfs集群申请volume。参考[`glusterfs-storageclass.yaml`](gluster-storage-class.yaml)
- 创建普通服务，用于管理服务发现[`es-discovery-svc.yaml`](es-discovery-svc.yaml),以及维护es-client节点状态，对外提供api[`es-svc.yaml`](es-svc.yaml)
- 创建headless-service, 用户维护stateful节点的状态。参考[`es-master-svc.yaml`](es-master-svc.yaml) 和[`es-data-svc.yaml`](es-data-svc.yaml)
- 首先创建master 节点，等到master 节点起来之后，在创建data和client节点。参考[`es-data-stateful-gfs.yaml`](es-data-stateful-gfs.yaml) 和 [`es-master-stateful-gfs.yaml`](es-master-stateful-gfs.yaml)

## 部署
首先创建storageclass，启动服务发现和master节点：
```
kubectl create -f gluster-storage-class.yaml
kubectl create -f es-discovery-svc.yaml
kubectl create -f es-svc.yaml
kubectl craete -f es-master-svc.yaml
kubectl create -f es-master-stateful-gfs.yaml
```
等到master 节点起来之后，部署client和data节点
```
kubectl create -f es-client.yaml
kubectl create -f es-data-svc.yaml
kubectl create -f es-data-stateful-gfs.yaml
```
## 测试
得到server的cluster-ip：
```
 kubectl get svc elasticsearch
NAME            TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)    AGE
elasticsearch   ClusterIP   10.100.220.56   <none>        9200/TCP   3m
```
测试es的健康状态：
```
$ curl http://10.100.220.56:9200/_cluster/health?pretty

```

