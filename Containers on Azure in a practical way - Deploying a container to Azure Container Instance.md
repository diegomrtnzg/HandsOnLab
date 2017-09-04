# Deploying a container to Azure Container Instance
*This lab is based on* [official documentation](https://docs.microsoft.com/en-us/azure/container-instances/).

Azure Container Instances offers the fastest and simplest way to run a container in Azure, without having to provision any virtual machines and without having to adopt a higher-level service. 

## Step by step
### Create an Azure Container Instance
#### Launch Azure Cloud Shell
The Azure Cloud Shell is a free Bash shell that you can run directly within the Azure portal. It has the Azure CLI preinstalled and configured to use with your account. Click the Cloud Shell button on the menu in the upper-right of the [Azure portal](https://portal.azure.com/).

![](media/Containers%20on%20Azure%20in%20a%20practical%20way/Deploying%20a%20container%20to%20Azure%20Container%20Instance/image1.png)

The button launches an interactive shell that you can use to run all of the steps in this topic:

![](media/Containers%20on%20Azure%20in%20a%20practical%20way/Deploying%20a%20container%20to%20Azure%20Container%20Instance/image2.png)

#### Create a resource group

Azure Container Instances are Azure resources and must be placed in an Azure resource group, a logical collection into which Azure resources are deployed and managed.

Create a resource group with the [az group create](/cli/azure/group#create) command. 

The following example creates a resource group named *myResourceGroup* in the *eastus* location.

```
az group create --name myResourceGroup --location eastus
```


#### Create a container

You can create a container by providing a name, a Docker image, and an Azure resource group. You can optionally expose the container to the internet with a public IP address. In this case, we'll use a container that hosts a very simple web app written in [Node.js](http://nodejs.org).

```
az container create --name mycontainer --image microsoft/aci-helloworld --resource-group myResourceGroup --ip-address public 
```

Within a few seconds, you should get a response to your request. Initially, the container will be in a **Creating** state, but it should start within a few seconds. You can check the status using the `show` command:

```
az container show --name mycontainer --resource-group myResourceGroup
```

At the bottom of the output, you will see the container's provisioning state and its IP address:

```json
...
"ipAddress": {
      "ip": "13.88.8.148",
      "ports": [
        {
          "port": 80,
          "protocol": "TCP"
        }
      ]
    },
    "osType": "Linux",
    "provisioningState": "Succeeded"
...
```

Once the container moves to the **Succeeded** state, you can reach it in the browser using the IP address provided. 

![](media/Containers%20on%20Azure%20in%20a%20practical%20way/Deploying%20a%20container%20to%20Azure%20Container%20Instance/image3.png)

### Pull the container logs

You can pull the logs for the container you created using the `logs` command:

```
az container logs --name mycontainer --resource-group myResourceGroup
```

Output:

```bash
listening on port 80
::ffff:10.240.255.105 - - [21/Jul/2017:00:01:46 +0000] "GET / HTTP/1.1" 200 1663 "-" "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/59.0.3071.115 Safari/537.36"
::ffff:10.240.255.105 - - [21/Jul/2017:00:01:46 +0000] "GET /favicon.ico HTTP/1.1" 404 150 "http://104.210.39.122/" "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/59.0.3071.115 Safari/537.36"
```

### Delete the container

When you are done with the container, you can remove it using the `delete` command:

```azurecli-interactive
az container delete --name mycontainer --resource-group myResourceGroup
```

## Next steps
[Deploy a container group](https://docs.microsoft.com/en-us/azure/container-instances/container-instances-multi-container-group)

[Mounting an Azure file share with Azure Container Instances](https://docs.microsoft.com/en-us/azure/container-instances/container-instances-mounting-azure-files-volume)

[Deploy to Azure Container Instances from the Azure Container Registry](https://docs.microsoft.com/en-us/azure/container-instances/container-instances-using-azure-container-registry)
