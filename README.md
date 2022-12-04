# Kubernetes (K8S)
Kubernetes is a container orchestration system. Kubernetes allows us to create containers on multiple servers (physical or virtual). It is done automatically by telling the Kubernetes how many container will be created based on specific image. Kubernetes takes care of the:

1. Automatic deployment of the containerized applications across different servers.
2. Distribution of the load across multiple server. This will avoid under/over utilization our servers.
3. Auto-scaling of the deployed applications
4. Monitoring and health check of the containers
5. Replacement of the failed containers.

Kubernates supports [some container runtimes](https://kubernetes.io/docs/setup/production-environment/container-runtimes/):
1. [Docker](https://www.docker.com/)
2. [CRI-O](https://cri-o.io/)
3. [Containerd](https://containerd.io/) (pronounced: Container-dee)

# Pod
In Docker, Container is the smallest unit. Meanwhile in Kubernetes the smallest unit is Pod.

<img src="https://github.com/neutrofoton/HiKubernetes/blob/main/images/ss_pod.png" alt="" width="75%"/>

A pod can be contain one or more containers. There are also shared volumes and network resources (for example ip address) among containers in the same Pod. 

In most common use case there is only a single container in a Pod. However sometime if some containers have to be tighten one another it can be serveral containers in a Pod. **Please keep in mind that one Pod should be in one Server**

# Node
A Node is a Server. It can contain one or more Pods.

<img src="https://github.com/neutrofoton/HiKubernetes/blob/main/images/ss_node.png" alt="" width="75%"/>

 

# Kubernetes Cluster
Kubernetes Cluster contains serveral Nodes. The nodes can be located in different location. Usually the nodes in a Kubernetes Cluster close to each other in order to perform jobs more efficiently. 

<img src="https://github.com/neutrofoton/HiKubernetes/blob/main/images/ss_kubernetes_cluster.png" alt="" width="75%"/>

In Kubernetes Cluster there is a **Master Node**. Other Nodes in the cluster are called **Worker Node**. 

The Master Node manages the Worker Nodes. It's the Master Node jobs to distribute load across other Woker Nodes. All Pods related to our applications are deployed in the Worker Nodes. The Master Node runs only System Pods which are reponsible for the Kubernetes Cluster jobs in general. In short, the Master Node is the control of the Worker Nodes in Kubernetes Cluster and does not run our applications. 

<img src="https://github.com/neutrofoton/HiKubernetes/blob/main/images/ss_master_worker_nodes.png" alt="" width="75%"/>

Every cluster has at least one worker node. In a node there are several [some services](https://kubernetes.io/docs/concepts/overview/components/). Roughly it's shown in the following pictures.

<img src="https://github.com/neutrofoton/HiKubernetes/blob/main/images/ss_kubernetes_service.png" alt=""/>

## Node Components:
Node components run on every node, maintaining running pods and providing the Kubernetes runtime environment.

1. ***Contriner Runtime*** <br/>
The container runtime is the software that is responsible for running containers.

2. ***kubelet*** <br/>
An agent that runs on each node in the cluster. It makes sure that containers are running in a Pod.

3. ***kube-proxy*** <br/>
kube-proxy is a network proxy that runs on each node in your cluster, implementing part of the Kubernetes Service concept.

## Control Plane Component
The control plane's components make global decisions about the cluster (for example, scheduling), as well as detecting and responding to cluster events (for example, starting up a new pod when a deployment's replicas field is unsatisfied).

1. ***kube-apiserver*** <br/>
The API server is a component of the Kubernetes control plane that exposes the Kubernetes API. The API server is the front end for the Kubernetes control plane.

2. ***kube-scheduler***<br/>
Control plane component that watches for newly created Pods with no assigned node, and selects a node for them to run on

3. ***kube-controller-manager*** <br/>
Control plane component that runs controller processes.

4. ***cloud-controller-manager*** <br/>
A Kubernetes control plane component that embeds cloud-specific control logic. The cloud controller manager lets you link your cluster into your cloud provider's API, and separates out the components that interact with that cloud platform from components that only interact with your cluster.
The cloud-controller-manager only runs controllers that are specific to your cloud provider. If you are running Kubernetes on your own premises, or in a learning environment inside your own PC, the cluster does not have a cloud controller manager.

5. ***etcd*** <br/>
Consistent and highly-available key value store used as Kubernetes' backing store for all cluster data.