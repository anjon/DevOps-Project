# DevOps Project 01

### Prerequisite
1. Jenkins Server
2. Apache Tomcat Server

### Jenkins Server Setup
For setting up the jenkins server please refer to the link https://github.com/anjon/Jenkins/tree/master/jenkins_install

### Apache Tomcat Sever Setup
For setting up the apache tomcat server refer to the link https://github.com/anjon/Jenkins/tree/master/tomcat_install

### Configure the Project on Jenkins
**Step 01:** Check the build stage on the Jenkins local system
- Login to the jenkins server with http://yourpublicip:8080.
- To create a maven project we need to install `Unleash Maven Plugin` and `Maven Invoker plugin`.
  - To install the plugin go to `Manage Jenkins` > `Manage Plugin` > `Available`. 
  - Search for `Unleash Maven Plugin` and `Maven Invoker plugin`. Check the box and install them.
- Create a new item with the name *"DevOps Project 01"* with the item type *"Maven Project"*.
- On the **Source Code Management** section select *Git* and put the *Repository URL* to "https://github.com/anjon/Maven-Web-App.git".
- In my case *Branches to build* is `/master`.
- In the **Build** section put `Root POM=pom.xml` and `Goals and options=clean install package`.
- Click `Apply` & `Save`.
- Now on the *"DevOps Project 01"* click `Build Now`.

**Step 02:** Deploy the Build to the Remote Tomcat Server
- To deploy to this `.war` file to the remote tomcat server we need to add the tomcat server credential and the Deploy to container Plugin.
- Add the remote tomcat server credential and Plugin
  - On the Jenkins home page. `Credentials` > `Jenkins` > `Global credentials (unrestricted)`.
  - On left `Add Credential`. Then set the `username`, `password`. The `ID` & `Description` are optional but I've set `Tomcat_Credential`.
  - In the tomcat server configuration I've set the user deployer. In this credential I'm using it.
  - The `Deploy to container Plugin` needs to be installed. The installation process is same as in **Step 01**
- Now on the Jenkins Home select the ongoing build i.e. ***DevOps-Project-01***. On the left click `configure`
- Now on the section **Post-build Actions** 
  - Set `WAR/EAR files=**/*.war`. 
  - On the *Containers* part add container *Tomcat 9x remote* form the drop down menu.
  - Set `Credentials` form drop down menu which we just set.
  - Provide `Tomcat URL=http://tomcat_server_ip:8090`. 
- Click `Apply` & `Save`. 
Now on the ***DevOps-Project-01*** if we click the `Build Now` then the app should be build as .war file in the Jenkins Server and deploy remotely to the Apache Tomcat Server. 

### Check and Verify
Go to the browser and put the apache tomcat server ip to check the webapp output. 
`http://apache_server_ip:8090/webapp`
The output should be similar like 
`Hello Jenkins.This is DevOps Project 01`.

### To Make Automated Build and Deploy
To make this project we can do some more tweak like below. 
- Go to the ***DevOps-Project-01***. The click `Configure`. 
- On the **Build Triggers** section select `Poll SCM` and set `Schedule=H/2 * * * *`
This option set the project git repository to be check for every 2 minutes if there is any change done or not. If there will be any change on the source code then it will trigger a new build automatically. 

