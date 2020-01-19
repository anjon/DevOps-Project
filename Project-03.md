# DevOps Project 03
![DevOps Project 3](https://github.com/anjon/DevOps-Project/blob/master/devops-project-03.jpg)

### Prerequisite
1. Jenkins Server (https://github.com/anjon/Jenkins/tree/master/Jenkins)
2. Docker Server

### Install & Configure Docker Server 
For this project we need to have a server hosting docker service. We I took an Amazon EC2 instance with amazon linux. Now login to the docker host instance and setup doceker for our project. 
```sh
yum install docker 
systemctl enable docker.service
systemctl start docker.service
systemctl status docker.service
```

Now create a new user for Docker management and add him to Docker (default) group.
```sh
useradd dockeradmin
echo "dockeradmin" | passwd --stdin dockeradmin
usermod -aG docker dockeradmin
sed -ie 's/PasswordAuthentication no/PasswordAuthentication yes/' /etc/ssh/sshd_config
systemctl restart sshd
```

Write a Docker file under /opt/docker
```sh
mkdir /opt/docker
chown -R dockeradmin.dockeradmin /opt/docker
vim Dockerfile
# Pull base image 
From tomcat:8-jre8 

# Maintainer
MAINTAINER "Anjon Sarker" 

# copy war file on to container 
COPY ./webapp.war /usr/local/tomcat/webapps
```

### Integrate Docker Server With Jenkins
Login to the Jenkins console and add Docker server to execute commands from Jenkins
- `Manage Jenkins` --> `Configure system`. On the `Publish over SSH` section add Docker server and credentials
- `Name=Docker_Host`, `Hostname=docker_server_ip`, `Username=dockeradmin`. CLick on the `Use password authentication` box 
- Set the dockeradmin passwod in `Passphrase / Password=dockeradmin`. Now from the below click `Test Configiration` and should be Okay.

### Create A Test Project To Validate The Configuration
From the Jenkins home page create new item named ***DevOps-Project-03*** with the type as Maven. Now we are going to configure this
- In the `Source Code Management` section select `Git` and `Repository URL=https://github.com/anjon/Maven-Web-App.git`. Use the default master branch.
- In the `Build` section `Root POM=pom.xml` and `Goals and options=clean install package`.
- In the `Post Steps` section select `Send files or execute commands over SSH` form the `add post-build steps` drop down menu. 
- Set `Name=Docker_Host`, `Source files=target/*.war`, `Remove prefix=target`, `Remote directory=//opt//docker`.
- In the Execute command section put the below commands
```sh
docker stop docker_demo;
docker rm -f docker_demo;
docker image rm -f docker_demo;
cd /opt/docker;
docker build -t docker_demo .
```
- From `Post Steps` section again select `Send files or execute commands over SSH` form the `add post-build steps` drop down menu.
- Set `Name=Docker_Host`.
- Here set a docker command to run the docker container form the image we just build it form the previous steps
```sh
Exec command=docker run -d --name docker_demo -p 8090:8080 docker_demo
```

Now check on the docker host what are the image are there and containers run over there.  
On the ***DevOps-Project-03*** home page run the `Build Now` and check for the success of the jenkins job. After the completion of  jenkins job please go to the web browser and then check the application  
`http://<DOCKER_PUB_IP>:8090/webapp`  

Thanks :whale2:
