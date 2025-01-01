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
| `kubectl describe pod`| Describe all PODs |
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
| `kubectl logs POD` | Get logs for a POD |
| `kubectl logs POD -c CONTAINER_INITCONTAINER` | Get logs of a specific container inside the POD |
| `kubectl exec -it POD -- COMMAND ARGUMENTS` | Run a command in a POD |
| `kubectl exec -it POD -c CONTAINER -- COMMAND ARGUMENTS` | Run a command in a multi container POD |
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

## APPLICATION LIFECYCLE
### ROLLING UPDATES and ROLLBACKS
* When one first creates a deployment it triggers a rollout
* At the time of application updates, new rollouts are triggered
* Any change in the app is seen as update wheather ahange in image or change in labels or anything else

> [!NOTE]
> * `Rolling`: All the pods are replaced one by one instead of all at once, this is the default update strategy
> * `Recreate`: All the PODs are taken doen and then they are all recreated
>
> * These differences can be see in the `kubectl describe DEPLOYMENT` output in the events section

> [!NOTE]
> #### HOW UPGRADES HAPPEN
> * There is a repoca set already, a new replica set is created and then one by one new POD is created in new replica set and one POD from older replica set is taked down
> * Both these replica sets can be seen when the upgrade is happening using the `kubectl get replicases`

| COMMAND | EFFECT |
| ------- | ------ |
| `kubectl rollout status DEPLOYMENT` | Get rollout status for deployment |
| `kubectl rollout history DEPLOYMENT` | Get rollout history for deployment |
| `kubectl rollout undo DEPLOYMENT` | Undo a rollout |

> [!NOTE]
> * Difference between time after rollout and rollback is that when we create a new rollout the new version, the newer replica set have all the active PODs and after the rollback all the active PODs are in the older replica set

> [!TIP]
> * Using the describe command `StrategyType` can be seen and if it is `RollingUpdate`, there is one more parameter `RollingUpdateStrategy` can be seen which tells about the strategy config for rolling updates telling maximum down instances and surge instances for the time the update is being rolled out

> [!IMPORTANT]
> * Update strategy can be configured as follows
> ```yaml
> apiVersion: apps/v1
> kind: Deployment
> metadata:
>   ...
> spec:
>   template:
>     ...
>   strategy:
>     rollingUpdate:
>       maxSurge: N%
>       maxUnavailable: N%
>     type: TYPE
> ```
> * Where the typ can be `Recreate` and `RollingUpdate`

### COMMANDS and ARGUMENTS IN DOCKER
* See the docker docs for `ENTRYPOINT` and `CMD` options and we can pass arguments in the container for command as below
```yaml
apiVersion: v1
kind: Pod
metadata:
  ...
spec:
  containers:
    - name: CONTAINER_NAME
      image: IMAGE_NAME
      args: ["ARG1", "ARG2", "ARG3"]
```
* Also command specification for `ENTRYPOINT` is possible as below
```yaml
apiVersion: v1
kind: Pod
metadata:
  ...
spec:
  containers:
    - name: CONTAINER_NAME
      image: IMAGE_NAME
      command: ["COMMAND", "ARG1", "ARG2", ...]
      args: ["ARG1", "ARG2", "ARG3", ...]
```

### ENVIRONMENT VARIABLES
* There are 3 ways to do this:
    - Plain key value
    - ConfigMap
    - Secrets
* Using plain key value
```yaml
apiVersion: v1
kind: Pod
metadata:
  ...
spec:
  containers:
    - name: CONTAINER_NAME
      image: IMAGE_NAME
      env:
        - name: VARIABLE1
          value: VALUE1
        - name: VARIABLE2
          value: VALUE2
        - name: VARIABLE3
          value: VALUE3
```
* Using `ConfigMap`
```yaml
apiVersion: v1
kind: Pod
metadata:
  ...
spec:
  containers:
    - name: CONTAINER_NAME
      image: IMAGE_NAME
      env:
        - name: VARIABLE
          valueFrom:
            configMapKeyRef:
              name: CONFIGMAP
              key: KEY
```
```yaml
apiVersion: v1
kind: Pod
metadata:
  ...
spec:
  containers:
    - name: CONTAINER_NAME
      image: IMAGE_NAME
      envFrom:
        - configMapRef:
            name: CONFIGMAP
```

* Using `Secrets`
```yaml
apiVersion: v1
kind: Pod
metadata:
  ...
spec:
  containers:
    - name: CONTAINER_NAME
      image: IMAGE_NAME
      env:
        - name: VARIABLE
          valueFrom:
            secretKeyRef:
              name: SECRET
              key: KEY
```
```yaml
apiVersion: v1
kind: Pod
metadata:
  ...
spec:
  containers:
    - name: CONTAINER_NAME
      image: IMAGE_NAME
      envFrom:
        - secretRef:
            name: SECRET
```

