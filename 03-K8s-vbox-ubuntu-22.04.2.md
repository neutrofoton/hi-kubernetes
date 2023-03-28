# 1. Configuring Virtual Box VMs
The configuration step of Virtual Box are as follow:

1. Installing a VM <code>k8s</code> using Ubuntu Linux 22.04.2 LTS as a VM reference.
2. Copying disk of the VM as <code>k8s-master.vdi</code>, <code>k8s-worker1.vdi</code>, <code>k8s-worker2.vdi</code>.
3. Using Windows Command Prompt, change the hardisk UUID of each hdd on step <code>1.2</code> with the following command
   ```
   cd C:\Program Files\Oracle\VirtualBox
   C:\..\VirtualBox>VBoxManage.exe internalcommands sethduuid "D:\VMs\disk\k8s-master.vdi"

    UUID changed to: 91ebaec7-c113-4bb0-bc12-baa557f0574e
   ```
4. Applying the step <code>1.3</code> for the  <code>k8s-worker1.vdi</code>, <code>k8s-worker2.vdi</code>.

5. Configuring Network Adapter.
   <img src="https://github.com/neutrofoton/HiKubernetes/blob/main/images-k8s-vbox/k8s-vbox-network-setup.png" alt=""/>
   <br/><br/>
   The *Host Only Adapter* network configuration as follow
   <img src="https://github.com/neutrofoton/HiKubernetes/blob/main/images-k8s-vbox/k8s-vbox-network-host-only.png" alt=""/>
   <br/><br/>
   In the *NAT Adapter* network configuration, create a NAT Network named *K8sCluster*
   <img src="https://github.com/neutrofoton/HiKubernetes/blob/main/images-k8s-vbox/k8s-vbox-network-nat.png" alt=""/>

6. Creating three VMs <code>k8s-master</code>, <code>k8s-worker1</code>, <code>k8s-worker2</code> using the existing disks namely <code>k8s-master.vdi</code>, <code>k8s-worker1.vdi</code>, <code>k8s-worker2.vdi</code>. Makesure the network of *NAT, Host Only Adapter, Bridge* and *NAT Network* enabled. 

   <img src="https://github.com/neutrofoton/HiKubernetes/blob/main/images-k8s-vbox/k8s-vbox-network-enable-adapter-NAT.png" alt=""/>
   <br/><br/>
   <img src="https://github.com/neutrofoton/HiKubernetes/blob/main/images-k8s-vbox/k8s-vbox-network-enable-adapter-host-only.png" alt=""/>
   <br/><br/>
   <img src="https://github.com/neutrofoton/HiKubernetes/blob/main/images-k8s-vbox/k8s-vbox-network-enable-adapter-bridge.png" alt=""/>
   <br/><br/>
   <img src="https://github.com/neutrofoton/HiKubernetes/blob/main/images-k8s-vbox/k8s-vbox-network-enable-adapter-NAT-network.png" alt=""/>

   > Don't forget to refresh the MAC address on each adapter.

7. To ensure the network adapter work well, check using ip check
   ```
   ip addr
   ```
   
   
    
    <sub>*Table 1: Node, IP and Hostname*</sub>
    | Node        | IP Host Only   |Host Name               |
    | :---        |     :---:      |    :---               | 
    | master      | 192.168.56.110 |master.neutro.io        | 
    | worker1     | 192.168.56.111 |worker-node-1.neutro.io |
    | worker2     | 192.168.56.112 |worker-node-2.neutro.io |

   Set the ip of host only adapter of master according Table 1 by editing the <code>/etc/netplan/00-installer-config.yaml</code>

   ``` bash
   vi /etc/netplan/00-installer-config.yaml
   ``` 
   ``` bash
    # This is the network config written by 'subiquity'
    network:
      ethernets:
        enp0s10:
          dhcp4: true
        enp0s3:
          dhcp4: true
        enp0s8:
          dhcp4: false
          addresses:
              - 192.168.56.110/24
        enp0s9:
          dhcp4: true
      version: 2
   ```
   ``` bash
   sudo netplan apply

   # check ip
   #ip addr
   ip --brief addr show
   ```
   
   Set the hostname for master with the following command.
   ``` bash
   # host name for master
   sudo hostnamectl set-hostname master.neutro.io
   exec bash
   ```

   > Apply the same things to the <code>worker1 node</code> and <code>worker2 node</code> by refering to the Table 1.

# 2. Tool installation
Updating the apt package index and installing the followingpackages that are needed in Kubernates and container installation.

``` bash
sudo -i
apt-get update && apt-get upgrade -y
apt-get install -y vim git curl wget apt-transport-https gnupg gnupg2 software-properties-common ca-certificates lsb-release

exit
```

> Repeat on all the other nodes.

# 3. Docker Installation
The Docker installation steps are as follow:

