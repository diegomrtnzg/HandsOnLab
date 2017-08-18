#Create Azure VMs

## Deploying a Linux VM 
If you want to have a copy of the Linux environment in your own Azure subscription to continue with the lab after the event you can follow these steps to have your own Linux VM configured.
1.  **Open Edge** or your favorite browser and open the Azure Portal present in the following URL: <http://portal.azure.com>
2. From the top left corner of the web page, select the *New* option. If you don’t see it you would need to click the “*hamburger”* menu to open the blade or click directly in the green plus sign.

    ![](media/Containers%20on%20Azure%20in%20a%20practical%20way/Discover%20what%20is%20a%20container/image1.PNG)

3. In the search box introduce *Ubuntu Server 16.04 LTS* and click *Intro* in your keyboard.

    ![](media/Containers%20on%20Azure%20in%20a%20practical%20way/Discover%20what%20is%20a%20container/image12.PNG)

4.  The search results would appear. Select the option that matches the text introduce in the step before. A new blade would open and you would need to click the *Create* button in order to configure the virtual machine.

    ![](media/Containers%20on%20Azure%20in%20a%20practical%20way/Discover%20what%20is%20a%20container/image3.PNG)

5.  The creation of the VM has four steps, fill the form with the following information. 
    1. **Basics:**
        1.  **Name:** iil-lab-xxx *(where XXX should be a random number like 387)*
        2.  **VM disk type:** HDD
        3.  **Username:** iil-lab iv. **Password:** Passw0rd!Passw0rd!
        4.  **Resource Group:** create a new resource group named iil-lab
        5.  **Location:** choose the datacenter closer to your actual location to obtain better performance and reduced latency.
        6.  **Size:** D1_V2 Standard
    2.  **Settings:**
        1.  **Use managed disk:** Yes
        2.  **Network:** keep default configuration
        3.  **Subnet:** keep default configuration iv. **Public IP Address:** keep default configuration
        4.  **Network Security Group (firewall):** keep default configuration
        5.  **Extensions:** keep default configuration
        6.  **High Availability:** keep default configuration
        7.  **Monitoring:** keep default configuration
    8.  **Summary:** Click the *Purchase* button at the end of the summary to create the new virtual machine.

Creating a new virtual machine takes between 4-6 minutes. You can check if the deployment has succeeded click in the bell
on the top right menu.

![](media/Containers%20on%20Azure%20in%20a%20practical%20way/Discover%20what%20is%20a%20container/image13.PNG)


### Installing Docker on Linux 
1.  In the search box on the top of the portal look for the VM **iil-lab-xxx** where **xxx** is the random number you chose previously. Select the virtual machine.

    ![](media/Containers%20on%20Azure%20in%20a%20practical%20way/Discover%20what%20is%20a%20container/image4.PNG)

2.  A new blade is opened with the details of the VM. Select **Connect** in the top menu and copy the IP at the end of the command.

    ![](media/Containers%20on%20Azure%20in%20a%20practical%20way/Discover%20what%20is%20a%20container/image5.PNG)

3.  Open the *Start Menu* and search for *Putty.* Open the program.

    ![](media/Containers%20on%20Azure%20in%20a%20practical%20way/Discover%20what%20is%20a%20container/image6.PNG)

4.  Introduce the IP in the *Host Name* text box. Select *SSH* as protocol and click *Open.*

    ![](media/Containers%20on%20Azure%20in%20a%20practical%20way/Discover%20what%20is%20a%20container/image7.PNG)

5.  Accept the alert box to include the server key in your local computer and complete the login introducing the username *iil-lab* and the password *Passw0rd!Passw0rd!*
6.  One inside the virtual machine we need to install the *Docker* package. Execute the following command as sudo and introduce again your password: `sudo apt-get install docker.io`

    ![](media/Containers%20on%20Azure%20in%20a%20practical%20way/Discover%20what%20is%20a%20container/image8.PNG)

7. It would take a few minutes to complete the installation of the package. You can check that everything is correct running the following command. You would see the current docker installed version: `sudo docker -v`

    ![](media/Containers%20on%20Azure%20in%20a%20practical%20way/Discover%20what%20is%20a%20container/image9.PNG)

## Deploying a Windows VM 

If you want to have a copy of the Windows environment in your own Azure
subscription to continue with the lab after the event you can follow these steps
to have your own Windows VM configured.

1.  **Open Edge** or your favorite browser and open the Azure Portal present in
    the following URL: <http://portal.azure.com>
!
2.  From the top left corner of the web page, select the *New* option. If you
    don’t see it you would need to click the “*hamburger”* menu to open the
    blade or click directly in the green plus sign.

    ![](media/Containers%20on%20Azure%20in%20a%20practical%20way/Discover%20what%20is%20a%20container/image10.PNG)

3.  In the search box introduce *Windows Server 2016 Datacenter - with
    Containers* and click *Intro* in your keyboard.

    ![](media/Containers%20on%20Azure%20in%20a%20practical%20way/Discover%20what%20is%20a%20container/image11.PNG)

4.  The search results would appear. Select the option that matches the text
    introduce in the step before. A new blade would open and you would need to
    click the *Create* button in order to configure the virtual machine.

    ![](media/Containers%20on%20Azure%20in%20a%20practical%20way/Discover%20what%20is%20a%20container/image12.PNG)

5.  The creation of the VM has four steps, fill the form with the following
    information. 
    1. **Basics:**
        1.  **Name:** iil-win-xxx *(where XXX should be a random number like 387)*
        2.  **VM disk type:** HDD
        3.  **Username:** iil-lab iv. **Password:** Passw0rd!Passw0rd!
        4.  **Resource Group:** select the previous resource group created under the
        name iil-lab
        5.  **Location:** choose the datacenter closer to your actual location to
        obtain better performance and reduced latency.
        6.  **Size:** D1_V2 Standard
    2.  **Settings:**
        1.  **Use managed disk:** Yes
        2.  **Network:** keep default configuration
        3.  **Subnet:** keep default configuration iv. **Public IP Address:**
            keep default configuration
        4.  **Network Security Group (firewall):** keep default configuration
        5.  **Extensions:** keep default configuration
        6.  **High Availability:** keep default configuration
        7.  **Monitoring:** keep default configuration
    3.  **Summary:** Click the *Purchase* button at the end of the summary to
        create the new virtual machine.

As previously mentioned, creating a new virtual machine takes between 4-6
minutes.