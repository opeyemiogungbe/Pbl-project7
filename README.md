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

Esc + :wq!

sudo exportfs -arv
```


