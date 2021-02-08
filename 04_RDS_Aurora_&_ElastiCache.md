# RDS, Aurora & ElastiCache

1. [RDS](https://github.com/aws-notes/AWS_SAA/blob/main/04_RDS_Aurora_&_ElastiCache.md#rds)
2. [Aurora](https://github.com/aws-notes/AWS_SAA/blob/main/04_RDS_Aurora_&_ElastiCache.md#aurora)

## RDS

* It allows you to create databases in the cloud that are managed by AWS
  * Postgres
  * MySQL
  * MariaDB
  * Oracle
  * Microsoft SQL Server
  * Aurora (AWS Proprietary database)
 
### Advantage over using RDS versus deploying DB on EC2
* RDS is a managed service:
* Automated provisioning, OS patching 
* Continuous backups and restore to specific timestamp (Point in Time Restore)!
* Monitoring dashboards
* Read replicas for improved read performance
* Multi AZ setup for DR (Disaster Recovery) 
* Maintenance windows for upgrades
* Scaling capability (vertical and horizontal) 
* Storage backed by EBS (gp2 or io1) 

### RDS Backups
* Backups are automatically enabled in RDS
* Automated backups:
  * Daily full backup of the database (during the maintenance window)
  * Transaction logs are backed-up by RDS every 5 minutes
  * => ability to restore to any point in time (from oldest backup to 5 minutes ago) 
  * 7 days retention (can be increased to 35 days)
* DB Snapshots:
  * Manually triggered by the user
  * Retention of backup for as long as you want

### RDS Read Replicas for read scalability
* Up to 5 Read Replicas
* Within AZ, Cross AZ or Cross Region
* Replication is ASYNC, so reads are **eventually consistent**
* Replicas can be promoted to their own DB
* Applications **must update the connection string** to leverage read replicas

### RDS Read Replicas – Use Cases
* You have a production database that is taking on normal load
* You want to run a reporting application to run some analytics
* You create a Read Replica to run the new workload there
* The production application is unaffected
* Read replicas are used for SELECT (=read) only kind of statements (not INSERT, UPDATE, DELETE)

### RDS Read Replicas – Network Cost
* In AWS there’s a network cost when data goes from one AZ to another 
* To reduce the cost, you can have your Read Replicas in the same AZ

### RDS Multi AZ (Disaster Recovery)
* SYNC replication
* One **DNS name** – automatic app failover to standby
* Increase availability
* Failover in case of loss of AZ, loss of network, instance or storage failure
* No manual intervention in apps
* Not used for scaling
* Note:The Read Replicas be setup as Multi AZ for Disaster Recovery (DR)

### RDS Security - Encryption
* **At rest encryption**
  * Possibility to encrypt the master & read replicas with AWS KMS - AES-256 encryption
  * Encryption has to be defined at launch time
  * If the master is not encrypted, the read replicas cannot be encrypted
  * Transparent Data Encryption (TDE) available for Oracle and SQL Server
* **In-flight encryption**
  * SSL certificates to encrypt data to RDS in flight
  * Provide SSL options with trust certificate when connecting to database 
  * To enforce SSL:
    * PostgreSQL: rds.force_ssl=1 in the AWS RDS Console (Parameter Groups)
    * MySQL:WithintheDB:*GRANT USAGE ON *.* TO 'mysqluser'@'%' REQUIRE SSL;*
    
### RDS Encryption Operations
* **Encrypting RDS backups**
  * Snapshots of un-encrypted RDS databases are un-encrypted 
  * Snapshots of encrypted RDS databases are encrypted
  * Can copy a snapshot into an encrypted one
* **To encrypt an un-encrypted RDS database:**
  * Create a snapshot of the un-encrypted database
  * Copy the snapshot and enable encryption for the snapshot
  * Restore the database from the encrypted snapshot
  * Migrate applications to the new database, and delete the old database

### RDS Security – Network & IAM
* **Network Security**
  * RDS databases are usually deployed within a private subnet, not in a public one
  * RDS security works by leveraging security groups (the same concept as for EC2 instances) – it controls which IP / security group can communicate with RDS
* **Access Management**
  * **IAM policies** help control who can manage AWS RDS (through the RDS API)
  * **Username and Password** can be used to login into the database
  * **IAM-based authentication** can be used to login into MySQL & PostgreSQL
  
### RDS - IAM Authentication
* IAM database authentication works with MySQL and PostgreSQL
* You don’t need a password, just an authentication token obtained through IAM & RDS API calls
* Auth token has a lifetime of 15 minutes
  1. Service assumes a role with the correct permissions
  2. Service calls to the RDS Service via API
  3. Service receives API Auth Token
  4. Service uses Token auth to RDS instance
* Benefits:
  * Network in/out must be encrypted using SSL
  * IAM to centrally manage users instead of DB
  * Can leverage IAM Roles and EC2 Instance profiles for easy integration
  
### RDS Security – Summary
* Encryption at rest:
  * Is done only when you first create the DB instance
  * or: unencrypted DB => snapshot => copy snapshot as encrypted => create DB from snapshot
* Your responsibility:
  * Check the ports / IP / security group inbound rules in DB’s SG
  * In-database user creation and permissions or manage through IAM
  * Creating a database with or without public access
  * Ensure parameter groups or DB is configured to only allow SSL connections
* AWS responsibility:
  * No SSH access
  * No manual DB patching
  * No manual OS patching
  * No way to audit the underlying instance
  
## Aurora
