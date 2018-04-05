# Hand On Lab: Empower your application lifecycle with containers and Azure with Azure Container Service (Kubernetes)

## Requirements
Azure subscription

## Introduction
Nowadays, one of the biggest challenges when developing modern applications for the cloud is being able to deliver these applications continuously. In this Hands on Lab, you will learn how to implement a full continuous integration and deployment (CI/CD) pipeline using:
- Azure Container Instances: run your containers without provision infrastructure.
- Azure Container Registry: your own private container repository to store your containers.
- Jenkins: the service to run your lifecycle application tasks.
This lab is based on a web application, available on [GitHub](https://github.com/diegomrtnzg/ai-duet-tensor), developed with Python which uses Magenta to play piano.

![](media/Empower%20your%20web%20application%20lifecycle%20with%20containers%20and%20Azure%20using%20Azure%20Container%20Instances%20and%20Jenkins/image1.png)

### Containers in Azure
The process to build a container is:
1. Pull the base container image to your dev environment
2. Customize / create your custom image using the base container image
3. Push your custom container to your registry
4. Deploy the container to your environments

![](media/Empower%20your%20web%20application%20lifecycle%20with%20containers%20and%20Azure%20using%20Azure%20Container%20Instances%20and%20Jenkins/image2.png)

In Azure, we have several services to use containers:
- Azure Container Registry. **Docker private registry** in the cloud.
- IaaS. **Create your own container environment** using IaaS services: Virtual Machines, VM Scale Set, Azure Batch...
- Azure Container Service. **Create your container environment in the easiest way** using Docker Swarm, DC/OS or Kubernetes orchestrator. The new Azure Container Service (AKS) is a managed Kubernetes environment. Azure Service Fabric: microservices orchestrator which allows you to use containers. The only platform with **GA for Windows containers**.
- Azure Container Instances preview. **Container as a Service**: run containers with a single command.
- Web App for Containers. **PaaS service to run containerized web app**.

![](media/Empower%20your%20web%20application%20lifecycle%20with%20containers%20and%20Azure%20using%20Azure%20Container%20Instances%20and%20Jenkins/image3.png)

### Step by step
The objective is to deliver this application continuously in a Kubernetes cluster using Visual Studio Team Services. This is a brief explanation of the steps:
1. Code changes are committed to the source code repository
2. Code repository triggers a project in Jenkins
3. Jenkins gets the latest version of the sources and builds all the images that make up the application
4. Jenkins pushes each image to a Docker registry created using the Azure Container Registry service
5. Jenkins triggers a project
6. The new project deploys a new version of the container in Azure Container Instances
7. The new version of the application is deployed

![](media/Empower%20your%20web%20application%20lifecycle%20with%20containers%20and%20Azure%20using%20Azure%20Container%20Instances%20and%20Jenkins/image4.png)

## Step 0: Prerequisites
### Install Docker in your machine (optional)
*You will need it in the Exercise 1 (optional).*
To install Docker in your local machine, you need to follow these steps:
- [Mac](https://docs.docker.com/docker-for-mac/install/)
- [Windows](https://docs.docker.com/docker-for-windows/install/) 
- [Ubuntu](https://docs.docker.com/install/linux/docker-ce/ubuntu/)
- [Debian](https://docs.docker.com/install/linux/docker-ce/debian/)
- [CentOS](https://docs.docker.com/install/linux/docker-ce/centos/)
- [Fedora](https://docs.docker.com/install/linux/docker-ce/fedora/)

### Install Azure CLI in your machine (optional)
*You will need it in the Exercise 1 (optional).*
To install Azure CLI in your machine, you need to follow these steps:
- [Windows, Mac and Linux](https://docs.microsoft.com/en-us/cli/azure/install-azure-cli?view=azure-cli-latest)

### Install Git in your machine (optional)
*You will need it in the Exercise 1 (optional).*
*If you are using Windows and you don´t have a SSH tool, you will need it in the exercise 2.*

To install Git in your machine, you need to follow these steps:
- [Linux, Mac and Windows](https://git-scm.com/book/en/v2/Getting-Started-Installing-Git)

### Create an GitHub account (optional)
You can use your own GitHub account to complete this lab, create a new one or use the default repository. If you want to create a new GitHub account, follow these steps:
1. Go to github.com.
2. **Sign up for GitHub** by introducing a username, email and password.

    ![](media/Empower%20your%20web%20application%20lifecycle%20with%20containers%20and%20Azure%20using%20Azure%20Container%20Instances%20and%20Jenkins/image5.png)

3.	Choose the **Unlimited public repositories for free** and click “**Continue**”.
4.	Click on “**Skip this step**”.
5.	You will receive an email (in the email address you enter in the second step) to verify your identity. Click “**Verify email address**” on it and enter your credentials.

### Copy the GitHub project (optional)
If you are using you GitHub account (new or old account), you need to copy the GitHub project in your account in order to use it via Jenkins. To copy it, follow these steps:
1. Go to https://github.com/prashanthmadi/ai-duet-tensor 
2. Click “**Fork**” on the top-right part of the GitHub project.

    ![](media/Empower%20your%20web%20application%20lifecycle%20with%20containers%20and%20Azure%20using%20Azure%20Container%20Instances%20and%20Jenkins/image6.png)
 
It will copy the GitHub project in your account. **Save the link to the project**: *https://github.com/[yourusername]/ai-duet-tensor*. 

### Create two Azure Resource Groups
You will need two Azure Resource Groups: one for the Jenkins VM and Azure Container Registry and another for Azure Container Instances. To create them, follow these steps:
1. Go to [portal.azure.com](http://portal.azure.com).
2. Click “**+**” on the top-left part of the Azure portal.

    ![](media/Empower%20your%20web%20application%20lifecycle%20with%20containers%20and%20Azure%20using%20Azure%20Container%20Instances%20and%20Jenkins/image8.png)
 
3. Search **Resource Group** and click it to create a new Azure Resource Group.

    ![](media/Empower%20your%20web%20application%20lifecycle%20with%20containers%20and%20Azure%20using%20Azure%20Container%20Instances%20and%20Jenkins/image7.png)

4. Complete the parameters with these values:
    - Resource group name: **WinterReady**
    - Subscription: your subscription name
    - Resource group location: **West US**
5. Click create.

Repeat the steps to create the second Azure Resource Group. Now, use the Resource group name **WinterReadyACI** in the fourth step.

### Create a Jenkins VM
In order to use Jenkins, we need an environment to run it. Azure Marketplace provides you a customized VM with Jenkins installed, so we will use it. To create it, follow these steps:
1. Go to [portal.azure.com](http://portal.azure.com).
2. Click “**+**” on the top-left part of the Azure portal.

    ![](media/Empower%20your%20web%20application%20lifecycle%20with%20containers%20and%20Azure%20using%20Azure%20Container%20Instances%20and%20Jenkins/image8.png)

3. Search **Jenkins** and click the first option (published by Microsoft) to create a new Jenkins VM. 

    ![](media/Empower%20your%20web%20application%20lifecycle%20with%20containers%20and%20Azure%20using%20Azure%20Container%20Instances%20and%20Jenkins/image9.png)

4. Complete the parameters with these values (don´t modify other values):
    - Name: **JenkisVM**
    - User name: **ready**
    - Password and Confirm password: **Password123$**
    - Subscription: **your subscription name**
    - Resource group: select **Use existing** and **WinterReady**
    - Location: **WestUS**
    - Domain name label: you need to write a **unique domain label** (custom). Example: jenkinsvm[youralias]
    - Integration Settings: **Auto(MSI)**
    - Enable Cloud Agents: **No**
5. **Create** the VM with these values.

We will need the **domain name label**. To check it, go to the WinterReady resource group, select the JenkinsVM and copy the value of DNS name:

![](media/Empower%20your%20web%20application%20lifecycle%20with%20containers%20and%20Azure%20using%20Azure%20Container%20Instances%20and%20Jenkins/image10.png)

### Create an Azure Container Registry
In order to use store container images, we need a container registry. Azure Container Registry is a container private registry. To create it, follow these steps:
1. Go to [portal.azure.com](http://portal.azure.com).
2. Click “**+**” on the top-left part of the Azure portal.

    ![](media/Empower%20your%20web%20application%20lifecycle%20with%20containers%20and%20Azure%20using%20Azure%20Container%20Instances%20and%20Jenkins/image8.png)
  
3. Search Container Registry and click the first option (published by Microsoft) to create a new Azure Container Registry. 

    ![](media/Empower%20your%20web%20application%20lifecycle%20with%20containers%20and%20Azure%20using%20Azure%20Container%20Instances%20and%20Jenkins/image11.png)

4. Complete the parameters with these values (don´t modify other values):
    - Registry name: you need to write a unique name (custom). Example: acr[youralias]
    - Subscription: your subscription name
    - Resource group: select Use existing and WinterReady
    - Location: WestUS
    - Admin user: Enable
5. Create the Azure Container Registry with these values.

We will need the info about the container registry. To check it, go to the WinterReady resource group, select your Azure Container Registry, go to Access keys and copy the values:
- Login server
- Username
- Password

![](media/Empower%20your%20web%20application%20lifecycle%20with%20containers%20and%20Azure%20using%20Azure%20Container%20Instances%20and%20Jenkins/image12.png)

## Step 1: Build the container image locally (optional)
Before start building our Jenkins pipeline, we are going to create our container image locally, save in Azure Container Registry and deploy manually to Azure Container Instances to understand the process.
To complete the process, we need a command line with Docker support, Git and Azure CLI. 

### Create the container image
The first step is creating the container image we will use in the Azure Container Instances using the GitHub project. We will use our Docker environment installed in the previous step:
1. **Open your local shell** (PowerShell, Bash…).
2. **Download locally the source code**. If you are using your own GitHub account, type this command (with your link project similar to https://github.com/[yourusername]/ai-duet-tensor):
```
git clone https://github.com/[yourusername]/ai-duet-tensor.git
```
If you are using the default repository, type this command:
```
git clone  https://github.com/prashanthmadi/ai-duet-tensor.git
```
3. **Navigate to the project folder**:
```
cd ai-duet-tensor
```
4. We are going to use a **[Dockerfile](https://docs.docker.com/engine/reference/builder/)** (text document with the commands which create the image) to create the container image. You can check the Dockerfile with this command:
```
cat Dockerfile
```
5. **Create the image** using the build command. We need some Azure Container Registry info (you saved it in the Step 0. Create an Azure Container Registry). If you are using your own GitHub account:
```
docker build -t [AzureContainerRegistryLoginServer]/ai-duet:local .
```

### Save in the container registry
We have the image built locally. Now we need to store it on Azure Container Registry in order to access it from Azure Container Instances.
1. **Authenticate to the Azure Container Registry** before push the image (you saved the Azure Container Registry info in the Step 0. Create an Azure Container Registry):
```
docker login [AzureContainerRegistryLoginServer] -u [AzureContainerRegistryUsername] -p [AzureContainerRegistryPassword]
```
2. **Push the image** to store it on Azure Container Registry
```
docker push [AzureContainerRegistryLoginServer]/ai-duet:local
```

### Deploy to Azure Container Instances
Azure CLI enables you to manage your services from the Bash terminal. Azure provides you an Azure CLI in the Cloud: Azure Cloud Shell. We are going to use it to deploy the container to Azure Container Instances.
1. Go to [portal.azure.com](http://portal.azure.com).
2. Click “**>_**” on the top-right part of the Azure portal.
3. **Execute this command** to deploy in Azure Container Instances with these options:
    - Azure Resource Group: WinterReadyACI
    - Azure Container Instances name: ai-duet-local
    - Image and repository: used in the previous steps
    - CPU: 1
    - Memory: 1.5
    - IP address: public
```
az container create -g WinterReadyACI --name ai-duet-local --image [AzureContainerRegistryLoginServer]/ai-duet:local --cpu 1 --memory 1.5 --registry-login-server [AzureContainerRegistryLoginServer] --registry-username [AzureContainerRegistryUsername] --registry-password [AzureContainerRegistryPassword] --ip-address public
```

### Test the deployment
To check the app running, follow these steps:
1. Go to [portal.azure.com](http://portal.azure.com).
2. Go to the WinterReadyACI resource group and **select ai-duet-local**
3. **Go to the Azure Container Instances IP address** using your browser when the State of it is “Running” (it could take some time)

    ![](media/Empower%20your%20web%20application%20lifecycle%20with%20containers%20and%20Azure%20using%20Azure%20Container%20Instances%20and%20Jenkins/image13.png)

4. You will see the container running.

    ![](media/Empower%20your%20web%20application%20lifecycle%20with%20containers%20and%20Azure%20using%20Azure%20Container%20Instances%20and%20Jenkins/image14.png)

## Step 2: Set up Jenkins environment
Jenkins has a web interface to manage it, but it´s not accessible from internet. To manage it, we need to create a SSH connection between our machine and Jenkins VM and a tunnel to redirect our http traffic to the Jenkins VM. We are going to use Ubuntu on Windows to do it. Also, we need some configuration before start creating the Jenkins projects in order to use GitHub, Docker and Azure CLI on them.

### First log in
To create the connection between our machine and Jenkins VM, follow these steps:
1. Jenkins doesn’t expose its services directly. An SSH tunnel to the VM is required. Open the **Git Bash** (or your SSH tool).
2. **Execute this command** (you saved the Jenkins VM Domain Name Label info in the Step 0. Create a Jenkins VM) to create the SSH connection and the HTTP tunnel. 
```
ssh -L 127.0.0.1:8080:localhost:8080 ready@[JenkinsVMDomainNameLabel]
```
3. It will ask you to accept the server’s key. Type yes and press enter. Then you’ll need to type the password that you set on the Jenkins VM creation. If you followed the lab, the password will be: **Password123$**
4. **Open your browser** and go to your Jenkins homepage: [localhost:8080](http://localhost:8080)
5. Click “**Log in**”:

    ![](media/Empower%20your%20web%20application%20lifecycle%20with%20containers%20and%20Azure%20using%20Azure%20Container%20Instances%20and%20Jenkins/image15.png)

6. You need User and Password to log in. The user is admin and you can check the password by executing this command in the current Git Bash (or your SSH tool) session:
```
sudo cat /var/lib/jenkins/secrets/initialAdminPassword
```

### Install plugins
To install the GitHub and Azure CLI plugins, follow these steps:
1. **Go to your Jenkins homepage**: [localhost:8080](http://localhost:8080)
2. Click “**Manage Jenkins**”

    ![](media/Empower%20your%20web%20application%20lifecycle%20with%20containers%20and%20Azure%20using%20Azure%20Container%20Instances%20and%20Jenkins/image16.png)

3. Click “**Manage Plugins**”

    ![](media/Empower%20your%20web%20application%20lifecycle%20with%20containers%20and%20Azure%20using%20Azure%20Container%20Instances%20and%20Jenkins/image17.png)
 
4. Click “**Available**” section

    ![](media/Empower%20your%20web%20application%20lifecycle%20with%20containers%20and%20Azure%20using%20Azure%20Container%20Instances%20and%20Jenkins/image18.png)
 
5. Search **GitHub plugin** and **Azure CLI Plugin**, and check the install checkbox.

    ![](media/Empower%20your%20web%20application%20lifecycle%20with%20containers%20and%20Azure%20using%20Azure%20Container%20Instances%20and%20Jenkins/image19.png)

    ![](media/Empower%20your%20web%20application%20lifecycle%20with%20containers%20and%20Azure%20using%20Azure%20Container%20Instances%20and%20Jenkins/image20.png)
 
 
6. Click “**Install without restart**” to install the selected plugins.

    ![](media/Empower%20your%20web%20application%20lifecycle%20with%20containers%20and%20Azure%20using%20Azure%20Container%20Instances%20and%20Jenkins/image21.png)
 
The process will start to install the plugins in your Jenkins VM. When it finishes the operation, you can use your GitHub account and Azure CLI in your projects.

### Install Docker
In order to use Docker in your projects, you will need to install the tool in the machines where you build will run. Because we only use one machine, we need to install docker on it. To install it, follow these steps:
1. Go to your current session on **Git Bash** (or your SSH tool).
2. Execute the following commands:
```
sudo apt-get update
sudo apt-get install \
    apt-transport-https \
    ca-certificates \
    curl \
    software-properties-common
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add –
sudo apt-key fingerprint 0EBFCD88
sudo add-apt-repository \
   "deb [arch=amd64] https://download.docker.com/linux/ubuntu \
   $(lsb_release -cs) \
   stable"
sudo apt-get update
sudo apt-get install docker-ce
sudo chmod 777 /run/docker.sock
```

You can find more info and installation process in [Get Docker CE for Ubuntu](https://docs.docker.com/engine/installation/linux/docker-ce/ubuntu/) in the Docker Documentation.

### Configure GitHub
You only need this section if you are using your own GitHub account. If you are using the default repository, you don´t have to follow these steps (you can´t do it). This option allows you to start a new build project each time you commit any change to GitHub.
To configure the GitHub plugin, follow these steps:
1.	GitHub needs access to VM port 8080 to trigger a webhook. To configure it, go to [portal.azure.com](http://portal.azure.com).
2.	Go to the WinterReady resource group and **select JenkinsVM-nsg**
3.	Click “**Inbound security rules**” section and “**Add**”

    ![](media/Empower%20your%20web%20application%20lifecycle%20with%20containers%20and%20Azure%20using%20Azure%20Container%20Instances%20and%20Jenkins/image22.png)
 
4.	We need the predefined rule: any source, any destination, destination port 8080. Change the **Priority** to 1000. Click “**Ok**”

    ![](media/Empower%20your%20web%20application%20lifecycle%20with%20containers%20and%20Azure%20using%20Azure%20Container%20Instances%20and%20Jenkins/image23.png)
 
5.	Jenkins needs to authenticate to GitHub using a token. To create it, go to the section to **create new personal access token** in your GitHub account: github.com/settings/tokens/new 
6.	Enter Jenkins as “**Token description**” and select **repo** and **admin:repo_hook** to give access to them. Click “**Generate token**”

    ![](media/Empower%20your%20web%20application%20lifecycle%20with%20containers%20and%20Azure%20using%20Azure%20Container%20Instances%20and%20Jenkins/image24.png)
 
7.	**Copy the token**. We will need it later.
8.	Go to your **Jenkins homepage** to configure it: [localhost:8080](http://localhost:8080)
9.	Click “**Manage Jenkins**”

    ![](media/Empower%20your%20web%20application%20lifecycle%20with%20containers%20and%20Azure%20using%20Azure%20Container%20Instances%20and%20Jenkins/image25.png)
 
10.	Click “**Configure System**”

    ![](media/Empower%20your%20web%20application%20lifecycle%20with%20containers%20and%20Azure%20using%20Azure%20Container%20Instances%20and%20Jenkins/image26.png)
 
11.	Go to the **Jenkins location** section and modify the Jenkins URL: http://[JenkinsVMDomainNameLabel]:8080/
12.	 Click ”**Apply**”

        ![](media/Empower%20your%20web%20application%20lifecycle%20with%20containers%20and%20Azure%20using%20Azure%20Container%20Instances%20and%20Jenkins/image27.png)
 
13.	Go to the **GitHub** section and click “**Add GitHub Server**”
14.	Click “**Add**” to add a new credentials and select **Jenkins**

    ![](media/Empower%20your%20web%20application%20lifecycle%20with%20containers%20and%20Azure%20using%20Azure%20Container%20Instances%20and%20Jenkins/image28.png)
 
15.	Create a **new credentials** with these values and click “**Add**”:
    - Kind: Secret text
    - Secret: your GitHub token
    - ID: GitHub

    ![](media/Empower%20your%20web%20application%20lifecycle%20with%20containers%20and%20Azure%20using%20Azure%20Container%20Instances%20and%20Jenkins/image29.png)
 
16.	Select **GitHub** as credentials and click “**Save**”

    ![](media/Empower%20your%20web%20application%20lifecycle%20with%20containers%20and%20Azure%20using%20Azure%20Container%20Instances%20and%20Jenkins/image30.png)
 
### Step 3: Create the BuildImage project
It´s time to create our first project in Jenkins. This project will download the code, create the image and store it on Azure Container Registry. It will start automatically when any change in the code is done. To create it, follow these steps:
1. Go to your Jenkins homepage: [localhost:8080](http://localhost:8080)
2. Click “**New Item**”
 
    ![](media/Empower%20your%20web%20application%20lifecycle%20with%20containers%20and%20Azure%20using%20Azure%20Container%20Instances%20and%20Jenkins/image31.png)
 
3. Enter the project name **BuildImage** and select **Freestyle project**. Click “**Ok**”
 
    ![](media/Empower%20your%20web%20application%20lifecycle%20with%20containers%20and%20Azure%20using%20Azure%20Container%20Instances%20and%20Jenkins/image32.png)
 
4. In the **project settings**, you need to configure:
    1. General: select **GitHub project**.
        - GitHub project: If you are using your own GitHub account, the Project url is `http://github.com/[yourusername]/ai-duet-tensor`. If you are using the default repository, the Project url is `https://github.com/diegomrtnzg/ai-duet-tensor`.
    2. Source Code Management: select **Git** to add your GitHub repository.
        - Git: If you are using your own GitHub account, the Repository URL is `http://github.com/[yourusername]/ai-duet-tensor.git`. If you are using the default repository, the Repository URL is `https://github.com/diegomrtnzg/ai-duet-tensor.git`.
    3. Build Triggers: select **GitHub hook trigger for GITScm polling** to trigger a new build project each time you commit any change to GitHub.
    4. Build Environment: select **Use secret text or file**.

5. On **Bindings**, click “**Add**” and select **Secret text**. We will add a secret text with the Docker Registry Server info.
 
    ![](media/Empower%20your%20web%20application%20lifecycle%20with%20containers%20and%20Azure%20using%20Azure%20Container%20Instances%20and%20Jenkins/image33.png)
 
    1. Enter dockerserver in **Variable** field
    2. Click “**Add**” to add a new credentials and select **Jenkins**
 
        ![](media/Empower%20your%20web%20application%20lifecycle%20with%20containers%20and%20Azure%20using%20Azure%20Container%20Instances%20and%20Jenkins/image34.png)
 
    3. Create **new credentials** with these values and click “**Add**”:
        - Kind: Secret text
        - Secret: your Azure Container Registry Login Server info (you saved the Azure Container Registry info in the Step 0. Create an Azure Container Registry)
        - ID: DockerRegistryServer
 
        ![](media/Empower%20your%20web%20application%20lifecycle%20with%20containers%20and%20Azure%20using%20Azure%20Container%20Instances%20and%20Jenkins/image35.png)
 
    4. Select **DockerRegistryServer** as credentials
    
        ![](media/Empower%20your%20web%20application%20lifecycle%20with%20containers%20and%20Azure%20using%20Azure%20Container%20Instances%20and%20Jenkins/image36.png)
  
6. On **Bindings**, click “**Add**” again and select **Username and password (separated)**. We will add a secret username and password with the Docker Registry username and password.
 
    ![](media/Empower%20your%20web%20application%20lifecycle%20with%20containers%20and%20Azure%20using%20Azure%20Container%20Instances%20and%20Jenkins/image37.png)
 
    1. Enter `dockeruser` in **Username Variable** field and `dockerpassword` in **Password Variable** field
    2. Click “**Add**” to add a new credentials and select **Jenkins**
 
        ![](media/Empower%20your%20web%20application%20lifecycle%20with%20containers%20and%20Azure%20using%20Azure%20Container%20Instances%20and%20Jenkins/image38.png)
 
    3. Create a **new credentials** with these values and click “**Add**”:
        - Kind: Username with Password
        - Username: your Azure Container Registry Username info (you saved the Azure Container Registry info in the Step 0. Create an Azure Container Registry)
        - Password: your Azure Container Registry Password info (you saved the Azure Container Registry info in the Step 0. Create an Azure Container Registry)
        - ID: DockerRegistryCredentials
 
        ![](media/Empower%20your%20web%20application%20lifecycle%20with%20containers%20and%20Azure%20using%20Azure%20Container%20Instances%20and%20Jenkins/image39.png)
 
    4. Select **the credentials which matches with your username**
 
        ![](media/Empower%20your%20web%20application%20lifecycle%20with%20containers%20and%20Azure%20using%20Azure%20Container%20Instances%20and%20Jenkins/image40.png)
 
7.	On Build, click “**Add build step**” and select **Execute shell**. It will add you an option to execute a command in the build execution.
 
    ![](media/Empower%20your%20web%20application%20lifecycle%20with%20containers%20and%20Azure%20using%20Azure%20Container%20Instances%20and%20Jenkins/image41.png)
 
8.	In the **Command area**, add the following instructions:
```
WEB_IMAGE_NAME="$dockerserver/ai-duet:${BUILD_NUMBER}"
WEB_IMAGE_NAME_LATEST="$dockerserver/ai-duet:latest"
docker build -t $WEB_IMAGE_NAME .
docker login $dockerserver -u $dockeruser -p $dockerpassword
docker push $WEB_IMAGE_NAME
docker tag $WEB_IMAGE_NAME $WEB_IMAGE_NAME_LATEST
docker push $WEB_IMAGE_NAME_LATEST
```
It will create two environment variables (the container image names, one with the build number tag and other one with the latest tag), build the image using the Dockerfile, login in the Container Registry, and push the two images with their tags.
9.	Click “**Save**” to finish the Project settings.

## Step 4: Create the DeployACI project
In the second Jenkins project, it will deploy a new version of the container image (stored in Azure Container Registry) to Azure Container Instances. It will start automatically when the BuildImage project build is successful.
First, we need to create an Azure Service Principal and check our Azure subscription info to authenticate Jenkins (Azure CLI plugin) to our Azure account. To create it, follow these steps:
1. Go to [portal.azure.com](http://portal.azure.com).
2. Click “>_” on the top-right part of the Azure portal.
3. **Execute this command** to create the Azure Service Principal: `az ad sp create-for-rbac --name Jenkins`. We will need the Azure Service Principal info, so copy the output of the command (appId, password and tenant).
4. Execute this command to check your Azure subscription info: `az account show`. We will need the Azure Subscription info, so copy the output of the command (id).

Once we have the Azure info, we can create the Jenkins project:
1. Go to your Jenkins homepage: localhost:8080
2. Click “**New Item**”
  
    ![](media/Empower%20your%20web%20application%20lifecycle%20with%20containers%20and%20Azure%20using%20Azure%20Container%20Instances%20and%20Jenkins/image42.png)
 
3. Enter the project name **DeployACI** and select **Freestyle project**. Click “**Ok**”
  
    ![](media/Empower%20your%20web%20application%20lifecycle%20with%20containers%20and%20Azure%20using%20Azure%20Container%20Instances%20and%20Jenkins/image43.png)
 
4. In the **project settings**, you need to configure:
    1. Build Triggers: select **Build after other projects are built** and write BuildImage as Projects to watch to start this project when BuildImage project finish its build.
    2. Build Environment: select **Use secret text or file**.
5. On **Bindings**, click “**Add**” and select **Secret text**. We will add a secret text with the Docker Registry Server info.
  
    ![](media/Empower%20your%20web%20application%20lifecycle%20with%20containers%20and%20Azure%20using%20Azure%20Container%20Instances%20and%20Jenkins/image44.png)
 
6. Enter dockerserver in **Variable** field and select **DockerRegistryServer** as credentials.
 
    ![](media/Empower%20your%20web%20application%20lifecycle%20with%20containers%20and%20Azure%20using%20Azure%20Container%20Instances%20and%20Jenkins/image45.png)
  
7. On **Bindings**, click “**Add**” again and select **Username and password (separated)**. We will add a secret username and password with the Docker Registry username and password.
  
    ![](media/Empower%20your%20web%20application%20lifecycle%20with%20containers%20and%20Azure%20using%20Azure%20Container%20Instances%20and%20Jenkins/image46.png)
 
8. Enter `dockeruser` in **Username Variable** field and `dockerpassword` in **Password Variable** field and select **the credentials which matches with your username**.
  
    ![](media/Empower%20your%20web%20application%20lifecycle%20with%20containers%20and%20Azure%20using%20Azure%20Container%20Instances%20and%20Jenkins/image47.png)
 
9. On Build, click “**Add build step**” and select **azure-cli(2.0.23)**. It will add you an option to execute an Azure CLI command in the build execution.
  
    ![](media/Empower%20your%20web%20application%20lifecycle%20with%20containers%20and%20Azure%20using%20Azure%20Container%20Instances%20and%20Jenkins/image48.png)
 
10. In the Login to Azure area, click “**Add**” to add a new Service Principal and select **Jenkins**
  
    ![](media/Empower%20your%20web%20application%20lifecycle%20with%20containers%20and%20Azure%20using%20Azure%20Container%20Instances%20and%20Jenkins/image49.png)
 
11.	Create a **new credentials** with these values and click “**Add**”:
    - Kind: Microsoft Azure Service Principal
    - Subscription ID: the id of your Azure subscription info (you saved the Azure subscription info in the Step 4. Create the DeployACI project).
    - Cliente ID: the appId of your Azure Service Principal (you saved the Azure Service Principal info in the Step 4. Create the DeployACI project).
    - Cliente Secret: the password of your Azure Service Principal
    - Tenant ID:  the tenant of your Azure Service Principal
    - ID: Azure
  
    ![](media/Empower%20your%20web%20application%20lifecycle%20with%20containers%20and%20Azure%20using%20Azure%20Container%20Instances%20and%20Jenkins/image50.png)
 
12.	Select **Azure** as Service Principal and click “**Save**”
  
    ![](media/Empower%20your%20web%20application%20lifecycle%20with%20containers%20and%20Azure%20using%20Azure%20Container%20Instances%20and%20Jenkins/image51.png)
 
13.	In the **Command area**, add the following instructions:
```
az container create -g WinterReadyACI --name ai-duet --image ${dockerserver}/ai-duet:latest --cpu 1 --memory 1.5 --registry-login-server ${dockerserver} --registry-username ${dockeruser} --registry-password ${dockerpassword} --ip-address public
```
It will create an Azure Container Instances named ai-duet in the Azure Resource Group WinterReadyACI using the image built (the latest tag version) in the Step 3 using 1 CPU, 1.5 Memory and a public IP address.

14.	Click “**Save**” to finish the Project settings.

## Step 5: Test the CI/CD pipeline
The configuration it´s done. It's time to test the new CI/CD pipeline. 
If you are using your own GitHub account, the easiest way to test it is to update the source code and commit the changes into GitHub repository. To do it, follow these steps:
1. **Go to your GitHub project**: github.com/[yourusername]/ai-duet-tensor
2. **Create a new file**.
   
    ![](media/Empower%20your%20web%20application%20lifecycle%20with%20containers%20and%20Azure%20using%20Azure%20Container%20Instances%20and%20Jenkins/image52.png)
 
3. Enter a file name and file body and click “**Commit new file**”
    
    ![](media/Empower%20your%20web%20application%20lifecycle%20with%20containers%20and%20Azure%20using%20Azure%20Container%20Instances%20and%20Jenkins/image53.png)
   
    ![](media/Empower%20your%20web%20application%20lifecycle%20with%20containers%20and%20Azure%20using%20Azure%20Container%20Instances%20and%20Jenkins/image54.png)
 
If you are using the default repository, the easiest way to test it is to trigger manually the build process. To do it, follow these steps:
1.	Go to your BuildImage project in Jenkins: http://localhost:8080/job/BuildImage/
2.	Click **Build Now**
   
    ![](media/Empower%20your%20web%20application%20lifecycle%20with%20containers%20and%20Azure%20using%20Azure%20Container%20Instances%20and%20Jenkins/image55.png)
 
A few seconds after you push the code or manually trigger the build process, you will see a new build running in Jenkins. Once completed successfully, a new version of the app will be deployed to Azure Container Instances.
To check the app running, follow these steps:
1.	Go to [portal.azure.com](http://portal.azure.com).
2.	Go to the WinterReadyACI resource group and **select ai-duet**
3.	**Go to the Azure Container Instances IP address** using your browser when the State of it is “Running”
  
    ![](media/Empower%20your%20web%20application%20lifecycle%20with%20containers%20and%20Azure%20using%20Azure%20Container%20Instances%20and%20Jenkins/image56.png)
 
4.	You will see the container running.  

    ![](media/Empower%20your%20web%20application%20lifecycle%20with%20containers%20and%20Azure%20using%20Azure%20Container%20Instances%20and%20Jenkins/image57.png)
 
## Next steps
This Instructor Led Lab is an introduction to DevOps with containers and Jenkins using Azure Container Instances and Azure Container Registry, but there are lot of options to customize the DevOps process. These are some ideas using Jenkins:
- [Deploy to Web App for Containers](https://docs.microsoft.com/en-us/azure/jenkins/java-deploy-webapp-tutorial)
- [Deploy to Azure Container Service (Kubernetes)](https://docs.microsoft.com/en-us/azure/container-service/kubernetes/container-service-kubernetes-jenkins?toc=%2Fen-us%2Fazure%2Fjenkins%2Ftoc.json&bc=%2Fen-us%2Fazure%2Fbread%2Ftoc.json)
- [Use Azure Container Instances as Build Agent](https://docs.microsoft.com/en-us/azure/jenkins/azure-container-agents-plugin-run-container-as-an-agent)

More information about Azure Container Instances in the [Azure Container Instances Documentation](https://docs.microsoft.com/en-us/azure/container-instances/).
More information about Azure Container Registry in the [Azure Container Registry Documentation](https://docs.microsoft.com/en-us/azure/container-registry/).
More info about Docker in the [Docker Documentation](https://docs.docker.com/).
More information about CI/CD with Jenkins in the [Jenkins User Documentation](https://jenkins.io/doc/).

