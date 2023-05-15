## **Create a Kubernetes Cluster**

### **Prepartion**

1. Create Connection from Bastion Server to each Kubernetes nodes
- 1.1   
  - at bastion Server
      
        ssh-copy-id -i ~/.ssh/id_rsa.pub root@$MASTER_1

  - Enter the VM's Password
 
        xxxxxxx
    - Repeat to MASTER_2, MASTER_3
    - Repeat to WORKER_1, WORKER_2, WORKER_3

2. The First Login to each Kubernetes nodes

- SSH Login to the each Kubernetes nodes

      ssh $MASTER_1

    - Repeat to MASTE_2, MASTER_3
    - Repeat to WORKER_1, WORKER_2, WORKER_3

- Change the Password 

  - as root user  

        passwd

  - enter new password (twice)

        xxxxxx
        xxxxxx

3. Declare Parameters
- at each Kubernetes nodes

      export NEXUS_0=10.xxx.xxx.xxx

      export ETCD_1=10.xxx.xxx.xxx
      export ETCD_2=10.xxx.xxx.xxx
      export ETCD_3=10.xxx.xxx.xxx

      export LB_1=10.xxx.xxx.xxx

      export MASTER_1=10.xxx.xxx.xxx
      export MASTER_2=10.xxx.xxx.xxx
      export MASTER_3=10.xxx.xxx.xxx

      export WORKER_1=10.160.22.188
      export WORKER_2=10.160.22.143
      export WORKER_3=10.160.22.166

---
### **Create Cluster**


1. Setup Repo

- 1.1 Download gpg-key

  - at Bastion-0

        wget https://packages.cloud.google.com/yum/doc/yum-key.gpg
        wget https://packages.cloud.google.com/yum/doc/rpm-package-key.gpg

        curl https://docs.projectcalico.org/archive/v3.24/manifests/calico.yaml -O


- 1.2 send gpg files to each Kubernetes nodes
  - for every Kubernetes nodes

        sftp $MASTER_1
        
        put *.gpg
        
        exit

  - for every Kubernetes nodes     
        
        ssh $MASTER_1

        mv *.gpg /etc/pki/rpm-gpg/

2. Install CRI

- 2.1 Prepare CRI Registry

  - at each Kubernetes nodes 


  - 2.1.2 : Set the Repository 

        cat <<EOF > /etc/yum.repos.d/nexus.repo

        [nexus]
        name=Nexus Proxy
        baseurl=http://${NEXUS_0}:8081/repository/yum-group/
        enabled=1
        gpgcheck=0

        EOF

        cat /etc/yum.repos.d/nexus.repo

        # sudo yum-config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo

  - 2.1.3 :  

        cat <<EOF | sudo tee /etc/modules-load.d/containerd.conf
        overlay
        br_netfilter
        EOF

  - 2.1.4 :  

        sudo modprobe overlay
        sudo modprobe br_netfilter

  - 2.1.5 :  

        cat <<EOF | sudo tee /etc/sysctl.d/99-kubernetes-cri.conf
        net.bridge.bridge-nf-call-iptables  = 1
        net.ipv4.ip_forward                 = 1
        net.bridge.bridge-nf-call-ip6tables = 1
        EOF

  - 2.1.6 :  

        sudo sysctl --system

- 1.2 Install CRI

  - at each Kubernetes nodes

  - 1.2.1 :

        sudo yum install -y containerd.io

  - 1.2.2 :
  
        sudo mkdir -p /etc/containerd
        sudo containerd config default | sudo tee /etc/containerd/config.toml

        sudo vi /etc/containerd/config.toml


  - 1.2.3 :

        cat <<EOF> temptext_for_configtoml.txt
                [plugins."io.containerd.grpc.v1.cri".registry.mirrors."docker.io"]
                  endpoint = ["http://${NEXUS_0}:5001"]
                [plugins."io.containerd.grpc.v1.cri".registry.mirrors."k8s.gcr.io"]
                  endpoint = ["http://${NEXUS_0}:5001"]
                [plugins."io.containerd.grpc.v1.cri".registry.mirrors."registry.k8s.io"]
                  endpoint = ["http://${NEXUS_0}:5001"]
                [plugins."io.containerd.grpc.v1.cri".registry.mirrors."quay.io"]
                  endpoint = ["http://${NEXUS_0}:5001"]
                [plugins."io.containerd.grpc.v1.cri".registry.mirrors."${NEXUS_0}:5001"]
                  endpoint = ["http://${NEXUS_0}:5001"]    
        EOF

        cat temptext_for_configtoml.txt

  - 1.2.3 :
  
        sudo systemctl restart containerd
  
        sudo systemctl enable containerd

        sudo systemctl status containerd    

