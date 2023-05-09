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


