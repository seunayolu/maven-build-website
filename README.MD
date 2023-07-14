# Maven and Tomcat installation on EC2 instance

### Pre-requisites
1. T2.Micro Amazon Linux 2023 EC2 instance with Java v17 - yum install java-17 -y 
2. Use find /usr/lib/jvm/java-17 | head -n 3 to find your jdk installation path
3. Edit the ~/.bash_profile script and add the following ENV - JAVA_HOME=/path_to_jdk_installation

### Install Apache Tomcat
1. Download tomcat packages from  https://tomcat.apache.org/download-90.cgi
   ```sh 
   # Change directory to /opt
   cd /opt
   wget https://dlcdn.apache.org/tomcat/tomcat-9/v9.0.78/bin/apache-tomcat-9.0.78.tar.gz
   tar -xvzf /opt/apache-tomcat-9.0.78.tar.gz
   ```
2. give executing permissions to startup.sh and shutdown.sh which are under bin. 
   ```sh
   chmod +x /opt/apache-tomcat-9.0.78/bin/startup.sh
   chmod +x /opt/apache-tomcat-9.0.78/bin/shutdown.sh
   ```

3. create link files for tomcat startup.sh and shutdown.sh 
   ```sh
   ln -s /opt/apache-tomcat-9.0.78/bin/startup.sh /usr/local/bin/tomcatup
   ln -s /opt/apache-tomcat-9.0.78/bin/shutdown.sh /usr/local/bin/tomcatdown
   ```
#### Check point :
Using unique ports for each application is best practice in a production environment. But tomcat and Jenkins runs on port 8080. Hence, lets change tomcat port number to 8090. Change port number in conf/server.xml file under tomcat directory /opt/apache-tomcat-9.0.78

   ```sh
 cd /opt/apache-tomcat-9.0.78/conf
 vi server.xml
# update port number in the "connecter port" field in server.xml
# start the tomcat after configuration update
tomcatup
   ```
#### Check point :
access tomcat application from browser on port 8090
 - http://<Public_IP>:8090
make sure to open port 8090 in your security group/firewall

4. Now Tomcat is accessible on port 8090. But tomcat application doesn't  allow to login from browser. changing a default parameter in context.xml does address this issue
   ```sh
   #search for context.xml
   cd /
   find / -name context.xml
   ```
5. The above command gives 4 webapp/..context.xml files. Comment out the valve section using (<!-- & -->) for the 4 files
   After that restart tomcat services to effect these changes
   ```sh 
   tomcatdown
   tomcatup
   ```
6. Update users information in the tomcat-users.xml file goto /opt/apache-tomcat-9.0.78/conf/tomcat-user.xml and add below users to the file
   ```sh
	<role rolename="manager-gui"/>
	<role rolename="manager-script"/>
	<role rolename="manager-jmx"/>
	<role rolename="manager-status"/>
	<user username="admin" password="admin" roles="manager-gui, manager-script, manager-jmx, manager-status"/>
	<user username="deployer" password="deployer" roles="manager-script"/>
	<user username="tomcat" password="s3cret" roles="manager-gui, admin-gui"/>
   ```
7. Restart serivce and try to login to tomcat application from the browser by clicking on the Manager app. This time it should be Successful

