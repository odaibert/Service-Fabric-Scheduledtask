# Running Scheduled Tasks on Service Fabric
This is a scenario guide for developers seeking to migrate existing Windows Server Scheduled Task's to run inside Windows containers on top of Service Fabric.

## Scenario guide
This code accompanies a scenario guide which provides more resources and a more comprehensive walkthrough of containerizing existing applications using Service Fabric.

## Features
* Export a Scheduled Task from a Windows Server machine
* Build Image container 
* Push Docker container image on your Docker Hub 
* Create a Service Fabric cluster
* Create the Service Fabric project
* Deploy the application to the cluster

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

>You can use the CreateTask.bat to create a scheduled task on your local computer

1. Go to ScheduledTask folder 
1. Export scheduled task
```shell
schtasks /Query /XML /TN TaskExample > Exp-TaskExample.xml
``` 
> or just execute ExportTask.bat on the same folder

```xml
Exp-TaskExample.xml

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

### Step 3: Deploying to a Service Fabric cluster

1. Right-click the `ScheduledTask` project and select `Publish`
1. Choose your Service Fabric cluster
1. Click `Publish`

### Creating a Service Fabric application

1. Right-click your solution and add a new project
1. Select `Visual C# -> Cloud -> Service Fabric Application`
1. Choose the `Container` template and specify the image name (ex: `example/scheduledtask:1.0.0`)
1. Update `ApplicationManifest.xml`

Example:
```xml
<ServiceManifestImport>
  <ServiceManifestRef ServiceManifestName="mvc5appPkg" ServiceManifestVersion="1.0.0" />
  <ConfigOverrides />
  <Policies>
    <ContainerHostPolicies CodePackageRef="Code">
      <RepositoryCredentials AccountName="TestUser" Password="12345" PasswordEncrypted="false" />
    </ContainerHostPolicies>
  </Policies>
</ServiceManifestImport>
```

5. Update `ServiceManifest.xml`

```xml
 <CodePackage Name="Code" Version="1.0.0">
    <EntryPoint>
      <!-- Follow this link for more information about deploying Windows containers to Service Fabric: https://aka.ms/sfguestcontainers -->
      <ContainerHost>
        <ImageName>samples/scheduledtask</ImageName>
      </ContainerHost>
    </EntryPoint>
    ...
```

* * *
This project has adopted the [Microsoft Open Source Code of Conduct](https://opensource.microsoft.com/codeofconduct/). For more information see the [Code of Conduct FAQ](https://opensource.microsoft.com/codeofconduct/faq/) or contact [opencode@microsoft.com](mailto:opencode@microsoft.com) with any additional questions or comments.
