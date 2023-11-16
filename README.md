# DevOps Essentials Lab Cheat Sheet
Welcome to the DevOps Essentials Lab Cheat Sheet! 

This guide provides step-by-step instructions for completing various DevOps labs, covering tasks like: 
`Setting up servers using Terraform,` `Working with Git and GitHub,` `Configuring Jenkins and GitWebHook,` and `Deploying code in Docker containers.`

### DevOps Essentials Lab Pre-requisites
1. Basic understanding of Linux commands,
2. Basic knowledge of a Cloud platform such as AWS,
3. It's good to have an AWS-Free Tier Account for Practice.
---
## Lab 1: Use Terraform to Setup the `Docker Server` and `Jenkins Server` for CICD Lab.

**Objective:**
The objective of this lab is to set up two AWS EC2 instances, one for Jenkins and one for Docker, using Terraform. This lab aims to provide a foundation for building a Continuous Integration/Continuous Deployment (CICD) environment.

The first step is to Manually Launch an EC2 Server using below configuration:

* Region: North Virginia (us-east-1).
* Use tag Name: `CICDLab-YourName`
* OS version as `Ubuntu 22.04 LTS`
* Instance type: `t2.micro`
* Create a new Keypair with the Name `CICDLab-Keypair-YourName`
* In security groups, include ports `22 (SSH)` and `80 (HTTP)` (Add remaining ports later)
* Configure Storage: 10 GiB
* Launch the Instance.
* Once the Instance is Launched, Connect to the Instance using `MobaXterm` or `Putty` with the username `ubuntu`.

**Note:** Ensure to add the remaining ports in the security group ie... 8080, 9999, and 4243.

#### Task 1: Installing Terraform on to Anchor Server.

