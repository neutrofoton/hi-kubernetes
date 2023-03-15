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

# apply the kube-flannel.yml
kubectl apply -f kube-flannel.yml
```

## 4. Joining the Worker Nodes to The Cluster