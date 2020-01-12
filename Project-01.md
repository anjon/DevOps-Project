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
- To create a maven project we need to install ```Unleash Maven Plugin``` and ```Maven Invoker plugin```
  - To install the plugin go to ```Manage Jenkins``` > ```Manage Plugin``` > ```Available``` 
  - Search for *Unleash Maven Plugin* and *Maven Invoker plugin*. Check the box and install them.
- Create a new item with the name *"DevOps Project 01"* with the item type *"Maven Project"*.
- On the **Source Code Management** section select *Git* and put the *Repository URL* to "https://github.com/anjon/Maven-Web-App.git".
- In my case *Branches to build* is */master*.
- In the **Build** section put *Root POM*=pom.xml and *Goals and options*=clean install package.
- Click *Apply* & *Save*.
- Now on the *"DevOps Project 01"* click *Build Now*.

**Step 02:** Deploy the Build to the Remote Tomcat Server

- Plugin required for this project are listed below. To install the plugin go to *Manage Jenkins > Manage Plugin > Available* and search for the below plugins. Check the box and install them. 

-- Deploy to container Plugin.
- For this project we need to add the credential to access the remote tomcat server. Now to add the Credential follow the below steps. 
-- In the Jenkins home page Click *Credentials > * 
- Now on the section **Post-build Actions** set *WAR/EAR files*=```**/*.war```. On the *Containers* part add container from the *Tomcat 9x remote* form the drop down menu.
