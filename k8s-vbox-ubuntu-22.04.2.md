# 1. Configuring Virtual Box VMs
The configuration step of Virtual Box are as follow:

1. Install a VM <code>k8s</code> using Ubuntu Linux 22.04.2 LTS as a VM reference.
2. Copy disk of the VM as <code>k8s-master.vdi</code>, <code>k8s-worker1.vdi</code>, <code>k8s-worker2.vdi</code>.
3. Using Windows Command Prompt, change the hardisk UUID of each hdd on step <code>1.2</code> with the following command
   ```
   cd C:\Program Files\Oracle\VirtualBox
   C:\..\VirtualBox>VBoxManage.exe internalcommands sethduuid "D:\VMs\disk\k8s-master.vdi"

    UUID changed to: 91ebaec7-c113-4bb0-bc12-baa557f0574e
   ```
4. Do the step <code>1.3</code> for the  <code>k8s-worker1.vdi</code>, <code>k8s-worker2.vdi</code>.

5. Configure Network Adapter.
<img src="https://github.com/neutrofoton/HiKubernetes/blob/main/images-k8s-vbox/k8s-vbox-network-setup.png" alt=""/>
<br/><br/>
Host Only Adapter network configuration
<img src="https://github.com/neutrofoton/HiKubernetes/blob/main/images-k8s-vbox/k8s-vbox-network-host-only.png" alt=""/>
<br/><br/>
In the NAT Adapter network configuration, create NAT Network K8sCluster
<img src="https://github.com/neutrofoton/HiKubernetes/blob/main/images-k8s-vbox/k8s-vbox-network-nat.png" alt=""/>