2. Install Kubernetes files

- 2.1 

  - at each Kubernetes nodes

  - 2.1.1 :

        cat <<EOF | sudo tee /etc/yum.repos.d/kubernetes.repo
        [kubernetes]
        name=Kubernetes
        baseurl=http://${NEXUS_0}:8081/repository/yumpxy-kubernetes/kubernetes-el7-\$basearch/
        enabled=1
        gpgcheck=1
        gpgkey=file:///etc/pki/rpm-gpg/yum-key.gpg file:///etc/pki/rpm-gpg/rpm-package-key.gpg
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

  - at each Kubernetes nodes

  - 2.2.1 : 

        sudo yum install -y kubelet kubeadm kubectl --disableexcludes=kubernetes

        sudo systemctl enable --now kubelet


  - 2.2.2 :

        sudo systemctl daemon-reload
        sudo systemctl restart kubelet
        sudo systemctl enable kubelet
        sudo systemctl status kubelet -l

3. Create Kubenetes Cluster

- 3.1 **Setup Kubernetes Master node 1**
    - to prepare initialize K8s cluster
    - **Prerequisite**
      - The external etcd cluster was built and ready 
      - All cert files of etcd copied in the bastion server under "~/files/ETCD-1/etc/etcd/pki" directory
      - The Load Balancer for Control Plane was ready
      - Check the IP address of LB_1

  - 3.1.1 : **get the etcd certs**
    - Copy the etcd cert from Bastion to MASTER_1 nodes
    - at the Bastion Server 

          cd ~/files/ETCD-1/etc/etcd/pki
          
          sftp $MASTER_1 

          put ca.*
          put apiserver-etcd-client.*
     
          exit

  - 3.1.2 : Move the etcd cert to right location in the MASTER_1 nodes
    - at the MASTER_1 nodes

          ssh $MASTER_1 

          mkdir -p /etc/kubernetes/pki/etcd
          mv ./ca.* /etc/kubernetes/pki/etcd
          mv ./apiserver-etcd-client.* /etc/kubernetes/pki

          ls -l /etc/kubernetes/pki

    - **Do Not Exit from MASTER_1 nodes**

  - 3.1.3 **Check the Load Balancer for the control plane is ready**
    - at the MASTER_1 nodes
    - If the MASTER_1 is restarted or reconnected, the Environment Parameter should be export again before execute 

          
          # nc -v LOAD_BALANCER_IP PORT

          yum install nc

          nc -v ${LB_1} 6443

          Ctrl-C


- 3.2 Initiate Kubernate Cluster

  - 3.2.1 : Create kubeadm-config.yaml file

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
        cgroupDriver: "systemd"
        EOF

  - 3.2.2 :

        sudo kubeadm init --config kubeadm-config.yaml --upload-certs

  - 3.2.3 :

        mkdir -p $HOME/.kube
        sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
        sudo chown $(id -u):$(id -g) $HOME/.kube/config

- 3.3 apply CNI

  - 3.3.1 :

  - at bastion-0
        
        cd

        sftp $MASTER_1

        put calico.yaml

        exit

  - 3.3.2 :      

        ssh $MASTER_1

        kubectl apply -f calico.yaml


