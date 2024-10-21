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
| `kubectl get services` | See services |
| `kubectl describe pod POD_NAME` | Get more info about a pod, including the pod events |
| `kubectl describe replicaset REPLICASET_NAME` | Get more info for replicaset |
| `kubectl describe deployment DEPLOYMENT_NAME` | Get more info about deployment |
| `kubectl describe service SERVICE_NAME` | Get info regarding service |
| `kubectl create -f DEFINATION_FILE --record` | Create kubernetes object (pod, service, replica set, deployment) |
| `kubectl delete pod POD_NAME` | Delete a pod |
| `kubectl delete replicaset REPLICASET_NAME` | Delete replicaset and underlying pods |
| `kubectl delete deployment DEPLOYMENT_NAME` | Delete a deployment and underlying objects such as replicaset and pods |
| `kubectl rollout status DEPLOYMENT_NAME` | See rollout status for the deployment |
| `kubectl rollout history DEPLOYMENT_NAME` | See history of rollouts |
| `kubectl replace -f REPLICASET_FILE` | Apply changes made to replicaset file |
| `kubectl replace --force -f RESOURCE_OBJECT` | Forcefully update and recreate the resource |
| `kubectl scale replicasets REPLICASET_NAME -replicas=N` | Scale without editing the replication file |
| `delete pods $(kubectl get pods --no-headers -o custom-columns=":metadata.name")`| Delete all pods |
| `kubectl edit pod POD_NAME` | Change the actual config of a pod which is inside the kubernetes framework |
| `kubectl edit replicaset REPLICASET_NAME` | Change the actual config file of a replicaset which is inside the kubernetes framework |
| `kubectl edit deployment DEPLOYMENT_NAME` | Change the actual config file of a deployment which is insode the kubernetes framework |
| `kubectl apply -f DEFINATION_FILE --record` | Apply changes to a resource after changing its defination file, creates the resource if it does not exist |
| `minikube service SERVICE_NAME --url` | Get url to access the service |

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

```yaml
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
```yaml
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
* It is a process that monitors and controlls the pods
* All things are same as the replicationcontroller except the `apiVersion` and `kind`
* Also a new parameter `selector` is introduced, this contains the labels of already created pods
* Replica set can control pods which were not created as per replica set creation, that is done using the selectors
* The PODs with all the `matchLabels` selectors having the same values will be scheduled, it is like and in ligical operation
```yaml
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
```yaml
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
* Enables connectivity between groups of PODs
* Enable loose coupling between microservices
* Service is like a virtual server inside the node, inside the cluster it has its own IP address and that is called `ClusterIP` of the service
* **Types of Services**
    - `NodePort`: Makes an internal pod accessable on node
    - `ClusterIP`: Service creates a virtual IP insode the cluster to enable communication between different application services
    - `LoadBalancer`: Balances load between different pods

> [!IMPORTANT]
> ### NODEPORT
> * Port on the pod that runs an app_service is called `targetPort`
> * Port on the service itself is called `port`
> * Port on the node that we use to access services is called `nodePort`, its range is [30000, 32767]
> * To link pods to the service, we copy the required labels of the pods and then paste them in the `selector` section
> * Services also support session affinity
> * When pods are distributed along different nodes, services also gets created on all nodes and takes care of all that
> * Map a port on the node to a port on the pod
> * The algorithm for loadbalancing is random
> ```yaml
> apiVersion: v1
> kind: Service
> metadata:
>   name: SERVICE_NAME
> spec:
>   type: NodePort
>   ports:
>     - targetPort: TARGET_PORT
>       port: PORT
>       nodePort: NODE_PORT
>   selector:
>     l1: v1
>     l2: v2
> ```

> [!IMPORTANT]
> ### CLUSTER IP
> * Groups pods and makes easy to access the pods
> ```yaml
> apiVersion: v1
> kind: Service
> metadata:
>   name: SERVICE_NAME
> spec:
>   type: ClusterIP
>   ports:
>     - targetPort: TARGET_PORT
>       port: PORT
>   selector:
>     l1: v1
>     l2: v2
> ```

> [!TIP]
> `kubernetes` service of `ClusterIP` is the default service and it is always running

> [!IMPORTANT]
> ### LOAD BALANCER
> * Balance load between the pods
> * Use native load balancers from cloud platforms, using it in un supported environment will have the same support as `NodePort`
> * Use external load balancer
> ```yaml
> apiVersion: v1
> kind: Service
> metadata:
>   name: SERVICE_NAME
> spec:
>   type: LoadBalancer
>   ports:
>     - targetPort: TARGET_PORT
>       port: PORT
>       nodePort: NODE_PORT
>   selector:
>     l1: v1
>     l2: v2
> ```

