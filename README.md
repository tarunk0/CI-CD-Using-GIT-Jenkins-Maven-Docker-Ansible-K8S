# CI-CD-Using-GIT-Jenkins-Maven-Docker-Ansible-K8S
Part-3 CI/CD Using Git, Jenkins, Maven, Docker, Ansible and Kubernetes!


![Screenshot 2022-08-25 at 4 08 55 PM](https://user-images.githubusercontent.com/92631457/186748574-dc55add2-64ee-4d95-88df-480e2e2b27bf.png)

![Screenshot 2022-08-25 at 4 08 37 PM](https://user-images.githubusercontent.com/92631457/186748609-76230aac-3ba8-45d1-8672-24a1a6f655a0.png)

## STEP: 1 
   ### Creating User on Jenkins and Ansible Instances:
   - Create 2 Instances, for this tutorial I have created them in GCP.
   - Jenkins Instance Name: jenkins
   - Ansible Instance Name: ansible-control-node
   - Generate SSH Keys and Create User "ansadmin"
      - Generate keys using putty key generator or similar tool. 
      - Key comments: ansadmin
      - Save private and public key. 
      - Copy Public Key to VM(Edit mode) in SSH Key block
      
      ![image](https://user-images.githubusercontent.com/92631457/186561759-4ff58ab9-0e62-4ae6-8e7c-1e6b8aef983b.png)

      - Copy Public Key to VM(Edit mode) in SSH Key block  
   - Connect to instances and set password for user "ansadmin".
```
   passwd ansadmin
```
   - Give sudo permission to user "ansadmin"
```
   vi sudo
   ansadmin ALL=(ALL)  NOPASSWD:ALL
```
   
![image](https://user-images.githubusercontent.com/92631457/186561531-43672b9d-9e0e-4d6c-8e9c-fa338146b363.png)

## STEP: 2
   ### Java Installation on Jenkins and Ansible instances, as It is a java based project:
```
  apt update
  apt install openjdk-8-jdk -y
  # Or you can run 
  apt install openjdk-8-jre-headless
  # check java version
  java -version
```
   ### Enable the Password Authentication on all three instances:(By default it is set to No)
   
``` 
   vi /etc/ssh/sshd_config
   PasswordAuthentication yes #by default it is set to no
   service sshd reload #to refresh the changes
```
![Screenshot 2022-08-25 at 8 13 56 AM](https://user-images.githubusercontent.com/92631457/186562504-92021aec-c3cb-481c-8635-c269c3251139.png)


 ## STEP: 3
  ### Maven Installation and configuration on Jenkins Instance:
     
```
   apt update
   cd /opt
   wget https://dlcdn.apache.org/maven/maven-3/3.8.6/binaries/apache-maven-3.8.6-bin.tar.gz 
   #replace with latest version from  this link: https://maven.apache.org/download.cgi

   tar -xvzf apache-maven-3.8.6-bin.tar.gz
   mv apache-maven-3.8.6 maven

   #set maven path of gcp ubuntu server for jenkins
   vi /etc/profile
   M2_HOME=/opt/maven
   M2=$M2_HOME/bin
   PATH=$PATH:$JAVA_HOME:$M2_HOME:$M2:$HOME/bin
   export PATH
```
![image](https://user-images.githubusercontent.com/92631457/186562736-3e9e885c-0ba2-43b7-b75c-41117a47e947.png)

```
   #for refreshing
   source /etc/profile
   echo $PATH
   echo $M2
  
```
![image](https://user-images.githubusercontent.com/92631457/186752282-4722293c-bc2f-4e87-a056-41fe7a9ce255.png)

   ### Jenkins Installation and configuration on Jenkins Instance:

```
  # This is the Debian package repository of Jenkins to automate installation and upgrade. To use this repository, first add the key to your system:
    curl -fsSL https://pkg.jenkins.io/debian-stable/jenkins.io.key |      sudo tee \
    /usr/share/keyrings/jenkins-keyring.asc > /dev/null
    
    #Then add a Jenkins apt repository entry:
    echo deb [signed-by=/usr/share/keyrings/jenkins-keyring.asc] \
    https://pkg.jenkins.io/debian-stable binary/ | sudo tee \
    /etc/apt/sources.list.d/jenkins.list > /dev/null
    
    #Update your local package index, then finally install Jenkins:
    sudo apt-get update
    sudo apt-get install fontconfig openjdk-11-jre
    sudo apt-get install jenkins
    
    # Give sudo permission to the jenkins user
    vi /etc/sudoers
   
    #add the following line beside root
    jenkins ALL=(ALL) NOPASSWD: ALL
```
![image](https://user-images.githubusercontent.com/92631457/186561374-c478fb09-e8d9-4b7c-9f65-c41c8e491c8a.png)
      
```
    #restart jenkins
    systemctl restart jenkins
   
    #Test Jenkins
    http://<jenkins-public-ip.:8080
```
![image](https://user-images.githubusercontent.com/92631457/186753002-8208945b-7ada-4c0d-b168-3c926f0f90f2.png)

![image](https://user-images.githubusercontent.com/92631457/186753403-34e5355a-cf5b-47aa-9c8a-f5c6a17456ca.png)

![image](https://user-images.githubusercontent.com/92631457/186753808-1438fc7c-7188-4b6d-96a2-c37d8318fd60.png)

 
   ### Jenkins Plugins installation and configuration of Jenkins:
  
   - Plugins to be installed. Default Plugins installation is not required as we need some specific plugins.
     - Github
     - Maven Invoker
     - Maven Integration
     - Publish over SSH. 
 
![image](https://user-images.githubusercontent.com/92631457/186754410-803502ad-f14e-4b1e-ad69-4a7f3d9e257b.png)

  ## STEP: 4
   ### Ansible Installation and configuration on Ansible Named Instance:   
   
``` 
   #Ansible installation
   apt install software-properties-common
   apt-add-repository --yes --update ppa:ansible/ansible
   apt install ansible -y
   ansible --version
```
   
```
   #add the targets groups in the ansible hosts file
   vi /etc/ansible/hosts
   [all_hosts]
   localhost
```
![image](https://user-images.githubusercontent.com/92631457/186755466-e277ddf0-f820-49ca-8a4b-f8cabd23f3c9.png)

  - Install docker and start docker services on ansible instance for building images on this instance:
  
  ```sh
     apt install docker.io -y
     docker --version
     service docker start
     service docker status
  ```
  ![image](https://user-images.githubusercontent.com/92631457/186757822-c77c8f1d-92df-448c-9c01-31969baf4060.png)

  
```sh  
   #create a directory in /opt in Ansible server using user 'ansadmin'
   sudo su - ansadmin
   
   #Add a user to docker group to manage docker
   usermod -aG docker ansadmin
   
   #Create a directory under /opt in Ansible server user "ansadmin" to store the files which will be transferred to this:
   cd /opt
   sudo mkdir kubernetes
   sudo chown -R ansadmin:ansadmin /opt/kubernetes
   ls -l /opt
```
![image](https://user-images.githubusercontent.com/92631457/186759170-4e4a9ede-a2fb-4570-a4a4-000d860595e3.png)

## STEP: 5
   ### Create a Password less connection between Ansible to ansible localhost:
   
   - Log into Ansible Instance using user "ansadmin" and generate ssh key:
```
    sudo su - ansadmin
    ssh-keygen
```
   - Copy Keys to Local Ansible Instance
``` 
    ssh-copy-id localhost
```
![image](https://user-images.githubusercontent.com/92631457/186760741-5b93057e-4128-47b9-8017-84eff28c940d.png)


   - Validation Test
``` 
   ansible all -m ping
``` 
![Screenshot 2022-08-26 at 1 49 44 AM](https://user-images.githubusercontent.com/92631457/186760902-110f851a-70b5-4cac-90f3-c27fe7e810a2.png)

    ### Create a Password less connection between Jenkins and Kubernetes Management Server to copy the playbooks to the k8s management server:

    - Enable Password Authentication on Kubernetes-Management-server node using root user:
    
 ```sh
    sudo vi /etc/ssh/sshd_config
    #make the changes like
    PasswordAuthentication yes
    PermitRootLogin yes
    
    #reload the service
    sudo service sshd reload
```
 
    - Set password for admin user on K8s Master node using admin user - Reset the k8s management server's server
 ``` sh
     sudo passwd root
```
   - Add the SSH server on jenkins by doing the following:
     - Manage Jenkins --> Configuration System --> SSH Servers
     - ADD
     - Name: k8s-server
     - Hostname: Public Ip of k8s management server
     - Username: root
     - Advances--> Select Password Authentication
     - Password: admin
     - Click on Test Connection. 

![image](https://user-images.githubusercontent.com/92631457/186763967-6745de5e-77aa-404d-bf25-e0c2ac2262e1.png)

![image](https://user-images.githubusercontent.com/92631457/186764249-4ea1c34b-9eaf-448c-b6fe-343202142ef0.png)


   ### Create a Password less connection between Ansible and K8s Management server by doing the following for Running the playbooks kept on K8s Management server:
   
   - Login to Ansible Server and copy public key onto K8s-management-server(This is another way if ssh-copy-id doesnot work)
     - Ansible User -- ansadmin & K8s User -- root
     
 ```sh
    sudo su - ansadmin
    cd .ssh
    ls
    cat id_rsa.pub
 ```
 ![image](https://user-images.githubusercontent.com/92631457/186766492-2ab7288b-baf8-4113-8c64-231dd0573efa.png)

   - Copy public key "id_rsa.pub" from ansible Instance and paste to "authorised_keys" on K8s-amangement_server
     
    
 ```sh
    cd ~/.ssh
    ls
    vi authorized_keys
    #paste the key here
 ```
 ![image](https://user-images.githubusercontent.com/92631457/186766870-407d3905-8987-4374-b210-f65222258151.png)

    - Test pasword less connection from ansible server to k8s management server
        - Login to K8s management server using root. 
    ```sh
       ssh -i ~/.ssh/id_rsa root@<k8s-management-server-public-IP>
    ```
 ![image](https://user-images.githubusercontent.com/92631457/186767364-d845abdf-a579-429c-a60b-4a65599c283e.png)

   ## STEP: 6
   ### Setup connection between Jenkins and Ansible and also set the MAVEN path
   
   - Manage Jenkins --> Configuration System --> SSH Servers
     - ADD
     - Name: ansible
     - Hostname: Public Ip of ansible
     - Username: ansadmin
     - Advances--> Select Password Authentication
     - Password: admin
     - Click on Test Connection. 
    
 ![Screenshot 2022-08-26 at 2 31 04 AM](https://user-images.githubusercontent.com/92631457/186768210-a7b80e6e-90b8-47e6-8796-ec11e5e26edd.png)



   - Mangage Jenkins - Global Tool configuration - Maven
     - Name: maven
     - MAVEN_HOME: /opt/maven/
     
     ![image](https://user-images.githubusercontent.com/92631457/186560358-477f03a2-554c-4a98-86cf-a641e115b264.png)
     
     ### Github Repository url:
    
   - Repository Url:
     - Your github url -- https://github.com/tarunk0/CI-CD-Using-GIT-Jenkins-Maven-Docker-Ansible-K8S.git
     - Branch: /main
     
     ![image](https://user-images.githubusercontent.com/92631457/186559828-49eb9f29-2558-40ee-8894-c527aa25c4fd.png)

   
   - Change hosts file in github and add k8s-management-server public ip
   
   ![image](https://user-images.githubusercontent.com/92631457/186790125-24fb1748-b233-48b2-81ac-e784e8bcb84b.png)

   - Change hosts file in github and k8s-management-server public ip:
   - Login with Docker-Hub Credentials on ansible istance using "root user"
     - docker login
     - Username: tarunk0
     - Password: *********
     
     ![image](https://user-images.githubusercontent.com/92631457/186560259-ba5a0e11-63f4-4436-a401-413b726e723a.png)

## STEP: 7
   ### Create a Jenkins Job -- CI Job(Maven Project)
   
   - Project name: deploy_on_k8s_ci
   - Github Url: https://github.com/tarunk0/CI-CD-Using-GIT-Jenkins-Maven-Docker-Ansible-K8S.git
 
   ![image](https://user-images.githubusercontent.com/92631457/186775991-d9ba55c7-ef0b-41a2-8e2a-392fca5e542a.png)

   - Build --> Root POM --> pom.xml
     - Goals and actions: clean install package
   - Post build actions:
     - Add post build action --> Send build artifacts over SSH.
     - SSH Server
       - Name: ansible-server
       - Source file: Kubernetes/Dockerfile, Kubernetes/hosts, Kubernetes/simple-devops-image.yml, Kubernetes/kubernetes-tarun-deployment-GCP.yml, Kubernetes/kubernetes-tarun-service-GCP.yml (Jenkins instance path: /var/lib/jenkins/workspace/Deploy_on_Kuberenetes_CI)
       - Remove prefix: Kubernetes
       - Remove Directory: //opt/kubernetes
       
     ![image](https://user-images.githubusercontent.com/92631457/186776133-2825be20-bde0-46a3-be3b-3826f2cf7543.png)

     - SSH Server
       - Name: ansible-server
       - Source file: webapp/target/*.war
       - Remove prefix: webapp/target
       - Remove Directory: //opt/kubernetes
       - Exec Command:
     ```sh
        ansible-playbook -i /opt/kubernetes/hosts /opt/kubernetes/create-simple-devops-image.yml
        ```
      ![image](https://user-images.githubusercontent.com/92631457/186776245-f585dda3-d56c-44b5-9320-cc3b7f5b140b.png)

     
   ### Create a Jenkins Job -- CD Job(Freestyle Project)

 - Project name: deploy_on_k8s_cd
      - Github Url: https://github.com/tarunk0/CI-CD-Using-GIT-Jenkins-Maven-Docker-Ansible-K8S.git
      
       ![image](https://user-images.githubusercontent.com/92631457/186775335-3913e6b0-45ac-42bc-a5cc-af7eab4b1ccd.png)

      - Post build actions:
      - Add post build action --> Send build artifacts over SSH.
       ![image](https://user-images.githubusercontent.com/92631457/186775495-678533d1-f074-47eb-b986-f6a8dd5e389a.png)

      - SSH Server
           - Name: k8s-server
           - Source file: Kubernetes/tarun-deploy.yml, Kubernetes/tarun-service.yml (Jenkins instance path: /var/lib/jenkins/workspace/Deploy_on_Kuberenetes_CI)
           - Remove prefix: Kubernetes
           - Remove Directory: //root
         - SSH Server
           - Name: ansible-server
           - Exec Command:
         ```sh
            ansible-playbook -i /opt/kubernetes/hosts /opt/kubernetes/kubernetes-tarun-deployment-GCP.yml;
            ansible-playbook -i /opt/kubernetes/hosts /opt/kubernetes/kubernetes-tarun-service-GCP.yml;
         ```
    ![image](https://user-images.githubusercontent.com/92631457/186775820-e7e3e092-700c-4a43-ab93-bcf3bb5d3ae1.png) 

   ## STEP: 8
      ### Link -- CI and CD jobs on Jenkins to make it a CI/CD Pipeline:
      
      - Add Post build action --> Build Other Projects
      - Projects to build -- deploy_on_k8s_CD
      - Select --> Trigger only if build is stable

 
      ![Screenshot 2022-08-26 at 5 40 15 AM](https://user-images.githubusercontent.com/92631457/186790303-80cfa501-2ff7-4696-88c2-1491ed5affeb.png)
        
      - Test the URL:
        
          - http://k8s-loadbalancer-public-ip>:8080/webapp
          - http://k8s-Management-server-pub-ip>:8080/webapp
          - http://k8s-node-1-public-ip>:8080/webapp
          - http://k8s-node-2-public-ip>:8080/webapp


  ![Screenshot 2022-08-26 at 5 27 50 AM](https://user-images.githubusercontent.com/92631457/186790455-c403594f-8214-4112-b774-8e5cf4b4d0b6.png)

  ![Screenshot 2022-08-26 at 5 28 21 AM](https://user-images.githubusercontent.com/92631457/186790502-527f486a-3555-4dc1-9e5c-ea2d8442d72d.png)
   
   - You can see the changes getting reflected in the above screenshot as soon as the commit was pushed on to the github. This completes our CI/CD Pipeline using the gcp cluster created by KOPS :)
   - Thank you :))
    
    
    
    
    
    
    
