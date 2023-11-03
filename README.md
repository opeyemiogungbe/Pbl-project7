# IMPLEMETING DEVOPS TOOLING WEBSITE SOLUTION
This project is created to help introduce some vital set of DevOps tools that will help our team in day to day activities in managing, developing, testing, deploying and monitoring different projects. These tools are the best and widely use around the world by multiple DevOps teams, so we are going to create a single DevOps Tooling Solution that will consist of:

**1. Jenkins** – Jenkins is a popular tool for automation and continuous integration in software development. It is an open-source server that runs on Java and allows developers to build, test, and deploy software in an automated way. Jenkins has many features and plugins that make it easy to configure and customize. Some of the benefits of using Jenkins are:

* It can run on different platforms and environments.

* It can integrate with various source code management tools, such as Git, SVN, etc.

* It can trigger builds based on various events, such as code changes, scheduled time, etc.

* It can execute different types of tests, such as unit tests, integration tests, functional tests, etc.

* It can generate reports and feedback on the build status and quality.

* It can support parallel and distributed builds across multiple machines.


**2. Kubernetes** – Kubernetes is a container orchestration platform that helps manage and deploy applications at scale. The system is designed to simplify life cycle management, allowing developers to focus on their application code rather than infrastructure maintenance. It offers features like self-healing, automatic scaling, and load balancing, making it an ideal solution for large-scale deployments. Kubernetes is also highly extensible, allowing users to add new functionality as needed. Additionally, Kubernetes is open source and backed by a large community of developers, making it a versatile and widely-used platform.


**3. Jfrog Artifactory** – JFrog Artifactory is a universal DevOps solution for hosting, managing, and distributing binaries and artifacts of any type of software in binary form – such as application installers, container images, libraries, configuration files, etc. – can be curated, secured, stored, and delivered using Artifactory. Artifactory also serves as your central hub for DevOps, integrating with your tools and processes to improve automation, increase integrity, and incorporate best practices along the way1. Artifactory supports over 30 natively integrated package and file types, as well as generic repositories for everything else. It can be deployed on-premises, in the cloud, or in a hybrid or multi-cloud environment and also enables you to control the full lifecycle of your binaries, from build to release to archival

**4. Rancher** – Rancher is a complete container management platform for Kubernetes, giving you the tools to successfully run Kubernetes anywhere. it is also an open source software platform that enables organizations to run and manage Docker and Kubernetes in production.

**5. Grafana** – Grafana is an open-source observability platform for visualizing metrics, logs, and traces collected from your applications. It's a cloud-native solution for quickly assembling data dashboards that let you inspect and analyze your stack.
**6. Prometheus** – An open-source monitoring system with a dimensional data model, flexible query language, efficient time series database and modern alerting approach.

**7. Kibana**  – Kibana is a free and open user interface that lets you visualize your Elasticsearch data and navigate the Elastic Stack.

**This project implementataion will consists of following components:**

* Infrastructure: AWS
* Webserver Linux: Red Hat Enterprise Linux 8
* Database Server: Ubuntu 20.04 + MySQL
* Storage Server: Red Hat Enterprise Linux 8 + NFS Server
* Programming Language: PHP
* Code Repository: GitHub

The diagram below gives us an insight on the blueprint of this project

