# **Air-gap**

## **Common Resources**

### **Shared Server list**

| Server | Hostname | Public IP | Private IP | Parameter |
| :--- | :--- | :--- | :--- | :--- |
| Bastion |bastion-0.k8s.ymlee.ibm	| 169.45.124.110	|10.160.22.167 | BASTION_0 |
| Bastion |bastion-win-0.ymlee.ibm	| 169.45.97.133	|10.168.40.27 | - |
| Nexus | nexus-0.k8s.ymlee.ibm | 169.45.97.147 | 10.160.22.160 | NEXUS_0 |
| Gitlab | gitlab-0.ymlee.ibm | 169.45.97.152 | 10.168.40.32 | GITLAB |
| Ansible | ansible-0.ymlee.ibm | - | 10.168.40.xx | ANSIBLE |
| Harbor | harbor-0.ymlee.ibm	| - | 10.168.40.xx | HARBOR |
| NFS | NFS-0.ymlee.ibm	| - |10.168.40.xx | NFS |

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


## **Echo**
  - ss

        export NEXUS_0=10.160.22.160
