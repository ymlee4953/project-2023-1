## **Create an External ETCD Cluster**

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

---

## **Create External etcd Cluster with etcdadm**

---
## **For External etcd**

1. Download Software Package for etcdadm
    
    1.1 install Git and make
    - yum install and make
    - at Bastion Server

          yum install -y make git

    1.2 Install Go
    - Download File

          wget https://dl.google.com/go/go1.20.4.linux-amd64.tar.gz
          tar -C /usr/local -xzf ./go1.20.4.linux-amd64.tar.gz

    - add Go Path

          vi ~/.bash_profile

    - edit Go Path
    
          ...
          PATH=$PATH:$HOME/bin:/usr/local/go/bin
          ...
    
    - apply profile 

          source ~/.bash_profile

    1.3. Download etcdadm source file and make
    - Air-gap 환경 설치임으로 Bastion 을 통해 etcdadm 소스 및 etcd 설치 파일 다운로드 후 etcd 서버로 전송

          git clone https://github.com/kubernetes-sigs/etcdadm.git

          cd etcdadm
          make etcdadm 

    1.4 Download etcd source file

    - Check the default etcd version of etcdadm
    
          cat constants/constants.go | grep DefaultVersion

    - the result : the default etcd version of etcdadm is "3.5.7"

          ==> DefaultVersion = "3.5.7"

    - Download etcd installation file

          wget https://github.com/coreos/etcd/releases/download/v3.5.7/etcd-v3.5.7-linux-amd64.tar.gz
          
    
    1.5 Copy the etcd source files for Ansible

    - Copy files
    
          mkdir -p ~/files/BASTION-0/etcdadm/

          cp ~/etcdadm/etcdadm ~/files/BASTION-0/etcdadm/
          cp ~/etcdadm/etcd-v3.5.7-linux-amd64.tar.gz ~/files/BASTION-0/etcdadm/


---

2. Copy files for etcd to each etcd servers 
      
    - **Prerequisite**
      - The executable "etcdadm" file was built and ready in the bastion server
             
    2.1. Copy etcd files with sftp connection
    
    - At the bastion/ansible Server

          sftp $ETCD_1    // for all ETCD_1,2,3

    - after connection, copy etcd files and close

          put /root/etcdadm/etcdadm
          put /root/etcdadm/etcd-v3.5.7-linux-amd64.tar.gz

          exit

    2.2. Move etcd files with ssh connection
    
    - At the bastion/ansible Server

          ssh $ETCD_1    // for all ETCD_1,2,3

    - after connection, create the directory and move etcd files

          mkdir -p /var/cache/etcdadm/etcd/v3.5.7/
          mv ./etcd-v3.5.7-linux-amd64.tar.gz /var/cache/etcdadm/etcd/v3.5.7/

          ls -l /var/cache/etcdadm/etcd/v3.5.7/

          exit


3. Initialize a etcd cluster

    3.1. Initialize a etcd cluster at ETCD-1

    - At the bastion/ansible Server

          ssh $ETCD_1    // for ETCD_1 only

    - after connection, initialize a etcd cluster 

          ./etcdadm init

    - after creation, check the etcd process

          systemctl status etcd

    - after creation, check those cert files and prepare to copy
    
          ls -l /etc/etcd/pki

          exit
         

4. Copy cert files to etcd member servers 

    4.1. Get those cert files from ETCD-1

    - At the bastion/ansible Server

          mkdir -p ~/files/ETCD-1/etc/etcd/pki

          sftp $ETCD_1    // for ETCD_1 only

    - after connection, copy cert files

          get /etc/etcd/pki/*

          exit

          ls -l

    - Copy cert files to etcd member servers

          sftp $ETCD_2  // for ETCD_2 and 3

          put ca.*

          exit

    - after copy, move the files to the right location 
    
          ssh $ETCD_2  // for ETCD_2 and 3

          mkdir -p /etc/etcd/pki/
          mv ca.* /etc/etcd/pki/

          ls -l /etc/etcd/pki/

    - Join to the etcd cluster one-by-one 
    - at ETCD-2, 3

          ./etcdadm join http://10.xxx.xxx.xxx:2379    // one-by-one
          
          exit

    - Check the ETCD Cluster is running
    
          ssh $ETCD_1    // for all ETCD_1,2,3

          /opt/bin/etcdctl.sh member list -w table

          exit

---
