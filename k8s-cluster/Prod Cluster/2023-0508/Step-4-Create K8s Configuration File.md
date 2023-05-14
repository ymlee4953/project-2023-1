## **Create a Kubernetes Cluster**

### **Prepartion**

1. Create Connection from Bastion Server 
- at bastion Server  
      
      ssh-copy-id -i ~/.ssh/id_rsa.pub root@$MASTER_1

    - Enter the VM's Password
    - Repeat to MASTER_2, MASTER_3

2. The First Login to MASTER_1

- SSH Login to the MASTER_1

      ssh $MASTER_1

    - Repeat to MASTE_2, MASTER_3

- Change the Password of MASTER_1

  - as root user  

        passwd

      enter new password (twice)

3. Declare Parameters
- for Master nodes

      export NEXUS_0=10.160.22.160

      export ETCD_1=10.160.22.185
      export ETCD_2=10.160.22.158
      export ETCD_3=10.160.22.138

      export LB_1=10.160.22.162

      export MASTER_1=10.160.22.168
      export MASTER_2=10.160.22.164
      export MASTER_3=10.160.22.171

      export WORKER_1=10.160.22.188
      export WORKER_2=10.160.22.143
      export WORKER_3=10.160.22.166

---

- ccc 

      yum install -y yum-utils
      yum-config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo

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

      # permissive 모드로 SELinux 설정(효과적으로 비활성화)
      sudo setenforce 0
      sudo sed -i 's/^SELINUX=enforcing$/SELINUX=permissive/' /etc/selinux/config

      sudo yum install -y kubelet kubeadm kubectl --disableexcludes=kubernetes

      sudo systemctl enable --now kubelet


- eee

      cat <<EOF> kubeadm-config.yaml
      apiVersion: kubeadm.k8s.io/v1beta3
      kind: InitConfiguration
      nodeRegistration:
      criSocket: "/run/containerd/containerd.sock"
      ---
      apiVersion: kubeadm.k8s.io/v1beta3
      kind: ClusterConfiguration
      clusterName: 2.dev.k8s.ymlee
      kubernetesVersion: "v1.27.1"
      networking:
      podSubnet: 192.168.0.0/16
      serviceSubnet: 20.96.0.0/16
      apiServer:
      certSANs:
      - "${LB_1}"
      controlPlaneEndpoint: "${LB_1}:6443"
      etcd:
      external:
      endpoints:
      - https://${ETCD_1}:2379
      - https://${ETCD_2}:2379
      - https://${ETCD_3}:2379
      caFile: /etc/kubernetes/pki/etcd/ca.crt
      certFile: /etc/kubernetes/pki/apiserver-etcd-client.crt
      keyFile: /etc/kubernetes/pki/apiserver-etcd-client.key
      ---
      apiVersion: kubelet.config.k8s.io/v1beta1
      kind: KubeletConfiguration
      cgroupDriver: systemd
      EOF