6. Create three VMs <code>k8s-master</code>, <code>k8s-worker1</code>, <code>k8s-worker2</code> using the existing disks <code>k8s-master.vdi</code>, <code>k8s-worker1.vdi</code>, <code>k8s-worker2.vdi</code>. Makesure the network of NAT, Host Only Adapter, Bridge and NAT Network enabled. 

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
   Set the ip of host only adapter to: <br/>
   
    | Node        | IP Host Only   |Host Name               |
    | :---        |     :---:      |    :---               | 
    | master      | 192.168.56.110 |master.neutro.io        | 
    | worker1     | 192.168.56.111 |worker-node-1.neutro.io |
    | worker2     | 192.168.56.112 |worker-node-2.neutro.io |

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
   # host name for master
   sudo hostnamectl set-hostname master.neutro.io
   exec bash
   
   # host name for worker 1
   sudo hostnamectl set-hostname worker1.neutro.io
   exec bash
   
   # host name for worker 2
   sudo hostnamectl set-hostname worker2.neutro.io
   exec bash
   ```
# 2. Tool installation
   Update the apt package index and install packages that are needed.

   ``` bash
   sudo -i
   apt-get update && apt-get upgrade -y
   apt-get install -y vim git curl wget apt-transport-https gnupg gnupg2 software-properties-common ca-certificates lsb-release
   ```

> Repeat on all the other nodes.

# 3. Docker Installation
The Docker installation steps are as follow:

1. Install docker 
   ``` bash
   # sudo apt install docker.io 
   ``` 

   ``` bash
   # https://docs.docker.com/engine/install/ubuntu/

   # 1. Add Docker’s official GPG key
   sudo mkdir -m 0755 -p /etc/apt/keyrings
   curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg

   # 2. Set up the repository
   echo \
   "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu \
   $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

   # 3. Update package apt package index
   sudo apt-get update

   # NOTE: if receiving a GPG error when running apt-get update?
   sudo chmod a+r /etc/apt/keyrings/docker.gpg
   sudo apt-get update

   # 4. Install Docker Engine, containerd, and Docker Compose.
   sudo apt-get install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
   ```

2. Check the docker version
   ``` bash
   docker version
   ```
3. Enable and start docker
   ``` bash
   sudo systemctl enable docker
   # sudo systemctl start docker
   sudo systemctl daemon-reload
   sudo systemctl restart docker
   ```

   Verify the docker status
   ``` bash
   sudo systemctl status docker
   ```

> Repeat on all the other nodes.

# 4. Kubernates Installation
The Kubernates installation steps are described in the following steps:

1. Add <code>kubernates</code> repository 
   
   ``` bash
   sudo curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | apt-key add
   sudo apt-add-repository "deb http://apt.kubernetes.io/ kubernetes-xenial main"
   ```
2. Install <code>kubeadm, kubelet, kubectl</code>
   ``` bash
   sudo apt install -y kubeadm kubelet kubectl
   sudo apt-mark hold kubeadm kubelet kubectl
   ```

   Verify the installation
   ``` bash
   kubectl version --client && kubeadm version
   ```

3. Disable the swap memory

   Turn off swap
   ``` bash
   sudo sed -i '/ swap / s/^\(.*\)$/#\1/g' /etc/fstab
   ```

   Now disable Linux swap space permanently in <code>/etc/fstab</code>. Search for a swap line and add # (hashtag/comment) sign in front of the line.

   ``` bash
   sudo vim /etc/fstab

   # remark the following line
   #/swap.img	none	swap	sw	0	0
   ```

   Confirm setting is correct.
   ``` bash
   # swapoff disables swapping on the specified devices and files. 
   # -a flag is given, swapping is disabled on all known swap devices and files (as found in /proc/swaps or /etc/fstab)

   sudo swapoff -a
   sudo mount -a
   free -h
   ```
4. Enable kernel modules and configure sysctl.
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

> Repeat for each server node.

# 4. Kubernetes Cluster Configuration

For building Kubernetes cluster, in this lab we use <code>kubeadm</code>. The steps are:

1. Configure Container runtime (Docker CE runtime:)
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
2. Install Mirantis cri-dockerd as Docker Engine shim for Kubernetes
   For Docker Engine you need a shim interface. Mirantis cri-dockerd CRI socket file path is <code>/run/cri-dockerd.sock</code>. This is what will be used when configuring Kubernetes cluster.

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

   Validate successful installation
   ``` bash
   cri-dockerd --version
   ```

   Configure systemd units for cri-dockerd
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
   
   Ensure the service is running
   ``` bash
   systemctl status cri-docker.socket
   ```

3. Initialize master node

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

   > These are the basic kubeadm init options that are used to bootstrap cluster.<br/><br/>
   > <code>--control-plane-endpoint</code> :  set the shared endpoint for all control-plane nodes. Can be DNS/IP <br/>
   > <code>--pod-network-cidr</code> : Used to set a Pod network add-on CIDR<br/>
   > <code>--cri-socket</code> : Use if have more than one container runtime to set runtime socket path<br/>
   > <code>--apiserver-advertise-address</code> : Set advertise address for this particular control-plane node's API server<br/>
   
3. Initialize Clustering
   ``` bash
   sudo kubeadm init --pod-network-cidr=10.244.0.0/16 --cri-socket /run/cri-dockerd.sock --ignore-preflight-errors=NumCPU
   ``` 

   Add Kubernates configuration to 
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

   Check the cluster status
   ``` bash
   kubectl get pods --all-namespaces
   ```

# References
1. https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/create-cluster-kubeadm/
2. https://kubernetes.io/docs/concepts/cluster-administration/addons/
3. https://stackoverflow.com/questions/44114854/virtualbox-cannot-register-the-hard-disk-already-exists
4. https://computingforgeeks.com/install-kubernetes-cluster-ubuntu-jammy/
5. https://docs.tigera.io/archive/v3.0/getting-started/kubernetes
6. https://www.vladimircicovic.com/2022/08/kubernetes-setup-on-ubuntu-2204-lts-jammy-jellyfish
7. https://www.letscloud.io/community/how-to-install-kubernetesk8s-and-docker-on-ubuntu-2004
8. https://itnext.io/kubernetes-on-ubuntu-on-virtualbox-60e8ce7c85ed
9. https://stackoverflow.com/questions/70571312/port-6443-connection-refused-when-setting-up-kubernetes
