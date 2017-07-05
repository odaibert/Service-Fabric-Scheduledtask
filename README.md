# Running Scheduled Tasks on Service Fabric
This is a scenario guide for developers seeking to migrate existing Windows Server Scheduled Task's to run inside Windows containers on top of Service Fabric.

## Scenario guide
This code accompanies a scenario guide which provides more resources and a more comprehensive walkthrough of containerizing existing applications using Service Fabric.

## Features
* Export a Scheduled Task
* Build and Push the container image to Docker Hub
* Create a Service Fabric cluster
* Deploying to a Service Fabric cluster

By containerizing and deploying to Service Fabric your Windows Scheduled Task's, you can migrate at your own pace while taking advantage of the reliability provided by Service Fabric Cluster.

## Requirements
* [Visual Studio 2017](https://www.visualstudio.com/downloads/)
* [Docker for Windows](https://www.docker.com/docker-windows)
* [Docker Hub account](http://hub.docker.com)
* [Azure Service Fabric SDK](https://docs.microsoft.com/en-us/azure/service-fabric/service-fabric-get-started)
* [Service Fabric cluster](#TODO:service_fabric_cluster)

## Quickstart

### Step 1: Clone the repository
```shell
#TODO:repository_location
git clone https://github.com/Microsoft/repository.git
```

### Step 2: Export Scheduled Task

> You can use the CreateTask.bat to create a scheduled task on your local computer

1. Go to ScheduledTask folder 
1. Export scheduled task
```shell
schtasks /Query /XML /TN TaskExample > Exp-TaskExample.xml
``` 

Example:
```xml
<?xml version="1.0" encoding="UTF-16"?>
<Task version="1.2" xmlns="http://schemas.microsoft.com/windows/2004/02/mit/task">
  <RegistrationInfo>
    <Date>2017-07-05T15:57:32</Date>
    <Author>author</Author>
    <URI>\TaskExample</URI>
  </RegistrationInfo>
  <Settings>
    <DisallowStartIfOnBatteries>true</DisallowStartIfOnBatteries>
    <StopIfGoingOnBatteries>true</StopIfGoingOnBatteries>
    <MultipleInstancesPolicy>IgnoreNew</MultipleInstancesPolicy>
    <IdleSettings>
      <Duration>PT10M</Duration>
      <WaitTimeout>PT1H</WaitTimeout>
      <StopOnIdleEnd>true</StopOnIdleEnd>
      <RestartOnIdle>false</RestartOnIdle>
    </IdleSettings>
  </Settings>
  <Triggers>
    <TimeTrigger>
      <StartBoundary>2017-07-05T15:57:00</StartBoundary>
      <Repetition>
        <Interval>PT1M</Interval>
      </Repetition>
    </TimeTrigger>
  </Triggers>
  <Actions Context="Author">
    <Exec>
      <Command>c:\ScheduledTask\Task.bat</Command>
    </Exec>
  </Actions>
</Task>
```

### Step 3: Build and Push the container image to Docker Hub

1. Build the container image
```shell  
docker build -t scheduledtask .
```

2. Login to Docker Hub: 
```shell
docker login

Login with your Docker ID to push and pull images from Docker Hub. If you don't have a Docker ID, head over to https://hub.docker.com to create one.

Username: user
Password: Password

Login Succeeded
```

3.  Push to your registry
```shell
docker push example/scheduledtask:1.0.0
```
### Step 4: Create a Service Fabric Cluster

You've got an application ready to run, now you need a cluster to run it on. We'll use PowerShell to create one in Azure. The New-AzureRmServiceFabricCluster cmdlet will automatically provision everything you need to get started. Make sure to give it a unique ResourceGroupName, as that will be used as part of the DNS name for your cluster.
```shell
# First, we need to login to an Azure subscription
Login-AzureRmAccount

# Create a SecureString to be used for our virtual machines
# We'll use the same password to protect our PFX
$password = ConvertTo-SecureString "Password1234!" -AsPlainText -Force

# Create the cluster and drop the generated certificate in a folder
New-AzureRmServiceFabricCluster -ResourceGroupName partycluster1 -Location eastus -VmPassword $password -CertificateOutputFolder C:\temp -CertificatePassword $password -OS WindowsServer2016DatacenterwithContainers

# Import the PFX to your certificate store (replace with your PFX file name)
Import-PfxCertificate -FilePath C:\temp\myCluster12320170512104014.pfx -Password $password -CertStoreLocation Cert:\CurrentUser\My -Exportable
```
[Click here for more info about how to create a Service Fabric Cluster](https://docs.microsoft.com/en-us/azure/service-fabric/service-fabric-get-started-azure-cluster)

### Step 5: Deploying to a Service Fabric cluster

1. If you are connecting to an unsecured cluster, all that's required is the cluster connection endpoint, such as partycluster1.eastus.cloudapp.azure.com:19000. In that case, the connection endpoint in the publish profile would look something like this:
```xml
<ClusterConnectionParameters ConnectionEndpoint="partycluster1.eastus.cloudapp.azure.com:19000" />
```

> If you are connecting to a secured cluster, you will also need to provide the details of the client certificate from the local store to be used for authentication. For more details, see [Configuring secure connections to a Service Fabric cluster](https://docs.microsoft.com/en-us/azure/service-fabric/service-fabric-visualstudio-configure-secure-connections).

2. Right-click the `ScheduledTask` project and select `Publish`
1. Choose your Service Fabric cluster
1. Click `Publish`



* * *
This project has adopted the [Microsoft Open Source Code of Conduct](https://opensource.microsoft.com/codeofconduct/). For more information see the [Code of Conduct FAQ](https://opensource.microsoft.com/codeofconduct/faq/) or contact [opencode@microsoft.com](mailto:opencode@microsoft.com) with any additional questions or comments.