### CONFIGMAPS
* Used to store and pass config data in key value pairs
* ConfigMap is injcted in the PODs
```yaml
apiVersion:
kind: ConfigMap
metadata:
  name: NAME
data:
  VARIABLE1: VALUE1
  ...
```
| COMMAND | EFFECT |
| ------- | ------ |
| `kubectl get configmaps` | See all ConfigMaps |
| `kubectl describe configmap CONFIGMAP` | See details of ConfigMap |

#### CONFIGMAPS THROUGH VOLUMES

### SECRETS
* Store sensitive info
```yaml
apiVersion: v1
kind: Secret
metadata:
  name: NAME
data:
  KEY1: B64_ENCODED_VALUE1
  KEY2: B64_ENCODED_VALUE2
  KEY3: B64_ENCODED_VALUE3
```

| COMMAND | EFFECT |
| ------- | ------ |
| `kubectl get secrets` | List secret files |
| `kubectl describe secret SECRET` | Get details of a secret |
| `kubectl get secret app-secret -o yaml` | Get the values of all secrets inside the secrets object |

> [!TIP]
> * Create base64 encoded values
> ```bash
> echo -n 'SECRET_VALUE' | base64
> ```
> * Convert back base64 encoded values to normal human readable format
> ```bash
> echo -n 'BASE64_ENCODED_VALUE' | base64 --decode
> ```

> [!TIP]
> * When doing desfribe command on secrets, we also get to see the type of secret

> [!NOTE]
> * Secrets are not encrypted, only encoded, so everyone who haveaccess to the system can access them, also the one who can create the kubernetes objects ca also see and edit the secrets
> * Secrets in the `ETCD` are not encrypted as no data in `ETCD` is encrypted by default, we need to configure that
> * Secrets must be encrypted at rest and see teh docs and articles on `Encrypting Secret Data at Rest`

### MULTI CONTAINER PODS
* Multicontainer PODs share networkspace and volumespace
* There are 3 multi container patterns
    - sidecar
    - adapter
    - ambassador

> [!NOTE]
> * If in a senario, one have a different POD running a service that generates logs and one another POD that analyzes the logs
> * They can't share a volumespace so there is need of configuration to make them interact
> * `volumeMounts` is the solution
> * Suppose POD 1 generates the logs and POD 2 analyzes them
> ```yaml
> # POD 1 CONFIG
> apiVersion: v1
> kind: Pod
> metadata:
>   ...
> spec:
>   containers:
>     - image: IMAGE_NAME
>       name: NAME
>       volumeMounts:
>         - mountPath: PATH_INSIDE_CONTAINER_POD1
>           name: VOLUME_MOUNT_NAME_IN_POD1
> ```
> ```yaml
> apiVersion: v1
> kind: Pod
> metadata:
>   ...
> spec:
>   containers:
>     - image: IMAGE_NAME
>       name: NAME
>       volumeMounts:
>         - mountPath: PATH_INSIDE_CONTAINER_POD2
>           name: VOLUME_MOUNT_NAME_IN_POD2
> ```

### INITCONTAINERS
* Some times there is situation where one must just want to run a process only once before the main application starts running which is defined in the same config file
* This is where `initContainers` comes handy
* Init containers are provided as a list, all execute sequentially
> [!NOTE]
> If a `initContainer` fails instead of completing successfully, the main apolication won't start
```yaml
apiVersion: v1
kind: Pod
metadata:
  ...
spec:
  containers:
    ...
  initContainers:
  - name: NAME
    image: IMAGE_NAME
    command: ["COMMAND", "ARG1", "ARG2", ...]
```

### SELF HEALING APPLICATIONS
* Kubernetes supports self-healing applications through ReplicaSets and Replication Controllers
* The replication controller helps ensure that a POD is re-created automatically when the application within the POD crashes
* It helps in ensuring enough replicas of the application are running at all times
* Kubernetes provides additional support to check the health of applications running within PODs and take necessary actions through Liveness and Readiness Probes

## STORAGE
### DOCKER STORAGE and VOLUME DRIVERS
* Docker stores everything in `/var/lib/docker`
* There is an layered architecture, there are image layers and then we have container layers
* If one tries to modify image files, it follows `COPY_ON_WRITE` mechanism and a copy of that file is created in container layer and modifications are done on the container layer copy of that file.
* Volumes are used to mount the persistent storage to the container at specific path, this is called `VOLUME MOUNTING`
* There are `BIND MOUNTING` as well where a direcctory form the host is mounted on to a container
* `STORAGE DRIVERS` are responsible for maintaining the layered architecture and many more things related to storage and the best driver is picked up automatically based on hardware and OS
* Volumes are not handled by the storage drivers, they are handled by volume drivers, research more about the volume drivers

