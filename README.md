# Azure Database Migration

## Table of contents
1. [Milestone 1: Set up the Environment](#Milestone_1)
2. [Milestone 2: Set up the Production Environment](#Milestone_2)
3. [Milestone 3: Migrate to Azure SQL Database](#Milestone_3)
4. [Milestone 4: Set up the Development Environment and Introduce Regular Backups](#Milestone_4)
5. [Milestone 5: Disaster Recovery Simulation](#Milestone_5)
6. [Milestone 6: Geo-Replication for Azure SQL Database](#Milestone_6)
7. [Milestone 7: Microsoft Entra Integration](#Milestone_7)

Within his project I establish a production environment database on a virtual machine with the AdventureWorks database. I then migrate this database to an Azure SQL Database. This ensures there is a backup which is easily accessible.

To increase data resiliance a developement enviroment is created on another virtual machine. The AdventureWorks database is imported from a blob storage container. Scheduled backups are also set up to increase resiliance further. 

A pivotal phase of the project involves simulating a disaster recovery scenario with potential data loss. This will demonstate how these backups can be utilised. Furthermore, it will explore the complexities of geo-replication and failover configuration to ensure data availability even under challenging conditions.

To enhance security, Microsoft Entra ID integration will then be employed to define access roles, adding an extra layer of control and protection.

## Milestone 1: Set up the Environment <a name="Milestone_1"></a>

- I set up a repository on GitHub to track progress.

## Milestone 2: Set up the Production Environment <a name="Milestone_2"></a>

- I set up a Windows Virtual Machine (VM) to serve as the cornerstone of the production environment. The specified VM uses the operating system Windows 11 Pro and is Standard B2ms (2 vcpus, 8 GiB memory).

## Milestone 3: Migrate to Azure SQL Database <a name="Milestone_3"></a>

- Created an Azure SQL Database, which served as the target for database migration.
- Installed and configured Azure Data Studio on the production Windows VM. I used this tool to establish a connection to the existing on-premise database.
- I leveraged this extension to compare and subsequently migrated the schema from the on-premise database to the Azure SQL Database.
- Installed the Azure SQL Migration extension within Azure Data Studio.
- This extension facilitated a smooth transfer of data from the on-premise database to the Azure SQL Database, ensuring a successful and seamless data migration process.

![Database Connections in Azure Data Studio](./images/SSMS-Person-Top1000.png)

- I confirmed the success of the database migration process by comparing the schema of tables. I also chose a table at random and ensured the first 1000 rows had identical entries.

![Looking at Original Person.Person table](./images/SSMS-Person-Top1000.png)

![Comparing with Azure Person.Person table](./images/Azure-Person-Top1000.png)

## Milestone 4: Set up the Development Environment and Introduce Regular Backups <a name="Milestone_4"></a>

- I began by generating a full backup of the production database hosted on the Windows VM.

![Creating a Backup of the Production Database](./images/SSMS-Backup.png)

- Once the backup was completed, it was stored in 'C:\Program Files\Microsoft SQL Server\MSSQL16.MSSQLSERVER\MSSQL\Backup/'.
- The file was then uploaded to a storage container within Microsoft Azure.
- I set up another virtual machine to act as the development environment. I installed SQL Server and SSMS just as I did on the production environment. I then downloaded the backup file and loaded it into SSMS.

![Blob Containing AdventureWorks Backup](./images/Blob-backup.png)

- I then automated backups from the specified blob. To do this, I required creating a SQL Server Credential in SSMS. A SQL Server Credential is a security object that allows SQL Server to access external resources securely. To create a SQL Server Credential in SSMS, I right-clicked on the server name and selected New Query to open a new query window. Then, I executed the following T-SQL command to create the credentials:

![Creating a Security Credential](./images/Creating-security-credentials.png)

- After doing this, backups from the blob could then be specified. This was done weekly on a Sunday night to minimize impact on the business.

![Automating Backups](./images/Maintenance-plan.png)


## Milestone 5: Disaster Recovery Simulation <a name="Milestone_5"></a>

- I deliberately removed critical data from the production database to replicate a scenario where data integrity is compromised. The SQL code executed for this action is as follows:

  ```sql
  -- Intentional Deletion
  DELETE TOP (100)
  FROM Sales.SalesOrderDetail;

  -- Data Corruption
  UPDATE TOP (100) Purchasing.Vendor
  SET PurchasingWebServiceURL = NULL;

- After executing this code, I confirmed its success by comparing it to the local SQL Database using the connection already established in Azure Data Studio. I observed that in the SalesOrderDetail table, entries had fallen from 121,317 to 121,217. Additionally, I confirmed the loss of several URLs.
- To initiate the recovery process, I accessed the database within the Azure Cloud and selected the "Restore database" window. I chose to restore the database to a state an hour before the changes had taken place. During this process, I ensured to [mention any specific options or configurations chosen during the restoration]. After completion, I confirmed that the tables were back to their original specifications.

## Milestone 6: Geo-Replication for Azure SQL Database <a name="Milestone_6"></a>

- I set up geo-replication for the production Azure SQL Database. This process involved creating a synchronized replica of your primary database. The replica resided on a separate SQL server located in a different geographical region from your primary database server. This geographical separation is crucial to bolsters redundancy and resilience, minimizing shared risks.
- I then orchestrated a planned failover to the secondary region. This act transitions operations to the secondary copy. I then evaluated the availability and data consistency of the failover database.
- Afterward, I performed a failback to the primary region, demonstrating the cyclical nature of my failover strategy.

## Milestone 7: Microsoft Entra Integration <a name="Milestone_7"></a>

- I configured Microsoft Entra for the Azure Database.
- I created a DB reader user so that users are free to analyze the database without it being compromised.


## License information

Copyright (c) 2024 Robert Kay

Permission is hereby granted, free of charge, to any person obtaining a copy of this software and associated documentation files (the "Software"), to deal in the Software without restriction, including without limitation the rights to use, copy, modify, merge, publish, distribute, sublicense, and/or sell copies of the Software, and to permit persons to whom the Software is furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in all copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS," WITHOUT WARRANTY OF ANY KIND, EXPRESS OR IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY, FITNESS FOR A PARTICULAR PURPOSE, AND NONINFRINGEMENT. IN NO EVENT SHALL THE AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES, OR OTHER LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT, OR OTHERWISE, ARISING FROM, OUT OF, OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE SOFTWARE.