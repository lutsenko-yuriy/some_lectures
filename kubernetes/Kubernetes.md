# Kubernetes Lecture

Based on [this tutorial](https://www.youtube.com/watch?v=X48VuDVv0do).

## Basic concepts

### Main components

A K8s consists of [**nodes**](https://kubernetes.io/docs/concepts/architecture/nodes/) - simple server, either physical or virtual machines.

Inside these nodes are smallest deployable units called [**pods**](https://kubernetes.io/docs/concepts/workloads/pods/) - some abstract containers (like Docker containers but more abstract in order not to get coupled to Docker). Each pod has its own IP address.

Pods can die in the middle of their work for some reasons. A new one would be created but it will have another IP address.

Because of that [**services**](https://kubernetes.io/docs/concepts/services-networking/service/) were created. They have permanent IP addresses and are getting attached to pods. Since services' and pods' lifecycles do not depend on each other, pods can die but services will remain.

Services can be internal (available only from inside the cluster) and external (available from outside of the cluster).

In order to perform name-based virtual hosting or load balancing when we talk about external services we use [**Ingresses**](https://kubernetes.io/docs/concepts/services-networking/ingress/).

### ConfigMaps and Secrets

In order to communicate to other pods a pod needs their IPs or URLs. If we decide to store them inside the pod itself we will encounter an unpleasant issue that every time an IP or a URL changes we have to redeploy the pod.

In order to minimize such operations a [**ConfigMap**](https://kubernetes.io/docs/concepts/configuration/configmap/) was introduced. We create one, add everything we need into this ConfigMap and then connect the ConfigMap with the pod. Now when an IP or a URL changes we just change whatever is inside the ConfigMap without any other manipulations.

Though sometimes such data (e.g., database credentials) can be sensitive and we don't want to store the data in a relatively insecure ConfigMap. We store such data in a [**Secret**](https://kubernetes.io/docs/concepts/configuration/secret/).

### Volumes

When a database pod gets restarted everything on the previous database pod is not restored unless we dedicate a separate [**Volume**](https://kubernetes.io/docs/concepts/storage/volumes/) - an abstraction attaching a physical storage in the node or outside of the node.

### Deployment and statefulsets

Most likely we will have several similar pods for redundancy purposes (usually both work but while one can go down the other one would still be running). They both can be connected to the same service so there is no need to adjust an endpoint in case of any changes.

Of course we can build and run two pods by ourselves but that is a tedious task. So it would be better to have some sort of "blueprint" to run pods based on this "blueprint".

A [**deployment**](https://kubernetes.io/docs/concepts/workloads/controllers/deployment/) is a "blueprint" for cases when pods are stateless (webservers).

When pods are stateful and share the same state (databases) we use [**statefulsets**](https://kubernetes.io/docs/concepts/workloads/controllers/statefulset/) which also control who should modify the state now.

Managing statefulsets can be pretty difficult so it is recommended not to keep databases in a K8s cluster but rather somewhere else.

## K8s architecture

### Worker nodes

One of main elements is a worker node. Each worker node runs one or several pods.

A worker node has at least 3 processes:

* A container runtime (e.g., docker)
* A [kubelet](https://kubernetes.io/docs/reference/command-line-tools-reference/kubelet/) which interacts (or can interact) with the runtime and the node itself. It starts a pod with a container inside.
* A [kube proxy](https://kubernetes.io/docs/reference/command-line-tools-reference/kube-proxy/) which organizes effectively forwarding requests from services to pods.

### Master Nodes

Master Nodes are responsible for organizing a cluster. That includes adding/removing/restarting pods or nodes as well as monitoring.

Unlike worker nodes, master nodes have 4 processes:

* An API server - a cluster's gateway as well as a gatekeeper for authentication.
* A scheduler - the process responsible for scheduling and assigning a pod to one of worker nodes (usually the least busy) while the kubelet in the worker node starts the pod.
* A controller manager - the process responsible for monitoring if any pods died. It finds ones and then makes a request to the scheduler so it would restart the pods.
* [etcd](https://etcd.io/) - a key-value store. Stores all the changes in the cluster. Only stores the cluster-related data. Data of the application running on the cluster is not stored there.

Usually there are also several master nodes for a redundancy.

**NB**: Starting with version [1.20.0](https://github.com/kubernetes/kubernetes/blob/master/CHANGELOG/CHANGELOG-1.20.md) the name _Master Node_ gets deprecated. Instead another term is used which is [**Control Plane**](https://kubernetes.io/docs/concepts/overview/components/).

The processes from the lecture are called almost the same but `kube-` is added to the beginning of each process (except etcd, it did not change). Also there is a new process called [cloud controller manager](https://kubernetes.io/docs/concepts/overview/components/#cloud-controller-manager) which links the cluster with a used cloud provider's API.

For the rest of the lecture this entity will still be referred to as a Master Node.

### [An example cluster](https://www.youtube.com/watch?v=X48VuDVv0do&t=33m8s)

As rule, simple applications start with two master nodes and three worker nodes. For obvious reasons master nodes require less computing power, even though they are crucial for the cluster.

## Practice

### Minikube and Kubectl

[**Minikube**](https://github.com/kubernetes/minikube) is a one node cluster running both master and worker processes. It is used to test a cluster locally without deploying it somewhere else. It has a docker preinstalled.

Minikube starts a virtual box (virtual machine?) and runs the node on the virtual box.

In order to interact with the Minikube a **Kubectl** is used. It is a command line tool for a K8s cluster. Basically it interacts with the API server running in a master node in any cluster, not only the API server in the Minikube node.

### [Useful commands](https://kubernetes.io/docs/reference/kubectl/)

`kubectl get`:

* `nodes` - show the list of the nodes in the connected cluster
* `pods` - show the list of the pods in the connected cluster
* `services` - show the list of the services in the connected cluster
* `replicaset` - show the [replicasets](https://kubernetes.io/docs/concepts/workloads/controllers/replicaset/) - another layer between deployment and a pod, which maintains a stable set of replica pods and guarantees that a certain specific amount of identical pods would be running at the same time.

`kubectl create`:

* `deployment` - creates a new deployment

`kubectl edit`:

* `deployment` - opens a text editor where you can edit a deployment. By default it is Vi.

`kubectl delete`:

* `deployment` - deletes the deployment and all the pods built based on this deployment.

`kubectl apply -f <file>` - applies the content of the file (YAML format) to the cluster

**NB**: The output of `kubectl get pods` looks like this:

```text
NAME                                READY   STATUS    RESTARTS   AGE
nginx-deployment-5fbdf85c67-mb6r5   1/1     Running   0          48s
```

while the output of `kubectl get replicaset` looks like this:

```text
NAME                          DESIRED   CURRENT   READY   AGE
nginx-deployment-5fbdf85c67   1         1         1       95s
```

Notice that the name of the pod starts with the name of the replicaset.

### Debugging

`kubectl logs <pod name>` - gets logs from the pod
`kubectl describe pod <pod name>` - shows info on the pod including which states the pod has gone through
`kubectl exec -it <pod name> -- bin/bash` - shows a terminal inside the pod

## Setting up a cluster with configuration files

We can build a cluster based on configuration files. These files are usually in a format YAML. Each configuration consists of these parts:

1) Version of an API for a created component (`apiVersion: apps/v1`)
2) What component should be created (`kind: Deployment`)
3) Metadata (like name)
4) Specification with component specific fields
5) A current status generated by Kubernetes required to modify the configuration (e.g., when you decided to have 2 replicas instead of 1, Kubernetes wi1l create another replica without touching the previous one). It gets updated continuously. The information is fetched from etcd.

### Deployments in configuration files

Specification for deployments has a field called `template`. Inside this template there are also metadata and specs. Since we have to create pods we add a component inside a component.

### Labels and selectors

In order to organize connection between components labels and selectors are used.

Labels are a part of metadata used to mark component.

Selectors are a part of specs used to find other component in order to connect to them.

## Namespaces

[**Namespace**](https://kubernetes.io/docs/concepts/overview/working-with-objects/namespaces/) is something to organize resources. They are used to isolate one group of resources from the rest of the cluster (another cluster inside the bigger cluster).

By default in a freshly created cluster there are 4 namespaces:

* `kube-system` for system processes, master processes or kubectl processes. You should not create or modify anything in this namespace
* `kube-public` for publicely accessible data like configmaps.
* `kube-node-lease` for information on heartbeats from nodes. Each node has an associated lease object in the namespace which determines the availability of a node.
* `default` for creating resources unless you specify other namespaces and create resources in them.

They can be defined either by `kubectl create namespace <name of a new namespace>` or by writing a `namespace` property in the component's metadata in a YAML configuration file. The latter will also create the component in this namespace while creating a component in a command line would require additional argument `-n <name of the namespace>`.

Namespaces can be useless in small projects but when the project is getting bigger and a lot of engineers are working on it and a lot of users use it.

Using namespaces has limitation

* You cannot simply work with resources such as ConfigMaps and Secrets from other namespaces. On the other hand you can connect to services from other namespaces but in this case a URL of this service would be [a standard URL + '.' + the other namespace name](https://www.youtube.com/watch?v=X48VuDVv0do&t=6972s).
* Some components cannot be created within a namespace, only in a global cluster. These are volumes and nodes

### Kubens

IF we want not to add this `-n <namespace>` all the time we can use [**kubens**](https://github.com/ahmetb/kubectx) to switch between namespaces and all the K8s operations will be performed in the new namespace.

## Ingress

We can connect to a K8s cluster by connecting to a one of the external services with their IPs and ports. This is enough when we test a cluster but we need a human-readable format when we run the cluster on prod. For this purposes [**Ingress**](https://kubernetes.io/docs/concepts/services-networking/ingress/) was created.

Ingress also allows us to refer to internal services instead of external ones. One of its specification properties is (routing) rules according to which redirecting requests happens.

In order to implement an Ingress we need to run another node or several nodes called [**Ingress controllers**](https://kubernetes.io/docs/concepts/services-networking/ingress-controllers/) which evaluate and process Ingress rules. The ingress will be an entry point to the cluster. There are a lot of different implementations of ingress controllers. By default the K8s cluster provides K8s Nginx Ingress Controller.

### Implementation order

First we should start the controller. If we want the one provided by default in Minikube we type
```
minikube addons enable ingress
```

In the video it has created the controller pod `ingress-nginx-controller` in `kube-system` namespace but since `Minikube v1.19` [it will create another namespace](https://kubernetes.io/docs/tasks/access-application-cluster/ingress-minikube/) called `ingress-nginx` and we will find the pod there.

### HTTPS

Another property is getting added into specs which is `tls`:

```yaml
spec:
  tls:
  - hosts:
    - myapp.com
    secretName: myapp-secret-tls
# ...
```

where `myapp-secret-tls` is a secret with data we define like this

```yaml
data:
  tls.crt: <base64 encoded cert>
  tls.ket: <base64 encoded key>
type: kubernetes.io/tls
```

## [Helm package manager](https://helm.sh/)

Helm is a package manager for Kubernetes like docker hub where you can find some already created YAMLs instead of writing your own YAMLs.

These YAMLs are organized into so called Helm Charts. These are useful when, for example, you want to build some [ELK Stack](https://www.elastic.co/what-is/elk-stack) with all the secrets and config maps.

Also Helm can work as a templating engine so we can define a common blueprint and in this blueprint we can replace some values with other ones coming from `values.yaml`.

### Helm chart structure

* Name of chart which is also a top level folder
* `Chart.yaml` - metadata on chart
* `values.yaml` - default values for charts
* `charts` - chart dependencies folder
* `templates` - the folder with actual templates

Besides that there can be some README or LICENCE files.

We deploy those YAML files with `helm install <chartname>`.

The `values.yaml` can be overriden. For example, when default `values.yaml` look like this

```yaml
imageName: myapp
port: 8080
version: 1.0.0
```

we can redefine some of values in `my-values.yaml` which looks like this

```yaml
version: 2.0.0
```

After that with a command `helm install --values=my-values.yaml <chartname>` we will deploy all the templates with values like this:

```yaml
imageName: myapp
port: 8080
version: 2.0.0
```

## Volumes

As already mentioned when a pod gets restarted everything on the previous pod is not restored unless we dedicate a separate [**Volume**](https://kubernetes.io/docs/concepts/storage/volumes/) - an abstraction attaching a physical storage in the node or outside of the node.

After a pod being killed volumes allow to persist the data for this pod. Since it is impossible to predict on which exact node a new pod will be created instead of the old one volumes are bound to a cluster but not to a single namespace or a node.

Among the requirements to the volumes should be:

1) Independency from the pod's lifecycle
2) Availability on all nodes
3) Being able to survive even the cluster crashes

**NB**: several volumes (not necessary of the same type) can be assigned to a single pod.

### Persistent volumes

[**Persistent volumes (or PVs)**](https://kubernetes.io/docs/concepts/storage/persistent-volumes/) can be treated as some sort of cluster resource (like RAM, CPU). In general it is an interface to a real storage, be it a local hard drive or some cloud storage like Amazon EBS.

Nevertheless if we are talking about local volume types they do not meet some of the requirements:

1) They are coupled to a single node
2) They die with the whole cluster crashed.

So it is recommended to use a remote storage.

It is obvious that persistent volumes should be created before a pod depending on it is created. Be it a local storage or remote storage, it is a responsibility of a K8s administrator to provide a required volume.

### Persistence volume claim

While PVs are created by a K8s administrator a K8s user can crated the so-called **persistent volume claim (or PVC)** which contains the info on a required storage (is it AWS? what is the required space in GiBs?) so the user can know that there is a storage meeting his requirements.

So the work of a PVC looks like this:

1) A pod requests a volume with a PVC
2) The PVC tries to find a PV meeting the requirements of the PVC in the cluster
3) The volume has now the actual storage and gets mounted to the pod

Unlike volumes, PVCs are coupled to namespaces and the PVC should be in the same namespace as the pod.

### Volumes' special cases

ConfigMaps and Secrets can also be looked at as special cases of volumes. They are local volumes, created neither via PV nor via PVC and managed by K8s.

### Storage classes

All the PVC are satisfied manually by default. When the project grows this process can get expensive and time-consuming.

For that reason [**Storage classes (or SCs)**](https://kubernetes.io/docs/concepts/storage/storage-classes/) were introduced.

SCs provide PVs dynamically in a background when a PVC claims one.

Now when an SC is introduced the process of getting a new volume looks like this:

1) A pod requests a volume with a PVC
2) The PVC requests storage from the SC
3) The SC creates a new PV meeting the requirements of the PVC in the cluster
4) The volume has now the actual storage and gets mounted to the pod

### ADDITIONAL: Main roles in the K8s cluster

* Administrators - the ones setting up and maintaining a cluster, including making sure the cluster has enough resources. Sysadmins and DevOps engineers are the one in this role.
* Users - deploy applications into the cluster either manually or automatically with CI/CD servers' help. These are DevOps teams in development teams.

## StatefulSets

A [**StatefulSet**](https://kubernetes.io/docs/concepts/workloads/controllers/statefulset/) is a special K8s component that runs a stateful app.

### Differences between StatefulSets and Deployments

Pods created by the same deployment

* Are identical and interchangable
* Can be created in random order with random hashes
* Can have a single service load balancing

Pods created by the same statefulset

* Cannot be created or deleted at the same time
* Cannot be randomly addressed
* Are not identical
* Have a sticky identity able to survive a recreation
* Are not interchangable

### Scaling DB applications

While read operations are idempotent and therefore can go in parallel write operations modify the state and potentially cause data inconsistency.

For that reason the mechanism was introduced with a master node which can both read from and write into the state and worker nodes with read-only replicas of the state. Besides that the replication mechanism is implemented to continuously synchronize the states.

### Pod state

Stateful pods have their own state where they store if they are masters or workers. When a pod dies it gets replaced by a new pod with the same ID. For that reason it is recommended to store states and data on remote volumes.

### Pod identity

As already mentioned stateless pods have the following name structure:

```bash
$(deployment name)-$(replicaset hash)-$(pod hash)
```

The stateful set pods look like this

```bash
$(statefulset name)-$(ordinal)
```

These pods are created in order (pod #(k + 1) will not be created until pod #k is created) and deleted in reverse order (pod #(k - 1) will not be deleted until pod #k is deleted).

Also important that unlike stateless pods which have only the same endpoint provided by a load balancing service each stateful pod has its own endpoint `${pod name}.${governing service domain}`.

**NB**: Although K8s makes a lot to handle stateful pods there is still some challenges for the dev team like the replication issues, backup issues, remote storage issues. The reason is the fitness of the containerized approach in general. It fits really well to stateless applications but not for stateful applications.