### CONTAINER STORAGE INTERFACE (CSI)
* Global standard developed to support many different storage solutions
* It defines a set of `RPC` (Remote Procedural Calls) that would be called by container orchestrator, these must be implemented by storage drivers

### VOLUMES
* PODs are transient in nature
* Example of volume mount for single node cluster
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: random-number-generator
spec:
  containers:
    - image: alpine
      name: alpine
      command: ["/bin/sh","-c"]
      args: ["shuf -i 0-100 -n 1 >> /opt/number.out;"]
      volumeMounts:
        - mountPath: /PATH_INSIDE_CONTAINER
          name: VOLUME_NAME
  volumes: # VOLUMES BLOCK
    - name: VOLUME_NAME
      hostPath:
        path: /PATH_ON_HOST
        type: Directory
```
* This is not recommended for use in multi node cluster as this would use `/data` to store the data on all the server and expect all of the `/data` directories on different servers to be the same, which in fact are not unless one implement any external replicated shared storage solution
* Kubernetes support different storage solutions: `NFS`, `Gluster FS`, `Flocker`, `AWS` and many more

* To use AWS EBS volumes we replace the volumes block as:
```yaml

apiVersion: v1
kind: Pod
metadata:
  name: POD_NAME
spec:
  containers:
    - image: alpine
      name: alpine
      command: ["/bin/sh","-c"]
      args: ["shuf -i 0-100 -n 1 >> /opt/number.out;"]
      volumeMounts:
        - mountPath: /PATH_INSIDE_CONTAINER
          name: VOLUME_NAME
  volumes:
    - name: VOLUME_NAME
      awsElasticBlockStore:
        volumeID: VOLUME_ID
        fsType: FS_TYPE #ext4, ...
```

### PERSISTENT VOLUMES
* While creating volumes, the volumes are defined within the pod definition file
* What if there are too many users, too many nodes on the cluster and a large and complex environment
* By using the normal volumes user would have to configure storege for all the pods on their own
* The better way is that the administrator would createa large pool of storage and then the users carve out pieces of it

* A persistent volume is a cluster wide pool of volumes configured by administrator
* Users can now use storage from this pool using persistent volume claims

```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: PERSISTENT_VOLUME_NAME
spec:
  accessModes:
    - ACCESS_MODE
    - ACCESS_MODE
    ...
  capacity:
    storage: STORAGE_CAPACITY

  persistentVolumeReclaimPolicy: PERSISTENT_VOLUME_RECLAIM_POLICY # Retain, Delete, Recycle

  awsElasticBlockStore:
    volumeID: VOLUME_ID
    fsType: FS_TYPE
```
```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: PERSISTENT_VOLUME_NAME
spec:
  accessModes:
    - ACCESS_MODE
    - ACCESS_MODE
    ...
  capacity:
    storage: STORAGE_CAPACITY

  persistentVolumeReclaimPolicy: PERSISTENT_VOLUME_RECLAIM_POLICY # Retain, Delete, Recycle

  hostPath:
    path: /PATH_ON_HOST
```

### PERSISTENT VOLUME CLAIMS
* Used to make a persistent volume available to a node
* Persistent volume and persistent volume claims are different objects
* Administrator creates persistent volumes and user creates persistent volume claims
* Kubernetes binds persistent volumes to claims based on request and properties set on the volumes
* Every `PVC` is bound to a single `PV`
* While binding process, kubernetes ensures that the properties are satisfied: Capacity, Access Modes, Volume Modes, Storage Class, Selector
* If a larger `PV` is bound to a `PVC` having low claims, no other claim can utilize the capacity as there is one to one mapping
```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: CLAIM_NAME
spec:
  accessModes:
    - ACCESS_MODE
    - ACCESS_MODE
    ...
  resources:
    requests:
      storage: STORAGE_CAPACITY
  selector:
    ...
```

### STORAGE CLASS
* While creating the persistent volume using the cloud storage solutions, where is requirement that the storage object should be pre provisioned
* To solve hat problem, storage class was created, it auto provisions the storage resource (dynamic provisioning)

* Provisioning the storage solutions in `GCP`:
```yaml
# TORAGE CLASS OBJECT
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: google-storage
provisioner: kubernetes.io/gce-pd
parameters:
  ...
```

```yaml
# persisteNt VOLUME CLAIM OBJECT
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: CLAIM_NAME
spec:
  accessModes:
    - ReadWriteOnce
  storageClassName: google-storage
  resources:
    requests:
      storage: STORAGE_CAPACITY
```
```yaml
# POD DEFINATION OBJECT
apiVersion: v1
kind: Pod
metadata:
  name: POD_NAME
spec:
  containers:
    - image: IMAGE_NAME
      name: CONTAINER_NAME
      volumeMounts:
        - mountPath: /opt
          name: VOLUME_NAME
  volumes:
    - name: VOLUME_NAME
      persistentVolumeClaim:
        claimName: CLAIM_NAME
