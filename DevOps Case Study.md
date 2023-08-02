# Genpact Capstone Project

## Python Task

#### Guideline for Python, API with Python, RDBMS ( Capestone_project ):

#### Code is provided for the tasks but somewhere code is missing, and the format is not appropriate. So, do it yourself and complete the provided tasks.

#### TASK 1: Create a Restful API in Python with Flask for displaying the list of employees with browser and Postman, and record the screenshots for both.

#### TASK 2: Create a Restful API in python with Flask for displaying a particular emp_no. record with browser and Postman, and record the screenshots for both.

#### TASK 3: Create a Restful API in Python with Flask for the CRUD operations specifically get, put, and delete with Postman only, and record the screenshots for three of the operations.

<a href="https://github.com/cloudthat-devops/genpact_capstone_batch1/blob/main/Python/Python_readme.pdf">See the Python task here</a>

<a href="https://github.com/cloudthat-devops/genpact_capstone_batch1/blob/main/Readme_python.md">See the Python task copyable link here</a>

<a href="https://github.com/cloudthat-devops/genpact_capstone_batch1/blob/main/Python/api.py">See the Python solution code here</a>

## AWS HAPROXY TASK

### Create two EC2 instances and install apache2 web-server in it. Launch the third EC2 instance and install HA-PROXY in it. The HA-PROXY instance should act as a load balancer for the two Apache web-server instances

<a href="https://github.com/cloudthat-devops/genpact_capstone_batch1/blob/main/AWS_HAPROXY/AWS%20Hands%20on%20Lab-Hproxy.pdf">See the solution here</a>

<a href="https://github.com/cloudthat-devops/genpact_capstone_batch1/blob/main/AWS_HAPROXY/haproxy_lab_notes.md">See the alternate solution here</a>

## Terraform Task :
### Problem Statement: Launch an Ubuntu EC2 instance (t2.micro) to be used as your terraform workstation.  From that WS, using terraform, launch an EC2 instance (instance type: t2.micro, OS: Red Hat Linux) to be used as an ansible workstation for the ansible task.  Ensure that you create a key (using ssh-keygen) and use it while launching the EC2, so that we can SSH into the ansible WS once it is created. 
### Hints: 
In your terraform WS, install terraform using following commands


```
$ sudo apt update
$ sudo apt install wget unzip -y
$ wget https://releases.hashicorp.com/terraform/1.0.11/terraform_1.0.11_linux_amd64.zip
$ unzip terraform_1.0.11_linux_amd64.zip
$ sudo mv terraform /usr/local/bin
```

install the aws cli

```
$ sudo apt-get install python3-pip -y
$ sudo pip3 install awscli 
```

use aws configure and give your credentials

create a directory and inside that directory create your terraform files to create a instance

<a href="https://github.com/cloudthat-devops/genpact_capstone_batch1/tree/main/terraform">See the Terraform files here</a>

Also remember to create key pair using 

```
$ ssh-keygen -f mykey
```

### ** Make sure to modify your ami depending upon region and instance you need. Also modify the VPC id **

Finally use the following commands to to create your instance

```
terraform init
terraform fmt
terraform validate
terraform plan 
terraform apply 
```

## Ansible Tasks:
### Problem Statement: Once you have created new instance using terraform (as part of terraform task), ssh into that instance and install ansible in it.   After that, you have to install httpd webserver in the managed node.  You don’t have separate managed nodes. So use your ansible workstation itself as the managed node by adding the below line in your host inventory file:
#### localhost ansible_connection = local 

### Hint
### Install ansible using the following commands
```
$ sudo yum check-update
$ sudo yum install python3.8 wget -y
$ sudo pip3 install awscli boto boto3 ansible
$ ansible --version
```

use aws configure and give your credentials

Create a inventory in the location /etc/ansible/hosts

```
localhost ansible_connection=local 
```
create a directory and inside that directory create your ansible playbook to install httpd webserver

<a href="https://github.com/cloudthat-devops/genpact_capstone_batch1/tree/main/ansible">See the Ansible files here</a>

use

```
ansible-playbook <playbook name.yaml> 
```  
Make sure the webserver is running
  
## Docker & Kubernetes Task:
### Build a docker image to use the python api and push it to the DockerHub. Create a pod and nodeport service with that Docker image.
  
###  Hint: A KOPS cluster would be provided to you. You can use the worker nodes to write DockerFile and build image
###  Hint: Use the DockerFile provided to you if needed to create the docker image
 
Create the DockerFile, requirements.txt and python api code in the same directory. use the following commands to build the image and push it to docker hub
  
<a href="https://github.com/cloudthat-devops/genpact_capstone_batch1/tree/main/Docker">See the Docker files here</a>
  
 ``` 
  $ docker login -u <username> 
  $ docker build -t <username>/test-flask-app:v1 . 
  $ docker push <username>/test-flask-app:v1 
```

In the jumper node create a pod that uses the above created image. Use the pod.yaml file.
  
<a href="https://github.com/cloudthat-devops/genpact_capstone_batch1/tree/main/kubernetes">See the Kubernetes pod file here</a>
  
Use the following command to create the pod and a service for that pod

```
$ kubectl apply -f <pod file name.yaml>
$ kubectl expose po <pod name> --type NodePort --port 5000
$ kubectl get svc
```
Use the public IP of the worker nodes and nodeport number to access in web page

#### Example

```
http://PublicIP:NodePort_no/v1/books/
http://PublicIP:NodePort_no/v1/books/navathe
http://PublicIP:NodePort_no/v1/books/hightower
http://PublicIP:NodePort_no/v1/books/ritchie
```