## CORE CONCEPTS
### CLUSTER ARCHITECTURE
* **Worker Nodes**: Host containers as containers
* **Master Noder**: Manage, plan, schedule
    - **ETCD**: Highly available key value database that stores all info regarding the cluster and the containers
    - **Kube-scheduler**: Schedules containers
    - **Controllers**:
        - **Node Controller**: Onboard nodes, delete, unavail, etc 
        - **Replication Controller**: Ensure specified number of containers are running at any time 
    - **kube-apiserver**: Enables communication between different components, orchestrations for the cluster
    - **Container Runtime**: Runs the contatners
    - **Kubelet**: Agent that runs on every node and manages the node and waits for instructions from `kube-apiserver`
    - **Kube-proxy service**: Allows inter node container communication
### ETCD
* Distributed highly available key value store that is simple, fast and secure
* It stores info regarding nodes, PODs, configs, secrets, accounts, roles, bindings and more
* There are different ways to setup kubernetes and ETCD, the kubeadm way and the scratch way
* WHen we install cluster using kubeadm the ETCD gets deployed as a POD

| COMMAND | EFFECT |
| ------- | ------ |
| `kubectl get pods -n kube-system` | Get PODs in kube-system namespace |
| `kubeclt exec etcd-master -n kube-system etcdctl get / --prefix -keys-only` | Get all the keys stored by kubernetes |

* ```kubectl exec etcd-controlplane -n kube-system -- sh -c "ETCDCTL_API=3 etcdctl get / --prefix --keys-only --limit=10 --cacert /etc/kubernetes/pki/etcd/ca.crt --cert /etc/kubernetes/pki/etcd/server.crt --key /etc/kubernetes/pki/etcd/server.key"```

### KUBE-APISERVER
* When we run a command or make a post requeat to `kube-apiserver` it first validates authenticates the user, validates request and then it fetches the state from ETCD and then processes the request by updating ETCD and then schedule and then kubelet is responded

| COMMAND | EFFECT |
| ------- | ------ |
| `cat /etc/kubernetes/manifests/kube-apiserver.yaml` | Get api-service options (kubeadm) |
| `cat /etc/systemd/system/kube-apiserver.service` | Get api-service options (scratch) |
| `ps -aux \| grep kube-apiserver` | Get running process and the options |

### KUBE CONTROLLER MANAGER
* Monitors the other component processes and works to bring the whole system to desired state
* Node controller monitors and maintains the state of the nodes.
    - Checks the status of the nodes every 5 seconds
    - If heart beat not reached marks it unreachable, it waits for default 40schedule
    - If the node does not comes back up then all the tasks running on that node are are scheduled on other nodes
    - All this is done with `kube-apiserver`

* There are more kind of Controllers, all these controllers are part of `Kube-Controller-Manager`

| CONTROLLERS |
| ----------- |
| Deployment-Controller |
| CronJob |
| Service-Account-Controller |
| Namespace-Controller |
| Job-Controller |
| Stateful-set |
| PV-Binder-Controller |
| PV-Protection-Controller
| Endpoint-Controller |
| Replicaset |
| Replication-Controller |

| COMMAND | EFFECT |
| ------- | ------ |
| `cat /etc/kubernetes/manifests/kube-controller-manager.yaml` | See kube-controller-manager options (kubeadm) |
| `cat /etc/systemd/system/kube-controller-manager.service` | See kube-controller-manager options ( scratch) |

### KUBE SCHEDULER
* Decides only that which POD goes on which node and does not actually put the POD on the node, that is the job of kubelet
* Uses the worst fit scheduling algo.

| COMMAND | EFFECT |
| ------- | ------ |
| `cat /etc/kubernetes/manifests/kube-scheduler.yaml` | See kube-scheduler options (kubeadm) |
| `cat /etc/systemd/system/kube-scheduler.service` | See kube-scheduler options ( scratch) |
| `ps -aux \| grep kube-scheduler` | See running process and the options |

### KUBELET
* Follows master nodes and reports back, also does all the heavy lifting of loading and unloading of containers and PODs.
* Also responsible for registering the node to the master node

| COMMAND | EFFECT |
| ------- | ------ |
| `ps -aux \| grep kubelet` | Get running process and options |

### KUBE-PROXY
* Every POD is reachable to each other POD
* This is achieved using a POD networking solution, it is a internal virtual network which spans  all the nodes in the cluster  to which all the PODs connect to
* Services can't connect to networks as it is a virtual thing and do not have any interface and it is just resides in kubernetes memory
* Kube proxy is a process running on each node, its job is to look for new services and everytime a service is created, it creates appropriate rules on eac node to forward traffic to those services
* Kubeadm installs the Kube-proxy as a daemonset in PODs

