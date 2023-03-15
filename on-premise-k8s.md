# Configuring Kubernetes Cluster

## 1. Changing Hostname for All machine
``` bash
# host name for master
sudo hostnamectl set-hostname master.example.com
exec bash

# host name for worker 1
sudo hostnamectl set-hostname worker-node-1.example.com
exec bash

# host name for worker 2
sudo hostnamectl set-hostname worker-node-2.example.com
exec bash
```

## 2. Installing Kubernetes
...... installation step here


## 3. Setting up the Master Node and configuring the cluster

Initiate <code>kubeadm</code>
``` bash
sudo kubeadm init --pod-network-cidr=191.168.0.0/16
``` 

<img src="https://github.com/neutrofoton/HiKubernetes/blob/main/images-on-premise/kubeadm-init.png" alt=""/>


> *if you are greeting error then run this*
>
> 1. Reset the <code>kubeadm</code>
> ``` bash
> sudo kubeadm reset
> ```
> 2. Create docker <code>daemon.json</code>
> ``` bash
> cat <<EOF | sudo tee /etc/docker/daemon.json
> {
>     "exec-opts":["native.cgroupdriver=systemd"],
>     "log-driver":"json-file",
>     "log-opts":{
>         "max-size":"100m"
>     },
>     "storage-driver":"overlay2"
> }
> EOF
> ```
> 
> 3. Restarting docker
> ``` bash
> sudo systemctl enable docker
> sudo systemctl daemon-reload
> sudo systemctl restart docker
> sudo systemctl status docker
>
> # swapoff disables swapping on the specified devices and files. 
> # -a flag is given, swapping is disabled on all known swap devices and files (as found in /proc/swaps or /etc/fstab)
>
> sudo swapoff -a
> 
> ```
> <br/>

Run the following commands on the master node to allow non-root users to access use kubeadm:

``` bash
mkdir -p $HOME/.kube 
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config

```
> If <code>cp</code> command ask to overrides, do yes.

For joining the worker nodes to the cluster, run the following command
``` bash
sudo kubeadm token create --print-join-command
```

<img src="https://github.com/neutrofoton/HiKubernetes/blob/main/images-on-premise/kubeadm-token.png" alt=""/>


Add the Weave Net CNI plugin for highly available cluster
``` bash

# download the kube-flannel.yml
curl https://github.com/flannel-io/flannel/releases/latest/download/kube-flannel.yml

```
> Don't forget change the network IP to 
> ```
> "Network": "192.168.0.0/16",
> ```
> according the <code>pod-network-cidr</code> parameter in the <code>kubeadm</code> initialization.
>

Then apply the <code>kube-flannel.yml</code>

``` bash
# apply the kube-flannel.yml
kubectl apply -f kube-flannel.yml
```

## 4. Joining the Worker Nodes to The Cluster

On the <code>worker-node-1</code> and <code>worker-node-2</code> run the following command to join to the cluster.

``` bash
kubeadm join 172.31.53.81:6443 --token zoimxu.lmx5bxxxxx --dicovery-token-ca-cert-hash sha256:485d79d1a78bxxxxxxxxxxxxxxxxxxxx
```

To checking the nodes whether joined to the cluster

``` bash
# get all joined nodes in the cluster
kubectl get nodes

# get the namespaces
kubectl get namespaces

# get pods with namespace
kubectl get pods --all-namespaces
kubectk get pods -A
```

<img src="https://github.com/neutrofoton/HiKubernetes/blob/main/images-on-premise/kubeadm-check-nodes.png" alt=""/>

> If you got status <code>ContainerCreating</code> for long time, 
you can conside doing a full reboot of VMs that are running the Kubernetes master node and Kubernetes worker nodes. 

<img src="https://github.com/neutrofoton/HiKubernetes/blob/main/images-on-premise/kubeadm-check-nodes-ok.png" alt=""/>

If further issue still occur with the pods you can check using
``` bash
kubectl describe pod <name>
```

# References
1. https://stackoverflow.com/questions/48190928/kubernetes-pod-remains-in-containercreating-status
