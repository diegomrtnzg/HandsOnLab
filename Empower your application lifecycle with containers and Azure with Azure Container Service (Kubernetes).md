# Hand On Lab: Empower your application lifecycle with containers and Azure with Azure Container Service (Kubernetes)

## Requirements
Microsoft Azure subscription

## Introduction
Nowadays, one of the biggest challenges when developing modern applications for the cloud is being able to deliver these applications continuously. 

In this Hands on Lab, you will learn how to implement a full continuous integration and deployment (CI/CD) pipeline using:
-   Azure Container Service with Kubernetes: your own infraestructure to run your containers.
-   Azure Container Registry: your own private container repository to store your containers.
-   Visual Studio Team Services: the service to run your lifecycle application tasks.

This lab is based on a multi-container web application, available on [GitHub](https://github.com/Azure-Samples/azure-voting-app-redis), developed with Python / Flask and the data component is using Redis.

![](media/Empower%20your%20application%20lifecycle%20with%20containers%20and%20Azure%20with%20Azure%20Container%20Service%20(Kubernetes)/image38.PNG)

### Containers in Azure
The process to build a container is:
1. Pull the base container image to your dev environment
2. Customize / create your custom image using the base container image
3. Push your customize container to your registry
4. Deploy the container to your environments

![](media/Empower%20your%20application%20lifecycle%20with%20containers%20and%20Azure%20with%20Azure%20Container%20Service%20(Kubernetes)/Slide6.PNG)

In Azure, we have several servicies to use containers:
- [Azure Container Registry](https://azure.microsoft.com/en-us/services/container-registry/): **Docker private registry** in the cloud.
- [IaaS](https://azure.microsoft.com/en-us/overview/what-is-iaas/): **create your own container environment** using IaaS services: Virtual Machines, VM Scale Set, Azure Batch...
- [Azure Container Service](https://azure.microsoft.com/en-us/services/container-service/): **create your container environment in the easiest way** using Docker Swarm, DC/OS or Kubernetes orchestrator. The new Azure Container Service (AKS) is a managed Kubernetes environment.
[Azure Service Fabric](https://azure.microsoft.com/en-us/services/service-fabric/): microservices orquestator which allows you to use containers. The only platform with **GA for Windows containers**.
- [Azure Container Instance](https://azure.microsoft.com/en-us/services/container-instances/) *preview*: **Container as a Service**: run containers with a single command.
- [Web App for Containers](https://azure.microsoft.com/en-us/services/app-service/containers/): **PaaS service to run containerized web app**.

![](media/Empower%20your%20application%20lifecycle%20with%20containers%20and%20Azure%20with%20Azure%20Container%20Service%20(Kubernetes)/Slide7.PNG)

### Step by step
The objective is to deliver this application continuously in a Kubernetes cluster using Visual Studio Team Services. This is a brief explanation of the steps:
1.  Code changes are committed to the source code repository
2.  Code repository triggers a build in Visual Studio Team Services
3.  Visual Studio Team Services gets the latest version of the sources and builds all the images that make up the application
4.  Visual Studio Team Services pushes each image to a Docker registry created using the Azure Container Registry service
5.  Visual Studio Team Services triggers a new release
6.  The release runs deploy and update kubectl commands on the Azure Container Service cluster master node
7.  Kubernetes on the cluster pulls the latest version of the images
8.  The new version of the application is deployed

![](media/Empower%20your%20application%20lifecycle%20with%20containers%20and%20Azure%20with%20Azure%20Container%20Service%20(Kubernetes)/Slide17.PNG)

## Step 0: Prerequisites
Before starting this lab, you need to complete the following tasks:

### Create a Kubernetes cluster in Azure Container Service
> **Update**: Microsoft recently launched [Azure Container Service (AKS)](https://docs.microsoft.com/en-us/azure/aks/), a new version of Azure Container Service with Kubernetes as a managed service, so you don´t need container orchestration expertise to maintenance tasks. If you want to try it, you can create your Azure Container Service (AKS) cluster instead of the regular ACS cluster. [Instructions at the end of the Hands on Lab](#updates).

To create a new Kubernetes cluster in Azure Container Service, follow these steps:
1. Go to [Azure Portal](https://portal.azure.com/)
2. Open the Cloud Shell and select Bash (Linux)

    ![](media/Empower%20your%20application%20lifecycle%20with%20containers%20and%20Azure%20with%20Azure%20Container%20Service%20(Kubernetes)/image2.PNG)

3. Type this command to create a new resource group:
```
az group create --name myACSGroup --location westeurope
```
4. Type this command to create a new ACS Kubernetes cluster and generate new SSH keys(it will take a while):
> **Note**: if you are using a trial subscription, add `--agent-count 1` at the end of the command to only create one agent node
```
az acs create --orchestrator-type kubernetes --resource-group myACSGroup --name myK8sCluster --generate-ssh-keys
```
5. Type this command to download the credentials you will need to connect via kubectl (the Kubernetes CLI):
```
az acs kubernetes get-credentials --resource-group=myACSGroup --name=myK8sCluster
```
6. Copy and save the content of the credentials file, we will need it later:
```
cd .kube
cat config
```
> **Note**: copy the empty lines at the beginning and the end of the file.

### Create an Azure Container Registry
To create a new Azure Container Registry, follow these steps:
1. Go to [Azure Portal](https://portal.azure.com/)
2. Open the Cloud Shell and select Bash (Linux)
   
    ![](media/Empower%20your%20application%20lifecycle%20with%20containers%20and%20Azure%20with%20Azure%20Container%20Service%20(Kubernetes)/image2.PNG)

3. Type this command to create a new Azure Container Registry:
> **Note**: replace [random] with any random string.
```
az acr create --name myContainerRegistry[random] --resource-group myACSGroup --sku Basic --admin-enabled true
```
It will show you some info about the Azure Container Registry. Copy and save the loginServer URL, we will need it later: *XXXX.azurecr.io*

### Create a Visual Studio Team Services account and team project
You need to create a Visual Studio Team Services account with the same email address which has a Microsoft Azure subscription associated. You could follow [these steps](https://www.visualstudio.com/en-us/docs/setup-admin/team-services/sign-up-for-visual-studio-team-services) to create the account and the team project.

## Step 1: Configure your Visual Studio Team Services account
In this section, you can configure your Visual Studio Team Services account. To configure VSTS Services Endpoints, in your Visual Studio Team Services project, click the **Settings** icon in the toolbar, and select **Services**.

![](media/Empower%20your%20application%20lifecycle%20with%20containers%20and%20Azure%20with%20Azure%20Container%20Service%20(Kubernetes)/image3.PNG)

### Connect Visual Studio Team Services and Azure account
Set up a connection between your VSTS project and your Azure account.
1.  On the left, click **New Service Endpoint** \> **Azure Resource Manager**.
2.  To authorize VSTS to work with your Azure account, select your  **Subscription** and click **OK**.
    
    ![](media/Empower%20your%20application%20lifecycle%20with%20containers%20and%20Azure%20with%20Azure%20Container%20Service%20(Kubernetes)/image6.1.png)

### Connect VSTS to your Azure Container Service cluster
The last step before getting into the CI/CD pipeline is to configure external connections to your Kubernetes cluster in Azure.
1.  Add an endpoint of type **Kubernetes**. Then enter the Kubernetes connection information of your Kubernetes cluster (master node). You saved it in **Step 0: Prerequisites - Create a Kubernetes cluster in Azure Container Service - Step 6**
    - **Connection name**: custom name to identify the cluster
    - **Server URL**: you can find it in the copied text with the name **server**: https://XXXX.cloudapp.azure.com
    - **Kubeconfig**: the whole copied text (with the empty lines at the beginning and the end of the file)

    ![](media/Empower%20your%20application%20lifecycle%20with%20containers%20and%20Azure%20with%20Azure%20Container%20Service%20(Kubernetes)/image4.PNG)

All the configuration is done. In the next steps, you can create the CI/CD pipeline that builds and deploys the application to the Kubernetes cluster.

## Step 2: Import the code
In this step, you can import the GitHub project to your VSTS project and use it in the next steps.
1.  To import the GitHub code, click **Code**, the **name of your project repository** (*MSReady* on the screen) and **Import repository**.
    
    ![](media/Empower%20your%20application%20lifecycle%20with%20containers%20and%20Azure%20with%20Azure%20Container%20Service%20(Kubernetes)/image5.PNG)

4.  Copy the GitHub project URL to import it and click **Import**. Clone URL: `https://github.com/Azure-Samples/azure-voting-app-redis`

    ![](media/Empower%20your%20application%20lifecycle%20with%20containers%20and%20Azure%20with%20Azure%20Container%20Service%20(Kubernetes)/image6.PNG)

3.  When VSTS finishes to import the code, you will see the the project:

    ![](media/Empower%20your%20application%20lifecycle%20with%20containers%20and%20Azure%20with%20Azure%20Container%20Service%20(Kubernetes)/image7.PNG)

4.  Edit the file *azure-vote-all-in-one-redis.yml* to update the container image on the azure-vote-front deployment to use our own image. The original yml file uses the Microsoft image (and different tag). Delete the content of the file and paste this:
> **Note**: this file is the kubernetes deployment manifest. We describe on it the objects we would like to create: azure-vote-back deployment (deployment of the azure-vote-back container and its configuration), azure-vote-back service (expose internally the azure-vote-back deployment), azure-vote-front deployment (deployment of the azure-vote-front container and its configuration) and azure-vote-front service (expose externally the azure-vote-front deployment). More info about deployments in the [Kubernetes documentation](#next-steps)

![](media/Empower%20your%20application%20lifecycle%20with%20containers%20and%20Azure%20with%20Azure%20Container%20Service%20(Kubernetes)/image9.PNG)

```
apiVersion: apps/v1beta1
kind: Deployment
metadata:
  name: azure-vote-back
spec:
  replicas: 1
  template:
    metadata:
      labels:
        app: azure-vote-back
    spec:
      containers:
      - name: azure-vote-back
        image: redis
        ports:
        - containerPort: 6379
          name: redis
---
apiVersion: v1
kind: Service
metadata:
  name: azure-vote-back
spec:
  ports:
  - port: 6379
  selector:
    app: azure-vote-back
---
apiVersion: apps/v1beta1
kind: Deployment
metadata:
  name: azure-vote-front
spec:
  replicas: 1
  strategy:
    rollingUpdate:
      maxSurge: 1
      maxUnavailable: 1
  minReadySeconds: 5 
  template:
    metadata:
      labels:
        app: azure-vote-front
    spec:
      containers:
      - name: azure-vote-front
        image: RegistryURL/azure-vote-front:Tag
        ports:
        - containerPort: 80
        resources:
          requests:
            cpu: 250m
          limits:
            cpu: 500m
        env:
        - name: REDIS
          value: "azure-vote-back"
      imagePullSecrets: 
        - name: azurecontainerreg
---
apiVersion: v1
kind: Service
metadata:
  name: azure-vote-front
spec:
  type: LoadBalancer
  ports:
  - port: 80
  selector:
    app: azure-vote-front
```
6. Click **Commit** to save the modification.
    
    ![](media/Empower%20your%20application%20lifecycle%20with%20containers%20and%20Azure%20with%20Azure%20Container%20Service%20(Kubernetes)/image10.PNG)

## Step 3: Create the build definition
In this step, you can set up a build definition for your VSTS project and define the build workflow for your container images

### Initial definition setup
1.  To create a build definition, connect to your Visual Studio Team Services project and click **Build & Release**. In the **Build definitions** section, click **+ New**.
    
    ![](media/Empower%20your%20application%20lifecycle%20with%20containers%20and%20Azure%20with%20Azure%20Container%20Service%20(Kubernetes)/image11.PNG)

2.  Select the **Empty process**.

    ![](media/Empower%20your%20application%20lifecycle%20with%20containers%20and%20Azure%20with%20Azure%20Container%20Service%20(Kubernetes)/image12.PNG)

3.  Then, click the **Variables** tab and create a new variable: **RegistryURL**. Paste the values of your Registry URL.  You saved it in **Step 0: Prerequisites - Create an Azure Container Registry**

> **Note**: the URL format is dnsname.azurecr.io, without http:// or https://

![](media/Empower%20your%20application%20lifecycle%20with%20containers%20and%20Azure%20with%20Azure%20Container%20Service%20(Kubernetes)/image13.PNG)

4.  On the **Build Definitions** page, open the **Triggers** tab and configure the build to use continuous integration with the *azure-voting-app-redis* project that you created in the Step 2. Make sure that you select *master* as the **Branch specification**.

    ![](media/Empower%20your%20application%20lifecycle%20with%20containers%20and%20Azure%20with%20Azure%20Container%20Service%20(Kubernetes)/image14.PNG)

5.  Finally, click the **Tasks** tab and configure the Agent queue to **Hosted Linux Preview**.

    ![](media/Empower%20your%20application%20lifecycle%20with%20containers%20and%20Azure%20with%20Azure%20Container%20Service%20(Kubernetes)/image15.PNG)

### Define the build workflow
The next steps define the build workflow. First, you need to configure the source of the code. To do it, select **This project** and your **repository** and **branch** (*master*, you will find it in *All branches*).

![](media/Empower%20your%20application%20lifecycle%20with%20containers%20and%20Azure%20with%20Azure%20Container%20Service%20(Kubernetes)/image16.PNG)

There is a container image (frontend) to build for the *Azure Voting App* (the backend container is prebuilt). This image is built using the Dockerfile located in the *azure-vote*:

You need two Docker steps for create the image, one to build the image, and one to push the image into the Azure Container Registry.
1.  To add a step in the build workflow, click **+ Add build step** and select **Docker**.

    ![](media/Empower%20your%20application%20lifecycle%20with%20containers%20and%20Azure%20with%20Azure%20Container%20Service%20(Kubernetes)/image17.PNG)

2.  Configure one step that uses the `docker build` command.

    ![](media/Empower%20your%20application%20lifecycle%20with%20containers%20and%20Azure%20with%20Azure%20Container%20Service%20(Kubernetes)/image18.PNG)

    For the build operation, select your Azure Container Registry, the **Build an image** action, the Dockerfile that defines each image, define the **Image Name**, and select **Include Latest Tag**. The Image Name has to be `$(RegistryURL)/azure-vote-front:$(Build.BuildId)`. 

3.  Configure a second step that uses the docker push command by adding a second Docker task.

    ![](media/Empower%20your%20application%20lifecycle%20with%20containers%20and%20Azure%20with%20Azure%20Container%20Service%20(Kubernetes)/image19.PNG)

    For the push operation, select your Azure container registry, the **Push an image** action, enter the **Image Name** that is built in the previous step and select **Include Latest Tag**. The Image Name has to be the same: `$(RegistryURL)/azure-vote-front:$(Build.BuildId)`. 

4.  After you configure the build and push steps for the image, add three more steps in the build workflow.

    ![](media/Empower%20your%20application%20lifecycle%20with%20containers%20and%20Azure%20with%20Azure%20Container%20Service%20(Kubernetes)/image20.PNG)

    a.   A command-line task that uses a bash script to replace the *RegistryURL* occurrence in the azure-vote-all-in-one-redis.yml file with the *RegistryURL* variable.
        `-c "sed -i 's/RegistryURL/$(RegistryURL)/g' azure-vote-all-in-one-redis.yml"`

    ![](media/Empower%20your%20application%20lifecycle%20with%20containers%20and%20Azure%20with%20Azure%20Container%20Service%20(Kubernetes)/image21.PNG)

    b.  A command-line task that uses a bash script to replace the *Tag* occurrence in the azure-vote-all-in-one-redis.yml file with the *BuildId* variable.
        `-c "sed -i 's/Tag/$(Build.BuildId)/g' azure-vote-all-in-one-redis.yml"`

    c.  A task that drops the updated YML file as a build artifact so it can be used in the release. See the following screen for details.

    ![](media/Empower%20your%20application%20lifecycle%20with%20containers%20and%20Azure%20with%20Azure%20Container%20Service%20(Kubernetes)/image22.PNG)

    ![](media/Empower%20your%20application%20lifecycle%20with%20containers%20and%20Azure%20with%20Azure%20Container%20Service%20(Kubernetes)/image23.PNG)

### Test the build
To test the build, you need to:
1.  Click **Save & queue** to test your build definition.

    ![](media/Empower%20your%20application%20lifecycle%20with%20containers%20and%20Azure%20with%20Azure%20Container%20Service%20(Kubernetes)/image24.PNG)

2.  Click **Queue.**

    ![](media/Empower%20your%20application%20lifecycle%20with%20containers%20and%20Azure%20with%20Azure%20Container%20Service%20(Kubernetes)/image25.PNG)

3.  You can check the build status by click on **#XX** in the green bar. If the **Build** is correct, you have to see a similar screen:

    ![](media/Empower%20your%20application%20lifecycle%20with%20containers%20and%20Azure%20with%20Azure%20Container%20Service%20(Kubernetes)/image26.PNG)

## Step 4: Create the release definition
Visual Studio Team Services allows you to [manage releases across environments](https://www.visualstudio.com/team-services/release-management/). You can enable continuous deployment to make sure that your application is deployed on your different environments (such as dev, test, pre-production, and production) in a smooth way. You can create an environment that represents your Azure Container Service (Kubernetes) cluster.

### Initial release setup
1.  To create a release definition, click **Releases** \> **+ New definition**

    ![](media/Empower%20your%20application%20lifecycle%20with%20containers%20and%20Azure%20with%20Azure%20Container%20Service%20(Kubernetes)/image27.PNG)

2.  Select the **Empty process**

    ![](media/Empower%20your%20application%20lifecycle%20with%20containers%20and%20Azure%20with%20Azure%20Container%20Service%20(Kubernetes)/image28.PNG)

3.  To configure the artifact source, Click **+ Add artifact**. Link this new release definition to the build that you defined in the previous step. After that, the *azure-vote-all-in-one-redis.yml* file will be available in the release process.

    ![](media/Empower%20your%20application%20lifecycle%20with%20containers%20and%20Azure%20with%20Azure%20Container%20Service%20(Kubernetes)/image29.PNG)

    ![](media/Empower%20your%20application%20lifecycle%20with%20containers%20and%20Azure%20with%20Azure%20Container%20Service%20(Kubernetes)/image30.PNG)

3.  To configure the release trigger, click the **thunderbolt** and enable the continuous deployment trigger. Set the trigger to use the kubernetes branch. This setting ensures that a new release starts when the build completes successfully.

    ![](media/Empower%20your%20application%20lifecycle%20with%20containers%20and%20Azure%20with%20Azure%20Container%20Service%20(Kubernetes)/image31.PNG)

### Define the release workflow
The release workflow is composed of one task that you need to add.
1.  Add a new **Deploy to Kubernetes** task.

    ![](media/Empower%20your%20application%20lifecycle%20with%20containers%20and%20Azure%20with%20Azure%20Container%20Service%20(Kubernetes)/image32.PNG)

2.  Configure it with these values (use your own Kubernetes Service Connection, Azure subscription, Azure Container Registry names and the *azure-vote-all-in-one-redis.yml* configuration file):

    ![](media/Empower%20your%20application%20lifecycle%20with%20containers%20and%20Azure%20with%20Azure%20Container%20Service%20(Kubernetes)/image33.PNG)

### Test the release
It´s time to test the release on the Azure Container Service:
1.  Click **Save** to save the release definition.

    ![](media/Empower%20your%20application%20lifecycle%20with%20containers%20and%20Azure%20with%20Azure%20Container%20Service%20(Kubernetes)/image34.PNG)

2.  Click **Release**, **Create Release** and **Create** to start the release.

    ![](media/Empower%20your%20application%20lifecycle%20with%20containers%20and%20Azure%20with%20Azure%20Container%20Service%20(Kubernetes)/image35.PNG)

    ![](media/Empower%20your%20application%20lifecycle%20with%20containers%20and%20Azure%20with%20Azure%20Container%20Service%20(Kubernetes)/image36.PNG)

4.  You can check the release status by click on **Release-X** in the green bar. If the **Release** is correct, the release status will show you a similar screen:

    ![](media/Empower%20your%20application%20lifecycle%20with%20containers%20and%20Azure%20with%20Azure%20Container%20Service%20(Kubernetes)/image37.PNG)

## Step 5: Test the CI/CD pipeline and access to the application
Now that you are done with the configuration, it's time to test this new CI/CD pipeline. The easiest way to test it is to update the source code and commit the changes into your Visual Studio Team Services repository. A few seconds after you push the code, you will see a new build running in Visual Studio Team Services. Once completed successfully, a new release is triggered and deployed the new version of the application on the Azure Container Service cluster.

### Access the application
To access the application, follow these steps:
1. Go to [Azure Portal](https://portal.azure.com/)
2. Open the Cloud Shell and select Bash (Linux)
    
    ![](media/Empower%20your%20application%20lifecycle%20with%20containers%20and%20Azure%20with%20Azure%20Container%20Service%20(Kubernetes)/image2.PNG)

3. Type this command to list your ACS cluster IPs (master node and services):
```
az network public-ip list -g myACSGroup --query "[].[dnsSettings.domainNameLabel, ipAddress]" --out table
```
4. Look for an IP without label (the IP with name XXXXmgmt is the master node), copy the ipAddress and access it via your web browser. You will see your home page of your app running on ACS on Microsoft Azure.

> **Note**: the public IP assignment process can take up to 5 minutes from the end of the release because Kubernetes has to request an IP to Azure and Azure has to create a new one.

![](media/Empower%20your%20application%20lifecycle%20with%20containers%20and%20Azure%20with%20Azure%20Container%20Service/image38.PNG)

### Modify the application
Now it´s time to modify our application and test the automatic pipeline. To do it, follow these steps:
1. Go to the **Code** section in VSTS:
2. **Edit** the *azure-vote/azure-vote/config_file.cfg* file with your own **TITLE**, **VOTE1VALUE** and **VOTE2VALUE**
3. **Commit** the changes
4. **Access your Web Application IP address** (from previous step) to access your modified web app.

> **Note**: the public IP assignment process can take up to 5 minutes from the commit because the build and release proccess have to be completed.

## Next steps
-   For more information about CI/CD with Visual Studio Team Services, see the [VSTS Build overview](https://www.visualstudio.com/docs/build/overview).
-   For more information about Azure Container Service, see the [Azure Container Service Documentation](https://docs.microsoft.com/en-us/azure/container-service/).
-   For more information about Azure Container Service (AKS), see the [Azure Container Service (AKS) Documentation](https://docs.microsoft.com/en-us/azure/aks/).
-   For more information about Kubernetes, see the [Kubernetes documentation](https://kubernetes.io/docs/home/).

## Updates
#### [NEW] Create a Kubernetes cluster in Azure Container Service (AKS)
Azure Container Service (AKS) is a new managed Kubernetes service. 

To create a new Kubernetes cluster in Azure Container Service (AKS), follow these steps:
1. Go to [Azure Portal](https://portal.azure.com/)
2. Open the Cloud Shell and select Bash (Linux)
    
    ![](media/Empower%20your%20application%20lifecycle%20with%20containers%20and%20Azure%20with%20Azure%20Container%20Service%20(Kubernetes)/image2.PNG)

3. Type this command to register the AKS provider:
```
az provider register -n Microsoft.ContainerService
```
4. Type this command to create a new resource group:
```
az group create --name myACSGroup --location westeurope
```
4. Type this command to create a new ACS Kubernetes cluster and generate new SSH keys(it will take a while):
> **Note**: if you are using a trial subscription, add `--agent-count 1` at the end of the command to only create one agent node
```
az aks create --resource-group myACSGroup --name myK8sCluster --generate-ssh-keys
```
6. Type this command to download the credentials to connect via kubectl (the Kubernetes CLI):
```
az aks kubernetes get-credentials --resource-group=myACSGroup --name=myK8sCluster
```
7. Copy and save the content of the credentials file, we will need it later:
```
cd .kube
cat config
```
> **Note**: copy the empty lines at the beginning and the end of the file.
