# Discover what is a container - Windows

## Requirements
Windows Containers environment:
-   Windows machine with Containers
-   Azure VM (Windows OS): [How to create](Containers%20on%20Azure%20in%20a%20practical%20way%20-%20Discover%20what%20is%20a%20container%20-%20Azure%20VM.md)
    -   RDP connection tool

## Step by step: Running our first container on Windows
1.  Open the your machine.
2.  Select the *Start Menu* and open a new *Windows PowerShell* session.

    ![](media/Containers%20on%20Azure%20in%20a%20practical%20way/Discover%20what%20is%20a%20container/image14.PNG)

3.  Check that Docker is installed and available to your user running the
    following command. The output should be similar to the image.

    ```docker -v```

    ![](media/Containers%20on%20Azure%20in%20a%20practical%20way/Discover%20what%20is%20a%20container/image15.PNG)

4. Run your first container with the following command

    ```docker run microsoft/dotnet-samples:dotnetapp-nanoserver```

    ![](media/Containers%20on%20Azure%20in%20a%20practical%20way/Discover%20what%20is%20a%20container/image16.PNG)
