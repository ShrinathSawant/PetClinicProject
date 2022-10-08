### Guide to CREATE A `CICD Pipeline` on Jenkins Using Docker & Ansible  

## Step1 -- `Pre-Requisite`

```
      Create a VM (on any cloud OR Virtual Box) with 1CPU 2GB RAM

      login to the VM Install below tools
```

## Step2 -- `Install Java & Jenkins`
```
sudo wget https://raw.githubusercontent.com/lerndevops/labs/master/scripts/installJenkins.sh -P /tmp
sudo chmod 755 /tmp/installJenkins.sh
sudo bash /tmp/installJenkins.sh
```

## Step3 -- `Install Maven` 
```
sudo wget https://raw.githubusercontent.com/lerndevops/labs/master/scripts/installMaven.sh -P /tmp
sudo chmod 755 /tmp/installMaven.sh
sudo bash /tmp/installMaven.sh
```

## Step4 -- `Install Docker`
```
sudo wget https://raw.githubusercontent.com/lerndevops/labs/master/scripts/installDocker.sh -P /tmp
sudo chmod 755 /tmp/installDocker.sh
sudo bash /tmp/installDocker.sh
```

## Step5 -- `Install Ansible`
```
sudo wget https://raw.githubusercontent.com/lerndevops/labs/master/scripts/installAnsible.sh -P /tmp
sudo chmod 755 /tmp/installAnsible.sh
sudo bash /tmp/installAnsible.sh

# modify the ansible config file to ensure disable host key checking 

vi /etc/ansible/ansible.cfg 

# uncomment this to disable SSH key host checking
host_key_checking = False
```

## Step6 -- `Login to Jenkins UI`

> **hit `http://IP:8080` in browser   ## incase of cloud please use Public IP ensure the Port is allowed to access**

```
	enter `initialAdminPassword` the page to login ( cat /var/lib/jenkins/secrets/initialAdminPassword )

	click on `Install Suggested Plugins`
	
	continue next and finish the setup. 
```

## Step7 -- `Install reqired Plugins (Install from Jenkins UI)`
```
install all these from Jenkins UI 

  Manage Jenkins --> manage plugins -- Available -- search & install the below
  	1) warnings NG
  	2) cobertura
  	3) Junit
  	4) Build Pipeline
  	5) Docker Piepeline
```

## Step8 -- `Create Credentials (Setup from Jenkins UI)`

```
  Manage Jenkins -->  Manage Credentials ==> Stores scoped to Jenkins - global ==> Add Credentials 
	--> kind: username with password 
	--> scope: Global
	--> username: <enter your docker hub id>
	--> password: <enter your docker hub password> 
	--> ID: DOCKER_HUB_LOGIN 
	--> Description: DOCKER_HUB_LOGIN
```
## Step9 -- `Configure JAVA - MAVEN - Git (Setup from Jenkins UI)`

```
Java configuration in Jenkins console 
	
	Manage Jenkins --> Global Tool Configuration --> JDK --> Add JDK
		Name: myjava ( can be any string )
		JAVA_HOME: /path/to/javahome ( ex: /usr/lib/jvm/java-8-openjdk-amd64 )
```
```
Maven Configuration in Jenkins console
	
	Manage Jenkins --> Global Tool Configuration --> Maven --> Add Maven
		Name: maven3.6 ( can be any string )
		MAVEN_HOME: /path/to/mavenhome ( ex: /opt/apache-maven-3.6.5 )
```
```
Git Configuration in Jenkins console
	
	Manage Jenkins --> Global Tool Configuration --> Git --> Add Git
		Name: git ( can be any string )
		MAVEN_HOME: /path/to/githome ( ex: /usr/bin/git )
```

## Step10 -- `configure Jenkins with Docker - from Jenkin Server CLI` 

> by default Jenkins process runs with Jenkins User, which mean any jenkins Jobs we run from jenkins console will be running jenkins user on Jenkins machine

> we need to configure Jenkins user can run the docker commands by adding jenkins user to docker group

```
    sudo usermod -aG docker jenkins
```		   

> **`restart the Jenkins Service`**  

```
    sudo service jenkins restart
```	

> validate, run docker command with jenkins

```
	su - jenkins           ## switch to jenkins user
	docker ps              ## to list any containers running
	docker pull nginx      ## pull a docker image 
```

##### if the above commands execute without any error then we configured jenkins user properly 

## Step11 -- `Setup Deployment Environments`

> **setup atleast one docker swarm / kubernetes cluster**

```
  create two VMs for QA, Install the required toos & setup the kubernetes cluster, one master - one worker node
```

## Step12 -- `Setup Ansible Inventory on Jenkins machine using CLI`

```
   vi /tmp/inv 
      enter your servers in groups like qa & prod 
   sudo chmod 755 /tmp/inv
   sudo chown jenkins:jenkins /tmp/inv
 
   // look at the sample inventory file under https://raw.githubusercontent.com/lerndevops/PetClinic/master/deploy/inv 
   
   Note: ensure to put only manager IPs in inventory file -- DO NOT PUT NODE IPs
```

## Step 13: Now Let's start creating CICD Pipeline Using Pipeline As Code Script

