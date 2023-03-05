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

Every cluster has at least one worker node. In a Node there are several [components or services](https://kubernetes.io/docs/concepts/overview/components/). Roughly it's shown in the following pictures.

<img src="https://github.com/neutrofoton/HiKubernetes/blob/main/images/ss_kubernetes_service.png" alt=""/>

## Node Components:
Node components run on every node, maintaining running pods and providing the Kubernetes runtime environment.

1. ***Contriner Runtime*** <br/>
The <code>container runtime</code> is the software that is responsible for running containers.

2. ***kubelet*** <br/>
The <code>kubelet</code> is an agent that runs on each node in the cluster. It makes sure that containers are running in a Pod.

3. ***kube-proxy*** <br/>
The <code>kube-proxy</code> is a network proxy that runs on each node in your cluster, implementing part of the Kubernetes Service concept. It is reponsible for network communication inside on each Node and between Nodes 

## Control Plane Component
The control plane's components make global decisions about the cluster (for example, scheduling), as well as detecting and responding to cluster events (for example, starting up a new pod when a deployment's replicas field is unsatisfied).

1. ***kube-apiserver*** <br/>
The <code>kube-apiserver</code> is a component of the Kubernetes control plane that exposes the Kubernetes API. The API Service is the main communication between different Node.

2. ***kube-scheduler***<br/>
The <code>kube-scheduler</code> is responsible for planning and distribution load in the cluster. It watches for newly created Pods with no assigned node, and selects a node for them to run on. 

3. ***kube-controller-manager*** <br/>
The <code>kube-controller-manager</code> runs controller processes. It controls each Node in the Cluster.

4. ***cloud-controller-manager*** <br/>
The <code>cloud-controller-manager</code> embeds cloud-specific control logic. The cloud controller manager lets you link your cluster into your cloud provider's API. If we are running Kubernetes on our own premises, or in a learning environment inside our own PC, the cluster does not have a cloud controller manager.

5. ***etcd*** <br/>
The <code>etcd</code> stores all cluster data in key value format.

# Installations

## Installing kubectl
Installation steps of <code>kubectl</code> can be found [here](https://kubernetes.io/docs/tasks/tools/)

``` bash
# install with brew
brew install kubectl

# check the version
kubectl version --client

# or the new command check version
kubectl version --output=json
```
## Installing minikube
Installation steps of <code>minikube</code> can be found [here](https://minikube.sigs.k8s.io/docs/start/)

``` bash
# install with brew
brew install minikube

# check the installation
minikube version
```

```
minikube version: v1.29.0
commit: ddac20b4b34a9c8c857fc602203b6ba2679794d3
```

Starting the <code>minikube</code>
``` bash
# starting minikube
minikube start
```
> The environtiment machine I use:
> 1. Macos Monterey 12.6.3
> 2. VirtualBox-6.1.42 for Macos (intel)
> 
> I got an issue when using version 7.0 of virtual box. The issue was <code>["The host-only adapter we just created is not visible"]( https://github.com/kubernetes/minikube/issues/15377)</code>. It is solved by downgrading the VirtualBox version to 6.1.42


When we successfully starting minikube, checking the version will give us server and client information

``` bash
kubectl version --output=json
```

``` json
{
  "clientVersion": {
    "major": "1",
    "minor": "25",
    "gitVersion": "v1.25.2",
    "gitCommit": "5835544ca568b757a8ecae5c153f317e5736700e",
    "gitTreeState": "clean",
    "buildDate": "2022-09-21T14:33:49Z",
    "goVersion": "go1.19.1",
    "compiler": "gc",
    "platform": "darwin/amd64"
  },
  "kustomizeVersion": "v4.5.7",
  "serverVersion": {
    "major": "1",
    "minor": "26",
    "gitVersion": "v1.26.1",
    "gitCommit": "8f94681cd294aa8cfd3407b8191f6c70214973a4",
    "gitTreeState": "clean",
    "buildDate": "2023-01-18T15:51:25Z",
    "goVersion": "go1.19.5",
    "compiler": "gc",
    "platform": "linux/amd64"
  }
}
```