![Screenshot 2023-10-27 094720](https://github.com/opeyemiogungbe/Pbl-project7/assets/136735745/77b42221-39c8-4428-b8a2-08e08b2eb179)

## STEP 1 – PREPARE NFS SERVER

Network File System (NFS) is a distributed file system protocol that allows a user on a client computer to access files over a network as if those files were on the client’s own computer. NFS is implemented using a client-server model, where the server is responsible for managing the file system and the client is responsible for accessing it. NFS is commonly used in Unix and Linux environments, but it can also be used in Windows environments using third-party software.

1. Spin up a new EC2 instance with RHEL Linux 8 Operating System and configure LVM on the Server (as we did in project 6)


* On our LVM configuration nstead of formating the disks as ext4 you will have to format them as xfs as sown in the image below:

![Screenshot 2023-08-17 110046](https://github.com/opeyemiogungbe/Pbl-project7/assets/136735745/89d7a094-f930-45e9-a436-405df52ca934)

* We will ensure there are 3 Logical Volumes. lv-opt lv-apps, and lv-logs as shown in the image below:

![Screenshot 2023-08-17 105105](https://github.com/opeyemiogungbe/Pbl-project7/assets/136735745/4c794132-9948-41a7-a96c-83f7537ca2fa)

* we will create mount points on /mnt directory for the logical volumes as follow:

  Mount lv-apps on /mnt/apps – To be used by webservers

  Mount lv-logs on /mnt/logs – To be used by webserver logs

  Mount lv-opt on /mnt/opt – To be used by Jenkins server in our next project 8 ( Note: We must create the/mnt directory firtst) see the image below:

  ![Screenshot 2023-08-17 110803](https://github.com/opeyemiogungbe/Pbl-project7/assets/136735745/a98ed569-7ea6-47cd-862e-7e8edef432a0)

2. Install NFS server, configure it to start on reboot and make sure it is u and running
```
sudo yum -y update
sudo yum install nfs-utils -y
sudo systemctl start nfs-server.service
sudo systemctl enable nfs-server.service
sudo systemctl status nfs-server.service
```
The image below shows the state of our NFS server after succesful configuration:

![Screenshot 2023-08-18 050404](https://github.com/opeyemiogungbe/Pbl-project7/assets/136735745/3d946be9-dec3-4e06-a106-73228bca1c68)

3. Now we are going to make sure we set up permission that will allow our Web servers to read, write and execute files on NFS running the command below:

```
sudo chown -R nobody: /mnt/apps
sudo chown -R nobody: /mnt/logs
sudo chown -R nobody: /mnt/opt

sudo chmod -R 777 /mnt/apps
sudo chmod -R 777 /mnt/logs
sudo chmod -R 777 /mnt/opt
```
* we need to restart our NFS server for our permission to take effect:
  
`sudo systemctl restart nfs-server.service`

![Screenshot 2023-08-18 080818](https://github.com/opeyemiogungbe/Pbl-project7/assets/136735745/3df209a3-57a8-48a2-ab5f-37c8dac9a0fb)

* We will also configure access to NFS for clients within the same subnet (we will use our subnet/cidr from AWS)

```
sudo vi /etc/exports

/mnt/apps <Subnet-CIDR>(rw,sync,no_all_squash,no_root_squash)
/mnt/logs <Subnet-CIDR>(rw,sync,no_all_squash,no_root_squash)
/mnt/opt <Subnet-CIDR>(rw,sync,no_all_squash,no_root_squash)

Esc + :wq!(this command is to write and quit the Vi editor)

sudo exportfs -arv
```

![Screenshot 2023-08-18 080818](https://github.com/opeyemiogungbe/Pbl-project7/assets/136735745/f875eebd-86cc-4d4e-b7ba-31c380af2688)

4. We are going to also check which port is used by NFS and open it using Security Groups (In our case we are going to open on NFS server TCP 111, UDP 111, TCP2049, UDP 2049)

## STEP 2 — CONFIGURE THE DATABASE SERVER

In this step we are going to do the following:

1. Install MySQL server

![Screenshot 2023-08-19 043841](https://github.com/opeyemiogungbe/Pbl-project7/assets/136735745/9dcb62e3-20d5-4d97-9c56-3c9434a2c677)

3. Create a database and name it tooling

![Screenshot 2023-08-19 044058](https://github.com/opeyemiogungbe/Pbl-project7/assets/136735745/ed2da650-8be2-4e7b-86a9-4606420d4657)

5. Create a database user and name it webaccess

![Screenshot 2023-08-19 050739](https://github.com/opeyemiogungbe/Pbl-project7/assets/136735745/2868f46a-d4e4-4f4a-824b-e33e6014ca59)

6. Grant permission to webaccess user on tooling database to do anything only from the webservers subnet cidr

![Screenshot 2023-08-19 051109](https://github.com/opeyemiogungbe/Pbl-project7/assets/136735745/2f575ff4-c42a-42c7-8854-29c045e6aea9)

Now that we are done with our database configuration we'll go ahead and work on our webservers

## Step 3 — Prepare the Web Servers

We need to make sure that our Web Servers can serve the same content from shared storage solutions, which ia our NFS Server and MySQL database. For storing shared files that our Web Servers will use – we will utilize NFS and mount previously created Logical Volume lv-apps to the folder where Apache stores files to be served to the users (/var/www).

  This approach will help us to add new webservers or remove them whenever we need, and the integrity of the data (in the database and on NFS) will be preserved.

During the next steps we will do following:

* Configure NFS client (this step must be done on all three servers)
* Deploy a Tooling application to our Web Servers into a shared NFS folder
* Configure the Web Servers to work with a single MySQL database

1. Install NFS client after launching 3 new EC2 instance with RHEL 8 Operating System for our webservers

`sudo yum install nfs-utils nfs4-acl-tools -y`

2. Mount /var/www/ and target the NFS server’s export for apps

```
sudo mkdir /var/www
sudo mount -t nfs -o rw,nosuid <NFS-Server-Private-IP-Address>:/mnt/apps /var/www
```
![Screenshot 2023-08-19 055100](https://github.com/opeyemiogungbe/Pbl-project7/assets/136735745/b91bfdcf-6655-4ac8-be58-6da245731afa)

The image above shows us how we make a /var/www directory and mount our NFS towards the created directory.

3. Verify that NFS was mounted successfully by running df -h. Make sure that the changes will persist on Web Server after reboot by updating the fstab file:

`sudo vi /etc/fstab`

add following line

`<NFS-Server-Private-IP-Address>:/mnt/apps /var/www nfs defaults 0 0
`

![Screenshot 2023-08-19 090529](https://github.com/opeyemiogungbe/Pbl-project7/assets/136735745/b4eb784d-36e6-4553-a56e-71634eeb41c6)

Note: in the image above we couldn't get the picture for the updating of the fstab file so we cat into it to see it's content

4. Install Remi’s repository, Apache and PHP (Don't forget we will install it on the 3 webservers)

```
sudo yum install httpd -y

sudo dnf install https://dl.fedoraproject.org/pub/epel/epel-release-latest-8.noarch.rpm

sudo dnf install dnf-utils http://rpms.remirepo.net/enterprise/remi-release-8.rpm

sudo dnf module reset php

sudo dnf module enable php:remi-7.4

sudo dnf install php php-opcache php-gd php-curl php-mysqlnd

sudo systemctl start php-fpm

sudo systemctl enable php-fpm

setsebool -P httpd_execmem 1
```

5. we will verify that our Apache files and directories are available on the Web Server in /var/www and also on the NFS server in /mnt/apps. i'll be creating a new file touch test.txt from one server and check if the same file is accessible from other Web Servers.

![Screenshot 2023-08-19 085636](https://github.com/opeyemiogungbe/Pbl-project7/assets/136735745/64df4ad5-3ed5-495f-8c39-70dbeb54c283)

![Screenshot 2023-08-19 085648](https://github.com/opeyemiogungbe/Pbl-project7/assets/136735745/a8a5d5bf-3b57-4fd2-8da5-8389ca9fdef9)

Fom the image above, the first picture is which is our webserver1 we change directory into /var/www and created a test.txt file in there. we can see the effect shows in our webserver2 when we list the content of /var/www directory. This means our NFS is mounted correctly

6. Now we are going to locate the log folder for Apache on the Web Server and mount it to NFS server’s export for logs. Repeat step №3 to make sure the mount point will persist after reboot.

7. We will Fork the tooling source code from Darey.io Github Account to your Github account.

![Screenshot 2023-08-19 093558](https://github.com/opeyemiogungbe/Pbl-project7/assets/136735745/9170bed8-94a4-4153-834a-5caaacacd2c3)

8. We deploy the tooling website’s code to the Webserver (we will do this using cloning the code from our remote repositoryon github to our webserver) and ensure that the html folder from the repository is deployed to /var/www/html.


![Screenshot 2023-08-19 094650](https://github.com/opeyemiogungbe/Pbl-project7/assets/136735745/5b447cfb-8a6a-488a-beb4-463f2d08d631)


![Screenshot 2023-08-19 094627](https://github.com/opeyemiogungbe/Pbl-project7/assets/136735745/6a103dc8-3cbd-46a1-94d3-3466fda4516d)

9. Ater TCP port 80 is opened on the Web Server, setting permissions to our /var/www/html folder and also disable SELinux

`sudo setenforce 0`

To make this change permanent we must open following config file sudo vi /etc/sysconfig/selinux and set SELINUX=disabled then restrt httpd.

![Screenshot 2023-08-23 130314](https://github.com/opeyemiogungbe/Pbl-project7/assets/136735745/bfd285ce-0476-4e1f-a116-57b14e453964)

10. we will update the website’s configuration to connect to the database (in /var/www/html/functions.php file). Apply tooling-db.sql script to your database using this command

`mysql -h <databse-private-ip> -u <db-username> -p <db-pasword> < tooling-db.sql`

11. Now we will open the website in our browser http://<Web-Server-Public-IP-Address-or-Public-DNS-Name>/index.php and login into the website

![Screenshot 2023-08-23 183250](https://github.com/opeyemiogungbe/Pbl-project7/assets/136735745/a3b261f6-cca2-4204-a658-962c576faf84)


![Screenshot 2023-08-29 053214](https://github.com/opeyemiogungbe/Pbl-project7/assets/136735745/7a42ba59-c839-46bc-b3ae-db77ac9329d5)

The abov image shows our configuration was succesful. we just implemented a web solution for a DevOps team using LAMP stack with remote Database and NFS servers