```












## CLUSTER SECURITY
### AUTHENTICATION
* There are 3 users : developer, admin and bots
* Kubernetes does not manages users natively, it relies on external sources like LDAP or kerberos, certificates. But it can manage service accounts natively
* kube-apiserver authenticates the requests before processing them

### TLS
* It guanrantees trust between users during transaction
* Files with the extension `.crt` or `.pem` a re the public certificates and those ending with `.key` or `key.pem` are private keys
* Interaction bet wen all the nodes must be secured (all the components as well)
* The components (`kube-apiserver`, `etcd-server`, `kubelet`, `kube-proxy`, `kube-controller`, `kube-scheduler`, `admins`) all have certificates generated by CA or local CA

### GENERATING CERTIFICATES
```bash
openssl genrsa -out KEY_NAME.key 1024
openssl rsa -in KEY_NAME.key -pubout > PUBLIC_KEY.pem
```

### TLS in KUBERNETES
#### GENERATING CERTIFICATES
* Generate key
```bash
openssl genrsa -out ca.key 2048
```
* Create self signing request
```bash
openssl req -new -key ca.key -subj "/CN=KUBERNETES-CA" -out ca.csr
```
* Sign the certificates
```bash
openssl x509 -req ca.csr -signkey ca.key -out ca.crt
```
Then generate all the certificates for all the components of kubernetes


### KUBECONFIG
* It is aconfig file for accessing kubernetes cluster which passes all the details that are required to access a cluster (IP, port, key, certificate, certificate authority, etc)
* Generally stoerd at `~/.kube/config`
* It is in yaml formt and have 3 parts user, cluster and contexts
* User part has the `client-certificate-data` and `client-key-data`
* Cluster part has `certificate-authority-data`, `name`, `server address` (certificate data is in base64 encoded format) 
* Context used to bind the cluster and users
> [!TIP]
> * Default context to use can be defined in top level by adding the following statement in the kube-config file `current-context: USER@CLUSTER` 

| COMMAND | EFFECT |
| ------- | ------ |
| `kubectl ... --kubeconfig=KUBECONFIG_PATH` | Pass required data to access a aremote cluster |
| `kubectl config view` | View config file for current cluster |
| `kubectl config use-context CONTEX_NAME --kubeconfig=KUBECONFIG_PATH` | Use a particular context from kubeconfig file |


### API GROUPS
* Kubernetes API is grouped in to different categories to (`/metrics`, `/healthz`, `/version`, `/api`, `/apis`, `/logs`, etc)
* The `/api` is the core group where all the core functionalities exist (namespaces, pods, rc, nodes, endpoints, events, bindings, PV, PVC, configmaps, secrets, services, etc)
* The `/apis` is the named group, it is more organized and all new features and resource objects will be made available through this one only

* `curl http://localhost:6443/apis -k | grep "name"` returns all named resource groups

### AUTHORIZATION
* Kubernetes provides 4 types of authorization (node based, ABAC, RBAC, webhook, AlwaysAllow, AlwaysDeny)
* Authentication based on webhook is to implement externally managed authorization mechanisms

### RBAC
* A `Role` defination file defines a list of `apiGroups`, `resources`, `verbs`
* Then a `RoleBinding` object is created to bind the users and roles

> [!NOTE]
> #### ROLE
> ```yaml
> apiVersion: rbac.authorization.k8s.io/v1
> kind: Role
> metadata:
>   name: ROLE_NAME
> rules:
>   - apiGroups: ["", "", ""] # 
>     resources: ["RESOURCE1", "RESOURCE2", ...] # pods, nodes, etc
>     verbs: ["VERB1", "VERB2", ...] # list, get, delete, create, update
>     resourceName: ["RESOURCE_NAME1", "RESOURCE_NAME2", ...] # Name of resources that one needs to give access to instead of all resources
>   ...
>   ...
>   ...
> ```

> [!NOTE]
> #### USER ROLE BINDING
> ```yaml
> apiVersion: rbac.authorization.k8s.io/v1
> kind: RoleBinding
> metadata:
>   name: ROLE_BINDING_NAME
> subjects:
>   - kind: User
>     name: USER_NAME
>     apiGroup: rbac.authorization.k8s.io
> roleRef:
>   kind: Role
>   name: ROLE_NAME
>   apiGroup: rbac.authorization.k8s.io
> ```


