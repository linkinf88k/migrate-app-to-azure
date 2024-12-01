# TechConf Registration Website

## Project Overview
The TechConf website allows attendees to register for an upcoming conference. Administrators can also view the list of attendees and notify all attendees via a personalized email message.

The application is currently working but the following pain points have triggered the need for migration to Azure:
 - The web application is not scalable to handle user load at peak
 - When the admin sends out notifications, it's currently taking a long time because it's looping through all attendees, resulting in some HTTP timeout exceptions
 - The current architecture is not cost-effective 

In this project, you are tasked to do the following:
- Migrate and deploy the pre-existing web app to an Azure App Service
- Migrate a PostgreSQL database backup to an Azure Postgres database instance
- Refactor the notification logic to an Azure Function via a service bus queue message

## Dependencies

You will need to install the following locally:
- [Postgres](https://www.postgresql.org/download/)
- [Visual Studio Code](https://code.visualstudio.com/download)
- [Azure Function tools V3](https://docs.microsoft.com/en-us/azure/azure-functions/functions-run-local?tabs=windows%2Ccsharp%2Cbash#install-the-azure-functions-core-tools)
- [Azure CLI](https://docs.microsoft.com/en-us/cli/azure/install-azure-cli?view=azure-cli-latest)
- [Azure Tools for Visual Studio Code](https://marketplace.visualstudio.com/items?itemName=ms-vscode.vscode-node-azure-pack)

## Project Instructions

### Part 1: Create Azure Resources and Deploy Web App
1. Create a Resource group
2. Create an Azure Postgres Database single server
   - Add a new database `techconfdb`
   - Allow all IPs to connect to database server
   - Restore the database with the backup located in the data folder
3. Create a Service Bus resource with a `notificationqueue` that will be used to communicate between the web and the function
   - Open the web folder and update the following in the `config.py` file
      - `POSTGRES_URL`
      - `POSTGRES_USER`
      - `POSTGRES_PW`
      - `POSTGRES_DB`
      - `SERVICE_BUS_CONNECTION_STRING`
4. Create App Service plan
5. Create a storage account
6. Deploy the web app

### Part 2: Create and Publish Azure Function
1. Create an Azure Function in the `function` folder that is triggered by the service bus queue created in Part 1.

      **Note**: Skeleton code has been provided in the **README** file located in the `function` folder. You will need to copy/paste this code into the `__init.py__` file in the `function` folder.
      - The Azure Function should do the following:
         - Process the message which is the `notification_id`
         - Query the database using `psycopg2` library for the given notification to retrieve the subject and message
         - Query the database to retrieve a list of attendees (**email** and **first name**)
         - Loop through each attendee and send a personalized subject message
         - After the notification, update the notification status with the total number of attendees notified
2. Publish the Azure Function

### Part 3: Refactor `routes.py`
1. Refactor the post logic in `web/app/routes.py -> notification()` using servicebus `queue_client`:
   - The notification method on POST should save the notification object and queue the notification id for the function to pick it up
2. Re-deploy the web app to publish changes

## Monthly Cost Analysis
Complete a month cost analysis of each Azure resource to give an estimate total cost using the table below:

| Azure Resource | Service Tier | Monthly Cost |
| ------------ | ------------ | ------------ |
| *Azure Postgres Database* | *General Purpose Tier - Flexible Server Deployment* | *$125.41* |
| *Azure Service Bus* | *Standard tier* | *$9.81* |
| *App Service* | *Standard Tier* | *$58.4* |
| *Azure Functions* | *Consumption Tier* | *...* |
| *Storage Accounts* | *Block Blob Storage, General Purpose V2* | *$21.84* |

## Architecture Explanation
### Overview
The architecture for the TechConf Registration Website leverages Azure's robust cloud services to provide a scalable, efficient, and cost-effective platform. Key components include an Azure Web App for hosting the web application, an Azure Function for processing notifications using a Service Bus queue, and supporting services such as Azure Database for PostgreSQL, Service Bus, and a Storage Account.

### Deployment
Azure Web App: The web application is hosted on Azure App Service, a fully managed platform offering seamless deployment, scaling, and automatic updates.
Azure Function: The notification service is implemented as a serverless compute function, triggered by messages in the Service Bus queue. This architecture eliminates the need for manual infrastructure management and supports on-demand execution.
### Cost Efficiency
The architecture emphasizes cost-effectiveness by utilizing Azure services optimized for small to medium workloads:

Azure Database for PostgreSQL: Configured in the Basic Tier for an ideal balance of performance and cost.
Azure App Service: Deployed on a Standard Tier plan (1 S2: 2 cores, 1.75 GB RAM, 50 GB storage) suitable for moderate traffic volumes.
Azure Function: Operates on the Consumption Plan, ensuring charges only for actual resource usage.
Email Service: Uses the Free Tier of SendGrid, accommodating email notifications at no cost within defined limits.
### Risks
Database Cost Escalation: Growth in database size or high traffic levels may increase costs.
Mitigation: Implement monitoring tools and data optimization techniques to manage costs.
Notification Delays: Large-scale notification processing could lead to Azure Function timeouts.
Mitigation: Configure appropriate timeout settings and utilize batch processing to handle large workloads efficiently.
### Future Expansion
The architecture is designed to support scalability and future enhancements:

Scalability: Azure Web App and Azure Function can be scaled horizontally to handle peak user loads. Service Bus queues can be scaled for higher messaging volumes.
Additional Features: Easily integrate additional services such as:
Azure Cache for Redis: Improve database performance by reducing query load.
Azure CDN: Accelerate content delivery and enhance user experience.
Azure API Management: Add API-level management, security, and analytics.
### Summary
This Azure-based architecture provides a scalable, flexible, and cost-effective solution for the TechConf Registration Website. By addressing current pain points and incorporating features for future growth, the platform ensures robust performance, efficient resource utilization, and seamless user experiences, all while minimizing risks and costs.
