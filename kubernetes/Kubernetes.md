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

#### NB

Starting with version [1.20.0](https://github.com/kubernetes/kubernetes/blob/master/CHANGELOG/CHANGELOG-1.20.md) the name _Master Node_ gets deprecated. Instead another term is used which is [**Control Plane**](https://kubernetes.io/docs/concepts/overview/components/).

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

#### NB

The output of `kubectl get pods` looks like this:

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