## **Create a Load Balancer for Master nodes**

### **Prepartion**

1. Create Connection from Bastion Server 
- at bastion Server  
      
      ssh-copy-id -i ~/.ssh/id_rsa.pub root@$LB_1

    - Enter the VM's Password

2. The First Login to LB_1

- SSH Login to the LB_1

      ssh $LB_1

- Change the Password of LB_1

  - as root user  

        passwd

      enter new password (twice)

3. Declare Parameters

- at the LB-1

  - Parameters

        sudo cat <<EOF >> ~/.bashrc

        export LB_1=110.xxx.xxx.xxx

        export MASTER_1=10.xxx.xxx.xxx
        export MASTER_2=10.xxx.xxx.xxx
        export MASTER_3=10.xxx.xxx.xxx

        EOF

        cat ~/.bashrc

        source  ~/.bashrc

### **Setup HA Proxy**

1. Base Package Installation

    - Install Base Package for HA Proxy
    - At The Bastion Server

          ssh $LB_1   

    - At HA Proxy Server

          yum install -y gcc pcre-static pcre-devel
          yum install -y wget
          yum install -y make
          yum install -y openssl-devel

2. Download and Copy the Installation File
    - At HA Proxy Server

          wget https://www.haproxy.org/download/1.7/src/haproxy-1.7.8.tar.gz


3. Install HA Proxy
    - At HA Proxy Server

          mv haproxy-1.7.8.tar.gz /opt
          cd /opt
          tar xzvf haproxy-1.7.8.tar.gz

          cd /opt/haproxy-1.7.8
          make TARGET=linux2628 USE_OPENSSL=1
          make install

          mkdir -p /etc/haproxy
          mkdir -p /var/lib/haproxy 
          touch /var/lib/haproxy/stats
          ln -s /usr/local/sbin/haproxy /usr/sbin/haproxy

          cp /opt/haproxy-1.7.8/examples/haproxy.init /etc/init.d/haproxy
          chmod 755 /etc/init.d/haproxy
            
          systemctl daemon-reload
          systemctl enable haproxy

4. Add user
    - **At the HA Proxy Server**
           
          useradd -r haproxy


5. Make Load Balancer configurations
    - **At the HA Proxy Server**

          cd /etc/haproxy

    - Create Common Configuration File

          cat <<EOF > /etc/haproxy/haproxy.cfg 
          global
                log /dev/log local0
                log /dev/log local1 notice
                chroot /var/lib/haproxy
                stats timeout 30s
                user haproxy
                group haproxy
                daemon
            
          defaults
                maxconn 20000
                mode    tcp
                option  dontlognull
                timeout http-request 10s
                timeout queue        1m
                timeout connect      10s
                timeout client       86400s
                timeout server       86400s
                timeout tunnel       86400s

          #---------------------------------------------------------------------
          # For a High Available kubenetes master node cluster 
          #---------------------------------------------------------------------

          frontend kubernetes
              bind ${LB_1}:6443
              mode tcp
              option tcplog
              default_backend kubernetes-backend

          backend kubernetes-backend
              mode tcp
              option tcp-check
              balance roundrobin
              server master-1 ${MASTER_1}:6443 check fall 3 rise 2
              server master-2 ${MASTER_2}:6443 check fall 3 rise 2
              server master-3 ${MASTER_3}:6443 check fall 3 rise 2

          frontend ksconsole
              bind *:30880
              mode tcp
              option tcplog
              default_backend ksconsole-backend

          backend ksconsole-backend
              mode tcp
              option tcp-check
              balance roundrobin
              server master-1 ${MASTER_1}:30880 check fall 3 rise 2
              server master-2 ${MASTER_2}:30880 check fall 3 rise 2
              server master-3 ${MASTER_3}:30880 check fall 3 rise 2          

          EOF

          cat /etc/haproxy/haproxy.cfg 


6. System restart
    - **At the HA Proxy Server**
            
          systemctl daemon-reload
          systemctl restart haproxy
          systemctl status haproxy


  
---
# **Done**