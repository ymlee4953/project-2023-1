## **Create a Kubernetes Cluster**

### **Prepartion**

1. Install CRI

- 1.1 Prepare CRI Registry

  - 1.1.1 Install yum utilities 

        sudo yum install -y yum-utils

  - 1.1.2 : Set the Repository 

        sudo yum-config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo

  - 1.1.3 :  

        cat <<EOF | sudo tee /etc/modules-load.d/containerd.conf
        overlay
        br_netfilter
        EOF

  - 1.1.4 :  

        sudo modprobe overlay
        sudo modprobe br_netfilter

  - 1.1.5 :  

        cat <<EOF | sudo tee /etc/sysctl.d/99-kubernetes-cri.conf
        net.bridge.bridge-nf-call-iptables  = 1
        net.ipv4.ip_forward                 = 1
        net.bridge.bridge-nf-call-ip6tables = 1
        EOF

  - 1.1.6 :  

        sudo sysctl --system

- 1.2 Install CRI

  - 1.2.1 :

        sudo yum install -y containerd.io

  - 1.2.2 :
  
        sudo mkdir -p /etc/containerd
        sudo containerd config default | sudo tee /etc/containerd/config.toml

        sudo vi /etc/containerd/config.toml


    - Cgroup setting in the "/etc/containerd/config.toml" File
      > Cgroup Drvier setting : Systemd 
          
          ...
          [plugins."io.containerd.grpc.v1.cri".containerd.runtimes.runc.options]
            SystemdCgroup = true
          ...  


  - 1.2.3 :
  
        sudo systemctl restart containerd
  
        sudo systemctl enable containerd

        sudo systemctl status containerd    

2. Install Kubernetes files

- 2.1 

  - 2.1.1 :

        cat <<EOF | sudo tee /etc/yum.repos.d/kubernetes.repo
        [kubernetes]
        name=Kubernetes
        baseurl=https://packages.cloud.google.com/yum/repos/kubernetes-el7-\$basearch
        enabled=1
        gpgcheck=1
        gpgkey=https://packages.cloud.google.com/yum/doc/yum-key.gpg https://packages.cloud.google.com/yum/doc/rpm-package-key.gpg
        exclude=kubelet kubeadm kubectl
        EOF

  - 2.1.2 :

        cat <<EOF | sudo tee /etc/modules-load.d/k8s.conf
        br_netfilter
        EOF

        cat <<EOF | sudo tee /etc/sysctl.d/k8s.conf
        net.bridge.bridge-nf-call-ip6tables = 1
        net.bridge.bridge-nf-call-iptables = 1
        EOF

        sudo sysctl --system

  - 2.1.3 : permissive 모드로 SELinux 설정(효과적으로 비활성화)
 
        sudo setenforce 0
        sudo sed -i 's/^SELINUX=enforcing$/SELINUX=permissive/' /etc/selinux/config

        sudo swapoff -a
        sudo sed -i '/ swap / s/^\(.*\)$/#\1/g' /etc/fstab

- 2.2 

  - 2.2.1 : 

        sudo yum install -y kubelet kubeadm kubectl --disableexcludes=kubernetes

        sudo systemctl enable --now kubelet


  - 2.2.2 :

        sudo systemctl daemon-reload
        sudo systemctl restart kubelet
        sudo systemctl enable kubelet
        sudo systemctl status kubelet -l

3. Create Kubenetes Cluster

- 3.1 Initiate Kubernate Cluster

  - 3.1.1 : Create kubeadm-config.yaml file

        sudo cat <<EOF> kubeadm-config.yaml
        apiVersion: kubeadm.k8s.io/v1beta3
        kind: InitConfiguration
        nodeRegistration:
          criSocket: "unix:///var/run/containerd/containerd.sock"
        ---
        apiVersion: kubeadm.k8s.io/v1beta3
        kind: ClusterConfiguration
        clusterName: test.k8s.ymlee
        kubernetesVersion: "v1.27.1"
        networking:
          podSubnet: 192.168.0.0/16
          serviceSubnet: 20.96.0.0/16
        ---
        apiVersion: kubelet.config.k8s.io/v1beta1
        kind: KubeletConfiguration
        cgroupDriver: "systemd"
        EOF

  - 3.1.2 :

        sudo kubeadm init --config kubeadm-config.yaml

  - 3.1.3 :

        mkdir -p $HOME/.kube
        sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
        sudo chown $(id -u):$(id -g) $HOME/.kube/config

- 3.2 apply CNI

  - 3.2.1 :

        curl https://docs.projectcalico.org/archive/v3.24/manifests/calico.yaml -O



  - 3.2.2 :      

        kubectl apply -f calico.yaml

- 3.3 Join Worker nodes

  - 3.3.1 :