| COMMAND | EFFECT |
| ------- | ------ |
| `kubectl get roles` | Get RBAC roles |
| `kubectl get rolebindings` | List all `RoleBindings` |
| `kubectl describe role ROLE_NAME` | Describe a `Role` |
| `kubectl describe rolebinding ROLE_BINDING_NAME` | Describe `RoleBindings` |
| `kubectl auth can-i COMMANDS` | See if there is access to certain commands |
| `kubectl auth can-i COMMANDS --as USER_NAME` | See if there is access to a certain command as a particular user |
| `kubectl auth can-i COMMANDS --as USER_NAME --namespace=NAMESPACE` | See if there is access to a certain command as a particular user in pasticular user |
| `cat /etc/kubernetes/manifests/kube-apiserver.yaml` | Inpect `kube-apiserver` config and see authorization modes ams many more parameters |

### CLUSTER ROLES
* Roles and role bindins are namespaced and limited to the namespaces
* Cluster roles are for cluster scoped resources (nodes, PVC, PV, certificate signing requests, namespaces)

> [!NOTE]
> #### ROLE
> ```yaml
> apiVersion: rbac.authorization.k8s.io/v1
> kind: ClusterRole
> metadata:
>   name: CLUSTER_ROLE_NAME
> rules:
>   - apiGroups: ["", "", ""] # 
>     resources: ["RESOURCE1", "RESOURCE2", ...] # nodes, etc
>     verbs: ["VERB1", "VERB2", ...] # list, get, delete, create, update
>     resourceName: ["RESOURCE_NAME1", "RESOURCE_NAME2", ...] # Name of resources that one needs to give access to instead of all resources
>   ...
>   ...
>   ...
> ```

> [!NOTE]
> #### CLUSTER ROLE BINDING
> ```yaml
> apiVersion: rbac.authorization.k8s.io/v1
> kind: ClusterRoleBinding
> metadata:
>   name: CLUSTER_ROLE_BINDING_NAME
> subjects:
>   - kind: User
>     name: CLUSTER_USER_NAME
>     apiGroup: rbac.authorization.k8s.io
> roleRef:
>   kind: ClusterRole
>   name: ROLE_NAME
>   apiGroup: rbac.authorization.k8s.io
> ```
kubectl api-resources --namespaced=true


### SERVICE ACCOUNTS
* Objects that are used by external application to interact with kubernetes api, it provides authentication to external application for acessing kubernetes api
* The token is stored as `SERVICE_ACCOUNT_NAME-token-kbbdm`
* Service token secret can be directly mounted as secret in PODs and containers to allow the application to access kubernetes api
* Every namespace has a default service account named as `default`, this default service account always and default mounted to all the containers and pods

* This auto mount behaviour can be removed by specifying `automountServiceAccountToken` as `false` in `POD` `spec` section

| COMMAND | EFFECT |
| ------- | ------ |
| `kubectl get serviceaccount` | Get service accounts |
| `kubectl create serviceaccount SERVICE_ACCOUNT_NAME` | Create service account |
| `kubectl create token SERVICE_ACCOUNT_NAME` | Create service account token |

### PRIVATE REGISTRIES
* For accessing private registry images, first create a kubernetes secret having all the values as below
```bash
kubectl create secret docker-registry SECRET_NAME \
--docker-server=REGISTRY_URL \
--docker-username=REGISTRY_USER \
--docker-password=REGISTRY_USER_PASSWORD \
--docker-email=EMAIL
```
* After that while creating pod pass this secret as
```yaml
spec:
  containers:
    - name: CONTAINER_NAME
      image: IMAGE_PATH_WITH_FULL_REGISTRY_ADDRESS
  imagePullSecrets:
    - name: SECRET_NAME
```

### SECURITY CONTEXTS
* Like docker, user and capabilities can also be defined in kubernetes too
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: web-pod
spec:
  containers:
    - name: ubuntu
      image: ubuntu
      command: ["sleep", "3600"]
      securityContext:
        runAsUser: 1000
        capabilities:
          add: ["CAPABILITY1", ...] # like MAC_ADMIN
```

## NETWORKING
### CNI (CONTAINER NETWORKING INTERFACE)
* It is a component of the kubernetes
* It standardizes the networking architecture for all orchestration frameworks
* It also lays down the guidelines to develop custom networking solutions in form of plugins
* There are many plugins available

> [!NOTE]
> * Plugins are stored at `/opt/cni/bin` (the path where the container runtime seeks for plugins)
> * CNI configurations are stored at `/etc/cni/net.d` in conf files named in the format `CONFIG_NAME.conflist`

### SERVICE NETWORKING
* Implemented using iptables
* Both services and PODs have non colliding CIDR range for networking

| COMMAND | EFFECT |
| ------- | ------ |
| `cat /var/log/kube-proxy.log` | Get entries and logs for `kube-proxy` |


### DNS
* We have already seen how services are resolved in kubernetes
* For resolving the pods , kubernetes re-formats the IP of pods for internal resolution by replacing all `.` by `-`
* Pods are resolved in similar manner too (FQDN) `PROTOCOL://KUBERNETES_RENAMED_IP.NAMESPACE.pod.ROOT`. Normally the `ROOT` is `cluater.local`

