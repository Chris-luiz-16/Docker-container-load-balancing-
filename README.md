# Docker containers Load balancing

This repo is based on common situation on how to load balance multiple containers either in a single ec2-instance or in a multiple ec2-instance.
**Let's get started**

## Table of Contents

* [Differnt scenario of Load balancing the containers](https://github.com/Chris-luiz-16/LB-Docker-container-dilemma/edit/main/README.md#multiple-scenario-of-load-balancing-the-containers)
  * [Single instance multiple container](https://github.com/Chris-luiz-16/LB-Docker-container-dilemma/edit/main/README.md#1-single-instance-multiple-container)
  * [Multiple instance single container](https://github.com/Chris-luiz-16/LB-Docker-container-dilemma/edit/main/README.md#2-multiple-instance-single-container)
* [Simple flask hello world to test the load balancing](https://github.com/Chris-luiz-16/LB-Docker-container-dilemma/edit/main/README.md#simple-flask-hello-world-to-test-the-load-balancing)
* [Deploying multiple containers and Load balancing in a single ec2-instance](https://github.com/Chris-luiz-16/LB-Docker-container-dilemma/edit/main/README.md#deploying-multiple-containers-and-load-balancing-in-a-single-ec2-instance)
  * [Make a Custom Network Bridge](https://github.com/Chris-luiz-16/Docker-container-load-balancing-#1-make-a-custom-network-bridge)
  * [Creating the containers ](https://github.com/Chris-luiz-16/Docker-container-load-balancing-#2-creating-the-containers)
  * [Time to load balance using the ALB](https://github.com/Chris-luiz-16/Docker-container-load-balancing-#3-time-to-load-balance-using-the-alb)
  * [Route 53 domain point to ALB endpoint](https://github.com/Chris-luiz-16/Docker-container-load-balancing-#route-53-domain-point-to-alb-endpoint)
  * [ Verification of single instance multiple container setup](https://github.com/Chris-luiz-16/Docker-container-load-balancing-/edit/main/README.md#verification-of-single-instance-multiple-container-setup)
* [Deploying multiple ec2-instance with a container and load balancin it](https://github.com/Chris-luiz-16/Docker-container-load-balancing-/edit/main/README.md#deploying-multiple-ec2-instance-with-a-container-and-load-balancing-it)
  *  [Docker container creation for multiple instance setup](https://github.com/Chris-luiz-16/Docker-container-load-balancing-/edit/main/README.md#docker-container-creation-for-multiple-instance-setup)
  * [Load balancing it using the ALB](https://github.com/Chris-luiz-16/Docker-container-load-balancing-/edit/main/README.md#load-balancing-it-using-the-alb)
  * [Route 53 domain point to ALB endpoint]()
  * [Verification](https://github.com/Chris-luiz-16/Docker-container-load-balancing-/edit/main/README.md#verification)



## Different scenario of Load balancing the containers 



Let's get into the diagram so that you can understand it better
<br />

![ALB](https://github.com/Chris-luiz-16/LB-Docker-container-dilemma/assets/128575317/01aca5f7-c53b-4d0d-a99f-3cc622bcb3a1)
<br />

### ***1] Single instance multiple container***
***

As you can see on the left side image where all the setup is in a single ec2-instance. There are **three** containers running in a single ec2-instance.

Containers are running in the same port **5000** and all are having the same flask enviornment setup. However, I've done a port mapping so that all the 3 containers will have differnt ports assigned while publishing the port publicily.

In this scenario, we'll load balance all the 3 containers and will point to a domain like **example.com** so that the end-user can access the domain with an SSL on it after load balancing it with the ALB.


### ***2] Multiple instance single container***
***

As you can see on the right side of the image where you have **three** ec2-instance and each one of them having a one container running in the **5000** port. 

This is most common setup of Load balancing multiple Ec2-instance by defining the same on the target group.

<br />

## Simple flask hello world to test the load balancing 

I've created a simple flask hello world project to test the load balancing of the application. 

```python
from flask import Flask, request
import os
import socket

app = Flask(__name__)

@app.route('/')
def index():
    hostname = socket.gethostname()
    return f'<h1><center>This is Flask Application running in the Hostname: {hostname}</center></h1>'

flask_port = os.getenv('FLASKPORT', 5000)
app.run(host='0.0.0.0', port=flask_port)
```
<br />
If you guys dont have the time to build the code from the scratch don't worry I got your back. You can simply execute the below docker command to pull from my public docker hub image. It's as simple as that

```sh
docker pull chrisluiz16/version
```
<br />
<br />

## Deploying multiple containers and Load balancing in a single ec2-instance

It's time for my most favourite part to deploy multiple container in a single ec2-instance. Let's go

### 1] Make a Custom Network Bridge
***

Making a custome network bridge helps you to communicate with all the containers using just the ***container name***. So it's very important that you don't miss this step

```sh
docker network create lb-net
```
<br />

### 2] Creating the containers 
***

After creating the custom network bridge, now it's time to create some containers to check whether we will be able to execute first infra setup mentioned above on [Single instance multiple container](https://github.com/Chris-luiz-16/LB-Docker-container-dilemma/edit/main/README.md#1-single-instance-multiple-container)

```sh
# Make sure to open 8081,8082 and 8083 port in your Ec2-instance Security group

docker container run --name flaskapp1 -d --restart always --network lb-net -p 8081:5000 chrisluiz16/version:latest
docker container run --name flaskapp2 -d --restart always --network lb-net -p 8082:5000 chrisluiz16/version:latest
docker container run --name flaskapp3 -d --restart always --network lb-net -p 8083:5000 chrisluiz16/version:latest
```

Once that's executed your container's will be listed like this

```sh
docker container ls -a
CONTAINER ID   IMAGE                        COMMAND            CREATED         STATUS         PORTS                                       NAMES
fefe57eba914   chrisluiz16/version:latest   "python3 app.py"   8 seconds ago   Up 7 seconds   0.0.0.0:8083->5000/tcp, :::8083->5000/tcp   flaskapp3
a5c87a8b2cc1   chrisluiz16/version:latest   "python3 app.py"   9 seconds ago   Up 8 seconds   0.0.0.0:8082->5000/tcp, :::8082->5000/tcp   flaskapp2
7867ecfa5356   chrisluiz16/version:latest   "python3 app.py"   9 seconds ago   Up 8 seconds   0.0.0.0:8081->5000/tcp, :::8081->5000/tcp   flaskapp1
```
<br />

### 3] Time to load balance using the ALB
***

It's time we deploy the ALB for our instance. Before deploying the AWS ALB we need to provision the target group first.Create the target group as below

1. Create Target group
2. Choose the target group as ***Instances***
3. I choose the target group name as ***lb-tg*** you can choose any name you wish
4. Set HTTP as **80** [we can set the redirection rule later]
5. Set VPC where your instance is placed, mine is ***default*** so I'm not changing it
6. Protocol version as HTTP1
7. Keep all values default and just click ***Next**
8. After clicking Next here comes the tricky part,First tick the instance and then ***Ports for the selected instances*** enter the ports that we port published
9. Change the default port 80 to 8081 and click ***Include as pending below***. Screenshot attached for reference![Screenshot 2023-06-02 005151](https://github.com/Chris-luiz-16/LB-Docker-container-dilemma/assets/128575317/38f0afa3-4db6-4f1b-90d1-d0ca9fe9bd36)
<br />

10. Once that's done repeat step 8 and 9 untill 8082 and 8083 is added as the targets. End result will be like this ![image](https://github.com/Chris-luiz-16/LB-Docker-container-dilemma/assets/128575317/b6b4c34b-ff56-444c-b881-0791c2e3a057)
11. Click ***Create Target group*** next

Time to Create the ALB and attach it to our target
***
1. Create Load balancer
2. Choose ***Application Load Balancer*** and click on Create
3. Choos whatever name you like to name your ALB
4. Set Scheme as Internet facing
5. IP address type as IPv4
6. Set VPC as default
7. Mappings select 2 subnet alteash in whichever region you are
8. Set Security group to allow ALB to have 80 and 443 
9. Listener as HTTP:443 with target group name as the name you set for target name as ***lb-tg***
10. Default SSL/TLS certificate load the certificate from ACM you can follow the doc [Requesting a public certificate](https://docs.aws.amazon.com/acm/latest/userguide/gs-acm-request-public.html)
11] Click on create load balancer and that's it job done


Time to allow force http to https redirect
***

1. Go to load balancer
2. Click on your alb
3. Choose the listner and then click listners option. Screenshot attached for reference. ![image](https://github.com/Chris-luiz-16/LB-Docker-container-dilemma/assets/128575317/91059555-cfb4-4e97-b418-a0d6d7bd1cf3)
4. After that ***Default actions*** from add action to Redirect
5. Choose protocol HTTPS with the port 443 and status code as 301 permanently moved
6. Just click add and that's been sorted

### Route 53 domain point to ALB endpoint
***

Point the ALB enpoint by aliasing it with your domain name as shown below
![image](https://github.com/Chris-luiz-16/LB-Docker-container-dilemma/assets/128575317/a474574e-b6a6-45eb-b3cc-deab3a54e123)

### Verification of single instance multiple container setup
***
<br />
Here you can see the final output where the domain flask.chrisich.fun is loadbalancing by dispyaing the hostnames of the containers

![ezgif com-gif-maker](https://github.com/Chris-luiz-16/LB-Docker-container-dilemma/assets/128575317/47fe7b01-cc81-49fe-91ea-0d7d42efeea5)

<br />

<br />

## Deploying multiple ec2-instance with a container and load balancing it

Here we should have 3 ec2-instance and all the 3 will be having the same flask container running in 5000 port and setting a port publishing to 80 port and then loadbalancing it as eplained in the right side of the diagram  [Different scenario of Load balancing the containers](https://github.com/Chris-luiz-16/LB-Docker-container-dilemma/edit/main/README.md#multiple-scenario-of-load-balancing-the-containers)

### Docker container creation for multiple instance setup
***

Make sure you run the below docker command on all the 3 ec2-instance
```sh
docker container run --name flaskapp1 --restart always -d -p 80:5000 chrisluiz16/version
```

### Load balancing it using the ALB

Time to create the target group in order to attach it to the load balancer

1. Create Target group
2. Choose the target group as ***Instances***
3. I choose the target group name as ***alb-tg*** you can choose any name you wish
4. Set HTTP as **80** [we can set the redirection rule later]
5. Set VPC where your instance is placed, mine is ***default*** so I'm not changing it
6. Protocol version as HTTP1
7. Keep all values default and just click ***Next**
8. Just select on all the 3 instance without changing the port of ***Ports for the selected instances***. Attaching an image for refernece. ![image](https://github.com/Chris-luiz-16/Docker-container-load-balancing-/assets/128575317/3fe910be-b416-4141-944f-610cb4fa1d6f)
9. After that click include as pending below and then click crete target group and that's done
<br />

<br />

Time to Create the ALB and attach it to our target
***

1. Create Load balancer
2. Choose ***Application Load Balancer*** and click on Create
3. Choos whatever name you like to name your ALB
4. Set Scheme as Internet facing
5. IP address type as IPv4
6. Set VPC as default
7. Mappings  make sure select atleast 2 subnet in whichever region you are provisioning the ALB
8. Set Security group to allow ALB to have 80 and 443 
9. Listener as HTTP:443 with target group name as the name you set for target name as ***alb-tg***
10. Default SSL/TLS certificate load the certificate from ACM you can follow the doc [Requesting a public certificate](https://docs.aws.amazon.com/acm/latest/userguide/gs-acm-request-public.html)
11] Click on create load balancer and that's it job done
<br />

<br />

Time to allow force http to https redirect
***

1. Go to load balancer
2. Click on your alb
3. Choose the listner and then click listners option. Screenshot attached for reference. ![image](https://github.com/Chris-luiz-16/LB-Docker-container-dilemma/assets/128575317/91059555-cfb4-4e97-b418-a0d6d7bd1cf3)
4. After that ***Default actions*** from add action to Redirect
5. Choose protocol HTTPS with the port 443 and status code as 301 permanently moved
6. Just click add and that's been sorted
<br />

### Route 53 domain point to ALB endpoint

Point the ALB enpoint by aliasing it with your domain name as shown below
![image](https://github.com/Chris-luiz-16/Docker-container-load-balancing-/assets/128575317/05e41bf1-d565-404a-b9cb-ca6d3143aa6c)

### Verification

I have set the aliasing for lb.chrisch.fun in route53 record. Lets' try calling it in the browser to check whether it's load balancing it or not.
<br />

![alb_AdobeExpress (1)](https://github.com/Chris-luiz-16/Docker-container-load-balancing-/assets/128575317/64752697-b460-474c-b4a8-43723994a438)