1. Installing Docker 

   ``` bash
   # https://docs.docker.com/engine/install/ubuntu/

   # 1. Adding Docker official GPG key
   sudo mkdir -m 0755 -p /etc/apt/keyrings
   curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg

   # 2. Setting up the repository
   echo \
   "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu \
   $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

   # 3. Updating package apt package index
   sudo apt-get update

   # NOTE: if receiving a GPG error when running apt-get update?
   sudo chmod a+r /etc/apt/keyrings/docker.gpg
   sudo apt-get update

   # 4. Installing Docker Engine, containerd, and Docker Compose.
   sudo apt-get install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
   ```

2. Checking the docker version
   ``` bash
   docker version
   ```

3. Enabling and starting docker
   ``` bash
   sudo systemctl enable docker
   # sudo systemctl start docker
   sudo systemctl daemon-reload
   sudo systemctl restart docker
   ```

   Verifying the docker status
   ``` bash
   sudo systemctl status docker
   ```

   > Repeat on all the other nodes.

# 4. Kubernates Installation
The Kubernates installation steps are described in the following steps:

1. Adding <code>kubernates</code> repository 
   
   ``` bash
   sudo -i
   sudo curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | apt-key add
   sudo apt-add-repository "deb http://apt.kubernetes.io/ kubernetes-xenial main"
   
   exit
   ```

2. Installing <code>kubeadm, kubelet, kubectl</code>
   ``` bash
   sudo apt install -y kubeadm kubelet kubectl
   sudo apt-mark hold kubeadm kubelet kubectl
   ```

   Verifying the installation
   ``` bash
   kubectl version --output=yaml
   kubeadm version
   ```

3. Disabling the swap memory

   Turn off swap
   ``` bash
   sudo sed -i '/ swap / s/^\(.*\)$/#\1/g' /etc/fstab
   ```

   To ensure we disable Linux swap space permanently, edit the <code>/etc/fstab</code> then remark the <code>swap</code> line by adding # (hashtag/comment) sign in front of the line.

   ``` bash
   sudo vim /etc/fstab

   # remark the following line
   #/swap.img	none	swap	sw	0	0
   ```

   To confirm the setting is correct, run the following command.
   ``` bash
   # swapoff disables swapping on the specified devices and files. 
   # -a flag is given, swapping is disabled on all known swap devices and files (as found in /proc/swaps or /etc/fstab)

   sudo swapoff -a
   sudo mount -a
   free -h
   ```

4. Enabling kernel modules and configuring sysctl.
   ``` bash
   # Enable kernel modules
   sudo modprobe overlay
   sudo modprobe br_netfilter

   # Add some settings to sysctl
   sudo vim /etc/sysctl.d/kubernetes.conf

   net.bridge.bridge-nf-call-ip6tables = 1
   net.bridge.bridge-nf-call-iptables = 1
   net.ipv4.ip_forward = 1
   
   # Reload sysctl
   sudo sysctl --system

   ```

   > Repeat steps 4.1 - 4.4 on each server node.

# 5. Kubernetes Cluster Configuration

For building Kubernetes cluster, in this lab we use <code>kubeadm</code>. The steps are:

1. Configuring container runtime (Docker CE runtime)
   ``` bash
   # Create required directories
   sudo mkdir -p /etc/systemd/system/docker.service.d
   
   # Create daemon json config file
   sudo vim /etc/docker/daemon.json


   {
       "exec-opts":["native.cgroupdriver=systemd"],
       "log-driver":"json-file",
       "log-opts":{
           "max-size":"100m"
       },
       "storage-driver":"overlay2"
   }
   
   # Start and enable Services
   sudo systemctl daemon-reload 
   sudo systemctl restart docker
   sudo systemctl enable docker
   ```

2. Installing **Mirantis cri-dockerd** as *Docker Engine shim* for Kubernetes

   For Docker Engine you need a shim interface. The Mirantis cri-dockerd CRI socket file path is <code>/run/cri-dockerd.sock</code>. This is what will be used when configuring Kubernetes cluster.


   ``` bash
   VER=$(curl -s https://api.github.com/repos/Mirantis/cri-dockerd/releases/latest|grep tag_name | cut -d '"' -f 4|sed 's/v//g')

   echo $VER

   ### For Intel 64-bit CPU ###
   wget https://github.com/Mirantis/cri-dockerd/releases/download/v${VER}/cri-dockerd-${VER}.amd64.tgz
   tar xvf cri-dockerd-${VER}.amd64.tgz

   ### For ARM 64-bit CPU ###
   wget https://github.com/Mirantis/cri-dockerd/releases/download/v${VER}/cri-dockerd-${VER}.arm64.tgz
   cri-dockerd-${VER}.arm64.tgz
   tar xvf cri-dockerd-${VER}.arm64.tgz
   ```

   Move the <code>cri-dockerd</code> binary package to <code>/usr/local/bin</code> directory
   ``` bash
   sudo mv cri-dockerd/cri-dockerd /usr/local/bin/
   ```

   To validate the successful installation, check the <code>cri-dockerd</code> version.
   ``` bash
   cri-dockerd --version
   ```

   The next step is configuring systemd units for cri-dockerd
   ``` bash
   wget https://raw.githubusercontent.com/Mirantis/cri-dockerd/master/packaging/systemd/cri-docker.service
   wget https://raw.githubusercontent.com/Mirantis/cri-dockerd/master/packaging/systemd/cri-docker.socket
   
   sudo mv cri-docker.socket cri-docker.service /etc/systemd/system/
   sudo sed -i -e 's,/usr/bin/cri-dockerd,/usr/local/bin/cri-dockerd,' /etc/systemd/system/cri-docker.service
   ```

   Reload and enable the services
   ```
   sudo systemctl daemon-reload
   sudo systemctl enable cri-docker.service
   sudo systemctl enable --now cri-docker.socket
   ```
   
   To ensure the service is running, check the status of <code>cri-docker</code>
   ``` bash
   systemctl status cri-docker.socket
   ```

