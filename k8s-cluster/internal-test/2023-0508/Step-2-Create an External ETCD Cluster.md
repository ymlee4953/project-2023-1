## **Sreate an External ETCD Cluster**

### **Prepartion**

1. Create Connection from Bastion Server 
- at bastion Server  
      
      ssh-copy-id -i ~/.ssh/id_rsa.pub root@$ETCD_1

    - Enter the VM's Password
    - Repeat to ETCD_2, ETCD_3

2. The First Login to ETCD_1

- SSH Login to the ETCD_1

      ssh $ETCD_1

    - Repeat to ETCD_2, ETCD_3

- Change the Password of ETCD_1

  - as root user  

        passwd

      enter new password (twice)

