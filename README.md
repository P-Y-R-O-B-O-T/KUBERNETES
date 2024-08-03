## KUBERNETES ARCHITECTURE
* `Nodes`: Hosts on which kubernetes is installed, previously called minions
* `Cluster`: A goupu of nodes grouped together
* `Master`: Another node which is configured as master/controller

> [!IMPORTANT]
> **KUBERNETES COMPONENTS**
>    - `API server`: user, CLI, management services interect with it
>    - `etcd service`: Distributed reliable key value store to store all data to manage the cluster, also ensured logging to avoid conflict
>    - `kubelet`: Agent that runs on each node in cluster, responsible for ensuring all containers are running as expected
>    - `Container runtime`: Underlying software to run container (generally docker) but others are also available like rocket and cri-o
>    - `Controller`: Brain of cluster, react to events in cluster such as failure of container, endpoints and many more, also makes decisions to bring back containers alive
>    - `Scheduler`: Distributes work and containers between nodes

* Master node have a `kube-apiserver` thats why it becomes master
* Kubelet agent on other nodes enable them to interact with master node
* All the information gathered is stored on master in `etcd` key value store

## DOCKER VS CONTAINERD
* Docker uses `containerd` or container daemon but the `containerd` can run independently of docker and it can be directly be utilised by kubernetes
* `containerd` is not as user friendly for us to use, so we have a tool called `nerdctl` which wraps over `containerd` and provides docker like CLI
* **nerdctl Features**
    - Docker compose
    - Lazy pulling
    - Encrypted images
    - P2P image distribution
    - Image sign and verify
    - Namespaces in kubernetes

### CRICTL
* A utility to interact with CRI (Container Runtime Interface) compatible runtimes
* Installed seperately
* To debug containers
* Works accross different runtimes

> [!IMPORTANT]
> ### CRICTL UNIX SOCKETS
> `unix:///run/containerd/containerd.sock`
> `unix:///run/crio/crio.sock`
> `unix:///var/run/cri-dockerd.sock`

## PODS
* Kubernetes never deploy an application directly on a machine
* Containers are encapsulated inside a kubernetes object `pod`
* It is a single instance of an application
* It is also teh smallest object in kubernetes

> [!TIP]
> Suppose we have a single instance of application running in kubernetes and over the time load increases. Where should we create a new container ?
> In the same pod or we should create a new pod ?
> We should create a new pod
>
> Pods have one to one relation with containers
> Pods can not have multiple homogeneous pods but they can have multiple pods that are of different kind o different service
> Those containers can refer each other by calling each other `localhost` as they share the same networkspace, they can share the same storage space too
> In kubernetes we do not need to link containers manually

> [!TIP]
> Multi container pods are rare usecase

## KUBERNETES COMMANDS
| COMMAND | EFFECT |
| ------- | ------ |
| `kubectl run POD_NAME --image IMAGE_NAME` | Create a pod runnging a image |
| `kubectl get all` | Get all kubernetes objects |
| `kubectl get pods -o wide` | Get a list of pods with pod IP address, pod's node |
| `kubectl get replicationcontroller -o wide` | Get a list of replicationcontrollers |
| `kubectl get replicaset -o wide` | Get a list of replicasets |
| `kubectl get POD_NAME` | Get state of pod |
| `kubectl get pods --no-headers -o custom-columns=":metadata.name"` | Get all pods name only |
| `kubectl get deployments` | See deployments |
| `kubectl describe pod POD_NAME` | Get more info about a pod, including the pod events |
| `kubectl describe replicaset REPLICASET_NAME` | Get more info for replicaset |
| `kubectl create -f DEFINATION_FILE --record` | Create kubernetes object (pod, service, replica set, deployment) |
| `kubectl delete pod POD_NAME` | Delete a pod |
| `kubectl delete replicaset REPLICASET_NAME` | Delete replicaset and underlying pods |
| `kubectl delete deployment DEPLOYMENT_NAME` | Delete a deployment and underlying objects such as replicaset and pods |
| `kubectl rollout status DEPLOYMENT_NAME` | See rollout status for the deployment |
| `kubectl rollout history DEPLOYMENT_NAME` | See history of rollouts |
| `delete pods $(kubectl get pods --no-headers -o custom-columns=":metadata.name")`| Delete all pods |
| `kubectl edit pod POD_NAME` | Change the actual config of a pod which is inside the kubernetes framework |
| `kubectl edit replicaset REPLICASET_NAME` | Change the actual config file of a replicaset which is inside the kubernetes framework |
| `kubectl edit deployment DEPLOYMENT_NAME` | Change the actual config file of a deployment which is insode the kubernetes framework |
| `kubectl apply -f DEFINATION_FILE --record` | Apply changes to a resource after changing its defination file |

> [!TIP]
> * While runngin `kubectl get pods` the `N/N` in `READY` state shows `Containers running/Pods running`

## KUBERNETES CONCEPTS
### PODS WITH YAML
* Mandatory fields
    - `apiVersions`: Version of kubernetes api we use generally set to `v1` for `pods`
    - `kind`: Tells what type of defination the configuration file defines, possible values are in the table below
    - `metadata`: Data ablut the object like `name` and `labels`
    - `spec`: All the properties of objects are defined here

