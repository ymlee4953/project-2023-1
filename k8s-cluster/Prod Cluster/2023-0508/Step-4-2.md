## **Create a Kubernetes Cluster**

### **Prepartion**



- step-01 : Prepare CRI Registry

      sudo yum install -y yum-utils

- step-01 : Prepare CRI Registry

      sudo yum-config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo

- step-02 : Prepare CRI Registry 

      cat <<EOF | sudo tee /etc/modules-load.d/containerd.conf
      overlay
      br_netfilter
      EOF

- step-02 : Prepare CRI Registry 

      sudo modprobe overlay
      sudo modprobe br_netfilter

- step-02 : Prepare CRI Registry 

      cat <<EOF | sudo tee /etc/sysctl.d/99-kubernetes-cri.conf
      net.bridge.bridge-nf-call-iptables  = 1
      net.ipv4.ip_forward                 = 1
      net.bridge.bridge-nf-call-ip6tables = 1
      EOF

- step-02 : Prepare CRI Registry 

      sudo sysctl --system

- step-02 : Prepare CRI Registry 

      sudo yum install -y containerd.io

- step-03 : Prepare CRI Registry 

      sudo mkdir -p /etc/containerd
      sudo containerd config default | tee /etc/containerd/config.toml

      sudo vi /etc/containerd/config.toml

- step-03 : Prepare CRI Registry 

      sudo systemctl restart containerd

      sudo systemctl enable containerd

      sudo systemctl status containerd    

- ddd

      cat <<EOF | sudo tee /etc/yum.repos.d/kubernetes.repo
      [kubernetes]
      name=Kubernetes
      baseurl=https://packages.cloud.google.com/yum/repos/kubernetes-el7-\$basearch
      enabled=1
      gpgcheck=1
      gpgkey=https://packages.cloud.google.com/yum/doc/yum-key.gpg https://packages.cloud.google.com/yum/doc/rpm-package-key.gpg
      exclude=kubelet kubeadm kubectl
      EOF


- ddd

      cat <<EOF | sudo tee /etc/modules-load.d/k8s.conf
      br_netfilter
      EOF

      cat <<EOF | sudo tee /etc/sysctl.d/k8s.conf
      net.bridge.bridge-nf-call-ip6tables = 1
      net.bridge.bridge-nf-call-iptables = 1
      EOF

      sudo sysctl --system

- ddd

      # permissive 모드로 SELinux 설정(효과적으로 비활성화)
      sudo setenforce 0
      sudo sed -i 's/^SELINUX=enforcing$/SELINUX=permissive/' /etc/selinux/config

      sudo swapoff -a
      sudo sed -i '/ swap / s/^\(.*\)$/#\1/g' /etc/fstab

- ddd

      sudo yum install -y kubelet kubeadm kubectl --disableexcludes=kubernetes

      sudo systemctl enable --now kubelet

- ddd

      sudo vi /usr/lib/systemd/system/kubelet.service.d/10-kubeadm.conf

- ddd

      Environment="KUBELET_CGROUP_ARGS=--cgroup-driver=systemd"

      $KUBELET_CGROUP_ARGS

- eee

      sudo systemctl daemon-reload
      sudo systemctl restart kubelet
      sudo systemctl enable kubelet
      sudo systemctl status kubelet

- eee

      sudo cat <<EOF> kubeadm-config.yaml
      apiVersion: kubeadm.k8s.io/v1beta3
      kind: InitConfiguration
      nodeRegistration:
      criSocket: "/run/containerd/containerd.sock"
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

- eee

      sudo kubeadm init --config kubeadm-config.yaml

- eee

      mkdir -p $HOME/.kube
      sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
      sudo chown $(id -u):$(id -g) $HOME/.kube/config

- fff

      curl https://raw.githubusercontent.com/projectcalico/calico/v3.25.1/manifests/calico.yaml -O
      
- fff      

      kubectl apply -f calico.yaml