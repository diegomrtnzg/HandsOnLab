# Hand On Lab: Empower your application lifecycle with containers and Azure with Azure Container Service

*This lab is based on* [Full CI/CD pipeline to deploy a multi-container application on Azure Container Service with Docker Swarm using Visual Studio Team Services](https://github.com/Microsoft/azure-docs/blob/master/articles/container-service/container-service-docker-swarm-setup-ci-cd.md) *documentation.*

## Requirements
Microsoft Azure subscription

## Step by step
Nowadays, one of the biggest challenges when developing modern applications for the cloud is being able to deliver these applications continuously. In this article, you will learn how to implement a full continuous integration and deployment (CI/CD) pipeline using:
-   Azure Container Service Engine with Docker Swarm Mode
-   Azure Container Registry
-   Visual Studio Team Services

This lab is based on a simple application, available on [GitHub](https://github.com/jcorioland/MyShop/tree/docker-linux), developed with ASP.NET Core. The application is composed of four different services: three web APIs and one web front end.

![](media/Empower%20your%20application%20lifecycle%20with%20containers%20and%20Azure%20with%20Azure%20Container%20Service/image1.png)

The objective is to deliver this application continuously in a Docker Swarm Mode cluster, using Visual Studio Team Services. Here is a brief explanation of the steps:
1.  Code changes are committed to the source code repository
2.  Code repository triggers a build in Visual Studio Team Services
3.  Visual Studio Team Services gets the latest version of the sources and builds all the images that make up the application
4.  Visual Studio Team Services pushes each image to a Docker registry created using the Azure Container Registry service
5.  Visual Studio Team Services triggers a new release
6.  The release runs some commands using SSH on the Azure container service cluster master node
7.  Docker Swarm Mode on the cluster pulls the latest version of the images
8.  The new version of the application is deployed using Docker Stack

## Step 0: Prerequisites
Before starting this lab, you need to complete the following tasks:

### Create a Swarm Mode cluster in Azure Container Service with ACS Engine
To create a new Docker Swarm mode cluster in Azure Container Service using ACS Engine, follow these steps:
1.  [Go to this link](https://github.com/Azure/azure-quickstart-templates/tree/master/101-acsengine-swarmmode)
2.  Click on "Deploy to Azure" button

    ![](media/Empower%20your%20application%20lifecycle%20with%20containers%20and%20Azure%20with%20Azure%20Container%20Service/image2.png)

3.  Log in with your Microsoft Azure credentials
4.  Fill the fields to create the cluster:
    1.  Subscription: your subscription of Microsoft Azure
    2.  Resource group: we recommend creating a new resource group which contain all the cluster resources
    3.  Location: West US 2 is the recommended location
    4.  Agent count: you have to select 1 node
    5.  Agent Endpoint DNS: fill with a unique name like "clusterdiego739agent". You can use random numbers to complete the unique name
    6.  Agent Subnet: the subnet for the agent nodes
    7.  Agent VM Size: you could use Standard_D2_v2
    8.  First Consecutive Static IP: first IP for master node
    9.  Linux Admin Username: username for master nodes
    10. Location: you could use westus2
    11. Master Endpoint DNS: fill with a unique name like "clusterdiego739master". You can use random numbers to complete the unique name
    12. Master VM Size: you could use Standard_D2_v2
    13. Name Suffix: it´s ok the predefined value
    14. SSH Key: complete with your public SSH key with this format: ssh-rsa XXXX...XXXX…. More info in the note section after the step 6.
    15. Target Environment: it´s ok the predefined value
5.  Accept the terms and conditions
6.  Click on "Purchase" button

> **Note**: if you don´t have an SSH Key you could create a new key (e.g. using [PuTTYgen](http://www.putty.org/)]) or use [the example key](resources/Empower%20your%20application%20lifecycle%20with%20containers%20and%20Azure%20with%20Azure%20Container%20Service/SSH%20Key). The format ofthe keys must be:
```
Public SSH Key format

ssh-rsa [publickey] rsa-key-[dateyyyymmdd]

Private SSH Key format

\-----BEGIN RSA PRIVATE KEY-----

[privatekey]

\-----END RSA PRIVATE KEY-----
```

The deployment will start to create and in a few minutes you will have it available. You can continue the lab in the meanwhile the ACS cluster is created.

You need to save your private SSH key because you will need it on Step 1. You also will need other data from the cluster in the next steps:
1.  User: is the field Linux Admin Username
2.  Host name: you could find it in a resource named swarmm-agent-ip-…

![](media/Empower%20your%20application%20lifecycle%20with%20containers%20and%20Azure%20with%20Azure%20Container%20Service/image3.png)

> **Note**: The Docker Swarm orchestrator in Azure Container Service uses legacy standalone Swarm. Currently, the integrated [Swarm mode](https://docs.docker.com/engine/swarm/) (in Docker 1.12 and higher) is not a supported orchestrator in Azure Container Service. For this reason, we are using [ACS
Engine](https://github.com/Azure/acs-engine/blob/master/docs/swarmmode.md), a community-contributed [quickstart template](https://azure.microsoft.com/resources/templates/101-acsengine-swarmmode/), or a Docker solution in the [Azure Marketplace](https://azuremarketplace.microsoft.com/).

### Create an Azure Container Registry
To create a new Azure Container Registry, you need to follow [these steps](https://docs.microsoft.com/en-us/azure/container-registry/container-registry-get-started-portal). Enable admin users in order for authentication.

You will need some info of the Azure Container Registry in the next steps:
1.  Username: you can find this info in the “Access keys” blade with the name “Username”
2.  Password: you can find this info in the “Access keys” blade with the name “Password”
3.  Registry: you can find this info in the “Access keys” blade with the name “Login server” (you need to add https:// at the beginning of the “Login server” if it isn´t recognized as a URL).

![](media/Empower%20your%20application%20lifecycle%20with%20containers%20and%20Azure%20with%20Azure%20Container%20Service/image4.png)

### Create a Visual Studio Team Services account and team project
You need to create a Visual Studio Team Services account with the same email address which has a Microsoft Azure subscription associated. You could follow [these steps](https://www.visualstudio.com/en-us/docs/setup-admin/team-services/sign-up-for-visual-studio-team-services) to create the account and the team project.

## Step 1: Configure your Visual Studio Team Services account
In this section, you can configure your Visual Studio Team Services account. To configure VSTS Services Endpoints, in your Visual Studio Team Services project, click the **Settings** icon in the toolbar, and select **Services**.

![](media/Empower%20your%20application%20lifecycle%20with%20containers%20and%20Azure%20with%20Azure%20Container%20Service/image5.png)

### Connect Visual Studio Team Services and Azure account
Set up a connection between your VSTS project and your Azure account.
1.  On the left, click **New Service Endpoint** \> **Azure Resource Manager**.
2.  To authorize VSTS to work with your Azure account, select your     **Subscription** and click **OK**.
    
    ![](media/Empower%20your%20application%20lifecycle%20with%20containers%20and%20Azure%20with%20Azure%20Container%20Service/image6.png)

### Connect VSTS to your Azure Container Service cluster
The last step before getting into the CI/CD pipeline is to configure external connections to your Docker Swarm cluster in Azure.
1.  For the Docker Swarm cluster, add an endpoint of type **SSH**. Then enter     the SSH connection information of your Swarm cluster (master node).
    
    ![](media/Empower%20your%20application%20lifecycle%20with%20containers%20and%20Azure%20with%20Azure%20Container%20Service/image7.png)

> **Note**: you need the data from Step 0: Prerrequisites, Create a Swarm Mode cluster in Azure Container Service with ACS Engine.

All the configuration is done. In the next steps, you can create the CI/CD pipeline that builds and deploys the application to the Docker Swarm cluster.

## Step 2: Import the code
In this step, you can import the GitHub project to your VSTS project and use it in the next steps.
1.  To import the GitHub code, click **Code**, the **name of your project     repository** (*HoL – ACS* in the screen) and **Import repository**.
    
    ![](media/Empower%20your%20application%20lifecycle%20with%20containers%20and%20Azure%20with%20Azure%20Container%20Service/image8.png)

1.  Copy the GitHub project URL to import it and click **Import**. Clone URL: `https://github.com/jcorioland/MyShop`

    ![](media/Empower%20your%20application%20lifecycle%20with%20containers%20and%20Azure%20with%20Azure%20Container%20Service/image9.png)

2.  When VSTS finishes to import the code, you will see the the project:

    ![](media/Empower%20your%20application%20lifecycle%20with%20containers%20and%20Azure%20with%20Azure%20Container%20Service/image10.png)

## Step 3: Create the build definition
In this step, you can set up a build definition for your VSTS project and define the build workflow for your container images

### Initial definition setup
1.  To create a build definition, connect to your Visual Studio Team Services project and click **Build & Release**. In the **Build definitions** section, click **+ New**.
    
    ![](media/Empower%20your%20application%20lifecycle%20with%20containers%20and%20Azure%20with%20Azure%20Container%20Service/image11.png)

2.  Select the **Empty process**.

    ![](media/Empower%20your%20application%20lifecycle%20with%20containers%20and%20Azure%20with%20Azure%20Container%20Service/image12.png)

3.  Then, click the **Variables** tab and create two new variables: **RegistryURL** and **AgentURL**. Paste the values of your Registry and Cluster Agents DNS.

    ![](media/Empower%20your%20application%20lifecycle%20with%20containers%20and%20Azure%20with%20Azure%20Container%20Service/image13.png)

> **Note**: you need the data from Step 0: Prerrequisites, Create a Swarm Mode cluster in Azure Container Service with ACS Engine and Create an Azure Container Registry.

4.  On the **Build Definitions** page, open the **Triggers** tab and configure the build to use continuous integration with the fork of the MyShop project that you created in the Step 2. Make sure that you select *docker-linux* as the **Branch specification**.

    ![](media/Empower%20your%20application%20lifecycle%20with%20containers%20and%20Azure%20with%20Azure%20Container%20Service/image14.png)

5.  Finally, click the **Tasks** tab and configure the Default agent queue to **Hosted Linux Preview**.

    ![](media/Empower%20your%20application%20lifecycle%20with%20containers%20and%20Azure%20with%20Azure%20Container%20Service/image15.png)

### Define the build workflow
The next steps define the build workflow. First, you need to configure the source of the code. To do it, select **This project** and your **repository** and **branch** (docker-linux, you will find it in *All branches*).

![](media/Empower%20your%20application%20lifecycle%20with%20containers%20and%20Azure%20with%20Azure%20Container%20Service/image16.png)

There are five container images to build for the *MyShop* application. Each image is built using the Dockerfile located in the project folders:
1.  ProductsApi
2.  Proxy
3.  RatingsApi
4.  RecommandationsApi
5.  ShopFront

You need two Docker steps for each image, one to build the image, and one to push the image into the Azure container registry.
1.  To add a step in the build workflow, click **+ Add build step** and select **Docker**.

    ![](media/Empower%20your%20application%20lifecycle%20with%20containers%20and%20Azure%20with%20Azure%20Container%20Service/image17.png)

2.  For each image, configure one step that uses the `docker build` command.

    ![](media/Empower%20your%20application%20lifecycle%20with%20containers%20and%20Azure%20with%20Azure%20Container%20Service/image18.png)

    For the build operation, select your Azure Container Registry, the **Build an image** action, and the Dockerfile that defines each image. Set the **Working Directory** as the Dockerfile root directory, define the **Image Name**, and select **Include Latest Tag**.

    The Image Name has to be in this format: `$(RegistryURL)/[NAME]:$(Build.BuildId)`. Replace **[NAME]** with the image name:
    -   `proxy`
    -   `products-api`
    -   `ratings-api`
    -   `recommendations-api`
    -   `shopfront`

3.  For each image, configure a second step that uses the docker push command by
    adding a second Docker task.

    ![](media/Empower%20your%20application%20lifecycle%20with%20containers%20and%20Azure%20with%20Azure%20Container%20Service/image19.png)

    For the push operation, select your Azure container registry, the **Push an image** action, enter the **Image Name** that is built in the previous step and select **Include Latest Tag**.

4.  After you configure the build and push steps for each of the five images,
    add three more steps in the build workflow.

    ![](media/Empower%20your%20application%20lifecycle%20with%20containers%20and%20Azure%20with%20Azure%20Container%20Service/image20.png)

    a.   A command-line task that uses a bash script to replace the *RegistryURL* occurrence in the docker-compose.yml file with the RegistryURL variable.
        `-c "sed -i 's/RegistryUrl/$(RegistryURL)/g' src/docker-compose-v3.yml"`

    ![](media/Empower%20your%20application%20lifecycle%20with%20containers%20and%20Azure%20with%20Azure%20Container%20Service/image21.png)

    b.  A command-line task that uses a bash script to replace the *AgentURL* occurrence in the docker-compose.yml file with the AgentURL variable.
        `-c "sed -i 's/AgentUrl/$(AgentURL)/g' src/docker-compose-v3.yml"`

    c.  A task that drops the updated Compose file as a build artifact so it can be used in the release. See the following screen for details.

    ![](media/Empower%20your%20application%20lifecycle%20with%20containers%20and%20Azure%20with%20Azure%20Container%20Service/image22.png)

    ![](media/Empower%20your%20application%20lifecycle%20with%20containers%20and%20Azure%20with%20Azure%20Container%20Service/image23.png)

### Test the build
To test the build, you need to:
1.  Click **Save & queue** to test your build definition.

    ![](media/Empower%20your%20application%20lifecycle%20with%20containers%20and%20Azure%20with%20Azure%20Container%20Service/image24.png)

2.  Click **Queue.**

    ![](media/Empower%20your%20application%20lifecycle%20with%20containers%20and%20Azure%20with%20Azure%20Container%20Service/image25.png)

3.  If the **Build** is correct, you have to see a similar screen:

    ![](media/Empower%20your%20application%20lifecycle%20with%20containers%20and%20Azure%20with%20Azure%20Container%20Service/image26.png)

## Step 4: Create the release definition
Visual Studio Team Services allows you to [manage releases across environments](https://www.visualstudio.com/team-services/release-management/). You can enable continuous deployment to make sure that your application is deployed on your different environments (such as dev, test, pre-production, and production) in a smooth way. You can create an environment that represents your Azure Container Service Docker Swarm Mode cluster.

![](media/Empower%20your%20application%20lifecycle%20with%20containers%20and%20Azure%20with%20Azure%20Container%20Service/image27.png)

### Initial release setup
1.  To create a release definition, click **Releases** \> **+ Release**
2.  To configure the artifact source, Click **Artifacts** \> **Link an artifact  source**. Here, link this new release definition to the build that you defined in the previous step. After that, the docker-compose.yml file is available in the release process.

    ![](media/Empower%20your%20application%20lifecycle%20with%20containers%20and%20Azure%20with%20Azure%20Container%20Service/image28.png)

3.  To configure the release trigger, click **Triggers** and select **Continuous Deployment**. Set the trigger on the same artifact source. This setting ensures that a new release starts when the build completes successfully.

    ![](media/Empower%20your%20application%20lifecycle%20with%20containers%20and%20Azure%20with%20Azure%20Container%20Service/image29.png)

4.  To configure the release variables, click **Variables** and select **+Variable** to create three new variables with the info of the registry: **docker.username**, **docker.password**, and **docker.registry**. Paste the values of the Azure Container Registry which you created in Step 0, Create an Azure Container Registry.

    ![](media/Empower%20your%20application%20lifecycle%20with%20containers%20and%20Azure%20with%20Azure%20Container%20Service/image30.png)

>   **IMPORTANT**: As shown on the previous screen, click the **Lock** checkbox in docker.password. This setting is important to restrict the view of the password.

### Define the release workflow
The release workflow is composed of two tasks that you need to add.
1.  Configure a task to securely copy the compose file to a *deploy* folder on the Docker Swarm master node, using the SSH connection you configured previously. See the following screen for details.
    Source folder: `$(System.DefaultWorkingDirectory)/MyShop-CI/drop`

    ![](media/Empower%20your%20application%20lifecycle%20with%20containers%20and%20Azure%20with%20Azure%20Container%20Service/image31.png)

2.  Configure a second task to execute a bash command to run docker and docker stack deploy commands on the master node. See the following screen for details.
    `docker login -u $(docker.username) -p $(docker.password) $(docker.registry) && export DOCKER_HOST=:2375 && cd deploy && docker stack deploy --compose-file docker-compose-v3.yml myshop --with-registry-auth`

    ![](media/Empower%20your%20application%20lifecycle%20with%20containers%20and%20Azure%20with%20Azure%20Container%20Service/image32.png)

    The command executed on the master uses the Docker CLI to do the following tasks:
    -   Log in to the Azure container registry (it uses three build variables that are defined in the **Variables** tab)
    -   Define the **DOCKER_HOST** variable to work with the Swarm endpoint (:2375).
    -   Navigate to the *deploy* folder that was created by the preceding secure copy task where it contains the docker-compose.yml file.
    -   Execute docker stack deploy commands that pulls the new images and creates the containers.

>   **IMPORTANT**: As shown on the previous screen, leave the **Fail on STDERR** checkbox unchecked. This setting allows us to complete the release process due to docker-compose prints several diagnostic messages, such as containers are stopping or being deleted, on the standard error output. If you check the checkbox, Visual Studio Team Services will reports that errors occurred during the release, even if all goes well.

### Test the release
It´s time to test the release on the Azure Container Service and check if it shows the app:
1.  Click **Save**, **Release** and **Create Release** to create a new release.

    ![](media/Empower%20your%20application%20lifecycle%20with%20containers%20and%20Azure%20with%20Azure%20Container%20Service/image33.png)

2.  Click **Create** to start the release

    ![](media/Empower%20your%20application%20lifecycle%20with%20containers%20and%20Azure%20with%20Azure%20Container%20Service/image34.png)

3.  To check the status of the release, double click the **blue button**.

    ![](media/Empower%20your%20application%20lifecycle%20with%20containers%20and%20Azure%20with%20Azure%20Container%20Service/image35.png)

4.  If the **Release** is correct, you will see a similar screen to the below:

    ![](media/Empower%20your%20application%20lifecycle%20with%20containers%20and%20Azure%20with%20Azure%20Container%20Service/image36.png)

5.  To see the app running, go to **Agent public IP address** resource on the [Azure Portal](https://portal.azure.com/) (swarm-agent-ip-…. resource) and access the **DNS name**.

    ![](media/Empower%20your%20application%20lifecycle%20with%20containers%20and%20Azure%20with%20Azure%20Container%20Service/image37.png)

6.  You will see your home page of your app running on ACS on Microsoft Azure.

    ![](media/Empower%20your%20application%20lifecycle%20with%20containers%20and%20Azure%20with%20Azure%20Container%20Service/image38.png)

## Step 5: Test the CI/CD pipeline
Now that you are done with the configuration, it's time to test this new CI/CD pipeline. The easiest way to test it is to update the source code and commit the changes into your Visual Studio Team Services repository. A few seconds after you push the code, you will see a new build running in Visual Studio Team Services. Once completed successfully, a new release is triggered and deployed the new version of the application on the Azure Container Service cluster.

## Next steps
-   For more information about CI/CD with Visual Studio Team Services, see the [VSTS Build overview](https://www.visualstudio.com/docs/build/overview).
-   For more information about ACS Engine, see the [ACS Engine GitHub repo](https://github.com/Azure/acs-engine).
-   For more information about Docker Swarm mode, see the [Docker Swarm mode overview](https://docs.docker.com/engine/swarm/).