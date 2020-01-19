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