Once the Anchor EC2 server is up and running, SSH into the machine and do the following:
```
sudo hostnamectl set-hostname CICDLab
bash
```
```
sudo apt update
```
```
sudo apt install wget unzip -y
```
```
wget https://releases.hashicorp.com/terraform/1.6.3/terraform_1.6.3_linux_amd64.zip
```
View the [Terraform's Latest Versions](https://developer.hashicorp.com/terraform/downloads)
```
unzip terraform_1.6.3_linux_amd64.zip
ls
```
```
sudo mv terraform /usr/local/bin
```
```
ls
terraform -v
```
```
rm terraform_1.6.3_linux_amd64.zip
```

#### Task 2: Install AWS CLI and Ansible
Install Python 3 and required packages:
```
sudo apt-get install python3-pip -y
```
```
sudo pip3 install awscli boto boto3 ansible
```
```
aws configure
```
#### Enter the Credentials as below. Example:
| **Access Key ID** | **Secret Access Key** |
| ----------------- | --------------------- |
| AKIAXMWJXSSHRD27T6SC | H4Vh0U5oenKfmJ/+FEUcbaGbDjcnGAmZvQLX7zTT |

---
#### Note: If you need to create new credentials, Follow the below steps:
1. Go to the AWS console. On the top right corner, click on your name or AWS profile ID.
2. When the menu opens, click on Security Credentials.
3. Under AWS IAM Credentials, click on **Create Access Key**.
4. If you already have two active keys, you can deactivate and delete the older one so that you can create a new one.
5. Complete the "aws configure" step
---
#### Once configured, do a smoke test to check if your credentials are valid
```
aws s3 ls
```
#### Create a host inventory file with the necessary permissions
 ```
 sudo mkdir /etc/ansible && sudo touch /etc/ansible/hosts
```
```
 sudo chmod 766 /etc/ansible/hosts
```

#### Task 3: Use Terraform to launch two servers.
Create the Terraform configuration and variables files as described.
* We need to create two additional servers (docker-server and jenkins-server, You can use **t2.micro** for Docker or Jenkins)
* For **git** we will use the anchor EC2 from where we are operating now 
#### Create the terraform directory and set up the config files in it
```
mkdir devops-labs && cd devops-labs
```
As a first step, create a key using ssh-keygen.
```
ssh-keygen -t rsa -b 2048 
```
**Note:**
1. This will create **id_rsa** and **id_rsa.pub** in **/home/ubuntu/.ssh/**
2. Keep the path as **/home/ubuntu/.ssh/id_rsa**; don't set up any passphrase, just hit the '**Enter**' key for the 3 questions it asks

#### Now prepare the Terraform config files.
```
vi DevOpsServers.tf
```
Type the below code into DevOpsServers.tf
```
provider "aws" {
  region = var.region
}

resource "aws_key_pair" "mykeypair" {
  key_name   = var.key_name
  public_key = file(var.public_key)
}

# to create 2 EC2 instances
resource "aws_instance" "my-machine" {
  # Launch 2 servers
  for_each = toset(var.my-servers)

  ami                    = var.ami_id
  key_name               = var.key_name
  vpc_security_group_ids = [var.sg_id]
  instance_type          = var.ins_type

  # Read from the list my-servers to name each server
  tags = {
    Name = each.key
  }

  provisioner "local-exec" {
    command = <<-EOT
      echo [${each.key}] >> /etc/ansible/hosts
      echo ${self.public_ip} >> /etc/ansible/hosts
    EOT
  }
}
```


Now, create the variables file with all variables to be used in the main config file.
```
vi variables.tf
```
Change following data in variables.tf File. 
1. Edit the **Allocated Region** (**Ex:** ap-south-1) & **AMI ID** of same region,
2. **security group id** and
3. **KeyPair Name** (Example: "**yourname**-CICDlab-key-pair")

```
variable "region" {
    default = "us-east-1"
}

# Change the SG ID. You can use the same SG id used for your CICD anchor server
# Basically the SG should open ports 22, 80, 8080, 9999 and 4243
variable  "sg_id" {
    default = "sg-06dc8863d3ed3d280" # us-east-1
}

# Choose a free tier Ubuntu AMI. You can use below. 
variable "ami_id" {
    default = "ami-04505e74c0741db8d" # us-east-1; Ubuntu
}

# We are only using t2.micro for this lab
variable "ins_type" {
    default = "t2.micro"
}

# Replace 'yourname' with your first name
variable key_name {
    default = "yourname-CICDlab-key-pair"
}

variable public_key {
    default = "/home/ubuntu/.ssh/id_rsa.pub"   #Ubuntu OS
}

variable "my-servers" {
  type    = list(string)
  default = ["jenkins-server", "docker-server"]
}
```
#### Now, execute the terraform config files to launch the servers
```
terraform init
```
```
terraform fmt
```
```
terraform validate
```
```
terraform plan
```
```
terraform apply -auto-approve
```
#### After the terraform code is executed, check hosts inventory file and ensure below output (sample)
```
cat /etc/ansible/hosts
```
It will show ip addresses of jenkins server and docker server as below.

* [jenkins-server]
44.202.164.153
* [docker-server]
34.203.249.54

To update Jenkins & Docker Public IP addresses (Optional)
```
sudo vi /etc/ansible/hosts 
```

#### Now ssh into jenkins-server and check they are accessible from anchor EC2
```
ssh ubuntu@<Jenkins ip address>
```
#### Set the hostname as
```
sudo hostnamectl set-hostname Jenkins
bash
```
```
sudo apt update
```
**Exit** from Jenkins Server

#### Now ssh into docker-server and check they are accessible from anchor EC2
```
ssh ubuntu@<Docker ip address>  
```
#### Set the hostname as
```
sudo hostnamectl set-hostname Docker
bash
```
```
sudo apt update
```
**Exit** from Docker Server

#### Task 4: Use Ansible to deploy respective packages onto each of the servers 
* Navigate to the ansible directory and download the playbook:
* Create a directory and change to it
```
cd ~
mkdir ansible && cd ansible
```
#### Now, Download the playbook, which will deploy packages into the servers.
```
wget https://devops-e-e.s3.ap-south-1.amazonaws.com/DevOpsSetup.yml
```
#### Run the above playbook to deploy the packages
```
ansible-playbook DevOpsSetup.yml
```
#### At the end of this step, the Docker-server and Jenkins-server will be ready for the 

#### Check if the Jenkins landing page is appearing.
* Use your respective Public IP address as shown: http://44.202.164.153:8080/ 

#### Check if docker is working
* Use your respective Public IP address as shown: http://34.203.249.54:4243/version 

---
**Summary:**
1. Launch two EC2 instances in AWS - one for Jenkins and one for Docker.
2. Install Terraform on the Jenkins server to automate infrastructure provisioning.
3. Configure AWS CLI and Ansible for managing resources.
4. Create a Terraform configuration to define the servers and their attributes.
5. Launch the servers using Terraform.
6. Update the Ansible hosts file with the server details.
7. Configure Jenkins and Docker servers with proper hostnames.
8. Use Ansible to install necessary software packages and dependencies on both servers.

#### =============================END of LAB-01=============================

## Lab 2: Git and GitHub Operations.

**Objective:**
This lab focuses on Git and GitHub operations, including initializing a Git repository, making commits, creating branches, and pushing code to a GitHub repository.

#### Pre-requisites:
1. Create a **GitHub Account** & **Empty Public Repository** with name as **"hello-world"** (To know how to create refer Course Material)
2. Basic familiarity with Git commands.

After that, let's operate in the local Git repository

#### Task-1: Initializing the local git repository and committing changes

On the CICD anchor EC2, do the below:
```
cd ~/
```
Check the GIT version. 
```
git --version
```
If it does not exist, then you can install it with the below command. Else no need to execute the below line:  
```
sudo apt install git -y
```
Download the Java code we are going to use in the CICD pipeline
```
wget https://devops-e-e.s3.ap-south-1.amazonaws.com/hello-world-master.zip
```
```
unzip hello-world-master.zip -d hello-world-master
```
```
ls
rm hello-world-master.zip
```
```
cd hello-world-master && cd hello-world-master
ls
```
```
git init .
```
Check the email and user name configurations.
```
git config user.email
git config user.name
```
If you need to change it, you can use below:
```
git config --global user.email "< Add your email >"
```
```
git config --global user.name "< Add your Name >"
```
```
git status
```
```
git add .
```
```
git status
```
```
git log
```
```
git commit -m "This is the first commit"
```
```
git log
```
```
git status
```
```
git remote add origin < Replace your Repository URL > 
```
**Ex:** git remote add origin **https://github.com/janjiralakirankumar/hello-world.git**
```
git remote show origin
```
#### Task 2: Pushing the Code to your Remote GitHub Repository  

* To push code to **Github**, You need to generate a **Personal Access Token** (PAT) in github.
* Go to your GitHub homepage and Click on settings on the top right profile Icon.
* Click on Developer settings (At the bottom on the left side menu). Click on Personal Access Token and then Click on Generate New Token.
* Under '**Select Scopes**' select all items. Click on '**generate token**'. Copy the token

**Example:** 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
```
git push origin master 
```
* when it asks for password, enter the **Personal Access Token** and Press Enter.
* **Note:** When you enter the token, PAT is invisible and It's the expected behavior.
 
#### Task 3: Creating a Git Branch and Pushing into the Remote Repository  

```
git branch dev
```
```
git branch
```
```
git checkout dev
```
```
git branch
```
```
vi index.html
```
Press INSERT and add the below content
```
<html>
<body>
<h1> Hi There! This file is added in dev branch </h1>
</body>
</html>
```
Save vi using ESCAPE + :wq!
```
git status
```
```
git add index.html
```
```
git status
```
```
git commit -m "adding new file index.html in new branch dev"
```
```
git log
```
```
git push origin dev
```
Enter PAT Token when it asks for a password and Press Enter. (PAT is invisible, when you paste)
```
git branch
```
```
git branch prod
```
```
git branch
```
```
git checkout prod
```
```
git branch
```
```
git merge dev
```
```
git push origin prod
```
Enter Token: <PAT>
```
git checkout master
```
```
git merge prod
```
```
git push origin master
```
Enter UserID and Token then Press Enter.

---
**Summary:**
1. Initialize a local Git repository on an AWS EC2 instance.
2. Configure Git settings for email and username.
3. Add and commit changes to the Git repository.
4. Create and switch between Git branches.
5. Make changes and commits in different branches.
6. Push the code to a GitHub repository.
7. Merge branches and push the changes to GitHub.

#### =============================END of LAB-02=============================

## Lab 3: Configure Jenkins

**Objective:**
The objective of this lab is to configure Jenkins to build and deploy applications. It includes setting up Jenkins, installing necessary plugins, and configuring Jenkins to build Maven projects.

Initially, Copy the **private key** to the Jenkins server. so, that we can SSH from from **Jenkins Server** to **Docker Server** and viseversa.
```
cd ~
```
```
ansible jenkins-server -m copy -a "src=/home/ubuntu/.ssh/id_rsa dest=/home/ubuntu/.ssh/id_rsa" -b
```
```
ansible docker-server -m copy -a "src=/home/ubuntu/.ssh/id_rsa dest=/home/ubuntu/.ssh/id_rsa" -b
```
SSH into the **Jenkins server** and get the **initial password** for Jenkins
```
ssh ubuntu@xx.xx.xx.xx
```
```
sudo cat /var/lib/jenkins/secrets/initialAdminPassword
```
**Example:** "afbe8d33e25b4b908c0b9f91546f09e6"

1. Now, go to the **browser** and enter the Jenkins URL as shown: **http://< Jenkin's Public IP>:8080/**
2. Under Unlock Jenkins, enter the above **initialAdminPassword** & click **Continue**.
3. Click on **Install suggested Plugins** on Customize Jenkins page.
4. Once the plugins are installed, it gives you the page where you can create a New **Admin User**. 
5. Enter the **User Id** and **Password** followed by **Name and E-Mail ID** then click on **Save & Continue**.
6. In the next step, on Instance Configuration Page, verify your **Jenkins Public IP** and **Port Number** then click on **Save and Finish**

#### Now you will be prompted to the Jenkins Home Page

1. Click on **Manage Jenkins** > **Plugins** > **Available Plugins** tab and search for **Maven**.
2. Select the "**Maven Integration**" Plugin and "**Unleash Maven**" Plugin and click **Install** (without restart).
3. Once the installation is completed, click on **Go back to the top page**.
4. On Home Page select **Manage Jenkins** > **Tool**.
5. Inside Tool Configuration, look for **Maven installations**, click **Add Maven**. 
6. Give the **Name as "Maven"**, choose **Version as 3.9.4**, and **Save** the configuration.
7. Now you need to make a project for your application build, for that select **New Item** from the Home Page of Jenkins
8. Enter an item name as **hello-world** and select the project as **Maven Project** and then **click OK.**
   ( You will be prompted to the configure page inside the hello-world project.)
9. Go to the "**Source Code Management**" tab, and select Source Code Management as **Git**, Now you need to provide the GitHub Repository **Master Branch URL** and **GitHub Account Credentials**.
10. In the Credentials field, you have to click **Add** then click on **Jenkins**.
11. Then you will be prompted to the **Jenkins Credentials Provider** page. Under Add Credentials, you can add your **GitHub Username**, **Password**, and **Description**. Then click on **Add**.
12. After returning to the Source Code Management Page, click on Credentials and Choose your **GitHub Credentials**.
13. Keep all the other values as default and select "**Build**" Tab and inside Goals and options write "**clean package**" and **save** the configuration.

#### Note: 'clean package' command clears the target directory, Builds the project, and packages the resulting JAR file into the target directory.

16. Now, you will get back to the Maven project "**hello-world**" and click on "**Build Now**" for building the **.war** file for your application

* You can go to **Workspace** > **dist** folder to see that the **.war** file is created there.
* war file will be created in **/var/lib/jenkins/workspace/hello-world/target/**

## Task 2: Installing and Configuring Tomcat for Deploying our Application on Jenkins Server

* Now, SSH into the Jenkins server (Make sure that you are root user and Install Tomcat web server)
* **Note:** (If you are already in Jenkins Server, again SSH is not needed.)

1. Follow below steps:
```
sudo apt update
```
```
sudo apt install tomcat9 tomcat9-admin -y
```
```
ss -ltn
```
```
sudo systemctl enable tomcat9
```
Now we need to navigate to **server.xml** to change the Tomcat port number from **8080 to 9999**.
(As port number 8080 is already being used by the Jenkins website)
```
sudo vi /etc/tomcat9/server.xml
```
If you are unable to open file then change the permissions by using below command.
```
sudo chmod 766 /etc/tomcat9/server.xml
```
#### Change 8080 to 9999
* press esc & Enter **":"** and copy paste below code and hit enter
```
g/8080/s//9999/g
```
Save vi using **ESCAPE + :wq!**

* To Verify whether the Port is changed, execute the below Command.
```
cat /etc/tomcat9/server.xml
```
If you are unable to open file then change the permissions by using below command.
```
sudo chmod 766 /etc/tomcat9/server.xml
```
Now restart the system for the changes to take effect
```
sudo service tomcat9 restart
```
```
sudo service tomcat9 status
```
**To exit**, press **ctrl+c**

* Once the Tomcat service restart is successful, go to your web browser and enter **Jenkins Server IP address** followed by **9999** port.

Example: **http://< Jenkins Public IP >:9999**     or     **http://184.72.112.155:9999**
* Now you can check the Tomcat running on **port 9999** on the same machine.
* We need to copy the **.war** file created in the previous Jenkins build from the Jenkins workspace to tomcat webapps directory to serve the web content
```
sudo cp -R /var/lib/jenkins/workspace/hello-world/target/hello-world-war-1.0.0.war /var/lib/tomcat9/webapps
```
* Once this is done, go to your browser and enter Jenkins Public IP address followed by port 9999 and path (URL:  **http://< Jenkins Public IP >:9999/hello-world-war-1.0.0/**).
* Now, you can see that Tomcat is now serving your web page
* Now, Stop tomcat9 and remove it. Otherwise, it will slow down Jenkins.
```
sudo service tomcat9 stop
```
```
sudo apt remove tomcat9 -y
```
---
**Summary:**
1. Install Jenkins on an AWS EC2 instance.
2. Unlock Jenkins and create an admin user.
3. Install plugins, including Maven integration.
4. Configure Jenkins to use a specific version of Maven.
5. Create a Maven project in Jenkins for the "hello-world" application.
6. Configure source code management with Git and GitHub.
7. Define build goals and options.
8. Build the project and verify the outcome.
9. Install and configure Apache Tomcat for serving web applications.
10. Deploy the built WAR file to Tomcat.

#### =============================END of LAB-03=============================

## Lab 4: Using GitWebHook to build your code automatically using Jenkins

**Objective:**
This lab focuses on configuring Git WebHooks to automatically trigger Jenkins builds when code changes are pushed to a GitHub repository.

#### Task 1: Configure Git WebHook in Jenkins

1. Go to Jenkins webpage and choose **Manage Jenkins** > **Plugins**
2. Go to the **Available** Tab, Search for **GitHub Integration**. Select on the **GitHub Integration Plugin** and then on **Install** (without restart).
3. Once the installation is complete, click on **Go back to the top page**.
4. In your **hello-world project**, Click on **Configure**.
5. Go to **Build Triggers** and enable the **GitHub hook trigger for GITScm polling**. Then **Save**.
6. Go to your **GitHub website**, and inside the **hello-world** repository > **Settings** > **Webhooks** and Click on the **Add Webhook**.
7. Now fill in the details as below.
#### Payload URL Example: 
* http://< jenkins-PublicIP >:8080/github-webhook/ (**Example:** http://184.72.112.155:8080/github-webhook/)
* **Content type:** application/JSON

Then, Click on **Add Webhook**.

#### Task 2: Verifying whether the WebHook is working or not by editing the Source Code

1. Now, Make a minor change and commit in GitHub's **hello-world repository** by editing **hello.txt** file.
2. As the source code gets changed, Jenkins gets triggered by the WebHook and starts building the new source code.
3. Go to Jenkins, and you can see a build is happening.
4. Observe the successful load build on the Jenkins page.
---
**Summary:**
1. Configure Git WebHooks in Jenkins for automatic triggering of builds.
2. Create a GitHub WebHook for a specific GitHub repository.
3. Make a minor code change in the GitHub repository to trigger a build in Jenkins.
4. Verify that Jenkins successfully starts a new build when changes are pushed.

#### =============================END of LAB-04=============================

## Lab 5: Add Docker Machine as Jenkins Slave, build and deploy code in Docker Host as a container

**Objective:**
In this lab, you will set up a Docker container as a Jenkins slave, build a Docker image for a Java web application, and deploy it in a Docker container.

1. Go to **Jenkin's home page** and click on the **Manage Jenkins** and **Nodes**.
2. Click on **New Node** in the next window. Give the node name as **docker-slave** and Select **"permanent agent"**
3. Fill out the details for the node docker-slave as given below.
* The name should be given as **docker-slave**,
* Remote Root Directory as **/home/ubuntu**,
* labels to be **Slave-Nodes**,
* usage to be given as **"use this node as much as possible"**
* Launch method to be set as **"launch agents via SSH"**.
* In the host section, give the **Public IP of the Docker instance**.
* For Credentials for this Docker node, click on the dropdown button named **Add** and then click on **Jenkins**;
* Then in the next window, in kind select **SSH username with private key** (give username as ubuntu),
* In **Private Key** Select **Enter directly** then Copy-Paste the Private Key and then click on **Add** .

**Note:**
To get the private key, Go to your **CICD-anchored EC2 machine** and run below command.
```
cd ~/.ssh
cat id_rsa
```
* Copy the entire content, including the **first and last lines**. Paste it into the space provided for the **private key** then click on **Add**.
* Now, In SSH Credentials, choose the newly created **Ubuntu** credentials.
* Host Key Verification Strategy Select as **"Non Verifying Verification Strategy"** and **Save** it.
* Click on the **Add** button.
* it's done.
  
#### Now In CLI, SSH into your Docker Host and Perform the below steps to create a "Dockerfile" in /home/ubuntu directory.
```
cd ~
```
```
vi Dockerfile
```
enter the below:
```
# Pull Base Image
FROM tomcat:8-jre8

# Maintainer
MAINTAINER "CloudThat"

# Copy the war file to the images tomcat path
ADD hello-world-war-1.0.0.war /usr/local/tomcat/webapps/
```
---
#### Explanatory Notes of Above Docker FIle:

The Dockerfile you provided is a basic Dockerfile for deploying a Java web application using the Tomcat web server as the base image. Here's what each part of the Dockerfile does:

1. `FROM tomcat:8-jre8`: This line specifies the base image for your Docker container. In this case, it uses the official Tomcat 8 image with the Java 8 runtime. This image contains the Tomcat web server and Java runtime environment.

2. `MAINTAINER "CloudThat"`: This line is a comment and specifies the maintainer or author of the Dockerfile.

3. `ADD hello-world-war-1.0.0.war /usr/local/tomcat/webapps/`: This line adds your Java web application's WAR file (`hello-world-war-1.0.0.war`) to the `/usr/local/tomcat/webapps/` directory inside the container. When Tomcat starts, it will automatically deploy the WAR file as a web application.

This Dockerfile is a simple example of how to package a Java web application in a Docker container. It uses an existing Tomcat image as the base, and you're adding your application's WAR file to it. You can further customize this Dockerfile based on your specific requirements, such as adding environment variables, configuring Tomcat, or specifying additional settings for your application.

Once you build this Docker image and run a container based on it, your Java web application should be accessible through Tomcat on port 8080 inside the container, as mentioned in your Jenkins job's post-build steps.

---
1. Go to your **Jenkins Home page**, click on the **drop-down** on **hello-world project**, and select Configure 
tab.
2. In **General Tab**, Select **Restrict where this project can be run** and enter Label Expression as 
**Slave-Nodes.**
3. Go to **Post Steps Tab**, select **"Run only if the build succeeds"** then click on **Add post-build** step select **Execute shell** from the drop-down and copy paste the below commands in the shell and **Save**

**Execute shell commands in Jenkins:**
#### Note: You may replace 'yourname' with your actual first name (lines 3 and 5).

```
cd ~
cp -f /home/ubuntu/workspace/hello-world/target/hello-world-war-1.0.0.war .
sudo docker container rm -f yourname-helloworld-container
sudo docker build -t helloworld-image .
sudo docker run -d -p 8080:8080 --name yourname-helloworld-container helloworld-image
```
---
#### Explanatory Notes of the above Commands:

The commands you provided are part of the Jenkins job's post-build steps, and they are responsible for building a Docker image and running a Docker container for your Java web application. Here's a breakdown of what each command does:

1. `cd ~`: Change the working directory to the user's home directory.

2. `cp -f /home/ubuntu/workspace/hello-world/target/hello-world-war-1.0.0.war .`: Copy the WAR file (presumably the artifact of your Java web application) from the Jenkins workspace to the current directory (`~`), where you'll perform the Docker build.

3. `sudo docker container rm -f yourname-helloworld-container`: Remove any existing Docker container with the name "yourname-helloworld-container" forcefully if it exists. You should replace "yourname" with your actual first name.

4. `sudo docker build -t helloworld-image .`: Build a Docker image with the tag "helloworld-image" based on the Dockerfile located in the current directory (`.`). The Dockerfile you created earlier specifies how the image should be built.

5. `sudo docker run -d -p 8080:8080 --name yourname-helloworld-container helloworld-image`: Run a Docker container named "yourname-helloworld-container" from the "helloworld-image" image. This container will be detached (`-d`) and will map port 8080 on the host to port 8080 inside the container. You should replace "yourname" with your actual first name.

After running these commands, your Java web application should be deployed inside a Docker container, and it will be accessible on port 8080 of your Docker host.

Make sure that Docker is properly configured on your host, and all the necessary dependencies for your Java web application are included in the WAR file. Additionally, ensure that the Dockerfile is correctly configured to set up your application inside the container.

---
* Now you can build your **hello-world project** Either by
1. Clicking on **"Build Now"** or
2. By **making a small change in Github files**.

* Once the build is successful, access the tomcat server page,

To access the Page In Browser Type **"http:// < Your Docker Host Public IP >:8080/hello-world-war-1.0.0/"** to see the website
* **Example:** http://3.95.192.77:8080/hello-world-war-1.0.0/
---
**Summary:**
1. Create a Jenkins slave node named "docker-slave."
2. Configure the slave node to use SSH for communication.
3. Set up a Dockerfile to define the Docker image for the Java web application.
4. Create a Jenkins job to build and deploy the Java web application in a Docker container.
5. Add post-build steps to the Jenkins job to copy the WAR file, build the Docker image, and run a Docker container.
6. Trigger the Jenkins job to build and deploy the application in the Docker container.
7. Access the deployed web application in the Docker container via the Docker host's public IP.
----------------------------------------------------------------------
Once Done, It's time to **Clean up** the Instances

We can now **terminate all the 3 instances** and we are Done with All Labs.

#### =============================END of LAB-05=============================

---
## Important Git Commands

Here are some important Git commands and their brief explanations:

1. **`git init`**
   - Initializes a new Git repository in the current directory.

   ```bash
   git init
   ```

2. **`git clone`**
   - Copies a repository from a remote source (like GitHub) to your local machine.

   ```bash
   git clone <repository_url>
   ```

3. **`git add`**
   - Adds changes in the working directory to the staging area for the next commit.

   ```bash
   git add <file>
   ```

4. **`git commit`**
   - Records changes in the repository with a message describing the changes.

   ```bash
   git commit -m "Your commit message"
   ```

5. **`git status`**
   - Shows the status of changes as untracked, modified, or staged.

   ```bash
   git status
   ```

6. **`git diff`**
   - Shows the differences between the working directory and the latest commit.

   ```bash
   git diff
   ```

7. **`git log`**
   - Displays a log of commits, showing commit messages, authors, dates, and commit hashes.

   ```bash
   git log
   ```

8. **`git branch`**
   - Lists existing branches or creates a new branch.

   ```bash
   git branch          # List branches
   git branch <branch_name>   # Create a new branch
   ```

9. **`git checkout`**
   - Switches to a different branch or restores working files from a specific commit.

   ```bash
   git checkout <branch_name>   # Switch to a branch
   git checkout -b <new_branch>   # Create and switch to a new branch
   ```

10. **`git merge`**
    - Combines changes from different branches.

    ```bash
    git merge <branch_name>
    ```

11. **`git pull`**
    - Fetches changes from a remote repository and integrates them into the current branch.

    ```bash
    git pull origin <branch_name>
    ```

12. **`git push`**
    - Uploads local branch commits to the remote repository.

    ```bash
    git push origin <branch_name>
    ```

13. **`git remote`**
    - Lists remote repositories that are connected to your local repository.

    ```bash
    git remote -v   # Show remote repositories with URLs
    ```

14. **`git fetch`**
    - Downloads changes from a remote repository without merging them into your working branch.

    ```bash
    git fetch origin
    ```

15. **`git reset`**
    - Resets the current branch to a specific commit, optionally preserving changes in the working directory.

    ```bash
    git reset --hard <commit_hash>
    ```

These are just a few fundamental Git commands. There are many more, each serving different purposes in the version control process.