> [!TIP]
> * The labes is very important in case of 100s of pods running to group them and also to filter them
> * It becomes difficult to group them once they are deployed

* **Table for `apiVersion` and `kind`**

| KIND | API VERSION |
| ---- | ----------- |
| `POD` | `v1` |
| `Service` | `v1` |
| `ReplicaSet` | `apps/v1` |
| `Deployment` | `apps/v1` |

```
apiVersion: v1
kind: Pod
metadata:
  name: POD_NAME
  labels:
    k1: v1
    k1: v2

spec:
  containers:
    - name: CONTAINER_NAME
      image: IMAGE_NAME

    - name: CONTAINER_NAME
      image: IMAGE_NAME
```

### REPLICATION CONTROLLER
* We need multiple instance or pod running for high availability
* Replication controller helps to run multiple instances of a single pod in kubernetes cluster
* This even works for single pod, it ensures that specified number of pods are running at any point of time
* It also helps in load balancing and scaling
* Replica sets replaced replication controller
* It creates multiple instances of pods
* Pods created by replicationcontroller get the same prefix name as the replicationcontroller
```
apiVersion: v1
kind: replicationController
metadata:
  name: huhu_replica
  labels:
    app: APP_NAME
    task: TASK_NAME
spec:
  template: # IT CONTAINS POD DEFINATION
    metadata:
      name: POD_NAME
      labels:
        l1: v1
        l2: v2
    spec:
      containers:
        - name: CONTAINER_NAME
          image: IMAGE_NAME

        - name: CONTAINER_NAME
          image: IMAGE_NAME
  replicas: N
```

### REPLICA SET
* It is a process that controlls the pods
* All things are same as the replicationcontroller except the `apiVersion` and `kind`
* Also a new parameter `selector` is introduced, this contains the labels of already created pods
* Replica set can control pods which were not created as per replica set creation
```
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: huhu_replica
  labels:
    app: APP_NAME
    task: TASK_NAME
spec:
  template: # IT CONTAINS POD DEFINATION
    metadata:
      name: POD_NAME
      labels:
        l1: v1
        l2: v2
    spec:
      containers:
        - name: CONTAINER_NAME
          image: IMAGE_NAME

        - name: CONTAINER_NAME
          image: IMAGE_NAME
  replicas: N
  selector:
    matchLabels:
      l1: v1
      l2: v2
```

> [!TIP]
> #### HOW TO SCALE ?
> * Edit the config file and run `kubectl replace -f DEFINATION_FILE`
> OR
> * Run `kubectl scale --replicas=N -f DEFINATION_FILE`
> OR
> * Run `kubectl scale --replicas=N replicaset REPLICASET_NAME`

> [!NOTE]
> * If we create a new pod externally with any single label common within the selector in replicaset defination, it kill the newly created as it was not supposed to be there and if we delete a pod with a common label with replicaset, it will create a new one to makesure that specified numbe of pods are running
> *
> * The `matchLabels` values and `metadata` `label` values for pod defination should have at least one intersection in the file defination, else it gives error

> [!TIP]
> #### KILL ALL PODS
> kubectl `delete pods $(kubectl get pods --no-headers -o custom-columns=":metadata.name")`

### DEPLOYMENTS
* These come into a higher hierarchy in kubernetes
* These allow to make rolling upgrades, updates, rollbacks and many more
* Here nothing much changes into files as compared to `replicasets` but only one parameter `kind: Deployment`
* `Deployment` creates `ReplicaSets` creates `Pods`
```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: huhu_replica
  labels:
    app: APP_NAME
    task: TASK_NAME
spec:
  template: # IT CONTAINS POD DEFINATION
    metadata:
      name: POD_NAME
      labels:
        l1: v1
        l2: v2
    spec:
      containers:
        - name: CONTAINER_NAME
          image: IMAGE_NAME

        - name: CONTAINER_NAME
          image: IMAGE_NAME
  replicas: N
  selector:
    matchLabels:
      l1: v1
      l2: v2

```
### UPDATES, UPGRADES and ROLLBACKS
* Rolling updates are default depoyment strategies

> [!IMPORTANT]
> While upgrading, kubernetes creates a new replica set and keeps removing pods from old one and creating in new one, this nature can be seen by `kubectl get replicasets`
> If we want to rollout `kubectl rollout undo DEPLOYMENT_NAME` to rollback to previous build

## SERVICES
* Enable communication between various components inside and outside of application
* Enables connect applications with other applications and users
* These are objects that listen on a port and then forward to another port and address
* It is like a virtual server insode the node
```mermaid
block-beta
  user space node_port_service space pods
  user-->node_port_service
  node_port_service-->pods
  space
  end
  app_service1 space ClusterIP_service space app_service2
  app_service1-->ClusterIP_service
  ClusterIP_service-->app_service2
```
* **Types of Services**
    - `NodePort`: Makes an internal pod accessable on node
    - `ClusterIP`: Service creates a virtual IP insode the cluster to enable communication between different application services
    - `LoadBalancer`:
