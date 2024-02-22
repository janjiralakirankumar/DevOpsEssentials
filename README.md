#  DevOps Essentials Lab Cheat Sheet
Welcome to the DevOps Essentials Lab Cheat Sheet!

![how-devops](https://github.com/janjiralakirankumar/DevOpsEssentials/assets/137407373/b0493ccd-3370-4f98-b08a-6e76626ca226)

This guide provides step-by-step instructions for completing various DevOps labs, covering tasks like: 
`Setting up servers using Terraform,` `Working with Git and GitHub,` `Configuring Jenkins and GitWebHook,` and `Deploying code in Docker containers.`

### DevOps Essentials Lab Pre-requisites
1. Basic understanding of Linux commands,
2. Basic knowledge of a Cloud platform such as AWS,
3. It's good to have an AWS-Free Tier Account for Practice.
---
## Lab 1: Use Terraform to Setup the `Docker Server` and `Jenkins Server` for CICD Lab.

**Objective:**
The objective of this lab is to set up two AWS EC2 instances, `one for Jenkins` and `one for Docker,` using Terraform. This lab aims to provide a foundation for building a Continuous Integration/Continuous Deployment (CICD) environment.

---------------------------------------------------------------------
### Task-0: The first step is to `Manually Launch an Anchor EC2 Server` with the below configuration:

#### Step-01: Pre-requisites:

Login to AWS Console, go to EC2 Dashboard, and under "Network & Security," create a key pair and a security group.

1. Create key pair with name: `DevOps-Keypair-YourName`
2. Create security group with name: `DevOps-SG-YourName`
   (Include Ports: `22 [SSH],` `80 [HTTP],` `8080 [Jenkins],` `9999 [Tomcat],` and `4243 [Docker]`)

Once you are ready, and while Manually Launching an `Anchor EC2 Instance`, select the above `DevOps-Keypair-YourName` and `DevOps-SG-YourName`

#### Step-02: Steps for Manually launching the Anchor EC2 Instance

* **Region:** North Virginia (us-east-1).
* **Use tag Name:** `Anchor-EC2-YourName`
* **AMI Type and OS Version:** `Ubuntu 22.04 LTS`
* **Instance type:** `t2.micro`
* Choose the existing Keypair with the Name: `DevOps-Keypair-YourName`
* In security groups, Choose the existing security group with name: `DevOps-SG-YourName`
* **Configure Storage:** 10 GiB
* Click on `Launch Instance.`
---------------------------------------------------------------------
### Task-1: Installing Terraform onto `Anchor Server` to automate the creation of 2 more EC2 instances.

Once the Anchor EC2 server is up and running, SSH into the machine using `MobaXterm` or `Putty` with the username `ubuntu` and do the following:

[Click here](https://mobaxterm.mobatek.net/download-home-edition.html) to download MobaXterm (**Note:** Choose `Installer Edition` and install on your Laptop) 

```
sudo hostnamectl set-hostname AnchorServer
bash
```
```
sudo apt update
```
```
sudo apt install wget unzip -y
```
```
wget https://releases.hashicorp.com/terraform/1.7.3/terraform_1.7.3_linux_amd64.zip
```
[Click here](https://developer.hashicorp.com/terraform/downloads) for Terraform's Latest Versions
```
unzip terraform_1.7.3_linux_amd64.zip
ls
```
```
sudo mv terraform /usr/local/bin
```
```
rm terraform_1.7.3_linux_amd64.zip
```
```
ls
terraform -v
```
```
terraform
```
---------------------------------------------------------------------
### Task-2: Install Python 3, pip, AWS CLI, and Ansible on to Anchor Server
Install Python 3 and the required packages:
```
sudo apt-get update
```
```
sudo apt-get install python3-pip -y
```
```
sudo pip3 install awscli boto boto3 ansible
```
For Authentication with AWS we need to provide `IAM User's CLI Credentials`
```
aws configure
```
#### Credentials Example:

| **Access Key ID** | **Secret Access Key** |
| ----------------- | --------------------- |
| AKIAXMWJXSSHRD27T6SC | H4Vh0U5oenKfmJ/+FEUcbaGbDjcnGAmZvQLX7zTT |

<details>
  <summary>To know how to create New Credentials, Click here:</summary>

##### Here is a step-by-step summary of the instructions:

1. Go to the AWS console. On the top right corner, click on your name or AWS profile ID.
2. Click on Security Credentials.
3. Under AWS IAM Credentials, click on **Create Access Key**.
4. If you already have two active keys, deactivate and delete the older one. Create a new one, download, and save it.
5. Complete the `aws configure` step using the newly created access key, secret key, region, and output format.
</details>

---------------------------------------------------------------------
#### Once configured, do a smoke test to check if your credentials are valid and got the access to AWS account.

You can check using any one command or both.
```
aws s3 ls
```
(Or)
```
aws iam list-users
```

#### Create a host inventory file with the necessary permissions (Ansible need this Inventory file for identifying the targets)
 ```
 sudo mkdir /etc/ansible && sudo touch /etc/ansible/hosts
```
```
 sudo chmod 766 /etc/ansible/hosts
```
**Note:** The above command gives `Read and Write` permissions to `Owner,` `Read and Write` permissions to the `Group,` and `Read and Write` permissions to `Others.`

---------------------------------------------------------------------
### Task-3: Utilizing Terraform, initiate the deployment of two new servers: `Docker-server` and `Jenkins-server` 

* For **Git Operations Lab** we will use the same **Anchor EC2** from where we are operating now 

**Step-01:** As a first step, create a keyPair using `ssh-keygen` Command.

(Same public will be attached to newly created EC2 Instances)

**Note:**
1. This will create `id_rsa` and `id_rsa.pub` in Anchor Machine in `/home/ubuntu/.ssh/` path.
2. While creating choose defaults like:
   * path as **/home/ubuntu/.ssh/id_rsa**,
   * don't set up any passphrase, and just hit the '**Enter**' key for 3 questions it asks.

```
ssh-keygen -t rsa -b 2048 
```

<details>
  <summary>Click here for explanation</summary>
  
* `-t rsa:` Specifies the type of key to create, in this case, RSA.
* `-b 2048:` Specifies the number of bits in the key, 2048 bits in this case. The larger the number of bits, the stronger the key.
</details>

 **Step-02:** Create the terraform directory and set up the config files in it

Create the Terraform configuration and variables files as described.

```
mkdir devops-labs && cd devops-labs
```

**Step-03:** Now create the Terraform config files.
```
vi DevOpsServers.tf
```
Copy and paste the below code into `DevOpsServers.tf`
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
Now, create the variables file with all variables to be used in the `DevOpsServers.tf` config file.
```
vi variables.tf
```
**Note:** Change the following Inputs in `variables.tf.`

1. Edit the **Allocated Region** (**Ex:** ap-south-1) & **AMI ID** of same region,
2. Replace the same **Security Group ID** Created for the Anchor Server
3. Add your Name for **KeyPair** ("**YourName**-CICDlab-KeyPair")

```
variable "region" {
    default = "us-east-1"
}

# Change the SG ID. You can use the same SG ID used for your CICD anchor server
# Basically the SG should open ports 22, 80, 8080, 9999, and 4243
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
    default = "YourName-Jenkins-Docker-KeyPair"
}

variable public_key {
    default = "/home/ubuntu/.ssh/id_rsa.pub"   #Ubuntu OS
}

variable "my-servers" {
  type    = list(string)
  default = ["jenkins-server", "docker-server"]
}
```
**Step-04:** Now, execute the terraform commands to launch the new servers
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
Once the Changes are Applies, Go to `EC2 Dashboard` and check that `2 New Instances` are launched.

**Step-05:** Once the Terraform commands are executed, check the `inventory file` and ensure the below output.
```
cat /etc/ansible/hosts
```
The above command displays the IP addresses of the `Jenkins server` and `docker server` as an example shown below.

* [jenkins-server]

  44.202.164.153
  
* [docker-server]

  34.203.249.54

**(Optional Step):** When you `Stop` and `Start` the EC2 Instances, the Public IP Changes. In that case, execute the below command to update Jenkin's & Docker's new Public IPs in `Inventory file.` 
```
sudo vi /etc/ansible/hosts 
```
Once Updated, Save the File.

**Step-06:**  Check the access from `Anchor to Jenkins` and `Anchor to Docker`

##### From `Anchor Server` SSH into `Jenkins-Server` and check they are accessible.

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
**Exit** only from the Jenkins Server, not the Anchor Server.

##### Now from `Anchor Server` SSH into `Docker-Server` and check they are accessible.

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
**Exit** only from the Docker Server, not the Anchor Server.

---------------------------------------------------------------------
### Task-4: Use `Ansible` to deploy respective packages onto each of the servers 

#### Step-01: In Anchor Server Create a directory and change to it
```
cd ~
mkdir ansible && cd ansible
```
#### Step-02: Now, Download the playbook, which will deploy packages onto the `Docker-server` and `Jenkins-Server.`
```
wget https://devops-e-e.s3.ap-south-1.amazonaws.com/DevOpsSetup.yml
```
#### Step-03: Run the above playbook to deploy the packages
```
ansible-playbook DevOpsSetup.yml
```
At the end of this step, the `Docker-Server` and `Jenkins-Server` will be ready for performing further Labs

#### Step-04:

**1. Verify the Jenkins Landing Page:**

   * Open a web browser and navigate to your Jenkins landing page using your Jenkins server's public IP address. Replace `<Your_Jenkins_IP>` with the actual public IP.
   
   ```
   http://<Your_Jenkins_IP>:8080/
   ```

**2. Verify the Docker Landing Page:**

   * Open a web browser and access the Docker landing page using your Docker server's public IP address. Replace `<Your_Docker_IP>` with the actual public IP.
   
   ```
   http://<Your_Docker_IP>:4243/version
   ```

#### =============================END of LAB-01=============================
---
## Lab 2: Git and GitHub Operations.

**Objective:**
This lab focuses on `Git` and `GitHub operations,` including `initializing a Git repository,` `making commits,` `creating branches,` and `pushing code to a GitHub repository.`

### Pre-requisites:
1. Create a **GitHub Account** & **Empty Public Repository** with name as **"hello-world"**

   **Note:** (To SignUp for GitHub Account, [Click Here](https://github.com/signup?ref_cta=Sign+up&ref_loc=header+logged+out&ref_page=%2F&source=header-home))
2. You need to generate a **Personal Access Token** (PAT) in GitHub.
   
   <details>
     <summary>Click Here To Generate the Token:</summary>
     
   * First, Go to your `GitHub homepage,` Click on the top right `profile Icon` and then `settings`
   * Click on `Developer settings` (At the bottom on the left side menu). Click on `Personal Access Token` and then Click on `Generate New Token.`
   * Under '**Select Scopes**' select all items. Click on '**generate token**'.
   * Once generated, **Copy and save the token in a safe location as it is not visible again in GitHub.**
   
      **Example:** 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
   
   </details>

3. Basic familiarity with **Git Commands.**

   Once the `GitHub Account` and `Empty repository` is ready, let's operate in the Anchor Server.

---------------------------------------------------------------------
### Task-1: Initializing the local git repository and committing changes

On the `Anchor Server,` do the below:
```
cd ~
```
Check the GIT version (Git comes pre-installed on certain Linux machines by default). 
```
git --version
```
**(Optional)** If it does not exist, then you can install it using the below command. (Else no need to execute the below line):  
```
sudo apt install git -y
```
Download the **Java Code** that we are going to use in the CICD pipeline.
```
wget https://devops-e-e.s3.ap-south-1.amazonaws.com/hello-world-master.zip
```
```
unzip hello-world-master.zip
```
```
rm hello-world-master.zip
```
```
cd hello-world-master
```
```
git init .
```
To set the `User Identity` ie... `email and user name.` you can execute below commands:

```
git config --global user.email "< Add your email >"
```
```
git config --global user.name "< Add your Name >"
```
```
git status
```
Move the code from `Working Directory` to `Staging Area (Index)`
```
git add .
```
```
git status
```
```
git log
```
Move the code from `Staging Area (Index)` to `Local Repository`
```
git commit -m "This is the first commit"
```
```
git log
```
```
git status
```
Currently our Code is in Local Repository.

---------------------------------------------------------------------
### Task-2: Pushing the Code to your Remote GitHub Repository  

1. Create `Alias` as `Origin` to GitHub's Remote repository URL.
```
git remote add origin <Replace your Repository URL> 
```
(**Example:** `git remote add origin https://github.com/janjiralakirankumar/hello-world.git`)

2. To view a specific alias use the below command.
```
git remote show origin
```
3. Now you can push your code from `Local Repository` to Remote Repository using the below command.
```
git push origin master 
```
**Note:** When it asks for a password, enter the **Personal Access Token** and Press Enter

   (When you paste, PAT is invisible and It's the expected behavior.)

---------------------------------------------------------------------
### Task 3: Creating a Git Branch and Pushing Code to the Remote Repository
* To create a new Branch
```
git branch dev
```
* To know the current branch
```
git branch
```
* To switch to another branch
```
git checkout dev
```
```
git branch
```
Creating a new file with content in current branch
```
vi index.html
```
change to `INSERT mode` and add the below content
```
<html>
<body>
<h1> Hi There! This file is added in dev branch </h1>
</body>
</html>
```
Save the file using `ESCAPE+:wq!`
```
git status
```
Move the file from `Working Directory` to `Staging Area (Index)`
```
git add index.html
```
```
git status
```
Move the file from `Staging Area (Index)` to `Local Repository`
```
git commit -m "adding new file index.html in new branch dev"
```
```
git log
```
Pushing the same file from `Local Repository` to Remote Repository
```
git push origin dev
```
#### When it asks for a `User ID` enter `GitHub's User ID,` and when it asks for a `password` Enter `PAT` and then Press Enter. 

(**Note:** PAT is invisible when you paste)
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
#### When it asks for a `User ID` enter `GitHub's User ID,` and when it asks for a `password` Enter `PAT` and then Press Enter. 

(**Note:** PAT is invisible when you paste)
```
git checkout master
```
```
git merge prod
```
```
git push origin master
```
#### When it asks for a `User ID` enter `GitHub's User ID,` and when it asks for a `password` Enter `PAT` and then Press Enter. 

(**Note:** PAT is invisible when you paste)

#### =============================END of LAB-02=============================
---
## Lab 3: Configuring Jenkins server and Installing Tomcat onto Jenkin's Server for Deploying our Application.

**Objective:**
The objective of this lab is to configure Jenkins to build and deploy applications. It includes `Setting up Jenkins,` `installing necessary plugins` and `configuring Jenkins to build Maven projects,` and `Installing Tomcat Server for viewing the Web Page.' 

---------------------------------------------------------------------
### Task-1: Configure Jenkins Server:

#### Step-00:

1. Initially, Copy the **private key** from **Anchor Server** to the **Jenkins Server** & **Docker Server**. so, that we can SSH from **Jenkins Server** to **Docker Server** and viseversa.
```
cd ~
```
```
ansible jenkins-server -m copy -a "src=/home/ubuntu/.ssh/id_rsa dest=/home/ubuntu/.ssh/id_rsa" -b
```
  
   <details>
     <summary>Here's a breakdown of the command:</summary>
     
   - `ansible`: The Ansible command-line tool.
   - `jenkins-server`: The target machine specified in your inventory file.
   - `-m copy`: Specifies the Ansible module to use, in this case, the `copy` module.
   - `-a "src=/home/ubuntu/.ssh/id_rsa dest=/home/ubuntu/.ssh/id_rsa"`: Specifies the arguments for the module, indicating the source and destination paths for the file copy.
   - `-b`: Run the Ansible command with elevated privileges.
   
   </details>

```
ansible docker-server -m copy -a "src=/home/ubuntu/.ssh/id_rsa dest=/home/ubuntu/.ssh/id_rsa" -b
```

   <details>
     <summary>Here's a breakdown of the command:</summary>
     
   - `ansible`: The Ansible command-line tool.
   - `docker-server`: The target machine specified in your inventory file.
   - `-m copy`: Specifies the Ansible module to use, in this case, the `copy` module.
   - `-a "src=/home/ubuntu/.ssh/id_rsa dest=/home/ubuntu/.ssh/id_rsa"`: Specifies the arguments for the module, indicating the source and destination paths for the file copy.
   - `-b`: Run the Ansible command with elevated privileges.
   
   </details>

#### Step-01:

1. Go to the **Web Browser** and open a new tab then enter the URL as shown:

   **http://< Jenkin's Public IP>:8080/** (It requests the **InitialAdminPassword** during the setup.)
   
2. To obtain the **InitialAdminPassword**, access the Jenkins Server by SSHing from the Anchor Server, utilizing Jenkins' Public IP.
```
ssh ubuntu@xx.xx.xx.xx
```
From Jenkins execute the below command.
```
sudo cat /var/lib/jenkins/secrets/initialAdminPassword
```
   (**Example:** "afbe8d33e25b4b908c0b9f91546f09e6")

#### Step-02:

1. Now, go back to jenkin's landing page in the **Web Browser**:
   
   (Enter the Jenkins URL as shown: **http://< Jenkin's Public IP>:8080/**)

2. Under Unlock Jenkins, enter the above **initialAdminPassword** & click **Continue**.
3. Click on **Install suggested Plugins** on the Customize Jenkins page.
4. Once the plugins are installed, it gives you the page where you can create a New **Admin User**. 
5. Enter the **User Id** and **Password** followed by **Name** and **E-Mail ID** then click on **Save & Continue**.
6. In the next step, on the Instance Configuration Page, verify your **Jenkins Public IP** and **Port Number** then click on **Save and Finish**

#### Step-03: Configuring Maven in Jenkins

Do the below in Jenkin's Dashboard:

1. Click on **Manage Jenkins** > **Plugins** > **Available Plugins** tab and search for **Maven**.
2. Select the "**Maven Integration**" Plugin and click **Install** (without restart).
3. Once the installation is completed, click on **Go back to the top page**.
4. On Home Page select **Manage Jenkins** > **Tool**.
5. Inside Tool Configuration, look for **Maven installations**, click **Add Maven**. 
6. Give the Name as **"Maven"**, Version-Default one (Latest), and **Save** the configuration.

#### Step-04:

1. Create a new project for your application build by selecting **New Item** from the Jenkins homepage.
2. Enter an item name as **hello-world** and select the project as **Maven Project** and then click **OK.**
   ( You will be prompted to the configure page inside the hello-world project.)
3. Go to the "**Source Code Management**" tab, and select Source Code Management as **Git**, Now you need to provide the GitHub Repository's **Master Branch URL** and **GitHub Account Credentials**.
4. In the Credentials field, you have to click **Add** and then click on **Jenkins**.
5. Then you will be prompted to the **Jenkins Credentials Provider** page. Under Add Credentials, you can add your **GitHub Username**, **Password**, and **Description**. Then click on **Add**.
6. In the Source Code Management page, navigate to Credentials, and select your GitHub credentials.
7. Leave all other values as default, navigate to the "**Build**" tab, and in the "Goals and options" section, input "**clean package**". Save the configuration.

   #### Note: 
   
   The 'clean package' command clears the target directory, Builds the project, and packages the resulting WAR file into the target directory.

8. Return to the Maven project "**hello-world**" and click on "**Build Now**" to initiate the build process for your application's **.war** file.

      * You can go to **Workspace** > **dist** folder to see that the **.war** file is created there.
      * war file will be created in **/var/lib/jenkins/workspace/hello-world/target/**

---------------------------------------------------------------------
### Task-2: Installing and Configuring Tomcat for Deploying our Application on the same Jenkins Server

![image](https://github.com/janjiralakirankumar/DevOpsEssentials/assets/137407373/d5dde194-f10d-4b4d-a20c-890e9ca3e392)

* SSH into the Jenkins server as the root user and install the Tomcat web server.
  **Note:** (If you are already in Jenkins Server, again SSH is not needed.)

#### Step-01: Install Tomcat on to Jenkins Server

```
sudo apt update
```
```
sudo apt install tomcat9 tomcat9-admin -y
```
```
ss -ltn
```
**Note:** when you run `ss -ltn` command you'll see a list of `TCP sockets` that are in a listening state, and the output will also include information such as the `local address,` `port,` and the `state of each socket.`
```
sudo systemctl enable tomcat9
```

#### Step-02: Navigate to **server.xml** and change the Tomcat port number from **8080 to 9999** since port 8080 is already in use by Jenkins

1. Navigate to **server.xml** and change the port number
```
sudo vi /etc/tomcat9/server.xml
```
To Change the port from `8080` to `9999`
   * press esc & Enter **":"** and copy paste below code and then hit Enter
```
g/8080/s//9999/g
```
Save the file using `ESCAPE+:wq!`

2. To Verify whether the Port is changed, execute the below Command.
```
cat /etc/tomcat9/server.xml
```
**(Optional step):** If you are unable to open the file then change the permissions by using the below command.
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
**To exit**, press **Ctrl+C**

3. Once the Tomcat service restart is successful, go to your web browser and enter **Jenkins Server's Public IP address** followed by **9999** port.

   (Example: **http://< Jenkins Public IP >:9999**     (Or)     **http://184.72.112.155:9999**)

   * Now you can check the Tomcat running on **port 9999** on the same machine and it shows the WebPage.
  
#### Step-03: Copy the previously built **.war**

1. Copy the previously built **.war** file from the Jenkins workspace to the Tomcat webapps directory to deploy the web content.
```
sudo cp -R /var/lib/jenkins/workspace/hello-world/target/hello-world-war-1.0.0.war /var/lib/tomcat9/webapps
```

   <details>
     <summary>Here's a breakdown of the command:</summary>
     
   The above command is copying a `WAR (Web Application Archive)` file from the Jenkins workspace to the Tomcat web apps directory. Let's break down the command:
   
   - `sudo`: Run the command with superuser (root) privileges, as copying files to system directories often requires elevated permissions.
   
   - `cp`: The copy command in Linux.
   
   - `-R`: Recursive option, used when copying directories. It ensures that the entire directory structure is copied.
   
   - `/var/lib/jenkins/workspace/hello-world/target/hello-world-war-1.0.0.war`: Source path, specifying the location of the WAR file in the Jenkins workspace.
   
   - `/var/lib/tomcat9/webapps`: Destination path, indicating the Tomcat webapps directory where the WAR file is being copied.
   
   </details>

   **Note:** Assuming your Jenkins job generated `hello-world-war-1.0.0.war` in the workspace, this command copies it to Tomcat's webapps directory, enabling deployment and execution.

2. Access your browser and enter the Jenkins Public IP address, followed by port 9999 and the path (URL: **http://< Jenkins Public IP >:9999/hello-world-war-1.0.0/**).
3. Confirm Tomcat is serving your web page.
4. Stop tomcat9 and remove it to prevent slowing down the Jenkins server.
```
sudo service tomcat9 stop
```
```
sudo apt remove tomcat9 -y
```
#### =============================END of LAB-03=============================
---
## Lab 4: Using GitWebHook to build your code automatically using Jenkins

**Objective:**
This lab focuses on configuring `Git WebHooks` to automatically `trigger Jenkins builds` when code changes are pushed to a GitHub repository.

### Task-1: Setting up GitHub Integration in Jenkins for Automated Builds:

1. Navigate to the Jenkins webpage and select **Manage Jenkins** > **Plugins**.
2. In the **Available** tab, search for **GitHub Integration**. Choose the **GitHub Integration Plugin** and click **Install** (without restart).
3. After installation, return to the top page.
4. In your **hello-world project**, click on **Configure**.
5. Under **Build Triggers**, enable **GitHub hook trigger for GITScm polling** and click **Save**.

### Task-2: Configuring GitHub Webhook in GitHub:

1. On your **GitHub website**, go to the **hello-world** repository > **Settings** > **Webhooks**. Click **Add Webhook**.
2. Fill in the details as follows:
   - **Payload URL Example:**
     - http://< jenkins-PublicIP >:8080/github-webhook/ (Example: http://184.72.112.155:8080/github-webhook/)
   - **Content type:** application/JSON
   - Click **Add Webhook**.

### Task-3: Verifying WebHook Functionality:

1. Make a minor change and commit in GitHub's **hello-world repository** by editing the **hello.txt** file.
2. The WebHook triggers Jenkins upon source code modification, initiating an automatic build.
3. Navigate to Jenkins, where you'll observe the ongoing build.
4. Monitor the Jenkins page for the successful completion of the build.

#### =============================END of LAB-04=============================
---
## Lab-5: Configuring Docker Machine as Jenkins Slave, build and deploy code in Docker Host as a container

**Objective:**
In this lab, you will set up a `Docker container as a Jenkins slave,`and `build a Docker image` for a Java web application, and `Deploy it in a Docker container.`

![image](https://github.com/janjiralakirankumar/DevOpsEssentials/assets/137407373/9139c0b6-2571-4606-84d0-22dac79d479e)

### Task-1: Configuring Docker Machine as Jenkins Slave.

1. In Jenkins Webpage, Navigate to **Jenkin's Dashboard** and click on the **Manage Jenkins** and **Nodes**.
2. Click on **New Node** in the next window. Give the node name as `docker-slave` and Select `Permanent Agent`
3. Fill out the details for the node docker-slave as given below.
   * The name should be given as **docker-slave**,
   * Remote Root Directory as **/home/ubuntu**,
   * labels to be **Slave-Nodes**,
   * usage to be given as **"use this node as much as possible"**
   * Launch method to be set as **"launch agents via SSH"**.
   * In the host section, give the **Public IP of the Docker instance**.
   * For Credentials for this Docker node, click on the dropdown button named **Add** and then click on **Jenkins**;
   * Then in the next window, in kind select **SSH username with private key** (Give username as `ubuntu`),
   * In **Private Key** Select **Enter directly**
   
   **Note:** To get the `Private Key` go to `Anchor Server` (Not Jenkins/Docker) and Execute the below command:
      ```
      cd ~/.ssh
      cat id_rsa
      ```
     (Copy the entire content of the Private Key, including the **First and Last line** till `5 hyphens` only.)
     
   * Once Copied, Paste it into the space provided for the **private key** then click on **Add**..
   * Now, In SSH Credentials, choose the newly created **Ubuntu** credentials.
   * Host Key Verification Strategy Select as **Non Verifying Verification Strategy** and **Save** it and it's done.

**Note:** Check whether the Slave Node is online/Offline.

   <details>
     <summary>(Optional Step): If still the slave node is offline, do as below:</summary>
     
   1. From Jenkin's server using Docker's Public IP SSH into Docker server.
   2. Now, Re-check whether the Slave node is Online/Offline.
   </details>

### Task-3: Build and deploy code in Docker Host on the container.

Once the Slave Node is Online, then continue the below.

#### Now, In the CLI, SSH into your Docker Host and execute the following steps to create a "Dockerfile" in the `/home/ubuntu` directory.
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
   <details>
     <summary>Explanatory Notes of Above Docker FIle:</summary>
     
   The provided Dockerfile is a basic configuration for deploying a Java web application using the Tomcat web server. Here's the breakdown of each part:

   1. `FROM tomcat:8-jre8`: Specifies the base image for the Docker container, utilizing the official Tomcat 8 image with Java 8 runtime. This image includes the Tomcat web server and Java runtime environment.

   2. `MAINTAINER "CloudThat"`: A comment indicating the maintainer or author of the Dockerfile.

   3. `ADD hello-world-war-1.0.0.war /usr/local/tomcat/webapps/`: Copies the Java web application's WAR file (`hello-world-war-1.0.0.war`) into the `/usr/local/tomcat/webapps/` directory within the container. Upon Tomcat startup, the WAR file is automatically deployed as a web application.
   
   </details>

This Dockerfile packages a Java web app in a container using a Tomcat base image. Customize it for your needs. After building and running a container, your app should be accessible via Tomcat on port 8080, following Jenkins job steps.

### Task-4:

1. Navigate to your **Jenkins Home page**, choose the **hello-world project** from the drop-down, and click on the Configure tab.
2. In the **General Tab**, choose **Restrict where this project can be run** and set the Label Expression to **Slave-Nodes.**
3. Move to the **Post Build Steps Tab**, select **"Run only if the build succeeds,"** and then select `add a post-build step`, choose **Execute shell** from the drop-down, paste the provided commands (below) into the shell, and click **Save.**

#### Note: You may replace 'yourname' with your actual first name (lines 3 and 5).

```
cd ~
cp -f /home/ubuntu/workspace/hello-world/target/hello-world-war-1.0.0.war .
sudo docker container rm -f yourname-helloworld-container
sudo docker build -t helloworld-image .
sudo docker run -d -p 8080:8080 --name yourname-helloworld-container helloworld-image
```
   <details>
     <summary>Click here for breakup of command</summary>
     
   The commands you provided are part of the Jenkins job's post-build steps, and they are responsible for building a Docker image and running a Docker container for your Java web application. Here's a breakdown of what each command does:
   
   1. `cd ~`: Change the working directory to the user's home directory.
   
   2. `cp -f /home/ubuntu/workspace/hello-world/target/hello-world-war-1.0.0.war .`: Copy the WAR file (presumably the artifact of your Java web application) from the Jenkins workspace to the current directory (`~`), where you'll perform the Docker build.
   
   3. `sudo docker container rm -f yourname-helloworld-container`: Remove any existing Docker container with the name "yourname-helloworld-container" forcefully if it exists. You should replace "yourname" with your actual first name.
   
   4. `sudo docker build -t helloworld-image .`: Build a Docker image with the tag "helloworld-image" based on the Dockerfile located in the current directory (`.`). The Dockerfile you created earlier specifies how the image should be built.
   
   5. `sudo docker run -d -p 8080:8080 --name yourname-helloworld-container helloworld-image`: Run a Docker container named "yourname-helloworld-container" from the "helloworld-image" image. This container will be detached (`-d`) and will map port 8080 on the host to port 8080 inside the container. You should replace "yourname" with your actual first name.
      
   </details>

Upon executing these commands, your Java web application should be deployed within a Docker container, accessible on port 8080 of your Docker host.

### Task-5: Building the **hello-world project**

Either:
1. Manually click on **"Build Now"** in Jenkins.
2. Make a small change in GitHub files.

After a successful build, access the Tomcat server page:

In your browser, type **"http:// < Your Docker Host Public IP >:8080/hello-world-war-1.0.0/"** to view the website.
*Example:* http://3.95.192.77:8080/hello-world-war-1.0.0/

---------------------------------------------------------------------
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

---
## DevOps Tools and Documentation Links:

Certainly! Here are the official websites and documentation links for some popular DevOps tools:

1. **Version Control:**
   - **Git:**
     - Website: [Git](https://git-scm.com/)
     - Documentation: [Git Documentation](https://git-scm.com/doc)

2. **Continuous Integration and Continuous Deployment:**
   - **Jenkins:**
     - Website: [Jenkins](https://www.jenkins.io/)
     - Documentation: [Jenkins Documentation](https://www.jenkins.io/doc/)
   - **Travis CI:**
     - Website: [Travis CI](https://travis-ci.com/)
     - Documentation: [Travis CI Documentation](https://docs.travis-ci.com/)

3. **Configuration Management:**
   - **Ansible:**
     - Website: [Ansible](https://www.ansible.com/)
     - Documentation: [Ansible Documentation](https://docs.ansible.com/)
   - **Puppet:**
     - Website: [Puppet](https://puppet.com/)
     - Documentation: [Puppet Documentation](https://puppet.com/docs/)

4. **Containerization and Orchestration:**
   - **Docker:**
     - Website: [Docker](https://www.docker.com/)
     - Documentation: [Docker Documentation](https://docs.docker.com/)
   - **Kubernetes:**
     - Website: [Kubernetes](https://kubernetes.io/)
     - Documentation: [Kubernetes Documentation](https://kubernetes.io/docs/)

5. **Infrastructure as Code:**
   - **Terraform:**
     - Website: [Terraform](https://www.terraform.io/)
     - Documentation: [Terraform Documentation](https://www.terraform.io/docs/index.html)

6. **Monitoring and Logging:**
   - **Prometheus:**
     - Website: [Prometheus](https://prometheus.io/)
     - Documentation: [Prometheus Documentation](https://prometheus.io/docs/introduction/overview/)
   - **ELK Stack (Elasticsearch, Logstash, Kibana):**
     - Website: [Elastic](https://www.elastic.co/)
     - Documentation: [Elastic Stack Documentation](https://www.elastic.co/guide/index.html)

Always check the official websites for the most up-to-date information and documentation. Additionally, many of these tools may have vibrant communities where you can find additional resources, tutorials, and support.

#### ============================= END OF All LABs =============================

---
In Linux, you can use the `date` command to change the system date and time. Open a terminal and use the following example as a guide:

```bash
sudo date MMDDhhmm[[CC]YY][.ss]
```

Here's a breakdown of the format:

- `MM`: Month (01-12)
- `DD`: Day (01-31)
- `hh`: Hour (00-23)
- `mm`: Minutes (00-59)
- `CC`: Century (20 for 21st century, 19 for 20th century, etc.)
- `YY`: Year within the century
- `ss`: Seconds (00-61)

For example, to set the date to November 22, 2023, 15:30:00, you would run:

```bash
sudo date 112215302023.00
```

Make sure to replace the values with the desired date and time components.

After executing the `date` command, the system date and time will be updated accordingly.
