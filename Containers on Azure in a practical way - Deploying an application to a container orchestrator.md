# Deploying an application to a container orchestrator

## Requirements
SSH connection tool:

-   Windows: [PuTTY](http://www.putty.org/)

## Step by step
In a production ready scenario, a single host
application is something not recommendable. Our application would lack of high
availability and a limited scalability to the compute and memory available on
that single host.

A multi host deployment would be the recommended solution. However, new
challenges raise when working in that kind of scenarios. Container orchestrator
solutions came to help us to manage multi host environments going from a few
numbers of nodes to several hundred or thousand for really big deployments.

There is not yet a clear winner in the container orchestrator battle. Docker
launched their own clustering and orchestrator solution called Docker Swarm.
However, there are other orchestrators like Kubernetes, an internal Google
solution donated to the Cloud Native Computing Foundation for automating
deployment, scaling and management of containerized applications, or DCOS, a
solution based on the popular Apache Mesos project.

Microsoft has not chosen one orchestrator over the others like AWS or Google
Cloud; in our case, we have decided to have a neutral approach and make
Azure the best platform to run any of them. Azure Container Service is our
product that ”*makes it simpler for you to create, configure, and manage a
cluster of virtual machines that are preconfigured to run containerized
applications. It uses an optimized configuration of popular open-source
scheduling and orchestration tools. This enables you to use your existing
skills, or draw upon a large and growing body of community expertise, to
deploy and manage container-based applications on Microsoft Azure.*

*Azure Container Service leverages the Docker container format to ensure
that your application containers are fully portable. It also supports your
choice of Marathon and DC/OS, Docker Swarm, or Kubernetes so that you can
scale these applications to thousands of containers, or even tens of
thousands.*

*By using Azure Container Service, you can take advantage of the
enterprise-grade features of Azure, while still maintaining application
portability--including portability at the orchestration layers.”*  [Documentation](https://docs.microsoft.com/en-us/azure/container-service/container-service-intro)

## Deploying DC/OS with Azure Container Service 
1.  **Open Edge** or your favorite browser and open the Azure Portal present in
    the following URL: http://portal.azure.com

2.  From the top left corner of the web page, select the *New* option. If you
    don’t see it you would need to click the “*hamburger”* menu to open the
    blade or click directly in the green plus sign.

    ![](media/Containers%20on%20Azure%20in%20a%20practical%20way/Discover%20what%20is%20a%20container/image30.PNG)

3.  In the search box introduce *Azure Container Service* and click *Intro* in
    your keyboard.

    ![](media/Containers%20on%20Azure%20in%20a%20practical%20way/Discover%20what%20is%20a%20container/image31.PNG)

4.  The search results would appear. Select the option that matches the text
    introduce in the step before. A new blade would open and you would need to
    click the *Create* button to configure our orchestrator

    ![](media/Containers%20on%20Azure%20in%20a%20practical%20way/Discover%20what%20is%20a%20container/image32.PNG)

5.  The configuration wizard has four steps. In the first one we need to define
    the basic settings for our deployment.

    1.  **Name:** introduce *iil-acs-dcos* or another name you like.

    2.  **Subscription:** select the subscription where the cluster would be
        deployed

    3.  **Resource Group:** select the resource group where resources would be
        provisioned

    4.  **Location:** select the region closest to you.

    ![](media/Containers%20on%20Azure%20in%20a%20practical%20way/Discover%20what%20is%20a%20container/image33.PNG)

1.  In the next step, we need to configure the masters and choose an
    orchestrator.

    1.  **Orchestrator:** choose DC/OS

    2.  **DNS Name Prefix:** introduce a unique name. You can try with
        *iil-acs-dcos-xxx* (where *XXX* should be a random number like 387)

    3.  **User name:** introduce *iillab*

    4.  **SSH Public Key:** Azure Container Service does not support user name
        and password. In order to authenticate to the service, a SSH Key pair is
        required. 

    5.  **Master Count**: leave the default configuration to *1*

    6.  **VM diagnostic**: leave the default configuration *Disabled*

> **Note**: if you don´t have an SSH Key you could create a new key (e.g. using [PuTTYgen](http://www.putty.org/)) or use [the example key](resources/Empower%20your%20application%20lifecycle%20with%20containers%20and%20Azure%20with%20Azure%20Container%20Service/SSH%20Key). The format ofthe keys must be:
```
Public SSH Key format

ssh-rsa [publickey] rsa-key-[dateyyyymmdd]

Private SSH Key format

\-----BEGIN RSA PRIVATE KEY-----

[privatekey]

\-----END RSA PRIVATE KEY-----
```

7.  In the next step, we need to configure the agents. Some subscriptions have a
    limit of the maximum number of cores than can be deployed. If you receive an
    error when deploying the cluster that the maximum number of cores has been
    reached, reduce the number or the size of the virtual machines.

    1.  **Agent Count**: modify the number to **1**

    2.  **Agent Virtual Machine Size**: leave the default configuration

8.  Finish the wizard selecting *OK.* Creating a new Azure Container Service
    instance takes several minutes while the virtual machines are provisioned
    and the cluster is deployed.

9.  After the deployment has completed, you would find a collection of resources
    inside your resource group. Look for the instance of *Container Service* and
    select it.

    ![](media/Containers%20on%20Azure%20in%20a%20practical%20way/Discover%20what%20is%20a%20container/image34.PNG)

10.  You would be able to see the information of our deployment. Copy the *Master
    FQDN*.

        ![](media/Containers%20on%20Azure%20in%20a%20practical%20way/Discover%20what%20is%20a%20container/image35.PNG)

11.  DC/OS doesn’t expose its services directly. An ssh tunnel to the master node is required. The next steps show how to connect using Windows. 
        1.  Open *Putty*. 

            ![](media/Containers%20on%20Azure%20in%20a%20practical%20way/Discover%20what%20is%20a%20container/image29.PNG)

        2.  In the *Host Name* introduce **iillab\@\<your master FQDN\>** and in the *Port* introduce **2200**

            ![](media/Containers%20on%20Azure%20in%20a%20practical%20way/Discover%20what%20is%20a%20container/image18.PNG)

        3.  In the SSH/Auth window, in the *Private key file for authentication*, browse and select your privatekey (ppk file).

            ![](media/Containers%20on%20Azure%20in%20a%20practical%20way/Discover%20what%20is%20a%20container/image27.PNG)

        4. In the SSH/Tunnels window, introduce **8080** on *Source port* and **localhost:80** on *Destination*, and then press *Add*.

            ![](media/Containers%20on%20Azure%20in%20a%20practical%20way/Discover%20what%20is%20a%20container/image28.PNG)

        5. Press *Open* to start the connection

12.  The previous command doesn’t provide any output, you can test that
    everything is working fine opening your browser and going directly to
    http://localhost:8080

        ![](media/Containers%20on%20Azure%20in%20a%20practical%20way/Discover%20what%20is%20a%20container/image36.PNG)

13.  In the dashboard, you can see the general status of our cluster. Because we
    have not deployed any workload everything is stable.

14.  In the left menu select **Nodes.** You can see that there are three nodes:
    one master and two slaves.

15.  In the left menu select **Services \> Services.** Choose **Run a service.**

16.  There are several options. You would deploy a **Single Container** as an
    example of how to run applications inside the orchestrator.

        ![](media/Containers%20on%20Azure%20in%20a%20practical%20way/Discover%20what%20is%20a%20container/image37.PNG)

17.  In the **Service Tab** select a name for our applicacion like
    *example-web-app.* As container image, you should choose *yeasy/simple-web.*

        ![](media/Containers%20on%20Azure%20in%20a%20practical%20way/Discover%20what%20is%20a%20container/image38.PNG)

18.  Select the **Networking** tab, choose **Add Service Endpoint,** uncheck
    **Assign Automatically** and introduce the number **80** in the **Host
    Port** text box. If a port is not defined, DCOS would choose a random port
    available on the host.

        ![](media/Containers%20on%20Azure%20in%20a%20practical%20way/Discover%20what%20is%20a%20container/image39.PNG)

19.  All the configuration that we are doing in the UI is generating a JSON file
    under the hoods. As an example, let’s add a parameter directly into the
    file. To do that, choose **JSON Editor** on the top left area of you screen.
    You can see the options that we have configured translated to the JSON file.

        ![](media/Containers%20on%20Azure%20in%20a%20practical%20way/Discover%20what%20is%20a%20container/image40.PNG)

20.  At the end of the JSON file add a colon after the last element of the JSON
    document and include the following text. We want to deploy our application
    in the public nodes of our cluster in order to be able to access from the
    internet.

```json
"acceptedResourceRoles": [ 
    "slave_public" 
    ]
```

21.  Click on **Review & Run** and wait for the application to be deployed.

        ![](media/Containers%20on%20Azure%20in%20a%20practical%20way/Discover%20what%20is%20a%20container/image41.PNG)

22.  After the status changes to **Running (1 instance)** you can select the
    service just deployed and review its configuration. For example, in the
    **Instance** tab you can see how many instances you have deployed and which
    node are associated with.

        ![](media/Containers%20on%20Azure%20in%20a%20practical%20way/Discover%20what%20is%20a%20container/image42.PNG)

23.  If you choose the instance you can review the information available like its
    configuration details and its logs. Being able to review the *stderr* and
    *stdout* logs is really useful to diagnose possible problems with our
    service deployed.

24.  On the **Configuration** tab, you can check the options configured when the
    services were created and modify some of them.

        ![](media/Containers%20on%20Azure%20in%20a%20practical%20way/Discover%20what%20is%20a%20container/image43.PNG)

25.  You can check the **Debug** tab to see activity information from our service
    like when it was last deployed, if it has been scaled, etc.

26.  Finally, if you want to see your app running you need to access the public
    IP of the load balancer associated with the public agent pool. You can get
    that information from the Azure Portal looking for the
    *dcos-agent-lb-xxxxxxx*

        ![](media/Containers%20on%20Azure%20in%20a%20practical%20way/Discover%20what%20is%20a%20container/image44.PNG)

27.  You would see something like this:

        ![](media/Containers%20on%20Azure%20in%20a%20practical%20way/Discover%20what%20is%20a%20container/image45.PNG)

>   *Note: It is possible that the first time you load the web page and error is
>   returned. It takes some time while the container is being downloaded,
>   executed and the load balancer discovered it. You can continue with the
>   exercise and come back to check it later.*

28.  A service can have one or more instances in order to scale and adapt to user
    demands. In this lab only one public node is available so it is not possible
    to bind more than one container to the same port. However, you can try the
    experience. Come back to the **Services** main page and select the gear that
    appear when you hover the service name. A new contextual menu would appear
    with the **Scale** option. If you select it a pop-up would appear and you
    would need to choose the number of instances. After that, DC/OS would deploy
    so many instances as desired on top of the cluster until resources are
    exhausted.

        ![](media/Containers%20on%20Azure%20in%20a%20practical%20way/Discover%20what%20is%20a%20container/image46.PNG)

To deploy more than one instance you need to integrate other DC/OS services
like Mesos DNS or Mesos Load Balancer in order to be able to properly route
the request to the containers running on different nodes and ports. That
scenario is out of scope but you can get more information on the “Deploying
Services and Pods” section of the DC/OS documentation
