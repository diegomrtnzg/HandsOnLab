# Containers and Azure App Service
Containers are not only being used by developers and operations teams to make it
easier the deployment and management of their workloads. We have also included
containers not only in our Azure Container Service as we have seen in the
previous exercise but also in other Platform as a Service like Azure App Service
or Service Fabric.

In this specific exercise, we are going to pay special attention on how
containers are integrated into the Azure App Service for Linux. You can deploy
your applications directly to the base container images published by Microsoft
inside the service or configure your own container.

## Step by step
### Creating a web app with a built-in container 
1.  **Open Edge** or your favorite browser and open the Azure Portal present in
    the following URL: <http://portal.azure.com>

1.  From the top left corner of the web page, select the *New* option. If you
    don’t see it you would need to click the “*hamburger”* menu to open the
    blade or click directly in the green plus sign.

    ![](media/Containers%20on%20Azure%20in%20a%20practical%20way/Containers%20and%20Azure%20App%20Service/image1.PNG)

1.  In the search box introduce *Web App On Linux* and click *Intro* in your
    keyboard.

    ![](media/Containers%20on%20Azure%20in%20a%20practical%20way/Containers%20and%20Azure%20App%20Service/image2.PNG)

1.  The search results would appear. Select the option that matches the text
    introduce in the step before. A new blade would open and you would need to
    click the *Create* button to configure the virtual machine.

    ![](media/Containers%20on%20Azure%20in%20a%20practical%20way/Containers%20and%20Azure%20App%20Service/image3.PNG)

1.  A unique name should be introduced in the *App Name* text box. If the name
    introduced is already being used a red icon would appear. You can try with
    *iil-linux-web-xxx* (where *XXX* should be a random number like 387)

2.  *Subscription, Resource Group* and *App Service plan* can be kept with the
    default values provided by the platform.

    ![](media/Containers%20on%20Azure%20in%20a%20practical%20way/Containers%20and%20Azure%20App%20Service/image4.PNG)

1.  The last step is *Configure container*. In this exercise, we would use a
    built-in container made available from Microsoft through the service. There
    are four stacks available to choose from: a. Node.js

    1.  PHP
    2.  .Net Core
    3.  Ruby

    ![](media/Containers%20on%20Azure%20in%20a%20practical%20way/Containers%20and%20Azure%20App%20Service/image5.PNG)

1.  Choose the default option *Nodejs 4.5* and select *OK* and *Create*

2.  After a few seconds, your web app would be deployed. You can access it
    through the following URL where you need to replace *xxx* with your
    previously chosen numbers.

    <http://iil-linux-web-xxx.azurewebsites.net/`>

    ![](media/Containers%20on%20Azure%20in%20a%20practical%20way/Containers%20and%20Azure%20App%20Service/image6.PNG)

### Creating a web app with a Docker Hub container 

In this case we would use an example application created by Damien Bod. It’s a
simple TO-DO list based on ASP.Net Core on Linux and Angular. You can find more
details in this article.

1.  **Open Edge** or your favorite browser and open the Azure Portal present in
    the following URL: <http://portal.azure.com>

1.  From the top left corner of the web page, select the *New* option. If you
    don’t see it you would need to click the “*hamburger”* menu to open the
    blade or click directly in the green plus sign.

    ![](media/Containers%20on%20Azure%20in%20a%20practical%20way/Containers%20and%20Azure%20App%20Service/image7.PNG)

1.  In the search box introduce *Web App On Linux* and click *Intro* in your
    keyboard.

    ![](media/Containers%20on%20Azure%20in%20a%20practical%20way/Containers%20and%20Azure%20App%20Service/image8.PNG)

1.  The search results would appear. Select the option that matches the text
    introduce in the step before. A new blade would open and you would need to
    click the *Create* button to configure the virtual machine.

    ![](media/Containers%20on%20Azure%20in%20a%20practical%20way/Containers%20and%20Azure%20App%20Service/image9.PNG)

1.  A unique name should be introduced in the *App Name* text box. If the name
    introduced is already being used a red icon would appear. You can try with
    *iil-linux-web-xxx* (where *XXX* should be a random number like 783)

2.  *Subscription, Resource Group* and *App Service plan* can be kept with the
    default values provided by the platform.

    ![](media/Containers%20on%20Azure%20in%20a%20practical%20way/Containers%20and%20Azure%20App%20Service/image10.PNG)

1.  The last step is *Configure container*. In this exercise, we would use a
    Docker Hub container. Containers repositories, the place where containers
    are stored can be public or private. In this example, we would use a *Public
    Repository.*

