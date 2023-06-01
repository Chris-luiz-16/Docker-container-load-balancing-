# LB-Docker-container-dilemma

This repo is based on common situation on how to load balance multiple containers either in a single ec2-instance or a single in a multiple ec2-instance.
**Let's get started**

## Table of Contents

* [Multiple scenario of Load balancing the containers] (https://github.com/Chris-luiz-16/LB-Docker-container-dilemma/blob/main/README.md#multiple-scenario-of-load-balancing-the-containers)
* [Docker command used for single instance multiple container]





## Multiple scenario of Load balancing the containers 



Let's get into the diagram so that you can understand it better



![ALB](https://github.com/Chris-luiz-16/LB-Docker-container-dilemma/assets/128575317/847a1e14-25f7-4cdb-b07b-94e3aaf68d8b)


### ***1] Single instance multiple container***
***

As you can see on the left side image where all the setup is in a single ec2-instance. There are **three** containers running in a single ec2-instance.

Containers are running in the port **5001,5002 & 5003** and all are having the same flask enviornment setup

In this scenarion, we'll load balance all the 3 containers and will point to a domain like **example.com** so that the end-user can access the domain with an SSL on it after load balancing it with the ALB.


### ***2] Multiple instance single container***
***

As you can see on the right side of the image where you have **three** ec2-instance and one common container running in the **5000** port. 

This is most common setup of Load balancing multiple Ec2-instance by defining the same on the target group.



