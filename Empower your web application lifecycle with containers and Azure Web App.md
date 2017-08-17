# Hand On Lab: Empower your web application lifecycle with containers and Azure Web App

## Requirements
Microsoft Azure subscription

## Step by step
Nowadays, one of the biggest challenges when developing modern applications for the cloud is being able to deliver these applications continuously. In this lab, you will learn how to implement a full continuous integration and deployment (CI/CD) pipeline using:
-   Azure Web App
-   Azure Container Registry
-   Visual Studio Team Services

This lab is based on ai-duet sample in Tensorflow Magenta project. [More info](https://github.com/prashanthmadi/ai-duet-tensor)

![](media/Empower%20your%20web%20application%20lifecycle%20with%20containers%20and%20Azure%20Web%20App/image1.png)

The objective is to deliver this application continuously in a Web App, using Visual Studio Team Services. The following figure details this continuous delivery pipeline:

![](media/cb8ee85e8b9c1e12b0e3d10219cc5c7b.png)

Here is a brief explanation of the steps:
1.  Code changes are committed to the source code repository (Visual Studio Team Services)
2.  Visual Studio Team Services triggers a new build
3.  Visual Studio Team Services gets the latest version of the sources and builds the application image
4.  Visual Studio Team Services pushes each image to a Docker registry created using the Azure Container Registry service
5.  Visual Studio Team Services triggers a new release
6.  The release connects to the Web App and pulls the latest version of the image
7.  The new version of the application is deployed

## Step 0: Prerequisites
Before starting this lab, you need to complete the following tasks:

### Create an Azure Web App running on Linux
To create a new Azure web app running on Linux, follow [this tutorial](https://docs.microsoft.com/en-us/azure/app-service-web/app-service-linux-how-to-create-web-app). You can select any Runtime Stack in the “Create blade”.

Once you have created the app, you can continue with the next prerequisite.

### Create an Azure Container Registry
To create a new Azure Container Registry, you need to follow [these steps](https://docs.microsoft.com/en-us/azure/container-registry/container-registry-get-started-portal). Enable admin users to authenticate in the next steps.

You will need some info of the Azure Container Registry in the next steps:
1.  Username: you can find this info in the “Access keys” blade with the name “Username”
2.  Password: you can find this info in the “Access keys” blade with the name “Password”
3.  Registry: you can find this info in the “Access keys” blade with the name “Login server” (you need to add https:// at the beginning of the “Login server” if it isn´t ecognized as a URL).

![](media/f704153ab5d226177217490b86967ab1.png)

### Create a Visual Studio Team Services account and team project
You need to create a Visual Studio Team Services (VSTS) account with the same  mail address which has a Microsoft Azure subscription associated. You could follow [these steps](https://www.visualstudio.com/en-us/docs/setup-admin/team-services/sign-up-for-visual-studio-team-services) to create the account and the team project..

## Step 1: Import the project and connect VSTS with Microsoft Azure
In this step, we will import the code project to our code repository (Visual Studio Team Services). This will allow us to work with a copy of the original project and modify if we need it.

### Import the code
You can import the GitHub project to your VSTS project and use it in the next steps.
1.  To import the GitHub code, click **Code**, the **name of your project repository** (*HoL – Web App* in the screen) and **Import repository**.

    ![](media/81d6c004cb13b41f6e29b51c04b9fefd.png)

2.  Copy the GitHub project URL to import it and click **Import**.     Clone URL: `https://github.com/prashanthmadi/ai-duet-tensor`

    ![](./media/image5.png)

3.  When VSTS finishes to import the code, you will see the the project:

    ![](./media/image6.png)

### Connect VSTS with Microsoft Azure
To use your Microsoft Azure account and resources in Visual Studio Team Services, you need to add it as a New Service. To do it, click on the **gear** located on top of your VSTS website. The services tab will prompt letting you configure your Endpoint. Click **New Service Endpoint -\> Azure Resource Manager.**

![](./media/image7.png)

Here you can select one from the list of your available subscriptions. Write a **name**, for example Microsoft Azure, and **select your Azure subscription**. Click **OK** to add the ARM Service Endpoint.

 ![](./media/image8.png)

## Step 2: Create the build definition
Visual Studio Team Services provides a [flexible and multiplatform build system](https://www.visualstudio.com/team-services/continuous-integration/) that allows us to create a powerful Continuous Integration (CI) pipeline in a simplest and efficient way. In this step of the journey we are going to create a build definition that will be triggered every time a new commit is pushed to the repository.

### Initial definition setup
1.  Select the **Build & Release** hub in your Visual Studio Team Services project, and then the **Builds** **Tab**. Click **+New** to create a new definition.

    ![](media/d1443d8b953ee7a027c6550ee3f2726d.png)

2.  Start with an **empty process**.

    ![](media/0f1ba93e217be4220c983d43ff35d517.png)

3.  Click **Process** and type a name. For example, HoL – Web Apps  

    ![](media/69f70f7947a697d768b172f5b3812b95.png)

4.  We have to define a variable that we will be using during the build process. Click **Variable** and click **Add** to add a new variable named “RegistryURL”.  

    ![](media/36f5743d480f0c1d9ea2ff83b2a25400.png)

>    **Note**: This variable will be the Container Registry URL in which we are publishing our container. To get that variable navigate in the Azure Portal to the Azure Container Registry which you created in Step 0.  

    ![](media/36f5743d480f0c1d9ea2ff83b2a25400.png)

5.  Now we need to specify when this build will run. Click **Triggers** and **enable the Continuous Integration** trigger. Doing so the build will run with every new commit pushed to the specified branch in Branch specification: master.  
    ![](media/056419ee9af60410a0f601ba83cd9d9e.png)

### Define the build workflow
The next steps define the build workflow. First, we need to define the Default agent queue where our build will run. In this project, the agent is Hosted Linux Preview.

![](media/12448e398bcfbb3baabb31c6c7752350.png)

Now, you need to configure the source of the code. To do it, select **This project** (Visual Studio Team Services) and your **repository** and **branch** (master).

![](media/9fa58d8a1b559886d1222ade759cbb92.png)

Next steps will build and push our container image to our repository. To do it, you need to add these tasks:

1.  To add the first step in the build workflow, click **+ Add Task** and select **Docker** (add two Docker tasks).

    ![](media/114632c3d1047659fceea86ae63fec0f.png)

2.  For the image, configure one step that uses the `docker build` command.

    ![](./media/image18.png)

    For the build operation, select your **Azure Container Registry** (first you need to select your Azure subscription), the **Build an image** action, and use the predefined  **Dockerfile**. Define the **Image Name** and select **Include Latest Tag**. The Image Name need to be: `$(RegistryURL)/aiduet:$(Build.BuildId)`.
3.  For the image, configure a second step that uses the docker push command by adding a second Docker task.

    ![](./media/image19.png)

    For the push operation, select your **Azure Container Registry** (first you need to select your Azure subscription), the **Push an image** action, enter the **Image Name** `$(RegistryURL)/aiduet:$(Build.BuildId)` and select **Include Latest Ta**g.

### Test the build
To test the build, you need to:
1.  Click **Save & queue** to test your build definition.  

    ![](media/2cb783ba43d1dc03a2fd422406c0a248.png)

2.  Click **Queue.**    

    ![](media/a22f3a2171d5cc04011c5e8c7e57a666.png)

3.  If the **Build** is correct, you have to see a similar screen:

    ![](./media/image22.png)

## Step 3: Create the release definition
Visual Studio Team Services allows you to [manage releases across environments](https://www.visualstudio.com/team-services/release-management/). You can enable continuous deployment to make sure that your application is deployed on your different environments (such as dev, test, pre-production, and production) in a smooth way. You can create an environment that represents your Azure Web App.

### Initial release setup
1.  To create a release definition, click **Releases** \> **+ Release**

    ![](./media/image23.png)

    ![](./media/image24.png)

2.  Select the Azure **App Service Deployment** template

    ![](media/5903aaf309b68d8d262c4791b9b579e8.png)

3.  Use the Build configured in the **Step 2** as artifact, and select
    **Continuous deployment**

    ![](media/af22c5c91ce417250f2700158d72b744.png)

4.  Click **Create** button to create the release

### Define the release workflow
The release workflow is composed of one task added with the Azure App Service Deployment template.
1.  Configure the task with this fields:
    -   Azure Subscription: select the Azure subscription with your Web App  
    -   App Service Name: select your Web App
    -   Registry: add the variable `$(docker.registry)`. This variable will contain the value of your registry URL.
    -   Repository: add your image name defined in the build: `aiduet`
    -   Tag: use the predefined tag: `$(Build.BuildId)`

    ![](media/be90cebc4a90f18ab97fe5bb689f71ef.png)

2.  Configure the **Application Settings** of the Web App with these values (format: name – value):
    -   `DOCKER_REGISTRY_SERVER_URL - https://$(docker.registry)`
    -   `DOCKER_REGISTRY_SERVER_USERNAME - $(docker.username)`
    -   `DOCKER_REGISTRY_SERVER_PASSWORD - $(docker.password)`

    ![](media/631cee1e468c1d5f3887ee0fb468d560.png)

>   **Note**: these App settings will change your Web App (Linux) App settings on Microsoft Azure when you release a new version.

![](media/c3d5533ccbbf9900f1da6e4a9be4ee51.png)

3.  To configure the release variables, click **Variables** and select **+Variable** to create three new variables with the info of the registry: **docker.username**, **docker.password**, and **docker.registry**. Use the values of the Azure Container Registry which you created in Step 0.

    ![](media/c36274839702d431c2ccc84304c213ce.png)

>   **IMPORTANT**: As shown on the previous screen, click the **Lock** checkbox in docker.password. This setting is important to restrict the password view.

### Test the release
It´s time to test the release on the Web App (Linux) and check if the it shows the app:
1.  Click **Save**, **Release** and **Create Release** to create a new release.

![](media/0f9412ebec93d59899db8319fb46ec8d.png)

2.  Click **Create** to start the release

![](media/fa324331625f92af8fe4f66fc1aba4c6.png)

3.  To check the status of the release, double click the **blue button**.

![](media/db32e843a13c5de92160e710fb36f828.png)

4.  If the **Release** is correct, you will see a similar screen to the below:

![](media/2ec17564cd6237944325f8483936b0ba.png)

5.  To see the Web App running, go to your **Web App** on the [Azure Portal](https://portal.azure.com/) and access the **URL**.

![](media/0a4bf9693cbbc16f9da38b7e5265ca74.png)

6.  You will see your home page of your web app running on Microsoft Azure.

![](media/93dfaf309db970dde8f0ef4847f82061.png)

## Step 4: Test the CI/CD pipeline
Now that you are done with the configuration, it's time to test this new CI/CD pipeline. The easiest way to test it is to update the source code and commit the changes into your Visual Studio Team Services repository. A few seconds after you push the code, you will see a new build running in Visual Studio Team Services. Once completed successfully, a new release will be triggered and it will deploy the new version of the application on the Azure Web App.

## Next steps
-   For more information about CI/CD with Visual Studio Team Services, see the [VSTS Build overview](https://www.visualstudio.com/docs/build/overview).
-   For more information about Azure Web App, see the [Web Apps Documentation](https://docs.microsoft.com/en-us/azure/app-service-web/).
-   For more information about Azure Container Registry, see the [Azure Container Registry Documentation](https://docs.microsoft.com/en-us/azure/container-registry/).
