# Lift and Shift - Adding Docker to Nerd Dinner

## Overview

When you decide to modernize your web applications and move them to the cloud (also known as "Lift and Shift".), you don’t necessarily have to fully re-architect your apps. Establishing an environment that emulates your on-premises architecture and putting your application "gets" you there, but doesn't accomplish much more than that.

Re-architecting an application by using an advanced approach like micro-services isn't always an option, because of cost and time restraints. Ripping apart the app and re-writing what sometimes can be years of work and iterations of people and business decisions probably wouldn't be the advised first step.

Depending on the type of application, re-architecting your apps might not be necessary. But adding a Dockerfile, and maybe migrating the database to a service offering like Azure's SQL Database require no code change other than connection strings.

In this Lab we will Use Nerd Dinner Application. Nerd Dinner is a Open Source ASP.NET MVC Project that helps nerds and computer people plan get-togethers. You can see the site running LIVE at http://www.nerddinner.com. We will move the application DB to Azure SQL instance and add the Docker support to the application to run the application in Azure Container Instances.

What's covered in this lab?

In this lab, you will


* Migrate the LocalDB to SQL Server in Azure
* Using the Docker tools in Visual Studio 2017, add the Docker support for the application
* Publish Docker Images to Azure Container Registry (ACR)
* Push the new Docker images from ACR to Azure Container Instances (ACI)

## Pre-requisites for the lab

