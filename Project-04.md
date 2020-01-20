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
In this part we are going to deploy and run a container to the docker server from the ansible host. For this we need to write a ansible playbook.
- Write an ansible playbook for deploy container to the docker host.
```yaml
mkdir /opt/playbooks && cd /opt/playbooks
chown -R ansadmin.ansadmin /opt/playbooks
vim create_docker_container.yml
---
- hosts: docker
  become: true
  tasks:
   - name: stop previous version docker
     shell: docker stop docker_demo
   - name: remove stopped container
     shell: docker rm -f docker_demo	  
   - name: remove docker images
     shell: docker image rm -f anjon/docker_demo
      
   - name: create docker image
     shell: docker run -d --name docker_demo -p 8090:8080 anjon/docker_demo
```
- Add this script to the jenkins job
  - On the project configure tab go to the section `Post Steps`.
  - Add `Send files or execute commands over SSH`
  - Name: `Ansible_Server`
  - Exec command: `ansible-playbook /opt/playbooks/create_docker_container.yml`
  
Now we can update out git repo and then run the jenkins job to check the status. It should be successful.  
To verify it correctly go to `http://<DOCKER_PUB_IP>:8090/webapp`

### Part 03: Deploy Container With Version Control
Untill now we used the latest image to deploy the container. In this stage we'll try to manage some taging and use some versioning so if something bad happen to the latest container we can swith to the previous container easily. 

To get this model we need to use to jenkins variable, which are 
- `BUILD_ID` - The current build id
- `JOB_NAME` - Name of the project of this build.

Now I'm going to edit some of the build command which was used to build the image
- Modify the Step 01 build part as the following
```sh
cd /opt/docker
docker build -t $JOB_NAME:v1.$BUILD_ID .
docker tag $JOB_NAME:v1.$BUILD_ID anjon/$JOB_NAME:v1.$BUILD_ID
docker tag $JOB_NAME:v1.$BUILD_ID anjon/$JOB_NAME:latest
docker push anjon/$JOB_NAME:v1.$BUILD_ID
docker push anjon/$JOB_NAME:latest
docker rmi $JOB_NAME:v1.$BUILD_ID anjon/$JOB_NAME:v1.$BUILD_ID anjon/$JOB_NAME:latest
```

With this we need to modify our ansible playbook which we used for deploying the container. 
```yaml
---
- hosts: docker
  become: true
  tasks:
   - name: stop previous version docker                         # Comment this line for the first build
     shell: docker stop docker_demo                             # Comment this line for the first build
   - name: remove stopped container                             # Comment this line for the first build
     shell: docker rm -f docker_demo	                          # Comment this line for the first build
   - name: remove docker images                                 # Comment this line for the first build
     shell: docker image rm -f anjon/devops_project_04:latest   # Comment this line for the first build
      
   - name: create docker image
     shell: docker run -d --name docker_demo -p 8090:8080 anjon/devops_project_04:latest
```

Now on the docker server we'll always have the latest docker container. But in case of any issue we can easily revert to the previous container.  
`http://<DOCKER_PUB_IP>:8090/webapp`