### CoreDNS
* We can enter hostname in all the nodes in the `/etc/hosts` file but it is not feasable to do on all nodes as a large number of pods and services are being created and destroyed
* So, its feasable to use a DNS server and enter the IP of dns server in `/etc/resolv.conf` on all nodes, done automaticlly by the `kubelet`
* It is implemented as a POD in the network and its config is at `/etc/coredns/Corefile`
* The PODs can also reach to the CoreDNS because there is a srvice also created for it named as `kube-dns` in the kubernetes namespace
* The kubelet config file also have the IP of `kube-dns` service so that it can reach it to resolve other services.
* Configuration for `kubelet` is stored at `/var/lib/kubelet/config.yaml` and it is passed to PODs as `configmap`

| COMMAND | EFFECT |
| ------- | ------ |
| `kubectl get sercvice -n kube-system` | See `kube-dns` service |
| `kubectl describe configmap -n kube-system` | See configmap for coredns |
| `kubectl get configmap -n kube-system` | Get all the configmap in the `kube-system` namespace |


### INGRESS
* It is a CRD of type `service`
* Helps users to access services using single externally accessable URL and maintain SSL security as well
* It can be exposed as `NodePort` or cloud native load balancer
* Nginx ingress controller needs to be installed manually in the cluster (controllers for HA Proxy, GCE and traefik are also available)

* Nginx controller can be deployed as either as deployment or a helm chart can be installed and then objects can be created, but for this to work, correct roles, configmap and service accounts are needed to be created. The better approach is to install a helm chart
* `Annotations` can be used in ingress objects to get special cloud features














## CLUSTER MAINTENANCE
### OS UPGRADES
* Senarios where we need to take down nodes:

* If one node was down for some purpose and it come back online quickly, there is no probelem but if not then after 5 minutes, the PODs are reschedule back on other nodes if they a part of replica set. But if the PODs are not a part of replica set they face issue if any other node is not serving the similar POD and the POD on the taken down node is not rescheduled

> [!NOTE]
> * We can take down a node and make a quick upgrade with 5 minutes if the node have capability to come back online in 5 minutes and the PODs on the node are part of a replica set

> [!IMPORTANT]
> * There is a safer way to upgrade a node where the node is drained, which in turn results in re scheduling of all work loads fom the node to other nodes
> * PODs are gracefully recreated, node is marked "cordoned" (unschedulable)
> * Now we can safely uugrade and reboot the node
> * After this we need to mark the node as "uncordoned" (schedulable)
> * The old PODs do not automatically fallback on the same node

| COMMAND | EFFECT |
| ------- | ------ |
| `kubectl drain NODE_NAME` | Drain a node |
| `kubectl get nodes` | Get nodes in the cluster |
| `kubectl cordon NODE_NAME` | Make node unschedulable but not reschedule old running containers |
| `kubectl uncordon NODE_NAME` | Uncordon the node |

### KUBERNETES VERSION CONTROL and CLUSTER UPGRADE
* Kubernetes follows a standard way of releases in the format `MAJOR.MINOR.PATCH`
* Ther are many core utilities and tools in the kubernetes core
    - kube-apiserver
    - Kube-Controller-Manager
    - kube-scheduler
    - kubelet
    - kube-proxy
    - kubectl
    - core DNS
    - ETCD

> [!NOTE]
> * It is not mandatory to have same version for all of these coer components
> * kube-apiserver is the primary component and is the component that other components that talk to
> * None of the other component should be at a higher version thatn the kube-apiserver
> * kube-controller-manager cand kube-scheduler can be at one version lower (1.10, 1.9)
> * kubelet and kube-proxy can be 2 versiona lower (1.10, 1.8)
> * kubctl can be one version up or low than the kube-apiserver

> [!IMPORTANT]
> * If kubernetes releases a new version (1.13) and one is at 1.10, one must upgrade it sequentially from to 1.11 and upgrade other core components then 1.12 and upgrade other core components then 1.13 and upgrade other core components, not directly to 1.13 as some other components may become become unsupported for a while which can lead to cluster instability for a while

> [!IMPORTANT]
> #### UPGRADATION STEPS
> * First upgradd the master nodes and then the worker nodes
> * As the master nodes go down, the control plane components like apiserver, scheduler, controller-managers go down briefly, workers keep working normally
> * During this time while the master nodes are down, if any POD on worker nodes go down, it is not recreated until the master nodes come back online
> * FOr upgrading the worker nodes, one must upgrade one at a time or add new nodes with latest software (supported) and drain the older nodes permanently

> [!IMPORTANT]
> #### UPGRADE USING KUBEADM
> * Run `kubeadm upgrade plan` to see the plan, where it lists the current version, and what versions it can be upgraded to.
> * Kubeadm also tells that we must manually upgrade the kubelet versions on each node as it does not upgrade kubelets
> * Kubeadm must be upgraded too before upgrading the cluster, it also follows the same kubernetes version
> * First upgrade the kubeadm `kubeadm upgrade -y kubeadm=1.12.0-00`
> * Then upgrade the cluster `kubeadm upgrade apply v1.12.0`, all the controlplane components are at the specified version, but incase of multiple controlplane the command to run is `sudo kubeadm upgrade node` on the other controlplanes
> * Run `kubectl get nodes` to see that the nodes are still at a older version including the master node as kubelet is not upgraded
> * One might not have the kubelets running on the master nodes depending on the way the cluster is setup, but while using the kubeadm tool the kubelets are running on the master node to run the controlplane components as PODs on the master node
> * When the cluster is setup from scratch, there may be no kubelet running on the master node
> * Upgrade the kubelet on master using `apt-get upgrade -y kubelet=1.12.0-00`
> * Restart the servce `systemctl restart kubelet`
> * Run `kubectl get nodes` to see the master node having the version as we upgraded but not the worker nodes
> * Now its time to upgrade the worker nodes
> * Shift workload from the nodes by draining them `kubectl drain NODE_NAME`
> * Upgrade the kubeadm tool `apt-get upgrade -y kubeadm=1.12.0-00`
> * Upgrade kubelet `apt-get upgrade -y kubelet=1.12.0-00`
> * Update node config for new kubelet version `kubelet upgrade node config --kubelet-version v1.12.0`
> * Restart kubelet service `systemctl restart kuberlet`
> * Uncordon the node `kubectl uncordon NODE_NAME`

* Drain the node
```bash
kubectl drain NODE_NAME
```
* Do not forget to change the `MAJOR`, `MINOR`, `PATCH` and `*` in the commands
* Now kubeadm needs to be updated
* Replace the apt repository definition so that apt points to the new repository instead of the Google-hosted repository
```bash
echo "deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] https://pkgs.k8s.io/core:/stable:/v1.MINOR_VERSION/deb/ /" | sudo tee /etc/apt/sources.list.d/kubernetes.list
```
* Download the public signing key for the Kubernetes package repositories. The same signing key is used for all repositories, so you can disregard the version in the URL
```bash
curl -fsSL https://pkgs.k8s.io/core:/stable:/v1.MINOR_VERSION/deb/Release.key | sudo gpg --dearmor -o /etc/apt/keyrings/kubernetes-apt-keyring.gpg
```
```bash
sudo apt-get update
```
* Check which latest version is available to upgrade
```bash
kubeadm upgrade plan
```
* Run the following to get the available releases to install kubeadm new version
```bash
sudo apt-cache madison kubeadm
```
* Install kubeadm new version
```bash
apt-get install kubeadm=VERSION
```bash
* Finally the kubeadm is updated, now tme to upgrade the the kubernetes
```bash
kubeadm upgrade plan v1.MINOR.0
kubeadm upgrade apply v1.MINOR.0
```
* Update the kubelet and kubectl
```bash
sudo apt-mark unhold kubelet kubectl && \
sudo apt-get update && sudo apt-get install -y kubelet='1.MINOR.PATCH-*' kubectl='1.MINOR.PATCH-*' && \
apt-get install kubelet=1.MINOR.PATCH-*.* \ 
apt-get install kubelet=1.MINOR.PATCH-*.* \ 
sudo apt-mark hold kubelet kubectl
```
* Restart kubelet
```bash
sudo systemctl daemon-reload
sudo systemctl restart kubelet
```
* Uncordon the node
```bash
kubeclt uncordon NODE_NAME
```
* For other controlplane nodes and the worker nodes the process is te same, except for some steps, just run the below command instead of `kubeadm upgrade node` and the nodes can only be cordoned or uncordoned through only controlplane, rest commands need to be run on the node itself
```
kubeadm upgrade node
```



