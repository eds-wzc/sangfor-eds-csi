# Sangfor EDS Block Storage CSI 1.0.0

[Container Storage Interface (CSI)](https://github.com/container-storage-interface/), driver and provisioner for the Sangfor EDS block storage blockplugin.

# Overview

Sangfor EDS CSI plugins implement an interface between CSI and Container Orchestrator (CO) - EDS cluster.

Current Sangfor EDS CSI plugins was tested in Kubernetes environment (requires Kubernetes 1.14+), and

the code can be run with any CSI enabled CO.

For details about configuration and deployment of the EDS CSI blockplugin, please refer the documentation.

For example usage of the CSI blockplugin, see examples below.

Before to go, you should have installed Sangfor EDS cluster.

You can get latest version of Sangfor EDS CSI blockplugin at docker hub by running docker CLI

download from http://nas.sangfor.org:5000/sharing/0ObohCaO4

**<font size=5>you should load images manually </font>**

`$ docker load -i eds-block-csi-1.0.0.tar.gz`

## Version Support

| Kubernetes | CSI   | chart | Sangfor EDS                        |
| ---------- | ----- | ----- | ---------------------------------- |
| v1.14      | 1.0.0 | 1.0.0 | 3.0.5.32custom_iscsi_csi or 3.0.6+ |
| v1.15      | 1.0.0 | 1.0.0 | 3.0.5.32custom_iscsi_csi or 3.0.6+ |
| v1.16+     | 1.0.0 | 1.0.1 | 3.0.5.32custom_iscsi_csi or 3.0.6+ |

# Deployment #

In this section，you will learn how to deploy the EDS CSI blockplugin.

## Prepare EDS cluster ##

Login to you EDS dashboard, your dashboard address should be https://your_domain_ip/index

### Dashbord ###

[dashboard](https://github.com/eds-wzc/sangfor-eds-csi/blob/master/image/dashbord.png)

### Create Block Storage ###

You need to remember the pool name

[create-block](https://github.com/eds-wzc/sangfor-eds-csi/blob/master/image/create-storage.png)

### Configure ISCSI ###

You need to remember the iqn and port

[configure-iscsi](https://github.com/eds-wzc/sangfor-eds-csi/blob/master/image/Configure-ISCSI.png)

### Configure access IP ###

You need to remember the access ip

[configure-access-ip](https://github.com/eds-wzc/sangfor-eds-csi/blob/master/image/Configure-access-IP.png)

 ### Create Account ###

You need to remember the account and password

[create-account](https://github.com/eds-wzc/sangfor-eds-csi/blob/master/image/Create-Account.png)

## Edit Yaml

You should download [chart](https://github.com/eds-wzc/sangfor-eds-csi/blob/master/charts/) and edit value.yaml.

The meaning of each parameter in the file, you can read the README.md.

`$ tar-zxvf eds-csi-provisioner && cd eds-csi-provisioner  `

`$ cat README.md`

`$ vi value.yaml`

## Depoly ##

If you have edited the yaml, you can depoly csi plugin ,create StorageClass and create secret

`$ helm version`

```
[root@k8s-node1 ~]# helm version
version.BuildInfo{Version:"v3.2.4", GitCommit:"0ad800ef43d3b826f31a5ad8dfbb4fe05d143688", GitTreeState:"clean", GoVersion:"go1.13.12"}
```

`$ helm install eds -f values.yaml . `

```
[root@k8s-node1 ysf]# helm install eds eds-csi-provisioner/
NAME: eds
LAST DEPLOYED: Wed Oct 28 05:38:06 2020
NAMESPACE: default
STATUS: deployed
REVISION: 1
TEST SUITE: None
NOTES:
To verify that vs-csi-provisioner has started, run:

  kubectl --namespace=default get pods -l "app=eds-block-csi-provisioner,release=eds"
```

### Verify Pod 

`$ kubectl get pod -o wide | grep csi  `

```
[root@k8s-node1 ysf]# kubectl get pod -o wide | grep csi
eds-eds-block-csi-provisioner-4n4d5                         2/2     Running   0          10m     10.174.13.81   k8s-node2   <none>           <none>
eds-eds-block-csi-provisioner-attacher-6cfb7c7f5b-7qw8h     2/2     Running   0          10m     10.244.2.73    k8s-node3   <none>           <none>
eds-eds-block-csi-provisioner-nmrxr                         2/2     Running   0          10m     10.174.13.82   k8s-node3   <none>           <none>
eds-eds-block-csi-provisioner-provisioner-8c7d56484-qrrwt   2/2     Running   0          10m     10.244.1.75    k8s-node2   <none>           <none>
```

### Verfiy StorageClass

`$ kubectl get sc -o wide `

```
[root@k8s-node1 ysf]# kubectl get sc -o wide
NAME                            PROVISIONER                 AGE
eds-eds-block-csi-provisioner   eds.csi.block.sangfor.com   10m
```

```
[root@k8s-node1 ysf]# kubectl get sc eds-eds-block-csi-provisioner -o yaml
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  annotations:
    meta.helm.sh/release-name: eds
    meta.helm.sh/release-namespace: default
  creationTimestamp: "2020-10-28T09:38:06Z"
  labels:
    app: eds-block-csi-provisioner
    app.kubernetes.io/managed-by: Helm
    chart: eds-block-csi-provisioner-1.0.1
    heritage: Helm
    release: eds
  name: eds-eds-block-csi-provisioner
  resourceVersion: "1399546"
  selfLink: /apis/storage.k8s.io/v1/storageclasses/eds-eds-block-csi-provisioner
  uid: 4895923b-1901-11eb-b45c-0050568c8739
parameters:
  acl: 10.174.13.80,10.174.13.81,10.174.13.82
  csi.storage.k8s.io/node-publish-secret-name: eds-eds-block-csi-provisioner-sc-secret
  csi.storage.k8s.io/node-publish-secret-namespace: default
  csi.storage.k8s.io/provisioner-secret-name: eds-eds-block-csi-provisioner-sc-secret
  csi.storage.k8s.io/provisioner-secret-namespace: default
  desc: iscsi
  fsType: ext4
  iqn: iqn.2018-11.vmirq.com.sangfor.eds
  iscsiInterface: default
  policyId: MuWJr+acrOm7mOiupOetlueVpQ==
  targetPortal: 10.174.41.36:3260
provisioner: eds.csi.block.sangfor.com
reclaimPolicy: Delete
volumeBindingMode: Immediate
```

### Verfiy Secret

`$ kubectl get secret -o wide `

```
[root@k8s-node1 ysf]# kubectl get secret -o wide
NAME                                                    TYPE                                  DATA   AGE
default-token-5cnvn                                     kubernetes.io/service-account-token   3      11d
eds-eds-block-csi-provisioner-attacher-token-jh6nv      kubernetes.io/service-account-token   3      11m
eds-eds-block-csi-provisioner-nodeplugin-token-jvczj    kubernetes.io/service-account-token   3      11m
eds-eds-block-csi-provisioner-provisioner-token-xhksp   kubernetes.io/service-account-token   3      11m
eds-eds-block-csi-provisioner-sc-secret                 Opaque                                4      11m
sh.helm.release.v1.eds.v1                               helm.sh/release.v1                    1      11m
```

```
[root@k8s-node1 ysf]# kubectl get secret eds-eds-block-csi-provisioner-sc-secret -o yaml
apiVersion: v1
data:
  restfulpassword: dGVzdDEyMy4=
  restfulurl: aHR0cHM6Ly8xMC4xNzQuMzAuMTEw
  restfulusername: dGVzdA==
  restfulvolumeid: dGVzdA==
kind: Secret
metadata:
  annotations:
    meta.helm.sh/release-name: eds
    meta.helm.sh/release-namespace: default
  creationTimestamp: "2020-10-28T09:38:06Z"
  labels:
    app: eds-block-csi-provisioner
    app.kubernetes.io/managed-by: Helm
    chart: eds-block-csi-provisioner-1.0.1
    heritage: Helm
    release: eds
  name: eds-eds-block-csi-provisioner-sc-secret
  namespace: default
  resourceVersion: "1399544"
  selfLink: /api/v1/namespaces/default/secrets/eds-eds-block-csi-provisioner-sc-secret
  uid: 48944004-1901-11eb-b45c-0050568c8739
type: Opaque
```

Congratulation! you have finished the deployment. Now you can use them to provide Sangfor EDS ISCSI service.

## Usage ##

In this section，you will learn how to provision ISCSI with Sangfor EDS CSI driver. 

Here will Assumes that you have installed Sangfor EDS Cluster.

## PVC 

### Edit yaml for PersistentVolumeClaim ###

```
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: eds-pvc01
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 2Gi
  storageClassName: eds-eds-block-csi-provisioner
```

### Create pvc ###

`$ kubectl apply -f pvc.yaml`

### Verify pv && pvc

Run kubectl check command

`$ kubectl get pvc`

```
[root@k8s-node1 ysf]# kubectl get pvc
NAME        STATUS    VOLUME                                     CAPACITY   ACCESS MODES   STORAGECLASS                    AGE
eds-pvc01   Bound     pvc-e447fce2-1901-11eb-b45c-0050568c8739   2Gi        RWO            eds-eds-block-csi-provisioner   90s
```

`$ kubectl get pv`

```
[root@k8s-node1 ysf]# kubectl get pv
NAME                                       CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS   CLAIM               STORAGECLASS                    REASON   AGE
pvc-e447fce2-1901-11eb-b45c-0050568c8739   2Gi        RWO            Delete           Bound    default/eds-pvc01   eds-eds-block-csi-provisioner            2m30s
```

## Depoly

### Edite yaml for deploy

```
[root@k8s-node1 ysf]# cat depoly.yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx
spec:
  containers:
  - image: harbor.msa.io/vs/nginx
    imagePullPolicy: Always
    name: nginx
    ports:
    - containerPort: 80
      protocol: TCP
    volumeMounts:
      - mountPath: /var/www
        name: iscsi-volume
  volumes:
  - name: iscsi-volume
    persistentVolumeClaim:
      claimName: eds-pvc01
```

## Create deploy

`$ kubectl create -f deploy.yaml`

### Verify deploy

 `$ kubectl get pods`

```
[root@k8s-node1 ysf]# kubectl get pods -o wide | grep nginx
nginx                                                       1/1     Running   0          38s     10.244.2.74    k8s-node3   <none>           <none>
```