2.  The only required parameter is *Image and optional tag*. We would introduce
    the following container *damienbod/aspnetcorethingsclient*

    ![](media/Containers%20on%20Azure%20in%20a%20practical%20way/Containers%20and%20Azure%20App%20Service/image11.PNG)

1.  Select *OK* and *Create*. We don’t need to specify a *Startup File* to have
    our example application running.

2.  After a few seconds, your web app would be deployed. You can access it
    through the following URL where you need to replace *xxx* with your
    previously chosen numbers.

    <http://iil-linux-web-xxx.azurewebsites.net/>

    ![](media/Containers%20on%20Azure%20in%20a%20practical%20way/Containers%20and%20Azure%20App%20Service/image12.PNG)

### Creating a web app with container from a Private Container Registry 

In this case we would use an example NodeJS application developed with SailsJS,
clone the GitHub repository using Git, build the image, and push it to Azure
Container Registry.

1.  **Open Edge** or your favorite browser and open the Azure Portal present in
    the following URL: <http://portal.azure.com>

1.  From the top left corner of the web page, select the *New* option. If you
    don’t see it you would need to click the “*hamburger”* menu to open the
    blade or click directly in the green plus sign.

    ![](media/Containers%20on%20Azure%20in%20a%20practical%20way/Containers%20and%20Azure%20App%20Service/image13.PNG)

1.  In the search box introduce *Azure Container Registry* and click *Intro* in
    your keyboard.

    ![](media/Containers%20on%20Azure%20in%20a%20practical%20way/Containers%20and%20Azure%20App%20Service/image14.PNG)

1.  The search results would appear. Select the option that matches the text
    introduced in the step before. A new blade would open and you would need to
    click the *Create* button to configure the container registry.

    ![](media/Containers%20on%20Azure%20in%20a%20practical%20way/Containers%20and%20Azure%20App%20Service/image15.PNG)

1.  A unique name should be introduced in the *Registry name* text box. If the
    name introduced is already being used a red icon would appear. You can try
    with *illregistryxxx* (where *XXX* should be a random number like 387)

2.  *Location*: choose the datacenter closer to your actual location to obtain
    better performance and reduced latency.

3.  Enable the *Admin user*. This will allow us to push a container using the
    docker tools.

4.  *Subscription* and *Storage Account* can be kept with the default values
    provided by the platform.

    ![](media/Containers%20on%20Azure%20in%20a%20practical%20way/Containers%20and%20Azure%20App%20Service/image16.PNG)

1.  Click **Create.**

2.  Once the Azure Container Registry is created we would navigate to the
    resource to get the registry data. In the search box on the top of the
    portal look for the Registry **illregistryxxx** where **xxx** is the random
    number you chose previously. Select the Container Registry.

    ![](media/Containers%20on%20Azure%20in%20a%20practical%20way/Containers%20and%20Azure%20App%20Service/image17.PNG)

1.  Click **Access Keys**. In this section, we can see the Registry Name
    (*illregistryxxx)*, Login Server (*illregistryxxx.azurecr.io)*, Username
    (*illregistryxxx)* and the Password. Copy or Write down this password as we
    would need it in a further step.

    ![](media/Containers%20on%20Azure%20in%20a%20practical%20way/Containers%20and%20Azure%20App%20Service/image18.PNG)

1.  Now we would use the Linux Machine with the docker environment. You can use the machine from these exercises:
    -   [Discover what is a container - Linux](Containers%20on%20Azure%20in%20a%20practical%20way%20-%20Discover%20what%20is%20a%20container%20-%20Linux.md)
    -   [Deploying an application to a container orchestrator](Containers%20on%20Azure%20in%20a%20practical%20way%20-%20Deploying%20an%20application%20to%20a%20container%20orchestrator.md)

    Connect to one of them and continue with the next step.

2.  Execute the following command to clone the repository that contains our
    application

    ```git clone https://github.com/cjaliaga/sails-docker-msready.git```

    ![](media/Containers%20on%20Azure%20in%20a%20practical%20way/Containers%20and%20Azure%20App%20Service/image21.PNG)

1.  Execute the following command to navigate to the just cloned repository’s
    folder

    ```cd sails-docker-msready```

    ![](media/Containers%20on%20Azure%20in%20a%20practical%20way/Containers%20and%20Azure%20App%20Service/image22.PNG)

1.  Now we would log in into the *Azure Container Registry* that we created
    before using the data that we copied in the step number 10. Execute the
    following command where **xxx** is the random number you chose previously

    ```sudo docker login illregistryxxx.azurecr.io```

    ![](media/Containers%20on%20Azure%20in%20a%20practical%20way/Containers%20and%20Azure%20App%20Service/image23.PNG)

1.  Once we have logged in into the Azure Container Registry we would build our
    image tagging it according to the registry where we would upload it. The tag
    would be *illregistryxxx.azurecr.io/illlab* where **xxx** is the random
    number you chose when the Container Registry was created. Execute the following command to build the image. Ensure that you write the final dot to specify the current folder.

    ```sudo docker build -t “illregistryxxx.azurecr.io/ill-lab” .```

    ![](media/Containers%20on%20Azure%20in%20a%20practical%20way/Containers%20and%20Azure%20App%20Service/image24.PNG)

1.  Wait until the image is being built. It could take a couple of minutes. Once
    the process ends you would see *“Successfully built”*

    ![](media/Containers%20on%20Azure%20in%20a%20practical%20way/Containers%20and%20Azure%20App%20Service/image25.PNG)

1.  Now that image is built we would push it into our Azure Container Registry specifying the image’s tag that we would push. The tag it’s the one specified in the step number 15:
*illregistryxxx.azurecr.io/ill-lab*. Execute the following command to push
the image

    ```sudo docker push illregistryxxx.azurecr.io/ill-lab```

    ![](media/Containers%20on%20Azure%20in%20a%20practical%20way/Containers%20and%20Azure%20App%20Service/image26.PNG)

1.  **Open Edge** or your favorite browser and open the Azure Portal present in
    the following URL: <http://portal.azure.com>

1.  From the top left corner of the web page, select the *New* option. If you
    don’t see it you would need to click the “*hamburger”* menu to open the
    blade or click directly in the green plus sign.

    ![](media/Containers%20on%20Azure%20in%20a%20practical%20way/Containers%20and%20Azure%20App%20Service/image27.PNG)

1.  In the search box introduce *Web App On Linux* and click *Intro* in your
    keyboard.

    ![](media/Containers%20on%20Azure%20in%20a%20practical%20way/Containers%20and%20Azure%20App%20Service/image28.PNG)

1.  The search results would appear. Select the option that matches the text
    introduce in the step before. A new blade would open and you would need to
    click the *Create* button to configure the Web App.

    ![](media/Containers%20on%20Azure%20in%20a%20practical%20way/Containers%20and%20Azure%20App%20Service/image29.PNG)

1.  A unique name should be introduced in the *App Name* text box. If the name
    introduced is already being used a red icon would appear. You can try with
    *iil-linux-web-xxx* (where *XXX* should be a random number like 783)

2.  *Subscription, Resource Group* and *App Service plan* can be kept with the
    default values provided by the platform.

    ![](media/Containers%20on%20Azure%20in%20a%20practical%20way/Containers%20and%20Azure%20App%20Service/image30.PNG)

1.  The last step is *Configure container*. In this exercise, we would use
    *Azure Container Registry (ACR)* as Private Registry. Select *Private
    Registry.*

2.  The parameters that we need to fill in this step are:

    1.  *Image and optional tag:* The image that we pushed before. Since we are
        using a Private Container the image name contains the Registry where
        it’s uploaded. It would be *illregistryxxx.azurecr.io/ill-lab:latest*
        where **xxx** is the random number you chose when the Container Registry
        was created.

    2.  *Server URL:* Our ACR Login Server. It would be
        https://illregistryxxx.azurecr.io where **xxx** is the random number you
        chose when the Container Registry was created.

    3.  *Login Username:* the ACR Username. It would be **illregistryxxx** where
        **xxx** is the random number you chose when the Container Registry was
        created.

    4.  *Password:* The password we copied from the Azure Container Registry -\>
        Access Keys section.

    ![](media/Containers%20on%20Azure%20in%20a%20practical%20way/Containers%20and%20Azure%20App%20Service/image31.PNG)

1.  Select *OK* and *Create*. We don’t need to specify a *Startup File* to have
    our example application running.

2.  After a few seconds, your web app would be deployed. You can access it
    through the following URL where you need to replace *xxx* with your
    previously chosen numbers.

    <http://iil-linux-web-xxx.azurewebsites.net/>

    ![](media/Containers%20on%20Azure%20in%20a%20practical%20way/Containers%20and%20Azure%20App%20Service/image32.PNG)
<<<<<<< HEAD

## Next steps
[SSH support for Azure Web App on Linux](https://docs.microsoft.com/en-us/azure/app-service-web/app-service-linux-ssh-support)
[Continuous deployment with Azure Web App on Linux](https://docs.microsoft.com/en-us/azure/app-service-web/app-service-linux-ci-cd)
=======
>>>>>>> b9fc60fae92e3c21bb8340a969a77b3bdf059814