| COMMAND | EFFECT |
| ------- | ------ |
| `cat /etc/kubernetes/manifests/kube-proxy.yaml` | See kube-proxy options (kubeadm) |
| `cat /etc/systemd/system/kube-proxy.service` | See kube-proxy options ( scratch) |
| `ps -aux \| grep kube-scheduler` | See running process and the options |
| `kubectl get daemonset -n kube-system` | Get the daemons running |

### POD

### REPLICA SETS

### DEPLOYMENTS

### SERVICES

### NAMESPACES
* Namespaces are like workspaces in kubernetes, they are used to isolate different kind of environments and workflows
* Kubernetes has its own seperate namespace for its internal services to isolate them form user and to prevent them to be modified externally by user, its is named as `kube-system`
* It also creates another namespace called `kube-public` which is used to create resources that are to be availed to all users
* It could be used to run production and test environment on same cluster to isolate environment and resources

> [!TIP]
* Resources can access the services on the same namespace just by the service name but to access the service in different namespace we need to use a more complex format `SERVICE_NAME.NAMESPACE.svc.CLUSTER_DOMAIN` where the local `CLUSTER_DOMAIN=cluster.local`, `svc` is the subdomain for service part

> [!TIP]
> * Namespace can be directly specified in the POD, service, replica set and depoyment yaml files in the metadata section in the format `namespace: NAMESPACE`

> [!IMPORTANT]
> Create a namespace using the following config file format
> ```yaml
> apiVersion: v1
> kind: Namespace
> metadata:
>   name: NAME
> ```
> * Then create  the resource using the create command

| COMMAND | EFFECT |
| ------- | ------ |
| `kubectl create namespace NAME` | Create a namespace |
| `kubectl get RESOURCE --namespace=NAMESPACE` | Get `RESOURCE` in the namespace where resource are PODs, services, deployments, replicasets etc |
| `kubectl get RESOURCE --all-namespaces` | Get resource list from all namespaces |
| `kubectl set-context $(kubctl config current-context) --namespace=NAMESPACE` | Set namespace so theat we not need to enter it manually in every command we run |
| `kubectl get namespaces` | Get namespaces list |

### RESOURCE QUOTA
* Used to limit the rsources of a namespace
```yaml
apiVersion: v1
kind: ResourceQuota
metadata:
  name: RESOURCE_QUOTA_NAME
  namespace: NAMESPACE
spec:
  hard:
    pods: "N"
    requests.cpu: "N"
    requests.memory: NG
    limits.cpu: "N"
    limits.memory: NG
    ...
    ...
    ...
```

### IMPERATIVE VS DECLARATIVE
* Imperative: Follow step by step instructions through the command line to get the final result
* Declarative: Define all steps in a file to get all results on a single command run

> [!TIP]
> * For example if one resource is deployed using a config file but one maintainer makes changes to the resource using kukectl edit command, the changes are not reflacted in the resource file and after this another maintainer makes the change to the resource in the resource file in different nfield value and apply the changes, the old value of the parameter that maintainer one changed will be set and the deploymnt will suffer inconsistency and mai even fail
> * The better way to do this is to first edit the resource file and then apply the changes and we must use the kubectl edit eay only if we know that the change we are making will be the last change that we will make in the config

> [!TIP]
> * Adding `--dry-run` at the end of any possible commands do not creates the resource but it tests if the resource is creatable or the command is right
> * Adding `-o yaml` outputs the command in yaml format in yaml format
> * Use the following parameter combinations to make the best

> [!TIP]
> Never mix imperative and declarative appraoches together

### KUBECTL APPLY COMMAND MECHANISM
* There is a kubernetes yaml config generated for the same config that we create for defination with more parameters
* One json file is also created for the config file that we created and it is called `last applied configuration`
* All these are compared to make decisions
* The `last applied configuration` file is required for comparing parameter value change and addition and deletion of parameters, it is stored in the live kubernetes config file as embeded document

## SCHEDULER
### MANUAL SCHEDULING
* Kubernetes adds a new parameter in the internal object config which is `nodeName`, it is normally not presentnin the userdefined config
* It traverses to all the PODs and the checks if a PODs is not having this value unset or set
* The PODs having this parameter unset are the candidates to be scheduled, the scheduler seeks the nodes and assigns the value to `nodeName`by creating a binding object