> [!IMPORTANT]
> * See cluster upgradation on the official docs [here](https://kubernetes.io/docs/tasks/administer-cluster/kubeadm/kubeadm-upgrade/)

### BACKUP and RESTORE
* Items to be backed up:
    - Resource configs
    - ETCD cluster
    - Persistent volumes

> [!NOTE]
> #### BACKUP RESOURCE CONFIG
> * Query the kube-apiserver using kubectl and save all resource config for all objects on the cluster as copy
> * There are tools like Ark and HeptIO to do resource config backups
>
> | COMMAND | EFFECT |
> | ------- | ------ |
> | `kubectl get all --all-namespaces -o yaml > FILE.yaml` | Backup all resource configs on a single file |

> [!NOTE]
> ### BACKUP ETCD
> * ETCD backup is not possible in managed clusters
> | COMMAND | EFFECT |
> | ------- | ------ |
> | `ETCDCTL_API=3 etcdctl snapshot save PATH/*/db` | BAckup ETCD |
> | `ETCD_API=3 etcdctl snapshot restore PATH/*.db --data-dir /var/lib/etcd-from-backup` | Restore ETCD data |
> | `kubectl describe pod etcd-controlplane -n kubesystem` | See details abt ETCD POD andspecially init params for commands and certificates |
> * Before restoring the ETCD data run `service kube-apiserver stop` as the restore process require to restart the ETCD cluster and th kube-apiserver depends on it
> * When ETCD restores from a backup, it initallizes a new cluster configuration and configures the members of ETCD as new members to a new cluster. This prevents a new member from joining a new member from joining the existing cluster
> * After restoring the backup run `nvim /etc/kubernetes/manifests/etcd.yaml` to edit the file and edit the volume mount having path `/var/lib/etcd` to the data directory we specified above
> * Then specify the new directory to initialize in the etcd cluster in the service file and restart the sstemctl daemon, ETCD service and then restart the kube-apiserver
> * If the etcd-controlplane does not come to a running state automatically, just delete theh POD and its replication controller will bring it back up

> [!TIP]
> * While using etcdctl command always provide endpoints, cacert, cert, key
```bash
ETCDCTL_API=3 etcdctl snapshot save PATH/*.db \
--endpoints=https://127.0.0.1:2379 \
--cacert=/etc/etcd/ca.crt \
--cert=/etc/etcd/etcd-server.crt \
--key=/etc/etcd/etcd-server.key
```

> [!TIP]
> #### IDENTIFY INTERNAL and EXTERNAL ETCD SERVER
> * Describe the POD for ETCD and see if the access URL is localhost or not if not then external ETCD



## DESIGN A CLUSTER
### QUESTIONS
* Purpose: education, testing, production
* Location: Cloud, on-premice
* Storage: SSD, network, shared volumes, node selectors with disk types
* Nodes: Virtual or bare metal
* Master Nodes: With ETCD integrated or seperate ETCD
* Openshift by redhat
* Implementation: Turnkey and hosted

### HIGH AVAILABILITY
* Multiple components running on multiple master nodes
* How do they chare work among themself
#### KUBE_APISERVER
* kube-apiserver is alive and running on all the nodes  at the same time
* Both have a different address as both of them are running on different hosts and request must be sent to one of them at once. They must ben load balanced. Then we point the kubectl utility to that load balancer
#### CONTROLLER MANAGER and SCHEDULER
* In case of multi master nodes, controller manager must run in active-standby mode
* There is a lock for selecting the leader or the active unit among the many

### ETCD TOPOLOGY
* Stacked topology: ETCD servers are hosted on hte master nodes or the control nodes themselves
* External ETCD: External ETCD servers are setup

#### READ and WRITE
* Running N ETCD servers
* Read can happen on any node
* Writing happens only through only one node called the leader node and if any write request goes to any other node, it is re directed to leader node, leader node is decided using RAFT concensus
* What if a node goes down? Data written to (N+1)/2 nodes is called a successful write operation which is also called quorum

#### SETUP ETCD EXTERNAL CLUSTER
* Installation
```bash
wget -q --https-only "https://github.com/coreos/etcd/releases/download/v3.3.9/etcd-v3.3.9-linux-amd64.tar.gz"
tar -xvf etcd-v3.3.9-linux-amd64.tar.gz
mv etcd-v3.3.9-linux-amd64/etcd /usr/local/bin/
mkdir -p /etc/etcd /var/lib/etcd
cp ca.pem kubernetes-key.pem kubernetes.pem /etc/etcd/
```
* Service file
```service
ExecStart=/usr/local/bin/etcd \\
--name ${ETCD_NAME} \\
--cert-file=/etc/etcd/kubernetes.pem \\ --key-file=/etc/etcd/kubernetes-key.pem \\
--peer-cert-file=/etc/etcd/kubernetes.pem \\
--peer-key-file=/etc/etcd/kubernetes-key.pem \\
--trusted-ca-file=/etc/etcd/ca.pem \\
--peer-trusted-ca-file=/etc/etcd/ca.pem \\
--peer-client-cert-auth \\
--client-cert-auth \\
--initial-advertise-peer-urls https://${INTERNAL_IP}:2380 \\
--listen-peer-urls https://${INTERNAL_IP}:2380 \\
-listen-client-urls https://${INTERNAL_IP}:2379, https://127.0.0.1:2379 \\
--advertise-client-urls https://$(INTERNAL_IP}:2379 \\
--initial-cluster-token etcd-cluster-0 \\
--initial-cluster peer-1=https://${PEER_IP}:2380,peer-2=https://${PEER_IP}:2380 \\
--initial-cluster-state new \\
--data-dir=/var/lib/etcd
```
* These above steps need to be performed on all the nodes serving ETCD and we need to specify other nodes as peers as in line 15


