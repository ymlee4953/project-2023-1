# **Air-gap**

## **Common Resources**

### **Shared Server list**

| Server | Hostname | Public IP | Private IP | Parameter |
| :--- | :--- | :--- | :--- | :--- |
| Bastion |bastion-0.k8s.demo.ymlee	| 169.45.124.110	|10.160.22.167 | BASTION_0 |
| Bastion |bastion-win-0.k8s.demo.ymlee	| 169.45.97.133	|10.168.40.27 | - |
| Nexus | nexus-0.k8s.demo.ymlee | 169.45.97.147 | 10.160.22.160 | NEXUS_0 |
| Gitlab | gitlab-0.k8s.demo.ymlee | 169.45.97.152 | 10.168.40.32 | GITLAB |
| Ansible | ansible-0.k8s.demo.ymlee | - | 10.168.40.xx | ANSIBLE |
| Harbor | harbor-0.k8s.demo.ymlee	| - | 10.168.40.xx | HARBOR |
| NFS | NFS-0.k8s.demo.ymlee	| - |10.168.40.xx | NFS |

---

### **K8s Cluster Server list**

| Server | Hostname | Public IP | Private IP | Parameter |
| :--- | :--- | :--- | :--- | :--- |
| ETCD |etcd-1.k8s.demo.ymlee	| -	|10.160.22.167 | ETCD_1 |
| ETCD |etcd-2.k8s.demo.ymlee	| -	|10.160.22.167 | ETCD_2 |
| ETCD |etcd-3.k8s.demo.ymlee	| -	|10.160.22.167 | ETCD_3 |
| ETCD |lb-1.k8s.demo.ymlee	| -	|10.160.22.167 | LB_1 |
| ETCD |master-1.k8s.demo.ymlee	| -	|10.160.22.167 | MASTER_1 |
| ETCD |master-2.k8s.demo.ymlee	| -	|10.160.22.167 | MASTER_1_2 |
| ETCD |master-3.k8s.demo.ymlee	| -	|10.160.22.167 | MASTER_1_3 |
---


## **Setup Bastion**

1. Log in into Bastino Server
2. Change Password
  - as root user  

        passwd

        # enter new password (twice)

3. Generate ssh-key to login to other VM's
  - Generate the ssh-key
        
        ssh-keygen

  - Check 

        cat /root/.ssh/id_rsa.pub


## **Echo Parameters**
  - Parameters

        export NEXUS_0=10.160.22.158

        export ETCD_1=10.160.22.185
        export ETCD_2=10.160.22.162
        export ETCD_3=10.160.22.138

        export LB_1=10.160.22.171

        export MASTER_1=10.160.22.188
        export MASTER_2=10.160.22.166
        export MASTER_3=10.160.22.143

        export WORKER_1=10.160.22.172
        export WORKER_2=10.160.22.174
        export WORKER_3=10.160.22.173
        
  - Nexus URL
            
        http://169.45.124.105:8081



  - Tutorial

        cat <<EOF | sudo tee /etc/yum.repos.d/kubernetes.repo
        [kubernetes]
        name=Kubernetes
        baseurl=https://packages.cloud.google.com/yum/repos/kubernetes-el7-\$basearch
        enabled=1
        gpgcheck=1
        gpgkey=https://packages.cloud.google.com/yum/doc/rpm-package-key.gpg
        exclude=kubelet kubeadm kubectl
        EOF


  - Set SELinux in permissive mode (effectively disabling it)

        sudo setenforce 0
        sudo sed -i 's/^SELINUX=enforcing$/SELINUX=permissive/' /etc/selinux/config

        sudo yum install -y kubelet kubeadm kubectl --disableexcludes=kubernetes

        sudo systemctl enable --now kubelet