3. Initializing master node

   In the master node, make sure that the <code>br_netfilter</code> module is loaded:
   ``` bash
   lsmod | grep br_netfilter
   ```

   Enable kubelet service.
   ``` bash
   sudo systemctl enable kubelet
   ```

   We now want to initialize the machine that will run the control plane components which includes etcd (the cluster database) and the API Server.
   ``` bash
   # sudo kubeadm config images pull
   sudo kubeadm config images pull --cri-socket /run/cri-dockerd.sock
   ```

   
4. Initializing Clustering

   ``` bash
   # without specifing control plane IP
   sudo kubeadm init --cri-socket /run/cri-dockerd.sock --pod-network-cidr=10.244.0.0/16 --ignore-preflight-errors=NumCPU

   # init with specific ip of control plane IP
   sudo kubeadm init --cri-socket /run/cri-dockerd.sock --control-plane-endpoint=192.168.56.110 --apiserver-advertise-address=192.168.56.110 --pod-network-cidr=10.244.0.0/16 --ignore-preflight-errors=NumCPU

   ``` 
   
    | Node                                          | IP Host Only   |
    | :---                                         |     :---       |
    | <code>--cri-socket</code>                    | use if have more than one container runtime to set runtime socket path |
    | <code>--control-plane-endpoint</code>        | set the shared endpoint for all control-plane nodes. Can be DNS/IP |
    | <code>--apiserver-advertise-address</code>   | set advertise address for this particular control-plane node's API server |
    | <code>--pod-network-cidr</code>              | set a Pod network add-on CIDR |

   In the log of <code>kubeadm init</code>, we see an instruction to add kube configuration in home directory. 
   ``` bash
   mkdir -p $HOME/.kube
   sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
   sudo chown $(id -u):$(id -g) $HOME/.kube/config
   ```
   
   Install flunnel as Kubernates network add on
   ``` bash
   curl -LO https://github.com/flannel-io/flannel/releases/latest/download/kube-flannel.yml
   vi kube-flannel.yml
   ```
   > Makesure IP in the <code>kube-flannel.yml</code> is the same as <code>--pod-network-cidr</code>
   
   ``` bash
   kubectl apply -f kube-flannel.yml
   ```

   Check the all pods running or not
   ``` bash
   kubectl get pods --all-namespaces
   ```
   
   <img src="https://github.com/neutrofoton/HiKubernetes/blob/main/images-k8s-vbox/k8s-vbox-all-pods-running.png" alt=""/>

   Finally we can check the master node is ready or not using the following command.
   ``` bash
   kubectl get nodes -o wide
   ```

# References
- https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/create-cluster-kubeadm/
- https://kubernetes.io/docs/concepts/cluster-administration/addons/
- https://docs.tigera.io/archive/v3.0/getting-started/kubernetes
- https://computingforgeeks.com/install-kubernetes-cluster-ubuntu-jammy/
- https://computingforgeeks.com/deploy-kubernetes-cluster-on-ubuntu-with-kubeadm/
- https://computingforgeeks.com/install-mirantis-cri-dockerd-as-docker-engine-shim-for-kubernetes/
- https://www.vladimircicovic.com/2022/08/kubernetes-setup-on-ubuntu-2204-lts-jammy-jellyfish
- https://www.letscloud.io/community/how-to-install-kubernetesk8s-and-docker-on-ubuntu-2004
- https://itnext.io/kubernetes-on-ubuntu-on-virtualbox-60e8ce7c85ed
- https://discuss.kubernetes.io/t/standard-k8s-installation-failed-centos7/20676
- https://ystatit.medium.com/deploy-kubernetes-with-specific-public-ip-address-for-control-plane-endpoint-cef1a54b2fbf
- https://stackoverflow.com/questions/44114854/virtualbox-cannot-register-the-hard-disk-already-exists
- https://stackoverflow.com/questions/70571312/port-6443-connection-refused-when-setting-up-kubernetes
- https://stackoverflow.com/questions/52609257/coredns-in-pending-state-in-kubernetes-cluster
- https://stackoverflow.com/questions/65995056/kubernetes-api-server-bind-address-vs-advertise-address