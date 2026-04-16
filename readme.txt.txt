Steps for jenkins server (Amazon Linux 2023 )
Note:
Create Swap 2Gb min
Create partition for /tmp and mount it ( appro. 5 GB )
=============================================================================================
Step1: set hostname
$sudo hostnamectl	  set-hostname	jenkins.devops.com
$sudo timedatectl   set-timezone   Asia/Kolkata
$sudo  su
-------------------------------------------------------------------------------------
Step2: Install Java
#sudo dnf install java-21-amazon-corretto -y
#java   -version
-------------------------------------------------------------------------------------
Step3: Create repo
#sudo wget -O /etc/yum.repos.d/jenkins.repo \
    https://pkg.jenkins.io/redhat-stable/jenkins.repo
#sudo rpm --import https://pkg.jenkins.io/redhat-stable/jenkins.io-2023.key
-------------------------------------------------------------------------------------
Step4: Install jenkins
#sudo dnf update  -y
#sudo dnf install jenkins -y
#sudo systemctl daemon-reload
-------------------------------------------------------------------------------------
Step5: start and enable jenkins
#sudo systemctl enable jenkins
#sudo systemctl start jenkins
#sudo systemctl status jenkins
-------------------------------------------------------------------------------------
Step6: Verify using browser
http://your_amazon_linux_instance_public_ip:8080
-------------------------------------------------------------------------------------
Step7: check default password of jenkins
#sudo cat /var/lib/jenkins/secrets/initialAdminPassword

install Jenkins with suggested plugins and set username password
username = admin
password = 111
-------------------------------------------------------------------------------------
Step8: Install git
#dnf   install   git   -y
-------------------------------------------------------------------------------------
Step9: Download maven

#cd /opt
#wget https://dlcdn.apache.org/maven/maven-3/3.9.14/binaries/apache-maven-3.9.14-bin.tar.gz
#tar -xvzf apache-maven-3.9.14-bin.tar.gz
#ls
#rm   -rvf    *.gz  (optional is not set properly)
#ls
-------------------------------------------------------------------------------------
Step10: Set maven and java path
#cd
#vim  .bash_profile
MAVEN_HOME=/opt/apache-maven-3.9.14/
M2=/opt/apache-maven-3.9.14/bin/
JAVA_HOME=/usr/lib/jvm/java-21-amazon-corretto.x86_64/
PATH=$PATH:$JAVA_HOME:$MAVEN_HOME:$M2:$HOME/bin
:wq

#source .bash_profile
#echo $PATH

Step11: Create directory for custom workspace
#mkdir  /registration
#chown jenkins:jenkins  /registration
-------------------------------------------------------------------------------------
Step12: Open jenkins -   Dashboard – manage jenkins - plugins - available plugins 
-  Maven Integration
-  Publish Over SSH
-  GitHub plugin

Restart Jenkins
--------------------------------------------------------------------------------------
Step13: Open jenkins - Dashboard – Manage Jenkins – Tools –

JDK Installations - Add JDK - Name ( java21 ) - JAVA_HOME ( /usr/lib/jvm/java-21-amazon-corretto.x86_64/ )

Git Installations 
 git – Name (default) – path to git execute ( /bin/git )

Maven Installations Add Maven – Name (mvn) – MAVEN_HOME( /opt/apache-maven-3.9.14/ )– Apply – Save.

--------------------------------------------------------------------------------------
Step14: create project for "pull code"

