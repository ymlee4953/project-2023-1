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
| Control Plane |master-1.k8s.demo.ymlee	|  169.45.124.110	|10.160.22.167 | MASTER_1 |
| Control Plane |master-2.k8s.demo.ymlee	|  169.45.124.110	|10.160.22.167 | MASTER_2 |
| Control Plane |master-3.k8s.demo.ymlee	|  169.45.124.110	|10.160.22.167 | MASTER_3 |
| Worker |worker-1.k8s.demo.ymlee	|  169.45.124.110	|10.160.22.167 | WORKER_1 |
| Worker |worker-2.k8s.demo.ymlee	|  169.45.124.110	|10.160.22.167 | WORKER_2 |
| Worker |worker-3.k8s.demo.ymlee	|  169.45.124.110	|10.160.22.167 | WORKER_3 |
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