4. Finalize Control plane with joining Kubernetes Master node 2, 3

    4.1 **Get the Certs from Master-1 for other Master members**
    - Copy the certs from MASTER_1 nodes to Bastion
    - at the Bastion Server 

          mkdir -p ~/files/MASTER-1/etc/kubernetes/pki/etcd/
          cd ~/files/MASTER-1/etc/kubernetes/pki/

          sftp $MASTER_1

          get /etc/kubernetes/pki/apiserver-etcd-client.crt /root/files/MASTER-1/etc/kubernetes/pki/
          get /etc/kubernetes/pki/apiserver-etcd-client.key /root/files/MASTER-1/etc/kubernetes/pki/
          get /etc/kubernetes/pki/ca.crt /root/files/MASTER-1/etc/kubernetes/pki/
          get /etc/kubernetes/pki/ca.key /root/files/MASTER-1/etc/kubernetes/pki/
          get /etc/kubernetes/pki/sa.key /root/files/MASTER-1/etc/kubernetes/pki/
          get /etc/kubernetes/pki/sa.pub /root/files/MASTER-1/etc/kubernetes/pki/
          get /etc/kubernetes/pki/front-proxy-ca.crt /root/files/MASTER-1/etc/kubernetes/pki/
          get /etc/kubernetes/pki/front-proxy-ca.key /root/files/MASTER-1/etc/kubernetes/pki/
          get /etc/kubernetes/pki/etcd/ca.crt /root/files/MASTER-1/etc/kubernetes/pki/etcd/
          get /etc/kubernetes/pki/etcd/ca.key /root/files/MASTER-1/etc/kubernetes/pki/etcd/
          get /etc/kubernetes/admin.conf /root/files/MASTER-1/etc/kubernetes/

          exit

    4.2 **Copy the Certs of Master-1 to other Master members**
    - Copy the certs of MASTER_1 to MASTER_2 and 3
    - at the Bastion Server 

          ssh $MASTER_2       // and MASTER_3
          
          mkdir -p /etc/kubernetes/pki/etcd/

          exit 

          sftp $MASTER_2       // and MASTER_3

          put /root/files/MASTER-1/etc/kubernetes/pki/apiserver-etcd-client.crt /etc/kubernetes/pki/
          put /root/files/MASTER-1/etc/kubernetes/pki/apiserver-etcd-client.key /etc/kubernetes/pki/
          put /root/files/MASTER-1/etc/kubernetes/pki/ca.crt /etc/kubernetes/pki/
          put /root/files/MASTER-1/etc/kubernetes/pki/ca.key /etc/kubernetes/pki/
          put /root/files/MASTER-1/etc/kubernetes/pki/sa.key /etc/kubernetes/pki/
          put /root/files/MASTER-1/etc/kubernetes/pki/sa.pub /etc/kubernetes/pki/
          put /root/files/MASTER-1/etc/kubernetes/pki/front-proxy-ca.crt /etc/kubernetes/pki/
          put /root/files/MASTER-1/etc/kubernetes/pki/front-proxy-ca.key /etc/kubernetes/pki/
          put /root/files/MASTER-1/etc/kubernetes/pki/etcd/ca.crt /etc/kubernetes/pki/etcd/
          put /root/files/MASTER-1/etc/kubernetes/pki/etcd/ca.key /etc/kubernetes/pki/etcd/
          put /root/files/MASTER-1/etc/kubernetes/admin.conf /etc/kubernetes/

          exit

    4.3 **Join the member nodes of control plane with copied token key**
    - at the Bastion Server 

          ssh $MASTER_2       // and MASTER_3
          
          kubeadm join 10.168.40.37:6443 --token v4hs9a.hstnvaq93w73pvo9 \
          --discovery-token-ca-cert-hash sha256:b6af0746091b649fd69ec64a4a2b9082f140e201304f6c7fdf9b2710e41a523e \
          --control-plane --certificate-key 14c77e611f4f15eec09194cafaa0994b5c0281a76bef0be4b09ea268afc4e3b8

    4.4 **Make Kubernetes access environment for master-2,3**
    - At the master-2,3 Server

          mkdir -p $HOME/.kube
          sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
          sudo chown $(id -u):$(id -g) $HOME/.kube/config

    4.5 **Verify the Kubernetes Cluster is Running**
    - At the master-1 Server

          kubectl get nodes
          kubectl get pods -o wide --all-namespaces



5. Join Worker nodes to the Kubernetes Cluster

    5.1 **Join the work nodes with copied token key**
    - at the Bastion Server 

          ssh $WORKER_1       // and WORKER_2,3
          
          kubeadm join 10.168.40.37:6443 --token v4hs9a.hstnvaq93w73pvo9 \
          --discovery-token-ca-cert-hash sha256:b6af0746091b649fd69ec64a4a2b9082f140e201304f6c7fdf9b2710e41a523e 

          exit

    5.2 **Verify the Kubernetes Cluster is Running**
    - At the master-1 Server

          kubectl get nodes
          kubectl get pods -o wide --all-namespaces


