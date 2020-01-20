![DevOps Project-04](https://github.com/anjon/DevOps-Project/blob/master/devops-project-04.jpg)


# DevOps Project 04
In this project, we will be see how to use Git, Jenkins, Ansible, DockerHub, Docker to DEPLOY on a docker container.

## PreRequisite
1. Jenkins Server. (https://github.com/anjon/Jenkins/tree/master/Jenkins)
2. Ansible Server. (https://github.com/anjon/Jenkins/tree/master/Ansible)
3. Docker Host.
4. DockerHub Account

### Part 01: Creating Docker Image & Push To DockerHub
In this part we are going to build a java war file by using the github source code. Then we are sending this source code to our ansible server. In the ansible server we are going to build a docker image using the war file and after the image is build push this image to the docker hub.

- Launch 2 EC2 instance one for ansible and another for docker host. 
- Install docker on both the ansible ans docker host. Create a user, add to sudoers, also to the docker group.
  ```sh
  # On both the ansible and docker server
  yum install docker
  useradd ansadmin
  echo 'ansadmin'| passwd --stdin ansadmin
  usermod -aG docker ansadmin
  echo 'ansadmin  ALL=(ALL)   NOPASSWD: ALL' > /etc/sudoers.d/ansadmin
  ```
- Add docker host in the ansible host file and configure passswordless authentication.
```sh
# On Both ansible & docker server
sed -ie 's/PasswordAuthentication no/PasswordAuthentication yes/' /etc/ssh/sshd_config
systemctl restart sshd

# On the ansible server
vim /etc/ansible/hosts
[docker]
DOCKER_PRI_IP

vim /eth/hosts
DOCKER_PRI_IP   docker

su - ansadmin
ssh-keygn
ssh-copy-id ansadmin@docker
mkdir /opt/playbooks
```

- Check the configuration form the ansible to docker host server.
```sh
ssh ansadmin@docker
ansible docker -m ping
```

- Add the ansible server in the jekins system configuration
  - `Manage Jenkins` --> `Configire systems` --> `Publish over SSH` section add `Ansible_Server. Click add and the values are
  - `Name=Ansible_Server`, `Hostname=<DOCKER_PRI_IP>`, `Username=ansadmin`. Click `Advanced`   
  - Select `Use password authentication, or use a different key` box and put the passsword for ansadmin.
  
- Install publish over ssh plugin.
  - `Manage Jenkins` --> `Manage Plugins` --> `Available` search for the "Publish Over ssh", select and install it.

- Connect to the docker hub from the ansible server cli. `docker login`. Provide the username and password.

Now We are ready to start our project. From the Jenkins Home create a new item name ***DevOps Project 04*** with the type as Maven. Then we are going to configure this project as below.
- *Source Code Management*
  - Repository URL: `https://github.com/anjon/Maven-Web-App.git`
  - Branch Specifier: `*/master`
- *Build*
  - Root POM: `pom.xml`
  - Goals and options: `clean install package`
- *Post Steps*  select `Send files or execute commands over SSH` from the `Add post-build step` drop down menu. 
  - NAME: `Ansible_Server`
  - Source files: `target/*.war`
  - Remove prefix: `target`
  - Remote directory: `//opt//docker`
- Add another `Send files or execute commands over SSH`
  - Name: `Ansible_Server`
  - Source files: `Dockerfile`
  - Remote directory: `//opt//docker`
  - Exec command: 
  ```sh
  cd /opt/docker
  docker build -t docker_demo .
  docker tag docker_demo anjon/docker_demo
  docker push anjon/docker_demo
  docker rmi docker_demo anjon/docker_demo
  ```
At this stage we can test the part 01 status of our project. From the ***DevOps Project 04*** home tab click `Build Now`
Check if the image is build local ansible server. Also login to the docker hub to check for the pushed image we have build in this part. Should be successful. 

### Part 02: Deploy Docker Image