Dashboard – New Item – Enter new item name (1.pullmycode) – freestyle project – ok – General - Advance -Use  Custom workspace ( /registration ) - source code management – git – repository url ( https://github.com/sharayuSP66/registration.git ) – Apply – Save.
--------------------------------------------------------------------------------------
Step15: create project for "build code"

Dashboard – New Item – Enter new item name (2.buildmycode) – freestyle project – ok – General - Advance -Use  Custom workspace ( /registration ) - Build Steps (invoke top-level maven targets ) – Maven version (mvn) – Goals ( clean install package ) – Apply – Save.

Dashboard run ( Build ) both project.
=============================================================================
Note-
Sample app path ( main branch )
https://github.com/sharayuSP66/regpro.git
you can clone code from my git repository and upload your git  repository
==================================================================================================
Steps for ansible server (Amazon Linux 2023 )
==================================================================================================
Step14. Set hostname
$sudo hostnamectl 	set-hostname    ansible.devops.com
$sudo timedatectl  set-timezone  Asia/Kolkata
$sudo su
$cd
-------------------------------------------------------------------------------------
Step15. Install ansible
#yum install ansible -y
#ansible  --version
-------------------------------------------------------------------------------------
Step16. Create user and add sudo privilege
#useradd  itadmin
#passwd itadmin
111
111
-------------------------------------------------------------------------------------
Step17. Provide sudo privilege to run sudo without password

#vim  /etc/sudoers
itadmin	ALL=(ALL)	NOPASSWD:  ALL
:wq!
-------------------------------------------------------------------------------------
Step18. Update ssh configuration file

#vim	/etc/ssh/sshd_config
PermitRootLogin	yes		[ line number 40 ]
PasswordAuthentication yes		[ line number 65 ]
:wq

#systemctl  restart   sshd
-------------------------------------------------------------------------------------
Step19.  Create directory for ansible project

#mkdir   /opt/Docker
#chown 	itadmin:itadmin	/opt/Docker
#su  itadmin
$cd /opt/Docker
-------------------------------------------------------------------------------------
Step20. Create inventory file

$vim   inventory
[docker]
docker_server
:wq
-------------------------------------------------------------------------------------
Step21. Create ansible.cfg file

$vim  ansible.cfg
[defaults]
inventory=/opt/Docker/inventory
remote_user=itadmin
host_key_checking=false
[privilege_escalation]
become=true
become_user=root
become_method=sudo
become_ask_pass=false
:wq
-------------------------------------------------------------------------------------
Step22. Manage jenkins and ansible server connectivity

Dashboard – manage Jenkins – System – SSH Servers (add ) – Name (Ansible_Server) Hostname (192.168.1.112) – username (itadmin) – Advanced - use password authentication or use a different key (111) – Test Configuration – Apply - Save.
-------------------------------------------------------------------------------------
Step23. Transfer git project data to ansible server using jenkins

Dashboard – New Item – Enter new item name (3. transfercodetoansible) – freestyle project – ok – General - Advance -Use  Custom workspace ( /registration ) - Now click on “Post Build Action” – send build artifacts over SSH – Name (Ansible_Server) – Source files (webapp/target/*.war) – Remove prefix (webapp/target/) – Remote directory (//opt/Docker)
Add Transfer Set
Source files (Dockerfile) – Remote directory (//opt/Docker) - Apply - Save.

now build project and open terminal of ansible server verify Dockerfile and webapp.war file
$ls	/opt/Docker/
==================================================================================================
Steps for Docker server (Amazon Linux 2023 )
==================================================================================================
Step24.  Set hostname
$sudo hostnamectl	set-hostname	docker.devops.com
$sudo timedatectl  set-timezone  Asia/Kolkata
$sudo su

Step25. Create user and add sudo privilege
#useradd  itadmin
#passwd itadmin
111
111
-------------------------------------------------------------------------------------
Step26. Provide sudo privilege to run sudo without password

#vim  /etc/sudoers
itadmin	ALL=(ALL)	NOPASSWD:  ALL
:wq!
-------------------------------------------------------------------------------------
Step27. Update ssh configuration file

#vim	/etc/ssh/sshd_config
PermitRootLogin	yes		[ line number 40 ]
PasswordAuthentication yes		[ line number 65 ]
:wq

#systemctl  restart   sshd
-------------------------------------------------------------------------------------
Step 28. Login into dockerhub 
https://hub.docker.com/

create repository with name "registration"

then open docker server terminal and docker login ( via root user )
#docker  login
username: sharayuSP66
password: xyz
=============================================================================================================
Ansible Server
=============================================================================================================
Step 29. Update host file on ansible server and update docker server private ip address
$sudo vim    /etc/hosts
172.xx.20.35  docker_server
:wq
-------------------------------------------------------------------------------------
Step30. Generate ssh key and transfer to docker server
#su  itadmin
$ssh-keygen

$ssh-copy-id   itadmin@docker_server
-------------------------------------------------------------------------------------
Step31. Create ansible playbook for create docker image and upload created image on docker hub
$cd /opt/Docker
$vim register.yml
---
 - name: build image and upload on dockerhub
   hosts: docker
   ignore_errors: yes
   tasks:
     - name: install docker
       yum:
         name: docker
         state: present
     - name: start and enable docker
       service:
         name: docker
         state: started
         enabled: yes
     - name: add user into group
       command: usermod -aG docker itadmin
     - name: Create directory for project
       file:
         path: /opt/Docker
         state: directory
     - name: transfer Dockerfile
       copy:
         src: Dockerfile
         dest: /opt/Docker/
     - name: transfer webapp war file
       copy:
         src: webapp.war
         dest: /opt/Docker/
     - name: change ownership
       command: chown -R itadmin:itadmin /opt/Docker
     - name: transfer password token file to docker server for dockerlogin purpose
       copy:
         src: pass.txt
         dest: /opt/Docker/
     - name: login into dockerhub
       shell: cd /opt/Docker; cat pass.txt | docker login -u sharayusp66 --password-stdin
     - name: build docker image
       shell: cd /opt/Docker; docker build -t registration:latest .
     - name: create tag to push image into dockerhub
       shell: docker tag registration:latest  sharayusp66/registration:latest
     - name: push docker image into dockerhub
       command: docker push  sharayusp66/registration:latest

:wq
---------------------------------------------------------------------------------------------------------------------------------------------
Step32: go to hub.docker.com and create token 

create pass.txt file and store token
#vim   /opt/Docker/pass.txt
( docker token )
:wq
=======================================================================================
Step33. run playbook using jenkins

Dashboard – New Item – Enter new item name (4. pushcontainertodockerhub) – freestyle project – ok –  Now click on “Post Build Action” – send build artifacts over SSH – Name (Ansible_Server) – Exec command ( cd /opt/Docker; ansible-playbook register.yml )
Save - Apply

Build project

after run project goto dockerhub "registration" repository and verify image successfully uploaded or not.
======================================================================================
Create DockerSwarm Cluster
======================================================================================
34. Create 3 EC2 Instance
1. Manager
2. worker1
3. worker2

install docker on all cluster nodes ( use following bootstrap )
#!/bin/bash
sudo yum update -y
sudo yum install docker -y
sudo systemctl start docker
sudo systemctl enable docker
sudo useradd itadmin
echo 111 | passwd --stdin itadmin
echo "itadmin ALL=(ALL)       NOPASSWD: ALL" >> /etc/sudoers
sudo gpasswd -a itadmin docker
echo 111 | passwd --stdin root
sed 's/PasswordAuthentication no/PasswordAuthentication yes/' -i /etc/ssh/sshd_config
echo PermitRootLogin yes >> /etc/ssh/sshd_config
systemctl restart sshd
docker swarm join --token xxxxxxxxxxxx 172.31.22.127:2377
------------------------------------------------------------------------------------------------------
35. Open terminal on manager
$sudo  hostnamectl   set-hostname  manager
$sudo  su

update host file
#vim  /etc/hosts
172.31.xx.xxx manager
172.31.xx.xx worker1
172.31.xx.xxx worker2
:wq
------------------------------------------------------------------------------------------------------
36. For create swarm on manager
#docker swarm init

copy generate token and execute on all worker node ( docker swarm join --token SWMTKN-1-3r8j2xxxxxxxxa0l5hkxxxxegxxxxxxxxxxxvlftsb8ut-xxxxxxxxxxxxxxzblgzplqis7htpv 172.31.35.254:2377 )

now verify on manager
#docker  node  ls

------------------------------------------------------------------------------------------------------
37. add worker into swarm
docker swarm join --token SWMTKN-1-1qeeapdfts1sx2xxxxxxxx4sb18mcri8zxxxxxxxxxxwe8k6st-xxxxxxxxxxxxxteg7 172.31.34.220:2377 


open terminal on worker1
$sudo  hostnamectl   set-hostname worker1
$sudo  su
#<paste here docker swarm join token>

open terminal on worker2
$sudo  hostnamectl   set-hostname worker2
$sudo  su
#<paste here docker swarm join token>
------------------------------------------------------------------------------------------------------
38. Update on Ansible hosts,inventory and Playbook
$sudo vim  /etc/hosts
172.31.34.8 manager
:wq

$vim  /opt/Docker/inventory
[docker]
docker_server
[swarm]
manager
[swarm:vars]
ansible_port=22
ansible_user=root
ansible_password=111
:wq

----------------------------------------------------------------------------------------------------
39. create playbook for deploy container image on docker swam cluster

$vim  /opt/Docker/deploycode.yml 

---
 - name: deploy project container image on docker swarm cluster
   hosts: swarm
   ignore_errors: yes
   tasks:
     - name: remove existing running service
       shell: docker service rm registration
     - name: create new service
       shell: docker service create --replicas=7  -d -p 80:8080 --name registration sharayuSP66/registration:latest
:wq
------------------------------------------------------------------------------------------------------
Step40. create project deploycode

Dashboard – New Item – Enter new item name (5. deploymycode) – freestyle project – ok –  Now click on “Post Build Action” – send build artifacts over SSH – Name (Ansible_Server) – Exec command ( cd /opt/Docker; ansible-playbook deploycode.yml )
Save - Apply
------------------------------------------------------------------------------------------------------
Step41. Goto Jenkins Dashboard and run project  and verify

For verify
open browser
http://<manager_public_ip>/webapp
=====================================================================================
======================================================================================