> **`Jenkins ( home page )`** 

```
  --> Click on New Item from left menu 
  --> Enter an item name: CICD-Pipeline 
  --> Choose: Pipeline 
  --> Click: ok
```

> **`insdie job parameters as below`**

```
-->  Click on Pipeline (TAB) on top 
-->  Definition (drop down): Pipeline Script from SCM
-->  SCM (drop down): Git
-->  Repositories --> Repositories URL --> https://github.com/lerndevops/PetClinic
-->  leave the other values Default for this Demo
-->  Script Path: cicd.gvy 
	--> Note: Script is already availabe at https://github.com/lerndevops/PetClinic/blob/master/cicd.gvy
-->  Click on Save 
-->  Build Now from left Menu 

```

![CICD-JPAC](https://github.com/lerndevops/labs/blob/master/cicd-flow/CICD-JPAC.png)


##### #Spring PetClinic Sample Application [![Build Status](https://travis-ci.org/spring-projects/spring-petclinic.png?branch=master)](https://travis-ci.org/spring-projects/spring-petclinic/)

###### Understanding the Spring Petclinic application with a few diagramsss
<a href="https://speakerdeck.com/michaelisvy/spring-petclinic-sample-application">See the presentation here</a>

## Running petclinic locally Noww
```
	git clone https://github.com/spring-projects/spring-petclinic.git
	cd spring-petclinic
	./mvnw tomcat7:run
```

You can then access petclinic here: http://localhost:9966/petclinic/

## In case you find a bug/suggested improvement for Spring Petclinic1
Our issue tracker is available here: https://github.com/spring-projects/spring-petclinic/issues


## Database configurations

In its default configuration, Petclinic uses an in-memory database (HSQLDB) which
gets populated at startup with data. A similar setup is provided for MySql in case a persistent database configuration is needed.
Note that whenever the database type is changed, the data-access.properties file needs to be updated and the mysql-connector-java artifact from the pom.xml needs to be uncommented.

You may start a MySql database with docker:

```
docker run -e MYSQL_ROOT_PASSWORD=petclinic -e MYSQL_DATABASE=petclinic -p 3306:3306 mysql:5.7.8
```

### Working with Petclinic in Eclipse/STS

### prerequisites
The following items should be installed in your system:
* Maven 3 (http://www.sonatype.com/books/mvnref-book/reference/installation.html)
* git command line tool (https://help.github.com/articles/set-up-git)
* Eclipse with the m2e plugin (m2e is installed by default when using the STS (http://www.springsource.org/sts) distribution of Eclipse)

Note: when m2e is available, there is an m2 icon in Help -> About dialog.
If m2e is not there, just follow the install process here: http://eclipse.org/m2e/download/


### Steps:

1) In the command line
```
git clone https://github.com/spring-projects/spring-petclinic.git
```
2) Inside Eclipse
```
File -> Import -> Maven -> Existing Maven project
```


## Looking for something in particular?

<table>
  <tr>
    <th width="300px">Java Config</th><th width="300px"></th>
  </tr>
  <tr>
    <td>Java Config branch</td>
    <td>
      Petclinic uses XML configuration by default. In case you'd like to use Java Config instead, there is a Java Config branch available <a href="https://github.com/spring-projects/spring-petclinic/tree/javaconfig">here</a>. Thanks to Antoine Rey for his contribution.     
    </td>
  </tr>
  <tr>
    <th width="300px">Inside the 'Web' layer</th><th width="300px">Files</th>
  </tr>
  <tr>
    <td>Spring MVC - XML integration</td>
    <td><a href="/src/main/resources/spring/mvc-view-config.xml">mvc-view-config.xml</a></td>
  </tr>
  <tr>
    <td>Spring MVC - ContentNegotiatingViewResolver</td>
    <td><a href="/src/main/resources/spring/mvc-view-config.xml">mvc-view-config.xml</a></td>
  </tr>
  <tr>
    <td>JSP custom tags</td>
    <td>
      <a href="/src/main/webapp/WEB-INF/tags">WEB-INF/tags</a>
      <a href="/src/main/webapp/WEB-INF/jsp/owners/createOrUpdateOwnerForm.jsp">createOrUpdateOwnerForm.jsp</a></td>
  </tr>
  <tr>
    <td>Bower</td>
    <td>
      <a href="/pom.xml">bower-install maven profile declaration inside pom.xml</a> <br />
      <a href="/bower.json">JavaScript libraries are defined by the manifest file bower.json</a> <br />
      <a href="/.bowerrc">Bower configuration using JSON</a> <br />
      <a href="/src/main/resources/spring/mvc-core-config.xml#L30">Resource mapping in Spring configuration</a> <br />
      <a href="/src/main/webapp/WEB-INF/jsp/fragments/staticFiles.jsp#L12">sample usage in JSP</a></td>
    </td>
  </tr>
  <tr>
    <td>Dandelion-datatables</td>
    <td>
      <a href="/src/main/webapp/WEB-INF/jsp/owners/ownersList.jsp">ownersList.jsp</a> 
      <a href="/src/main/webapp/WEB-INF/jsp/vets/vetList.jsp">vetList.jsp</a> 
      <a href="/src/main/webapp/WEB-INF/web.xml">web.xml</a> 
      <a href="/src/main/resources/dandelion/datatables/datatables.properties">datatables.properties</a> 
   </td>
  </tr>
  <tr>
    <td>Thymeleaf branch</td>
    <td>
      <a href="http://www.thymeleaf.org/petclinic.html">See here</a></td>
  </tr>
  <tr>
    <td>Branch using GemFire and Spring Data GemFire instead of ehcache (thanks Bijoy Choudhury)</td>
    <td>
      <a href="https://github.com/bijoych/spring-petclinic-gemfire">See here</a></td>
  </tr>
</table>

<table>
  <tr>
    <th width="300px">'Service' and 'Repository' layers</th><th width="300px">Files</th>
  </tr>
  <tr>
    <td>Transactions</td>
    <td>
      <a href="/src/main/resources/spring/business-config.xml">business-config.xml</a>
       <a href="/src/main/java/org/springframework/samples/petclinic/service/ClinicServiceImpl.java">ClinicServiceImpl.java</a>
    </td>
  </tr>
  <tr>
    <td>Cache</td>
      <td>
      <a href="/src/main/resources/spring/tools-config.xml">tools-config.xml</a>
       <a href="/src/main/java/org/springframework/samples/petclinic/service/ClinicServiceImpl.java">ClinicServiceImpl.java</a>
    </td>
  </tr>
  <tr>
    <td>Bean Profiles</td>
      <td>
      <a href="/src/main/resources/spring/business-config.xml">business-config.xml</a>
       <a href="/src/test/java/org/springframework/samples/petclinic/service/ClinicServiceJdbcTests.java">ClinicServiceJdbcTests.java</a>
       <a href="/src/main/webapp/WEB-INF/web.xml">web.xml</a>
    </td>
  </tr>
  <tr>
    <td>JdbcTemplate</td>
    <td>
      <a href="/src/main/resources/spring/business-config.xml">business-config.xml</a>
      <a href="/src/main/java/org/springframework/samples/petclinic/repository/jdbc">jdbc folder</a></td>
  </tr>
  <tr>
    <td>JPA</td>
    <td>
      <a href="/src/main/resources/spring/business-config.xml">business-config.xml</a>
      <a href="/src/main/java/org/springframework/samples/petclinic/repository/jpa">jpa folder</a></td>
  </tr>
  <tr>
    <td>Spring Data JPA</td>
    <td>
      <a href="/src/main/resources/spring/business-config.xml">business-config.xml</a>
      <a href="/src/main/java/org/springframework/samples/petclinic/repository/springdatajpa">springdatajpa folder</a></td>
  </tr>
</table>

<table>
  <tr>
    <th width="300px">Others</th><th width="300px">Files</th>
  </tr>
  <tr>
    <td>Gradle branch</td>
    <td>
      <a href="https://github.com/whimet/spring-petclinic">See here</a></td>
  </tr>
</table>


## Interaction with other open source projects

One of the best parts about working on the Spring Petclinic application is that we have the opportunity to work in direct contact with many Open Source projects. We found some bugs/suggested improvements on various topics such as Spring, Spring Data, Bean Validation and even Eclipse! In many cases, they've been fixed/implemented in just a few days.
Here is a list of them:

<table>
  <tr>
    <th width="300px">Name</th>
    <th width="300px"> Issue </th>
  </tr>

  <tr>
    <td>Spring JDBC: simplify usage of NamedParameterJdbcTemplate</td>
    <td> <a href="https://jira.springsource.org/browse/SPR-10256"> SPR-10256</a> and <a href="https://jira.springsource.org/browse/SPR-10257"> SPR-10257</a> </td>
  </tr>
  <tr>
    <td>Bean Validation / Hibernate Validator: simplify Maven dependencies and backward compatibility</td>
    <td>
      <a href="https://hibernate.atlassian.net/browse/HV-790"> HV-790</a> and <a href="https://hibernate.atlassian.net/browse/HV-792"> HV-792</a>
      </td>
  </tr>
  <tr>
    <td>Spring Data: provide more flexibility when working with JPQL queries</td>
    <td>
      <a href="https://jira.springsource.org/browse/DATAJPA-292"> DATAJPA-292</a>
      </td>
  </tr>  
  <tr>
    <td>Eclipse: validation bug when working with .tag/.tagx files (has only been fixed for Eclipse 4.3 (Kepler)). <a href="https://github.com/spring-projects/spring-petclinic/issues/14">See here for more details.</a></td>
    <td>
      <a href="https://issuetracker.springsource.com/browse/STS-3294"> STS-3294</a>
    </td>
  </tr>    
</table>


# Contributing

The [issue tracker](https://github.com/spring-projects/spring-petclinic/issues) is the preferred channel for bug reports, features requests and submitting pull requests.

For pull requests, editor preferences are available in the [editor config](https://github.com/spring-projects/spring-petclinic/blob/master/.editorconfig) for easy use in common text editors. Read more and download plugins at <http://editorconfig.org>.




