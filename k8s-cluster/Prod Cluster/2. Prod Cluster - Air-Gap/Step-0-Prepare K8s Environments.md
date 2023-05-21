# **Air-gap**

## **Common Resources**

### **Shared Server list**

| Server | Hostname | Public IP | Private IP | Parameter |
| :--- | :--- | :--- | :--- | :--- |
| Bastion |bastion-0.k8s.demo.ymlee	| 169.45.124.110	|10.160.22.167 | BASTION_0 |


---

### **K8s Cluster Server list**

| Server | Hostname | Public IP | Private IP | Parameter |
| :--- | :--- | :--- | :--- | :--- |
| ETCD |etcd-1.k8s.demo.ymlee	| -	|10.160.22.167 | ETCD_1 |
| ETCD |etcd-2.k8s.demo.ymlee	| -	|10.160.22.167 | ETCD_2 |
| ETCD |etcd-3.k8s.demo.ymlee	| -	|10.160.22.167 | ETCD_3 |
| Load Balancer |lb-1.k8s.demo.ymlee	|  169.45.124.110	|10.160.22.167 | LB_1 |
| Control Plane |master-1.k8s.demo.ymlee	|  -	|10.160.22.167 | MASTER_1 |
| Control Plane |master-2.k8s.demo.ymlee	|  -	|10.160.22.167 | MASTER_2 |
| Control Plane |master-3.k8s.demo.ymlee	|  -	|10.160.22.167 | MASTER_3 |
| Worker |worker-1.k8s.demo.ymlee	|  -	|10.160.22.167 | WORKER_1 |
| Worker |worker-2.k8s.demo.ymlee	|  -	|10.160.22.167 | WORKER_2 |
| Worker |worker-3.k8s.demo.ymlee	|  -	|10.160.22.167 | WORKER_3 |
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

        sudo cat <<EOF >> ~/.bashrc

        export NEXUS_0=10.xxx.xxx.xxx

        export ETCD_1=10.xxx.xxx.xxx
        export ETCD_2=10.xxx.xxx.xxx
        export ETCD_3=10.xxx.xxx.xxx

        export LB_1=110.xxx.xxx.xxx

        export MASTER_1=10.xxx.xxx.xxx
        export MASTER_2=10.xxx.xxx.xxx
        export MASTER_3=10.xxx.xxx.xxx

        export WORKER_1=10.xxx.xxx.xxx
        export WORKER_2=10.xxx.xxx.xxx
        export WORKER_3=10.xxx.xxx.xxx

        EOF

        cat ~/.bashrc

        source  ~/.bashrc

