## **Setup Nexus Server**

### **Prepartion**

1. Create Connection from Bastion Server 
- at bastion Server
      
      ssh-copy-id -i ~/.ssh/id_rsa.pub root@$NEXUS_0

    - Enter the VM's Password

2. The First Login to NEXUS_0

- SSH Login to the EXUS_0

      ssh $NEXUS_0

- Change the Password of NEXUS_0

  - as root user  

        passwd

      enter new password (twice)

### **Install Nexus**
1. Base Package Installation
   - **At the Nexus server**
  
          yum update -y
          yum -y install java-1.8.0-openjdk java-1.8.0-openjdk-devel


2. Nexus Software Download
    - **At the Nexus server**
    - Nexus Version 3.xx.x-xx
     
          cd /opt
          wget https://download.sonatype.com/nexus/3/latest-unix.tar.gz
          tar -xvzf latest-unix.tar.gz

          mv nexus-3.* nexus
          mv sonatype-work nexusdata
          
          useradd --system nexus
          
          chown -R nexus:nexus /opt/nexus
          chown -R nexus:nexus /opt/nexusdata



3. **Nexus Configuration Files**
      
    3.1 Moidfy file :  ***/opt/nexus/bin/nexus.vmoptions***  
    - **At the Nexus server**      
    - change value : sonatype-work → nexusdata
  
          mv /opt/nexus/bin/nexus.vmoptions /opt/nexus/bin/nexus.vmoptions.bak

          sed 's/sonatype-work/nexusdata/g' /opt/nexus/bin/nexus.vmoptions.bak > /opt/nexus/bin/nexus.vmoptions

          cat /opt/nexus/bin/nexus.vmoptions 

      
    3.2 Moidfy file : ***/opt/nexus/bin/nexus.rc***
    - **At the Nexus server**      
    - change value : Set run_as_user value as "nexus"

          mv /opt/nexus/bin/nexus.rc /opt/nexus/bin/nexus.rc.bak
          
          sed 's/#run_as_user=""/run_as_user="nexus"/g' /opt/nexus/bin/nexus.rc.bak > /opt/nexus/bin/nexus.rc

          cat /opt/nexus/bin/nexus.rc



    3.3 Moidfy file : ***/etc/security/limits.conf***
    - **At the Nexus server**      
    - add limit value to "/etc/security/limits.conf"

          cp /etc/security/limits.conf /etc/security/limits.conf.bak
          
          cat <<EOF >> /etc/security/limits.conf
          nexus - nofile 65536
          EOF

          cat /etc/security/limits.conf



4. Create "Nexus Service" file
  
    4.1 create file :  ***/etc/systemd/system/nexus.service***
    - **At the Nexus server**

          cat <<EOF > /etc/systemd/system/nexus.service 
          [Unit]
          Description=Nexus Service
          After=syslog.target network.target

          [Service]
          Type=forking
          LimitNOFILE=65536
          ExecStart=/opt/nexus/bin/nexus start
          ExecStop=/opt/nexus/bin/nexus stop
          User=nexus
          Group=nexus
          Restart=on-failure
          
          [Install]
          WantedBy=multi-user.target
          EOF

          cat /etc/systemd/system/nexus.service 

5. Start Nexus Service

    5.1 daemon-reload / Enable / Start Service
    - **At the Nexus server**

          systemctl daemon-reload
          systemctl enable nexus.service
          systemctl start nexus.service


    - Check 

          tail -f /opt/nexusdata/nexus3/log/nexus.log

         then
         
         if nexus service is successfully started, you can see the message like this, ... 
 
 
          -------------------------------------------------
          
          Started Sonatype Nexus OSS 3.xx.x-xx
          
          -------------------------------------------------

6. Initial Connection via browser

    6.1 get password
    - **at the nexus server**

          cat /opt/nexusdata/nexus3/admin.password

      then the initial password will show-up like 
 
          275bdee3-0c71-4b60-9947-eea20d946e1f
            
          # copy the initial password
          # you need to change the password at the first login 

    6.2 First log in via browser
    - **at the web browser**
    - Connect with port :8081
      
          Public : http://***.***.***.***:8081/
          Private : http://***.***.***.***:8081/

    6.3 Change the password in the First login
    - **at the browser**

          # user: admin
          # Password : paste copied the initial password and change to new password through Wizard


7. Repository Set up

    7.1 Yum Repository

    - yum (proxy)

      | Source | Repository Name | Remote Storage (URL) |
      | :--- | :--- | :--- |
      | CentOS – Base | yumpxy-centos-base | http://mirror.centos.org/centos/7/os/x86_64/ |
      | CentOS – Extra | yumpxy-centos-extra | http://mirror.centos.org/centos/7/extras/x86_64/ |
      | Docker-CE | yumpxy-docker-ce | https://download.docker.com/linux/centos/7/x86_64/stable |
      | K8s.gcr.io | yumpxy-k8s-gcr-io | https://packages.cloud.google.com/yum/repos/kubernetes-el7-x86_64 |

    - yum (hosted)
      | Repository Name | Repository Depth |
      | :--- | :--- |
      | yum-hosted | 1 |
      
    - yum (group)
      | Repository Name | Member Repositories |
      |  :--- | :--- |
      | yum-group | yumpxy-centos-base |
      | | yumpxy-centos-extra |
      | | yumpxy-docker-ce |
      | | yum-hosted |
      | | yumpxy-k8s-gcr-io |


    7.2 Docker Repository
    - docker (proxy)  
      | Source | Repository Name | Remote Storage (URL) | Extra Oprtions|
      | :--- | :--- | :--- | :--- |
      | Docker.io | dockerpxy-docker-io | https://registry-1.docker.io/ | Allow anonymous docker pull |
      | Gcr.io | dockerpxy-gcr-io | https://gcr.io/ | Allow anonymous docker pull |
      | Github.io | dockerpxy-ghcr-io | http://ghcr.io/ | Allow anonymous docker pull |
      | K8s.gcr.io | dockerpxy-k8s-gcr-io | https://k8s.gcr.io/ | Allow anonymous docker pull |
      | Quay.io | dockerpxy-quay-io | http://quay.io/ | Allow anonymous docker pull |

    - docker (hosted)
      | Repository Name | HTTP | Extra Oprtions|
      | :--- | :--- | :--- |
      | docker-hosted | 5000 | Allow anonymous docker pull |
      
      - 무료 Nexus 버전에서는 Docker Group 으로 Push 가 안되어 docker (hosted)로 직접 해야함
      
    - docker (group)
      | Repository Name | HTTP | Member Repositories | Extra Oprtions|
      |  :--- | :--- | :--- | :--- |
      | docker-group | 5001 | docker-hosted | Allow anonymous docker pull |
      | | | dockerpxy-docker-io |
      | | | dockerpxy-gcr-io |
      | | | dockerpxy-ghcr-io |
      | | | dockerpxy-k8s-gcr-io |
      | | | dockerpxy-quay-io|

    7.3 Pypi Repository
    - pypi (proxy)  

      | Source | Repository Name | Remote Storage (URL) |
      | :--- | :--- | :--- |
      | Python Package Index | pypi-proxy | https://pypi.org | 

    7.4 add Realms
    - Administration > Security > Realms       
      
          # Select Realms Menu
         
          # Move "Docker Bearer Token Realm" from Available to Active

8. Restart Nexus Service

    8.1 Restart Sevice to apply the change
    - **at the Nexus Server**

          systemctl restart nexus.service

--- 
# **Done**      