1.  **Microsoft Azure Account**: You will need a valid and active Azure account for the Azure labs. If you do not have one, you can sign up for a [free trial](https://azure.microsoft.com/en-us/free/){:target="_blank"}

    * If you are a Visual Studio Active Subscriber, you are entitled for a $50-$150 credit per month. You can refer to this [link](https://azure.microsoft.com/en-us/pricing/member-offers/msdn-benefits-details/) to find out more including how to activate and start using your monthly Azure credit.

    * If you are not a Visual Studio Subscriber, you can sign up for the FREE [Visual Studio Dev Essentials](https://www.visualstudio.com/dev-essentials/)program to create **Azure free account** (includes 1 year of free services, $200 for 1st month).


2. **Visual Studio 2017** latest version  with **.Net Core SDK** and **Azure Development Tools** for Visual Studio are installed.

3. **Docker for windows** is installed. Click [here](https://docs.docker.com/docker-for-windows/install/#download-docker-for-windows) for download and install instructions for **Docker for windows**

## Setting up the Environment

1. Clone the application repo from https://github.com/spboyer/nerddinner-mvc4 in your local machine and open the solution in Visual Studio 2017.

     ![cloneandopensolution](images/cloneandopensolution.png)

2. Rebuild the solution and run the application locally to ensure that the application is working fine.
The application looks like as below.
   
   ![apphomepage](images/apphomepage.png)

## Exercise 1: Migrate the LocalDB to SQL Server in Azure
  In this exercise we will create a SQL azure instance and  migrate the application LoaclDB to SQL Server in Azure.

1. Create a new SQL Azure instance in the Azure portal following the below document. 

   [Create an Azure SQL database in the Azure portal](https://docs.microsoft.com/en-us/azure/sql-database/sql-database-get-started-portal).

2. Once the SQL database is provisioned in Azure open the **SQL Server Object Explorer** in Visual Studio.
Click on **Add Server** icon and connect to the Azure SQL server which you have deployed in previuous step.

    ![connecttoazuresql](images/connecttoazuresql.png)
3. To get the schema moved from the LocalDB to the new SQL Azure instance right-click on the LocalDB Instance and select the Schema Compare.

   ![schemacompare](images/schemacompare.png)

   In the schema compare wizard select target as Azure sql database and click on compare.

   ![schemacompare2](images/schemacompare2.png)

   Click on **Update** in the next wizard to update the schema to Azure SQL database.

   ![updateschema](images/updateschema.png)
   ![updateschema2](images/updateschema2.png)

4. Similarly to get the data moved from the LocalDB to the SQL Azure instance right-click on the LocalDB Instance and select the Data Compare tool and walk through the simple wizard.
      
      ![datacompare](images/datacompare.png)

      ![datacompare2](images/datacompare2.png)

      ![datacompare3](images/datacompare3.png)

5. In order to accomplish the **zero code change mantra**, using web.config transforms is the best way to accomplish this at this times. Here we'll add a new **web.release.config** with a new entry. Open the **web.release.config** in Visual Studio and add the below entry.
```csharp
<connectionStrings>  
      <add name="DefaultConnection" connectionString="Data Source=nerddinnerlabsql.database.windows.net;Initial Catalog=nerddinnerlab;Integrated Security=False;User ID=yourUserID;Password=yourdbpassword;Connect Timeout=30;Encrypt=True;TrustServerCertificate=False;ApplicationIntent=ReadWrite;MultiSubnetFailover=False" providerName="System.Data.SqlClient"  
        xdt:Transform="SetAttributes" xdt:Locator="Match(name)"/>
</connectionStrings>  

```
> **Note**: Replace the connection string with your Azure SQL database connection string.

Now we  have successfully migrated the application LocalDB to Azure SQL Db.

## Exercise 2: Add the Docker Support and run the application locally & debug within the Docker container using Visual Studio

1. Visual Studio has great support for Docker. All you have to do is right-click on the project, select **Add->Docker Support**

   ![adddockersupport](images/adddockersupport.png)

2. Visual Studio then add the Docker file, compose files and a specific Docker project to the solution. It also inspects the project to determine the proper base image to use for your project.

   ![dockersupportfiles](images/dockersupportfiles.png)

   In the case of Nerd Dinner, it chose to use microsoft/aspnet:4.7.1-windowsservercore-ltsc2016. Here is the complete file.

```csharp
FROM microsoft/aspnet:4.7.1-windowsservercore-ltsc2016
ARG source
WORKDIR /inetpub/wwwroot
COPY ${source:-obj/Docker/publish} .
```

3. To run the application locally and debug within the Docker container using Visual Studio and to test the connectivity to the SQL Azure instance set the **docker-compose** as startup project and click on **Docker**.
![rundocker](images/rundocker.png)

   Visual Studio downloads the base images and subsequently builds your dev images

   ![downloadingimages](images/downloadingimages.png)

   Once the images downloaded and build is done you will see the application launching in local browser.

    ![applicationlaunch](images/applicationlaunch.png)

   Now the application is running in local docker. To see the list of docker images you can run the following commad 
   ```csharp
    docker images
   ```

     You will see the images similar to this
   
   ![dockerimages](images/dockerimages.png)


## Exercise 3: Publish Docker Images to Azure Container Registry (ACR)

 Now you have validated that the application is running successfully in local Docker, now we can publish the image to a new or existing Azure Container Registry using the publishing wizard in Visual Studio.

1. Right-click on the project, select **Publish**

   ![clickpublish](images/clickpublish.png)

2. In the Publish wizard select **Container Registry** and select **Create New Azure Container Registry** and click on **Publish**

   ![publishtarget](images/publishtarget.png)

3. Fill the required details and click on **Create**

   ![acrdetails](images/acrdetails.png)

4. When publishing, the production Docker image is created and pushed to the Azure Container Registry.
  ![pushrefstoacr](images/pushrefstoacr.png)

5. Once the publish is successfull navigate to the deployeded ACR in Azure portal. You will see **nerddinner** repository with image Tag is published.

   ![acrrepositories](images/acrrepositories.png)


6. Selcect **Access Keys** in ACR and copy the **password**. This password is required in the next exercise.

   ![accesskey](images/accesskey.png)

## Exercise 4: Push the new Docker images from ACR to Azure Container Instances (ACI)

  In this exercise we will create Azure Container Instance and push the new Docker image from ACR to Azure Container Instance.

 we have options as to where the application can be deployed.
* Azure Container Instances
* Azure Container Service
* Service Fabric

In this case, we will use a Windows Container on ACI to bring up Nerd Dinner.

1. We will use Azure CLI to create and push image to Azure Container Instance. Click on **Cloud Shell** in Azure portal.
![cloudshell](images/cloudshell.png)
2. Run the following command to create a new resource group for ACI
   ```csharp
    az group create --name nerddinnerapp --location westus
   ```

     ![acirgcreate](images/acirgcreate.png)

3. Run the following command to Create windows ACI and push the Nerd Dinner image from ACR
   ```csharp
   az container create --name nerddinnerapp --resource-group nerddinnerapp --os-type windows --image {your acr name}.azurecr.io/nerddinner:{Image Tag} --ip-address public 
   ```
   ![deployaci](images/deployaci.png)

   When prompted for **image registry password** paste the password which we have copied in the previous exercise
   > Replace **your acr name** and **Image tag** with your resources details

   It would take approximately 5-10 minutes to deploy ACI.

4. Navigate to the resource group where we have deployed ACI and select the **nerddinnerapp** container group.

    ![acistatus](images/acistatus.png)

    Once the **State** of the container is **Running** you can access the deployed **Nerd Dinner** application using **IP address**. Copy the **IP address** and paste in any browser to see the application running.

    ![finaloutput](images/finaloutput.png)