> [!TIP]
> * If a system has no scheduler, one can schedule the POD manually by setting this parameter in the user defined config

> [!TIP]
> * What if a POD is already assigned to a node with no `nodeName` specified in the user defined cinfig and needed to be rescheduled
> * The way is to create a binding object that would be sent to `kube-apiserver`
> ```yaml
> apiVersion: v1
> kind: Binding
> metadata:
>   name: BINDING_NAME
> target:
>   apiVersion: v1
>   kind: node
>   name: NODE_NAME
> ```
> * COnver the above config file to equivalent json format and then make request to `kube-apiserver`
> ```bash
> curl --header "Content-Type:application/json" --request POST --data 'JSON' http://$SERVER/api/v1/namespaces/NAMESPACE/pods/$PODNAME/binding/
> ```
> * Or the better way is to edit the user defined config file and then force recreste the POD

> [!CAUTION]
> * If a POD is not scheduled it may be that the scheduler may not be running

### LABELS and SELECTORS
* Group objects together on base of properties and criteria
* Labels are the properties attached to objects
* Selectors help to filter these objects based on conditions we provide
* Labels are defined in the metadata section
* While defining the `replicaSet` we define labels at 2 locations, one for replica set and one for POD, the `matchLabels` field in sepec of `replicaSet` a single match in the labels with the POD would do the thing 

| COMMAND | EFFECT |
| ------- | ------ |
| `kubectl get RESOURCE --selector KEY=VALUE,KEY=VALUE,...` | Get custom grouped resources |
| `kubectl label nodes NODE_NAME KEY=VALUE` | Label a node |
| `kubectl label nodes NODE_NAME --overwrite KEY=VALUE` | Update a label |
| `kubectl describe node NODE_NAME` | See labels |

#### ANNOTATIONS
* These are not used to select anything but used to make important note on different type of events
* These can be defined inside the metadata section as `annotations` and all the key value pairs can be listed inside the the `annotations`

### TAINT and TOLERATIONS
* Used to place specific groups of PODs on specific nodes
* Taints are set on nodes and tolerations set on nodes

| COMMAND | EFFECT |
| ------- | ------ |
| `kubectl taint nodes NODE_NAME KEY OPERATOR VALUE:TAINT_EFFECT` | Taint the node with specific taint effect and (key, value) |
| `kubectl describe node NODE_NAME \| grep taint` | See taints on the nodes |
| `kubectl taint node NODE_NAME TAINT` | Remove the taint from node and the `TAINT` can be directly copied from the node describe command |
> [!TIP]
> * **OPERATOR**: =, !, <, > 

> [!NOTE]
> #### TAINT EFFECTS
> * **NoSchedule**: PODs will not be scheduled on the node
> * **PreferNoSchedule**: Try not to sbhedule the PODs on the node but it is not guarenteed
> * **NoExecute**: PODs won't be scheduled on he node and the already running PODs on he particular node will be evicted

> [!TIP]
> Usefule while cluster maintenance

#### TOLERATIONS
* Tolerations can be added to pode defination as below
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: NAME
spec:
  containers:
    ...
  tolerations:
    - key: "KEY"
      operator: "OPERATOR_NAME"
      value: "VALUE"
      effect: "TAINT_EFFECT"
    
```

> [!CAUTION]
> * No PODs are scheduled on the master controller node beacuse it is tainted to all apps by default and which is a good practice as no workload should run on master node
> * To see the taint `kubectl describe node kubemaster | grep taint`

### NODE SELECTORS
* Allows to schedule the PODs on particular nodes according to the appliction demand and node capability
* `nodeSelector` also follows the inclusive rule of and logic
* Tho following format for setting `nodeSelector` parameter for PODs
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: NAME
spec:
  containers:
    ...
  nodeSelector:
    KEY: VALUE
```
### NODE AFFINITY
* Complex operations not allowed in `nodeSelector`
* We can create a complex chain of requirements for placing a POD
```yaml
apiVersion: v1
kind: POD
metadata:
  name: NAME
spec:
  containers:
  ...
  affinity:
    nodeAffinity:
      NODE_AFFINITY_TYPE:
        nodeSelectorTerms:
          - matchExpressions:
            - key: KEY
              operator: OPERATOR
              values:
                - VALUE1
                - VALUE2
                ...
```
> [!TIP]
> #### OPEARATORS
> * Operators are `In`, `NotIn`, `Exists`

> [!IMPORTANT]
> #### NODE AFFINITY TYPES
> * **`requiredDuringSchedulingIgnoreDuringExecution`**:
    - Matching affinity is mandatory while creation of POD, if can not match, POD will not be scheduled
    - If an environment change is made and some labels are changed or added or removed, the POD will not be rescheduled, it will keep rnning where it is
