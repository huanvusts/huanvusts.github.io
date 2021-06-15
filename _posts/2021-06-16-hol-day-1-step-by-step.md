---
layout: page
title: DevOps - Hands-on lab Day 1
---

Jun 2021

**Contents**

<!-- TOC -->

- [App modernization hands-on lab step-by-step](#app-modernization-hands-on-lab-step-by-step)
  - [Abstract and learning objectives](#abstract-and-learning-objectives)
  - [Overview](#overview)
  - [Solution architecture](#solution-architecture)
  - [Requirements](#requirements)
  - [Exercise 1: Setting up Azure Migrate](#exercise-1-setting-up-azure-migrate)
  - [Exercise 2: Migrate your application with App Service Migration Assistant](#exercise-2-migrate-your-application-with-app-service-migration-assistant)
    - [Task 1: Perform assessment for migration to Azure App Service](#task-1-perform-assessment-for-migration-to-azure-app-service)
    - [Task 2: Migrate the web application to Azure App Service](#task-2-migrate-the-web-application-to-azure-app-service)
  - [Exercise 3: Migrate the on-premises database to Azure SQL Database](#exercise-3-migrate-the-on-premises-database-to-azure-sql-database)
    - [Task 1: Perform assessment for migration to Azure SQL Database](#task-1-perform-assessment-for-migration-to-azure-sql-database)
    - [Task 2: Retrieve connection information for SQL Databases](#task-2-retrieve-connection-information-for-sql-databases)
    - [Task 3: Migrate the database schema using the Data Migration Assistant](#task-3-migrate-the-database-schema-using-the-data-migration-assistant)
    - [Task 4: Migrate the database using the Azure Database Migration Service](#task-4-migrate-the-database-using-the-azure-database-migration-service)
    - [Task 5: Configure the application connection to SQL Azure Database](#task-5-configure-the-application-connection-to-sql-azure-database)

<!-- /TOC -->

# App modernization hands-on lab step-by-step

## Abstract and learning objectives

In this hands-on lab, you implement the steps to modernize a legacy on-premises application, including upgrading and migrating the application and the database to Azure and updating the application to take advantage of serverless and cloud services.

At the end of this hands-on lab, your ability to build solutions for modernizing legacy on-premises applications and infrastructure using cloud services will be improved.

## Overview

Parts Unlimited is an online auto parts store. Founded in Spokane, WA, in 2008, they are providing both genuine OEM and aftermarket parts for cars, sport utility vehicles, vans, and trucks, including new and remanufactured complex parts, maintenance items, and accessories. Its mission is to make buying vehicle replacement parts easy for consumers and professionals. Parts Unlimited has 185 stores in the US, with plans to scale to Mexico and Brazil.

Parts Unlimited has a hosted web application on its internal infrastructure and using a Windows Server, Internet Information Services (IIS), and Microsoft SQL Server to host the solution. Beyond the initial effort and costs, these applications incur ongoing maintenance costs in hardware, operating system updates, and licensing fees. These maintenance costs make Microsoft Azure App Service an attractive alternative. Their team is looking to migrate Microsoft ASP.NET applications and any SQL Server database to Azure App Service and Azure SQL Database. However, they are worried that their application might not be supported. Their website is built on a .NET Core version that hit the end of life on December 23, 2019. They wonder if they can move to the cloud now and migrate their application later or if the old version will be a show stopper.

Additionally, Parts Unlimited has plans to increase its marketing investment, currently on hold because of scaling issues. The company is stuck and can't grow without increasing its infrastructure footprint. Their CEO wants to finalize their cloud vs. on-premises decision based on the current migration effort's success. The engineering team is worried about their order processing subsystem. Currently, they have a strongly coupled order processing system that runs synchronously during checkout. When moved to the cloud, they don't want to be worried about their order processing system's scalability. They are looking for a modern approach with the least migration effort possible.

Finally, Parts Unlimited is looking to invest in DevOps practices to decrease human error in deployments. They are looking for options to have a staging environment to test functionality before shipping to production. However, their team does not have any experience in building CI/CD pipelines. They are not sure if this goal is achievable in the short term, and they do not want it to hold up their migration to the cloud.

## Solution architecture

Below is a high-level architecture diagram of the solution you implement in this hands-on lab. Please review this carefully to understand the whole of the solution as you are working on the various components.

![This solution diagram includes a high-level overview of the architecture implemented within this hands-on lab.](/assets/devops/architecture-diagram.png "Solution architecture diagram")

> **Note:** The solution provided is only one of many possible, viable approaches.

The solution begins with setting up Azure Migrate as the central assessment and migration hub for Parts Unlimited's E-Commerce website. Using the App Service Migration Assistant tool from Azure Migrate, Parts Unlimited found that their website is fully compatible with Azure App Service. As a next step, they use App Service Migration Assistant to provision an Azure App Service environment and deploy their application to Azure. Following the success in moving the web application, Parts Unlimited uses the Data Migration Assistant (DMA) assessment to determine that they can migrate into a fully-managed SQL Database service in Azure. The assessment reveals no compatibility issues or unsupported features that would prevent them from using Azure SQL Database.

Next, Parts Unlimited sets up a private GitHub repository and pushes their codebase to GitHub. They set up deployment slots to have a staging environment to test functionality before releasing to production. As a CI/CD solution, they decide to use GitHub Actions and Workflows.

Finally, Parts Unlimited decides to decouple its order processing system and move to an event-driven serverless compute platform. Following a [web-queue-worker architecture](https://docs.microsoft.com/en-us/azure/architecture/guide/architecture-styles/web-queue-worker), they build an Azure Function and use Azure Storage Queue to process orders and create invoices asynchronously. When new orders come in, the web front end adds jobs into the queue consumed by Azure Functions. The Functions App scales independently based on the number of jobs in the queue, helping Parts Unlimited elastically handle a variable amount of orders.

## Requirements

- Microsoft Azure subscription must be pay-as-you-go or MSDN.
  - Trial subscriptions will _not_ work.
  - **IMPORTANT:** To complete this lab, you must have sufficient rights within your Azure AD tenant to register resource providers in your Azure Subscription.
- An active GitHub Account.

## Exercise 1: Setting up Azure Migrate

Duration: 10 minutes

Azure Migrate provides a centralized hub to assess and migrate on-premises servers, infrastructure, applications, and data to Azure. It provides a single portal to start, run, and track your migration to Azure. Azure Migrate comes with a range of assessment tools and migration that we will use during our lab. We will use Azure Migrate as the central location for our assessment and migration efforts.

1. In the [Azure portal](https://portal.azure.com), navigate to your lab resource group. Select **Add** to add a new resource.

   ![Lab resource group is open. Resource Add button is highlighted.](/assets/devops/portal-add-resource.png "Lab Resource Group")

2. Type **Azure Migrate** into the search box and hit **Enter** to start the search.

   ![Azure Portal new resource page is open. The search box is filled with Azure Migrate.](/assets/devops/azure-migrate-search.png "Marketplace Search for Azure Migrate")

3. Select **Create** to continue.

   ![Azure Migrate resource creation screen is open. Create button is highlighted.](/assets/devops/azure-migrate-create.png "Creating Azure Migrate")

4. As part of our migration project for Parts Unlimited, we will first assess and migrate their Web Application living on IIS, on a VM. Select **Web Apps** to continue.

   ![Azure Migrate is open. Web Apps section is highlighted.](/assets/devops/azure-migrate-web-apps.png "Azure Migrate Web Apps")

5. Select **Create project**.

   ![Azure Migrate is open. Web Apps section is selected. Create project button is highlighted.](/assets/devops/azure-migrate-create-project.png "Azure Migrate Create project")

6. Type **partsunlimitedweb** as your project name. Select **Create** to continue.

   ![Azure Migrate project settings page is shown. The project name is set to partsunlimitedweb. Create button is highlighted.](/assets/devops/azure-migrate-create-project-settings.png "Azure Migrate Project Creation")

7. Once your project is created **Azure Migrate** will show you default **Web App Assessment** and **Web App Migration** tools (You might need to refresh your browser). For the Parts Unlimited web site, **App Service Migration Assistant** is the one we have to use. Download links are presented on Azure Migrate's Web Apps page. In our case, our lab environment comes with App Service Migration Assistant pre-installed on Parts Unlimited's web server.

   ![Azure Migrate Web App assessment and migration tools are presented.](/assets/devops/azure-migrate-web-app-migration.png "Azure Migrate Web Apps Capabilities")

8. Another aspect of our migration project will be the database for Parts Unlimited's website. We will have to assess the database's compatibility and migrate to Azure SQL Database. Let's switch to the **SQL Server (only)** section in Azure Migrate. Select **Click here** hyperlink for Assessment tools.

   ![Azure Migrate is open. The databases section is selected. Click here link for assessment tools is highlighted.](/assets/devops/azure-migrate-database-assessment.png "Azure Migrate Databases")

9. We will use **Azure Migrate: Database Assessment** to assess Parts Unlimited's database hosted on a SQL Server 2008 R2 server. Pick **Azure Migrate: Database Assessment (1)** and select **Add tool (2)**.

   ![Azure Migrate Database Assessment option is selected for Azure Migrate tools. Add tool button is highlighted.](/assets/devops/azure-migrate-database-assessment-tool.png "Azure Migrate Database Assessment Tools")

10. Now, we can see a download link for the **Data Migration Assessment (1)** tool under assessment tools in Azure Migrate. In our case, our lab environment comes with the Data Migration Assessment pre-installed on Parts Unlimited's database server. Select **Click here (2)** under the **Migration Tools** section to continue.

    ![Data Migration Assessment tool's download link is shown. Click here link for migration tools is highlighted.](/assets/devops/azure-migrate-database-migration.png "Azure Migrate DMA Download")

11. We will use **Azure Migrate: Database Migration** to migrate Parts Unlimited's database to an Azure SQL Database. Pick **Azure Migrate: Database Assessment (1)** and select **Add tool (2)**.

    ![Azure Migrate Database Migration option is selected for Azure Migrate tools. Add tool button is highlighted.](/assets/devops/azure-migrate-database-migration-tool.png "Azure Migrate Database Migration Tool")

12. Now, we have all the assessment and migration tools/services we need for Parts Unlimited ready to go under the Azure Migrate umbrella.

    ![Azure Migrate databases section is open. Azure Migrate Database Assessment and Database Migration tools are presented.](/assets/devops/azure-migrate-database-migration-ready.png "Azure Migrate Database Migration and Assessment Tools")

## Exercise 2: Migrate your application with App Service Migration Assistant

Duration: 20 minutes

The first step for Parts Unlimited is to assess whether their apps have dependencies on unsupported features on Azure App Service. In this exercise, you use an **Azure Migrate** tool called the [App Service migration assistant](https://appmigration.microsoft.com/) to evaluate Parts Unlimited's website for a migration to Azure App Service. The assessment runs readiness checks and provides potential remediation steps for common issues. Once the assessment succeeds, we will proceed with the migration as well. You will use a simulated on-premises environment hosted in virtual machines running on Azure.

### Task 1: Perform assessment for migration to Azure App Service

Parts Unlimited would like an assessment to see what potential issues they might need to address in moving their application to Azure App Service. You will use the [App Service migration assistant](https://appmigration.microsoft.com/) to assess the application and run various readiness checks.

1. In the [Azure portal](https://portal.azure.com), navigate to your **WebVM** VM by selecting **Resource groups** from Azure services list, selecting the **hands-on-lab-SUFFIX** resource group, and selecting the **WebVM** VM from the list of resources.

   ![The WebVM virtual machine is highlighted in the list of resources.](/assets/devops/webvm-selection.png "WebVM Selection")

2. On the WebVM Virtual Machine's **Overview (1)** blade, copy the **Public IP address (2)**.

   ![The WebVM VM blade is displayed, Public IP Address copy button is highlighted.](/assets/devops/web-vm-ip.png "WebVM Overview and Public IP")

3. Open a new browser window and navigate to the IP Address you copied.

   ![The WebVM VM blade is displayed, Public IP Address copy button is highlighted.](/assets/devops/parts-umlimited-web-site.png "Parts Unlimited Web Site")

   > For testing purposes, you might want to create yourself an account on the Parts Unlimited website and purchase some products. Use the coupon code **FREE** to buy everything for free.

4. Go back to the Azure Portal. On the WebVM Virtual Machine's **Overview** blade, select **Connect (1)** and **RDP (2)** on the top menu.

   ![The WebVM VM blade is displayed, with the Connect button highlighted in the top menu.](/assets/devops/connect-rdp-webvm.png "WebVM RDP Connect")

5. Select **Download RDP File** on the next page, and open the downloaded file.

   > **Note**: The first time you connect to the WebVM Virtual Machine, you will see a blue pop-up terminal dialog taking you through a couple of software installs. Don't be alarmed, and wait until the installs are complete.

   ![RDP Window is open. Download RDP File button is highlighted.](/assets/devops/rdp-download.png "WebVM RDP File Download")

6. Select **Connect** on the Remote Desktop Connection dialog.

   ![In the Remote Desktop Connection Dialog Box, the Connect button is highlighted.](/assets/devops/remote-desktop-connection-webvm.png "Remote Desktop Connection dialog")

7. Enter the following credentials with your password when prompted, and then select **OK**:

   - **Username**: demouser
   - **Password**: {YOUR-ADMIN-PASSWORD}

   > **Note**: default password is `Password.1!!`

   ![The credentials specified above are entered into the Enter your credentials dialog.](/assets/devops/rdp-credentials-webvm.png "Enter your credentials")

8. Select **Yes** to connect, if prompted that the identity of the remote computer cannot be verified.

   ![In the Remote Desktop Connection dialog box, a warning states that the remote computer's identity cannot be verified and asks if you want to continue anyway. At the bottom, the Yes button is circled.](/assets/devops/remote-desktop-connection-identity-verification-webvm.png "Remote Desktop Connection dialog")

9. Once logged into the WebVM VM, a script will execute to install the various items needed for the remaining lab steps.
10. Once the script completes, open **AppServiceMigrationAssistant** that is located on the desktop.

    ![AppServiceMigrationAssistant is highlighted on the desktop.](/assets/devops/appservicemigrationassistant-desktop.png "App Service Migration Assistant")

11. Once App Service Migration Assistant discovers the websites available on the server, choose **Default Web Site (1)** for migration and select **Next (2)** to start the assessment.

    ![AppServiceMigrationAssistant is open. Default Web Site is selected. The next button is highlighted.](/assets/devops/appservicemigration-choose-site.png "App Service Migration Assistant Web Site selection")

12. Observe the result of the assessment report. In our case, our application has successfully passed 13 tests **(1)** with no additional actions needed. Now that our assessment is complete, select **Next (2)** to proceed to the migration.

![Assessment report result is shown. There are 13 success metrics presented. The next button is highlighted.](/assets/devops/appservicemigration-report.png "Assessment Report")

> For the details of the readiness checks, see [App Service Migration Assistant documentation](https://github.com/Azure/App-Service-Migration-Assistant/wiki/Readiness-Checks).

### Task 2: Migrate the web application to Azure App Service

After reviewing the assessment results, you have ensured the web application is a good candidate for migration to Azure App Service. Now, we will continue the process with the migration of the application.

1. In order to continue with the migration of our website Azure App Service Migration Assistant needs access to our Azure Subscription. Select **Copy Code & Open Browser** button to be redirected to the Azure Portal.

   ![Azure App Service Migration Assistant's Azure Login dialog is open. A device code is presented. Copy Code & Open Browser button is highlighted.](/assets/devops/appservicemigration-azure-login.png "Azure Login")

2. At its first launch, you will be asked to choose a browser. Select **Microsoft Edge (1)** and check **Always use this app (2)** checkbox. Select **OK (2)** to move to the next step.

   ![Browser choice dialog is shown. Microsoft Edge is selected. Always use this app checkbox is checked. OK button is highlighted.](/assets/devops/browser-choice.png "Default Browser Selection")

3. Right-click the text box and select **Paste (1)** to paste your login code. Select **Next** to give subscription access to App Service Migration Assistant.

   ![Azure Code Login website is open. Context menu for the code textbox is shown. Paste command from the context menu is highlighted. The next button is highlighted as a second step. ](/assets/devops/appservicemigration-azure-login-code.png "Enter Authentication Code")

4. Continue the login process with your Azure Subscription credentials. When you see the message that says **You have signed in to the Azure App Service Migration Assistant application on your device** close the browser window and return to the App Service Migration Assistant.

   ![Azure Login process is complete. A message dialog is shown that indicates the login process is a success.](/assets/devops/appservicemigration-azure-login-complete.png "App Service Migration Assistant authentication approval")

5. Select the Azure Migrate project we created **(1)** in the previous exercise to submit our migration results. Select **Next** to continue.

   ![Azure Migrate Project is set to partsunlimitedweb. The next button is highlighted.](/assets/devops/appservicemigration-azure-migrate.png "Azure Migrate Hub integration")

6. In order to migrate Parts Unlimited website, we have to create an App Service Plan. The Azure App Service Migration Assistant will take care of all the requirements needed. Select **Use existing (1)** and select the lab resource group as your deployment target. App Service requires a globally unique Site Name. We suggest using a pattern that matches `partsunlimited-web-{uniquesuffix}` **(2)**. Select **Migrate** to start the migration process.

   ![Deployment options are presented. The existing lab resource group is selected as the destination. The destination site name is set to partsunlimited-web-20X21. Migrate button is highlighted.](/assets/devops/appservicemigration-migrate.png "Azure App Service Migration Assistant Options")

   > **WARNING:** If your migration fails with a **WindowsWorkersNotAllowedInLinuxResourceGroup (1)** try the migration process again, but this time selecting a different Resource Group for your deployment. If that is not possible, select a different Region.
   >
   > ![Migration failed error screen is shown. WindowsWorkersNotAllowedInLinuxResourceGroup message is highlighted.](/assets/devops/app-migration-windowsworkersnotallowed.png "Migration failed")

7. We have just completed the Parts Unlimited website's migration from IIS on a Virtual Machine to Azure App Service. Congratulations. Let's go back to the Azure Portal and look into Azure Migrate. Search for `migrate` **(1)** on the Azure Portal and select **Azure Migrate (2)**.

   ![Azure Portal is open. The search box is filled with the migrate keyword. Azure Migrate is highlighted from the result list.](/assets/devops/find-azure-migrate.png "Azure Migrate on Azure Portal Search")

8. Switch to the **Web Apps (1)** section. See the number of discovered web servers, assessed websites **(2)** and migrated websites change **(3)**. Keep in mind that you might need to wait for 5 to 10 minutes for results to show up. You can use the **Refresh** button on the page to see the latest status.

   ![Azure Migrate shows web app assessment and migration reports.](/assets/devops/azure-migrate-web-app-migration-done.png "Azure Migrate Web Apps Tools")

## Exercise 3: Migrate the on-premises database to Azure SQL Database

Duration: 55 minutes

The next step of Part Unlimited's migration project is the assessment and migration of its database. Currently, the database lives on a SQL Server 2008 R2 on a virtual machine. You will use an **Azure Migrate: Database Assessment** tool called **Microsoft Data Migration Assistant (DMA)** to assess the `PartsUnlimited` database for a migration to Azure SQL Database. The assessment generates a report detailing any feature parity and compatibility issues between the on-premises database and Azure SQL Database. After the assessment, you will use an **Azure Migrate: Database Migration** service called **Azure Database Migration Service (DMS)**. During the exercise, you will use a simulated on-premises environment hosted in virtual machines running on Azure.

### Task 1: Perform assessment for migration to Azure SQL Database

Parts Unlimited would like an assessment to see what potential issues they might need to address in moving their database to Azure SQL Database. In this task, you will use the [Microsoft Data Migration Assistant](https://docs.microsoft.com/sql/dma/dma-overview?view=sql-server-2017) (DMA) to assess the `PartsUnlimited` database against Azure SQL Database (Azure SQL DB). Data Migration Assistant (DMA) enables you to upgrade to a modern data platform by detecting compatibility issues that can impact database functionality on your new version of SQL Server or Azure SQL Database. It recommends performance and reliability improvements for your target environment. The assessment generates a report detailing any feature parity and compatibility issues between the on-premises database and the Azure SQL DB service.

> **Note**: The Database Migration Assistant is already installed on your SqlServer2008 VM. It can be downloaded through Azure Migrate or from the [Microsoft Download Center](https://go.microsoft.com/fwlink/?linkid=2090807) as well.

1. Connect to your SqlServer2008 VM with RDP. Your credentials are the same as the WebVM , `demouser` with `Password.1!!` password.

   ![The SQLServer2008 virtual machine is highlighted in the list of resources.](/assets/devops/find-sqlserver2008-resource.png "SqlServer2008 Selection")

2. Launch DMA from the Windows Start menu by typing "data migration" into the search bar and then selecting **Microsoft Data Migration Assistant** in the search results.

   > **Note**: If you do not see the migration assistant, install it from the `c:\DataMigrationAssistant.msi` file.

   > **Note**: There is a known issue with screen resolution when using an RDP connection to Windows Server 2008 R2, which may affect some users. This issue presents itself as very small, hard-to-read text on the screen. The workaround for this is to use a second monitor for the RDP display, allowing you to scale up the resolution to make the text larger.

   ![In the Windows Start menu, "data migration" is entered into the search bar, and Microsoft Data Migration Assistant is highlighted in the Windows start menu search results.](/assets/devops/windows-start-menu-dma.png "Data Migration Assistant")

3. In the DMA dialog, select **+** from the left-hand menu to create a new project.

   ![The new project icon is highlighted in DMA.](/assets/devops/dma-new.png "New DMA project")

4. In the New project pane, set the name of the project **(1)** and make sure the following value are selected:

   - **Project type**: Select Assessment.
   - **Project name (1)**: Enter **Assessment**.
   - **Assessment type**: Select Database Engine.
   - **Source server type**: Select SQL Server.
   - **Target server type**: Select Azure SQL Database.

   ![New project settings for doing an assessment of a migration from SQL Server to Azure SQL Database.](/assets/devops/dma-new-project-to-azure-sql-db.png "New project settings")

5. Select **Create (2)**.

6. On the **Options** screen, ensure **Check database compatibility (1)** and **Check feature parity (1)** are both checked, and then select **Next (2)**.

   ![Check database compatibility and check feature parity are checked on the Options screen.](/assets/devops/dma-options.png "DMA options")

7. On the **Sources** screen, select **Add sources**.
8.
9. Enter the following into the **Connect to a server** dialog that appears on the right-hand side:

   - **Server name (1)**: Enter **SQLSERVER2008**.
   - **Authentication type (2)**: Select **SQL Server Authentication**.
   - **Username (3)**: Enter **PUWebSite**
   - **Password (4)**: Enter **{YOUR-ADMIN-PASSWORD}** `Password.1!!`
   - **Encrypt connection**: Check this box if not checked.
   - **Trust server certificate (5)**: Check this box.

   ![In the Connect to a server dialog, the values specified above are entered into the appropriate fields.](/assets/devops/dma-connect-to-a-server.png "Connect to a server")

10. Select **Connect (6)**.

11. On the **Add sources** dialog that appears next, check the box for `PartsUnlimited` **(1)** and select **Add (2)**.

    ![The PartsUnlimited box is checked on the Add sources dialog.](/assets/devops/dma-add-sources.png "Add sources")

12. Select **Start Assessment**.

    ![Start assessment](/assets/devops/dma-start-assessment-to-azure-sql-db.png "Start assessment")

13. Take a moment to review the assessment for migrating to Azure SQL DB. The SQL Server feature parity report **(1)** shows that Analysis Services and SQL Server Reporting Services are unsupported **(2)**, but these do not affect any objects in the `PartsUnlimited` database, so they won't block a migration.

    ![The feature parity report is displayed, and the two unsupported features are highlighted.](/assets/devops/dma-feature-parity-report.png "Feature parity")

14. Now, select **Compatibility issues (1)** so you can review that report as well.

    ![The Compatibility issues option is selected and highlighted.](/assets/devops/dma-compatibility-issues.png "Compatibility issues")

    The DMA assessment for migrating the `PartsUnlimited` database to a target platform of Azure SQL DB reveals that no issues or features are preventing Parts Unlimited from migrating their database to Azure SQL DB.

15. Select **Upload to Azure Migrate** to upload assessment results to Azure.

    ![Upload to Azure Migrate button is highlighted.](/assets/devops/dma-upload-azure-migrate.png "Azure Migrate Upload")

16. Select the right Azure environment **(1)** your subscription lives. Select **Connect (2)** to proceed to the Azure login screen.

    ![Azure is selected as the Azure Environment on the connect to Azure screen. Connect button is highlighted.](/assets/devops/dma-azure-migrate-upload.png "Azure Environment Selection")

17. Select your subscription **(2)** and the `partsunlimited` Azure Migrate project **(3)**. Select **Upload (4)** to start the upload to Azure.

    ![Upload to Azure Migrate page is open. Lab subscription and partsunlimited Azure Migrate Project are selected. Upload button is highlighted.](/assets/devops/dma-azure-migrate-upload-2.png "Azure Migrate upload settings")

    > **Note**: If you encounter **Failed to fetch subscription list from Azure, Strong Authentication required (1)** you might not see some of your subscriptions because of MFA limitations. You should still be able to see your lab subscription.

18. Once the upload is complete select **OK** and navigate to the Azure Migrate page on the Azure Portal.

    ![Assessment Uploaded dialog shown.](/assets/devops/dma-upload-complete.png "Assessment Uploaded")

19. Select the **Databases (1)** page on Azure Migrate. Observe the number of assessed database instances **(2)** and the number of databases ready for Azure SQL DB **(2)**. Keep in mind that you might need to wait for 5 to 10 minutes for results to show up. You can use the **Refresh** button on the page to see the latest status.

    ![Azure Migrate Databases page is open. The number of assessed database instances and the number of databases ready for Azure SQL DB shows one.](/assets/devops/dma-azure-migrate-web.png "Azure Migrate Database Assessment")

### Task 2: Retrieve connection information for SQL Databases

In this task, you will retrieve the IP address of the SqlServer2008 VM and the Fully Qualified Domain Name for the Azure SQL Database. This information is needed to connect to your SqlServer2008 VM and Azure SQL Database from Azure Data Migration Service and Azure Data Migration Assistant.

1. In the [Azure portal](https://portal.azure.com), navigate to your **SqlServer2008-ip** resource by selecting **Resource groups** from the Azure services list, selecting the **hands-on-lab-SUFFIX** resource group, and selecting the **SqlServer2008-ip** Public IP address from the list of resources.

   ![The SqlServer2008-ip IP address is highlighted in the list of resources.](/assets/devops/sqlip-selection.png "SqlServer2008 public IP resource")

2. In the **Overview** blade, select **Copy** to copy the public IP address and paste the value into a text editor, such as Notepad.exe, for later reference.

   ![SqlServer2008-ip resource is open. Public IP Address copy button is highlighted.](/assets/devops/sqlip-copy-public-ip.png "SqlServer2008 public IP")

3. Go back to the resource list and navigate to your **SQL database** resource by selecting the **parts** SQL database resource from the resources list.

   > **Note**: If you do not see the database resource, navigate to the SQL Server resource, then select **SQL Databases** and then the **parts** database.

   ![The parts SQL database resource is highlighted in the list of resources.](/assets/devops/resources-azure-sql-database.png "SQL database")

4. On the Overview blade of your SQL database, copy the **Server name** and paste the value into a text editor, such as Notepad.exe, for later reference.

   ![The server name value is highlighted on the SQL database Overview blade.](/assets/devops/sql-database-server-name.png "SQL database")

### Task 3: Migrate the database schema using the Data Migration Assistant

After reviewing the assessment results and ensuring the database is a candidate for migration to Azure SQL Database, use the Data Migration Assistant to migrate the schema to Azure SQL Database.

1. On the SqlServer2008 VM, return to the Data Migration Assistant and select the New **(+)** icon in the left-hand menu.

2. In the New project dialog, enter the following:

   - **Project type (1)**: Select Migration.
   - **Project name (2)**: Enter Migration.
   - **Source server type**: Select SQL Server.
   - **Target server type**: Select Azure SQL Database.
   - **Migration scope (3)**: Select Schema only.

   ![The above information is entered in the New Project dialog box.](/assets/devops/data-migration-assistant-new-project-migration.png "New Project dialog")

3. Select **Create (4)**.

4. On the **Select source** tab, enter the following:

   - **Server name (1)**: Enter **SQLSERVER2008**.
   - **Authentication type (2)**: Select **SQL Server Authentication**.
   - **Username (3)**: Enter **PUWebSite**
   - **Password (4)**: Enter **{YOUR-ADMIN-PASSWORD}** `Password.1!!`
   - **Encrypt connection**: Check this box.
   - **Trust server certificate (5)**: Check this box.
   - Select **Connect (6)**, and then ensure the `PartsUnlimited` database is selected **(7)** from the list of databases.

   ![The Select source tab of the Data Migration Assistant is displayed, with the values specified above entered into the appropriate fields.](/assets/devops/data-migration-assistant-migration-select-source.png "Data Migration Assistant Select source")

5. Select **Next (7)**.

6. On the **Select target** tab, enter the following:

   - **Server name (1)**: Paste the server name of your Azure SQL Database you copied into a text editor in the previous task.
   - **Authentication type (2)**: Select SQL Server Authentication.
   - **Username (3)**: Enter **demouser**
   - **Password (4)**: Enter **{YOUR-ADMIN-PASSWORD}** `Password.1!!`
   - **Encrypt connection**: Check this box.
   - **Trust server certificate (5)**: Check this box.
   - Select **Connect (6)**, and then ensure the `parts` database is selected **(7)** from the list of databases.

   ![The Select target tab of the Data Migration Assistant is displayed, with the values specified above entered into the appropriate fields.](/assets/devops/data-migration-assistant-migration-select-target.png "Data Migration Assistant Select target")

7. Select **Next (8)**.

8. On the **Select objects** tab, leave all the objects checked **(1)**, and select **Generate SQL script (2)**.

   ![The Select objects tab of the Data Migration Assistant is displayed, with all the objects checked.](/assets/devops/data-migration-assistant-migration-select-objects.png "Data Migration Assistant Select target")

9. On the **Script & deploy schema** tab, review the script. Notice the view also provides a note that there are no blocking issues **(1)**.

   ![The Script & deploy schema tab of the Data Migration Assistant is displayed, with the generated script shown.](/assets/devops/data-migration-assistant-migration-script-and-deploy-schema.png "Data Migration Assistant Script & deploy schema")

10. Select **Deploy schema (2)**.

11. After the schema is deployed, review the deployment results, and ensure there were no errors.

    ![The schema deployment results are displayed, with 23 commands executed and 0 errors highlighted.](/assets/devops/data-migration-assistant-migration-deployment-results.png "Schema deployment results")

12. Launch SQL Server Management Studio (SSMS) on the SqlServer2008 VM from the Windows Start menu by typing "sql server management" **(1)** into the search bar and then selecting **SQL Server Management Studio 17 (2)** in the search results.

    ![In the Windows Start menu, "sql server management" is entered into the search bar, and SQL Server Management Studio 17 is highlighted in the Windows start menu search results.](/assets/devops/smss-windows-search.png "SQL Server Management Studio 17")

13. Connect to your Azure SQL Database by selecting **Connect->Database Engine** in the Object Explorer and then entering the following into the Connect to server dialog:

    - **Server name (1)**: Paste the server name of your Azure SQL Database you copied above.
    - **Authentication type (2)**: Select SQL Server Authentication.
    - **Username (3)**: Enter **demouser**
    - **Password (4)**: Enter **{YOUR-ADMIN-PASSWORD}** `Password.1!!`
    - **Remember password (5)**: Check this box.

    ![The SSMS Connect to Server dialog is displayed, with the Azure SQL Database name specified, SQL Server Authentication selected, and the demouser credentials entered.](/assets/devops/ssms-connect-azure-sql-database.png "Connect to Server")

14. Select **Connect (6)**.

15. Once connected, expand **Databases**, and expand **parts**, then expand **Tables**, and observe the schema has been created **(1)**. Expand **Security > Users** to observe that the database user is migrated as well **(2)**.

    ![In the SSMS Object Explorer, Databases, parts, and Tables are expanded, showing the tables created by the deploy schema script. Security, Users are expended to show database user PUWebSite is migrated as well.](/assets/devops/ssms-databases-contosoinsurance-tables.png "SSMS Object Explorer")

### Task 4: Migrate the database using the Azure Database Migration Service

At this point, you have migrated the database schema using DMA. In this task, you migrate the data from the `PartsUnlimited` database into the latest Azure SQL Database using the Azure Database Migration Service.

> The [Azure Database Migration Service](https://docs.microsoft.com/azure/dms/dms-overview) integrates some of the functionality of Microsoft's existing tools and services to provide customers with a comprehensive, highly available database migration solution. The service uses the Data Migration Assistant to generate assessment reports that provide recommendations to guide you through the changes required before performing a migration. When you're ready to begin the migration process, Azure Database Migration Service conducts all necessary steps.

1. In the [Azure portal](https://portal.azure.com), navigate to your Azure Database Migration Service by selecting **Resource groups** from the Azure services list, selecting the **hands-on-lab-SUFFIX** resource group, and then selecting the **contoso-dms-UniqueId** Azure Database Migration Service in the list of resources.

   ![The contoso-dms Azure Database Migration Service is highlighted in the list of resources in the hands-on-lab-SUFFIX resource group.](/assets/devops/resource-group-dms-resource.png "Resources")

2. On the Azure Database Migration Service blade, select **+New Migration Project**.

   ![On the Azure Database Migration Service blade, +New Migration Project is highlighted in the toolbar.](/assets/devops/dms-add-new-migration-project.png "Azure Database Migration Service New Project")

3. On the New migration project blade, enter the following:

   - **Project name (1)**: Enter DataMigration.
   - **Source server type**: Select SQL Server.
   - **Target server type**: Select Azure SQL Database.
   - **Choose type of activity**: Select **Offline data migration** and select **Save**.

   ![The New migration project blade is displayed, with the values specified above entered into the appropriate fields.](/assets/devops/dms-new-migration-project-blade.png "New migration project")

4. Select **Create and run activity (2)**.

5. On the Migration Wizard **Select source** blade, enter the following:

   - **Source SQL Server instance name (1)**: Enter the IP address of your SqlServer2008 VM that you copied into a text editor in the previous task. For example, `51.143.12.114`.
   - **Authentication type (2)**: Select SQL Authentication.
   - **Username (3)**: Enter **PUWebSite**
   - **Password (4)**: Enter **{YOUR-ADMIN-PASSWORD}** `Password.1!!`
   - **Connection properties (5)**: Check both Encrypt connection and Trust server certificate.

   ![The Migration Wizard Select source blade is displayed, with the values specified above entered into the appropriate fields.](/assets/devops/dms-migration-wizard-select-source.png "Migration Wizard Select source")

6. Select **Next: Select databases >> (6)**.

7. PartsUnlimited database comes preselected. Select **Next: Select target >>** to continue.

   ![The Migration Wizard Select database blade is displayed. PartsUnlimited database is selected. Next: Select target >> button is highlighted.](/assets/devops/dms-migration-wizard-select-database.png "Migration Wizard Select databases")

8. On the Migration Wizard **Select target** blade, enter the following:

   - **Target server name (1)**: Enter the `fullyQualifiedDomainName` value of your Azure SQL Database (e.g., parts-xwn4o7fy6bcbg.database.windows.net), which you copied in the previous task.
   - **Authentication type (2)**: Select SQL Authentication.
   - **Username (3)**: Enter **demouser**
   - **Password (4)**: Enter **{YOUR-ADMIN-PASSWORD}** `Password.1!!`
   - **Connection properties**: Check Encrypt connection.

   ![The Migration Wizard Select target blade is displayed, with the values specified above entered into the appropriate fields.](/assets/devops/dms-migration-wizard-select-target.png "Migration Wizard Select target")

9. Select **Next: Map to target databases >> (5)**.

10. On the Migration Wizard **Map to target databases** blade, confirm that **PartsUnlimited (1)** is checked as the source database, and **parts (2)** is the target database on the same line, then select **Next: Configuration migration settings >> (3)**.

    ![The Migration Wizard Map to target database blade is displayed, with the ContosoInsurance line highlighted.](/assets/devops/dms-migration-wizard-map-to-target-databases.png "Migration Wizard Map to target databases")

11. On the Migration Wizard **Configure migration settings** blade, expand the **PartsUnlimited (1)** database and verify all the tables are selected **(2)**.

    ![The Migration Wizard Configure migration settings blade is displayed, with the expand arrow for PartsUnlimited highlighted, and all the tables checked.](/assets/devops/dms-migration-wizard-configure-migration-settings.png "Migration Wizard Configure migration settings")

12. Select **Next: Summary >> (3)**.

13. On the Migration Wizard **Summary** blade, enter the following:

    - **Activity name**: Enter PartsUnlimitedDataMigration.

    ![The Migration Wizard summary blade is displayed, with PartsUnlimitedDataMigration entered into the name field.](/assets/devops/dms-migration-wizard-migration-summary.png "Migration Wizard Summary")

14. Select **Start migration**.

15. Monitor the migration on the status screen that appears. Select the refresh icon in the toolbar to retrieve the latest status.

    ![On the Migration job blade, the Refresh button is highlighted, and a status of Full backup uploading is displayed and highlighted.](/assets/devops/dms-migration-wizard-status-running.png "Migration status")

    > The migration takes approximately 2 - 3 minutes to complete.

16. When the migration is complete, you should see the status as **Completed**.

    ![On the Migration job blade, the status of Completed is highlighted.](/assets/devops/dms-migration-wizard-status-complete.png "Migration with Completed status")

17. When the migration is complete, select the **PartsUnlimited** migration item.

    ![The ContosoInsurance migration item is highlighted on the PartsUnlimitedDataMigration blade.](/assets/devops/dms-migration-completion.png "PartsUnlimitedDataMigration details")

18. Review the database migration details.

    ![A detailed list of tables included in the migration is displayed.](/assets/devops/dms-migration-details.png "Database migration details")

19. If you received a status of "Warning" for your migration, you can find more details by selecting **Download report** from the ContosoDataMigration screen.

    ![The Download report button is highlighted on the DMS Migration toolbar.](/assets/devops/dms-toolbar-download-report.png "Download report")

    > **Note**: The **Download report** button will be disabled if the migration is completed without warnings or errors.

20. The reason for the warning can be found in the Validation Summary section. In the report below, you can see that a storage object schema difference triggered a warning. However, the report also reveals that everything was migrated successfully.

    ![The output of the database migration report is displayed.](/assets/devops/dms-migration-wizard-report.png "Database migration report")

### Task 5: Configure the application connection to SQL Azure Database

Now that we have both our application and database migrated to Azure. It is time to configure our application to use the SQL Azure Database.

1. In the [Azure portal](https://portal.azure.com), navigate to your `parts` SQL Database resource by selecting **Resource groups** from Azure services list, selecting the **hands-on-lab-SUFFIX** resource group, and selecting the `parts` SQL Database from the list of resources.

   ![The parts SQL database resource is highlighted in the list of resources.](/assets/devops/resources-azure-sql-database.png "SQL database")

2. Switch to the **Connection strings (1)** blade, and copy the connection string by selecting the copy button **(2)**.

   ![Connection string panel if SQL Database is open. Copy button for ADO.NET connection string is highlighted.](/assets/devops/sql-connection-string-copy.png "Database connection string")

3. Paste the value into a text editor, such as Notepad.exe, to replace the Password placeholder. Replace the `{your_password}` section with your admin password. Copy the entire connection string with the replaced password for later use.

   ![Notepad is open. SQL Connection string is pasted in. {your_password} placeholder is highlighted.](/assets/devops/sql-connection-string-password-replace.png "Database connection string")

4. Go back to the resource list, navigate to your `partsunlimited-web-{uniquesuffix}` **(2)** App Service resource. You can search for `partsunlimited-web` **(1)** to find your Web App and App Service Plan.

   ![The search box for resource is filled in with partsunlimited-web. The partsunlimited-web-20 Azure App Service is highlighted in the list of resources in the hands-on-lab-SUFFIX resource group.](/assets/devops/resource-group-appservice-resource.png "Resources")

5. Switch to the **Configuration (1)** blade, and select **+New connection string (2)**.

   ![App service configuration panel is open. +New connection string button is highlighted.](/assets/devops/app-service-settings.png "App Service Configuration")

6. On the **Add/Edit connection string** panel, enter the following:

   - **Name(1)**: Enter `DefaultConnectionString`.
   - **Value**: Enter SQL Connection String you copied in Step 3.
   - **Type (3)**: Select **SQLAzure**
   - **Deployment slot setting (4)**: Check this option to make sure connection strings stick to a deployment slot. This will be helpful when we add additional deployment slots during the next exercises.

   ![Add/Edit Connection string panel is open. The name field is set to DefaultConnectionString. The value field is set to the connection string copied in a previous step. Type is set to SQL Azure. The deployment slot setting checkbox is checked. OK button is highlighted. ](/assets/devops/app-service-connection-string.png "Adding connection string")

7. Select **OK (5)**.

8. Select **Save** and **Continue** for the following confirmation dialog.

   ![App Service Configuration page is open. Save button is highlighted.](/assets/devops/app-service-settings-save.png "App Service Configuration")

9. Switch to the **Overview (1)** blade, and select **URL (2)** to navigate to the Parts Unlimited website hosted in our Azure App Service using Azure SQL Database.

   ![Overview panel for the App Service is on screen. URL for the app service is highlighted.](/assets/devops/app-service-navigate-to-app-url.png "App Service public URL")

You should follow all steps provided _after_ attending the Hands-on lab.
