# RDS, Aurora & ElastiCache

## RDS

* It allows you to create databases in the cloud that are managed by AWS
  * Postgres
  * MySQL
  * MariaDB
  * Oracle
  * Microsoft SQL Server
  * Aurora (AWS Proprietary database)
 
## Advantage over using RDS versus deploying DB on EC2
* RDS is a managed service:
* Automated provisioning, OS patching 
* Continuous backups and restore to specific timestamp (Point in Time Restore)!
* Monitoring dashboards
* Read replicas for improved read performance
* Multi AZ setup for DR (Disaster Recovery) 
* Maintenance windows for upgrades
* Scaling capability (vertical and horizontal) 
* Storage backed by EBS (gp2 or io1) 

## RDS Backups
* Backups are automatically enabled in RDS
* Automated backups:
  * Daily full backup of the database (during the maintenance window)
  * Transaction logs are backed-up by RDS every 5 minutes
  * => ability to restore to any point in time (from oldest backup to 5 minutes ago) • 7 days retention (can be increased to 35 days)
* DB Snapshots:
  * Manually triggered by the user
  * Retention of backup for as long as you want

## RDS Read Replicas for read scalability
* Up to 5 Read Replicas
* Within AZ, Cross AZ or Cross Region
* Replication is ASYNC, so reads are **eventually consistent**
* Replicas can be promoted to their own DB
* Applications **must update the connection string** to leverage read replicas

## RDS Read Replicas – Use Cases
* You have a production database that is taking on normal load
* You want to run a reporting application to run some analytics
* You create a Read Replica to run the new workload there
* The production application is unaffected
* Read replicas are used for SELECT (=read) only kind of statements (not INSERT, UPDATE, DELETE)

## RDS Read Replicas – Network Cost
* In AWS there’s a network cost when data goes from one AZ to another 
* To reduce the cost, you can have your Read Replicas in the same AZ

## RDS Multi AZ (Disaster Recovery)
* SYNC replication
* One **DNS name** – automatic app failover to standby
* Increase availability
* Failover in case of loss of AZ, loss of network, instance or storage failure
* No manual intervention in apps
* Not used for scaling
* Note:The Read Replicas be setup as Multi AZ for Disaster Recovery (DR)

## RDS Security - Encryption
* At rest encryption
  * Possibility to encrypt the master & read replicas with AWS KMS - AES-256 encryption
  * Encryption has to be defined at launch time
  * If the master is not encrypted, the read replicas cannot be encrypted
  * Transparent Data Encryption (TDE) available for Oracle and SQL Server
* In-flight encryption
  * SSL certificates to encrypt data to RDS in flight
  * Provide SSL options with trust certificate when connecting to database • To enforce SSL:
    * PostgreSQL: rds.force_ssl=1 in the AWS RDS Console (Parameter Groups)
    * MySQL:WithintheDB:*GRANT USAGE ON *.* TO 'mysqluser'@'%' REQUIRE SSL;*
    
## RDS Encryption Operations
* Encrypting RDS backups
  * Snapshots of un-encrypted RDS databases are un-encrypted 
  * Snapshots of encrypted RDS databases are encrypted
  * Can copy a snapshot into an encrypted one
* To encrypt an un-encrypted RDS database: 
  * Create a snapshot of the un-encrypted database
  * Copy the snapshot and enable encryption for the snapshot
  * Restore the database from the encrypted snapshot
  * Migrate applications to the new database, and delete the old database