> * **`preferredDuringSchedulingIgnoreDuringExecution`**
    - Matching affinity rules is not mandatory while creation of POD, if can not match, POD will be scheduled at any available node
    - If an environment change is made and some labels are changed or added or removed, the POD will not be rescheduled, it will keep rnning where it is
> * **`requiredDuringSchedulingRequiredDuringExecution`**
    - Matching affinity is mandatory while creation of POD, if can not match, POD will not be scheduled
    - If an environment change is made and some labels are changed or added or removed, the POD will be evicted
> * **`preferredDuringSchedulingIgnoreDuringExecution`**
    - Matching affinity rules is not mandatory while creation of POD, if can not match, POD will be scheduled at any available node
    - If an environment change is made and some labels are changed or added or removed, the POD will be evicted

### RESOURCE REQUIREMENTS and LIMITS
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: NAME
  labels:
    ...
spec:
  containers:
    ...
  resources:
    requests:
      memory: "NUNIT"
      cpu: N
    limits:
      memory: "NUNIT"
      cpu: N
```
> [!NOTE]
> * For Memory we can define units of memory as `G`, `K`, `M`, `Gi`, `Ki`, `Mi`

> [!NOTE]
> * N can take any value for cpu from 0.1 to N
> * The N for the cpu is equivalent to either of these depending on the environments:
    - Hyperthreads
    - 1 AWS/Azure/GCP vCPU

> [!CAUTION]
> * If a POD consumes more memory then the limit then the POD is terminated and if cpu limit is exceeded then the POD is throttled

> [!TIP]
> #### BEHAVIOUR CPU
> * **REQUEST ABSENT LIMIT ABSENT**:
>     - One POD may over consume and other may thrive
> * **REQUEST ABSENT LIMIT PRESENT**:
>     - Some PODs may not perform at best
> * **REQUEST PRESENT LIMIT ABSENT**: IDEAL FOR MOST CASES
>     - No POD thrives no POD throttled
> * **REQUEST PRESENT LIMIT PRESENT**:
>     - No POD thrives but one may not perform the best
> #### BEHAVIOUR MEMORY
> * **REQUEST ABSENT LIMIT ABSENT**:
>     - One POD may over consume and other may thrive
> * **REQUEST ABSENT LIMIT PRESENT**:
>     - Some PODs may not perform at best
> * **REQUEST PRESENT LIMIT ABSENT**:
>     - No POD thrives no POD throttled
> * **REQUEST PRESENT LIMIT PRESENT**: IDEAL FORMOST CASES
>     - No POD thrives but one may not perform the best


### LIMIT RANGE
* Ensure that the PODs running have some defaults set for requests and limits, even for the PODs having no requests and limits set
* These are implamented at a namespace level
* It does not affect existing PODs, on;y the new PODs
> [!NOTE]
> #### CPU
> ```yaml
> apiVersion: v1
> kind: LimitRange
> metadata:
>   name: NAME
>   namespace: NAMESPACE
> spec:
>   limits:
>     - default:
>         cpu: N
>       defaultRequest:
>         cpu: N
>       max:
>         cpu: N
>       min: N
>       type: Container
> ```

> [!NOTE]
> #### MEMORY
> ```yaml
> apiVersion: v1
> kind: LimitRange
> metadata:
>   name: NAME
>   namespace: NAMESPACE
> spec:
>   limits:
>     - default:
>         memory: NUNIT
>       defaultRequest:
>         memory: NUNIT
>       max:
>         memory: NUNIT
>       min: NUNIT
>       type: Container
> ```

### RESOURCE QUOTA
* Used to limit the resources of the namespace, the total resources being consumed can't exceed the value defined
```yaml
apiVersion: v1
kind: ResourceQuota
metadata:
  namespace: NAMESPACE
  name: NAME
spec:
  hard:
    requests.cpu: N
    requests.memory: NUNIT
    limits.cpu: N
    limits.memory: NUNIT
```

### DAEMONSETS
* Resource objecy that runs one copy of the POD on each node
* Useful for monitoring solutions and logging

> [!TIP]
> * Weave-net also requires daemon-sets

> [!NOTE]
> * Kube-proxy is also an daemon-set

```yaml
apiVersion: v1
kind: DaemonSet
metadata:
  name: NAME
spec:
  template:
    metadata:
      labels:
        K1: V1
        K2: V2
    spec:
      containers:
        ...
  selector:
    matchLabels:
      K1: V1
      K2: V2
      ...
```
