# Kubernetes Lecture

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