# DevOps Project 02
![DevOps Project 02](https://github.com/anjon/DevOps-Project/blob/master/devops-project-02.jpg)

This project objective is to build a java web app and then deploy it to the tomcat server using ansible.

### Prerequisites:
1. Jenkins Server. (https://github.com/anjon/Jenkins/tree/master/jenkins_install)
2. Ansible Server. (https://github.com/anjon/Jenkins/tree/master/ansible_install)
3. Tomcat Server.  (https://github.com/anjon/Jenkins/tree/master/tomcat_install)

#### Stage 01: Build Web Application form Maven
In this stage we are going to build a war file by using Jenkins server. 
- Login to the jenkins server with http://yourpublicip:8080.
- To create a maven project we need to install `Unleash Maven Plugin` and `Maven Invoker plugin`.
  - To install the plugin go to `Manage Jenkins` > `Manage Plugin` > `Available`. 
  - Search for `Unleash Maven Plugin` and `Maven Invoker plugin`. Check the box and install them.
- Create a new item with the name *"DevOps Project 02"* with the item type *"Maven Project"*.
- On the **Source Code Management** section select *Git* and put the *Repository URL* to "https://github.com/anjon/Maven-Web-App.git".
- In my case *Branches to build* is `/master`.
- In the **Build** section put `Root POM=pom.xml` and `Goals and options=clean install package`.
- Click `Apply` & `Save`.
- Now on the *"DevOps Project 01"* click `Build Now`.

#### Stage 02: Send the Web Application to the Deployment Server
In this step we are sending the webapp to our deployment server i.e. Ansible. 
- Login to the ansible server and create a directory `/opt/plabooks`. This We'll user for receive webapp form jenkins and send this to the remote tomcat server.
- For this we need to add ssh server in the Jenkins configuration.
  - Jenkins Home > Manage Jenkins > Configure System
  - In the bottom SSH Servers > Add > 
  - Name=Ansible_Server, Hostname=ansibleserverip, Username=ansadmin
  - Go Advance options, select box `Use password authentication, or use a different key`.
  - Provide `Passphrase / Password = ansadmin_password`
  - In the down click `Test Configuration`. The results should be `Success`.
- Need to install the `Publish Over SSH` plugin. 
  - Jenkins Home > Manage Jenkins > Manage Plugin > Available.
  - Search for `Publish Over SSH` plugin and select it. Then install it. 

Now form the Jenkins Home page go to the ***DevOps Project 02***.
- Click `Configure` and then from the **Post Steps** section `Add post-build step` dropdown menu select `Send files or execute commands over SSH`.
  - SSH Server `Name=Ansible_Server`, `Source files=target/*.war`, `Remote directory=//opt//playbooks`. 
- Click `Apply` and `Save`

#### Stage 03: Deploy webapp to the Tomcat Server. 
For this we need an user to the tomcat server by using which we are going to deploy the webapp there. 
- Login to the Tomcat server to add an user and add target tomcat server ip in ansible configuration.
```sh 
useradd ansadmin
echo "password" | passed --stdin ansadmin
echo "ansadmin ALL=(ALL) NOPASSWD: ALL" >> /etc/sudoers.d/ansadmin
sed -ie 's/PasswordAuthentication no/PasswordAuthentication yes/' /etc/ssh/sshd_config
systemctl restart sshd
```
Add this tomcat server ip address to the ansible host file
```sh
cat /etc/ansible/hosts
[tomcat]
xx.xx.xx.xx
```
- Now we need to write an ansible playbooks to send the webapp file to the tomcat server. 
```sh
vim /opt/playbooks/copywarfile.yml
---
- hosts: tomcat 
  become: true
  tasks: 
    - name: copy jar/war onto tomcat servers
      copy:
        src: /opt/playbooks/target/webapp.war 
        dest: /opt/apache-tomcat-9.0.30/webapps
```
- Add this playbook to the ***DevOps Project 02***
  - Click Configure and then from the **Post Steps** section `Add post-build step` dropdown menu select `Send files or execute commands over SSH`.
  - `Exec command=ansible-playbook /opt/playbooks/copywarfile.yml`
  - Click `Apply` & `Save`

We are now end of our project configuration. Now ifwe click to the `Build Now` under the ***DevOps Project 02*** then this will work like the following way
1. Build the webapp form the git repo which we have added in this project.
2. Publish this webapp to the Ansible server.
3. Send the webapp.war form the Ansible to the Tomcat server by using the `copywarfile.yml` file.

We can verify this configuration form the browser.
`http://tomcat-public-ip:8090/webapp`
