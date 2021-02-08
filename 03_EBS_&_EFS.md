# EBS

## EBS - Elastic Block Storage

* An EC2 machine loses its root volume when terminated
* EBS (Elastic Block Store) Volume is a network drive you can attach to your instances to persist data
* It’s a network drive attached storage not local. There is read write latancy.
* It can be detacted and attached freely
* It’s locked to an Availability Zone (you can snapshot then copied to another)
* Have a provisioned capacity (GB or IOPS) 
  * Your billed for provisioned capacilty and not storage used
 
 ### EBS Volume Types
1. **GP2*** (SSD): General purpose SSD volume that balances price and performance for a wide variety of workloads
2. **IO1*** (SSD): Highest-performance SSD volume for mission-critical low-latency or high - throughput workloads
3. **IST1** (HDD): Low cost HDD volume designed for frequently accessed, throughput - intensive workloads
4. **ISC1** (HDD): Lowest cost HDD volume designed for less frequently accessed workloads

**Only GP2 and IO1 can be used as boot volumes**

### GP2
* Use case
  * Recommended for most workloads
  * System boot volumes
  * Virtual desktops
  * Low-latency interactive apps
  * Development and test environments
* 1 GiB - 16TiB
* Small gp2 volumes can burst IOPS to 3000
* Max IOPS is 16,000...
* 3 IOPS per GB, means at 5,334GB we are at the max IOPS

### IO1
* Use case
  * Critical business applications that require sustained IOPS performance, or more than 16,000 IOPS per volume (gp2 limit)
  * Large database workloads, such as: MongoDB, Cassandra, Microsoft SQL Server, MySQL, PostgreSQL, Oracle
* 4 GiB - 16TiB
* IOPS is provisioned (PIOPS) – MIN 100 - MAX 64,000 (Nitro instances) else MAX 32,000 
* The maximum ratio of provisioned IOPS to requested volume size (in GiB) is 50:1

### IST1
* Use case
  * Streaming workloads requiring consistent, fast throughput at a low price. 
  * Big data, Data warehouses, Log processing Apache Kafka
  * Cannot be a boot volume
* 500 GiB - 16TiB
* Max IOPS is 500
* Max throughput of 500 MiB/s – can burst

### ISC1
* Use case
  * Throughput-oriented storage for large volumes of data that is infrequently accessed
  * Scenarios where the lowest storage cost is important
  * Cannot be a boot volume
* 500 GiB - 16TiB
* Max IOPS is 250
* Max throughput of 250 MiB/s – can burst

### EBS –VolumeTypes Summary
* **gp2:** General Purpose Volumes (cheap)
  * 3 IOPS / GiB, minimum 100 IOPS, burst to 3000 IOPS, max 16000 IOPS 
  * 1 GiB – 16TiB ,+1TB = +3000 IOPS
* **io1:** Provisioned IOPS (expensive)
  * Min 100 IOPS, Max 64000 IOPS (Nitro) or 32000 (other) 
  * 4 GiB - 16 TiB. Size of volume and IOPS are independent
* **st1:** Throughput Optimized HDD
  * 500 GiB – 16 TiB , 500 MiB /s throughput
* **sc1:** Cold HDD, Infrequently accessed data 
  * 500 GiB – 16 TiB , 250 MiB /s throughput
  
### EBS Snapshots
* Incremental – only backup changed blocks
* EBS backups use IO and you shouldn’t run them while your application is handling a lot of traffic
* Snapshots will be stored in S3 (but you won’t directly see them)
* Not necessary to detach volume to do snapshot, but recommended
* Max 100,000 snapshots
* Can make Image (AMI) from Snapshot
* EBS volumes restored by snapshots need to be pre-warmed (using fio or dd command to read the entire volume)
* Snapshots can be automated using Amazon Data Lifecycle Manager

### EBS Migration
* EBS Volumes are only locked to a specific AZ
* To migrate it to a different AZ (or region):
  * Snapshot the volume
  * (optional) Copy the volume to a different region
  * Create a volume from the snapshot in the AZ of your choice
  
### EBS Encryption
* When you create an encrypted EBS volume, you get the following:
  * Data at rest is encrypted inside the volume
  * All the data in flight moving between the instance and the volume is encrypted 
  * All snapshots are encrypted
  * All volumes created from the snapshot
* Encryption and decryption are handled transparently (you have nothing to do)
* Encryption has a minimal impact on latency
* EBS Encryption leverages keys from KMS (AES-256)
* Copying an unencrypted snapshot allows encryption
* Snapshots of encrypted volumes are encrypted

### Encryption: encrypt an unencrypted EBS volume
* Create an EBS snapshot of the EBS volume
* Create a copy of the snapshot and Encrypt the new copy
* Create new ebs volume from the snapshot 
* Attach the encrypted volume to the original instance

### EBS vs Instance Store(local storage)
* Some instance do not come with Root EBS volumes
* Instead, they come with “Instance Store” (= ephemeral storage)
* Instance store is physically attached to the machine (EBS is a network drive)
* Pros:
  * Better I/O performance (EBS gp2 has an max IOPS of 16000, io1 of 64000)
  * Good for buffer / cache / scratch data / temporary content 
  * Data survives reboots
* Cons:
  * On stop or termination, the instance store is lost
  * You can’t resize the instance store
  * Backups must be operated by the user

### Local EC2 Instance Store
* Physical disk attached to the physical server
* Very High IOPS (because physical not network attached)
* Disks up to 7.5 TiB, stripped to reach 30 TiB
* Block Storage (just like EBS)
* Cannot be increased in size
* Risk of data loss if hardware fails

### EBS RAID Options
* EBS is already redundant storage (replicated within an AZ)
* Really high IOPS - 100 000 IOPS
* Mirror your EBS volumes - mount volumes in parallel in RAID settings
* RAID is possible as long as your OS supports it
* Some RAID options are: 
  * RAID 0 (strips disks - no redundancy)
  * RAID 1 (redundent)
 
### RAID 0 (Increase IOPS Performance)
* **Not Redundent** - if a disk fails you will lose data
* Combines 2 or more volumes 
  * 2 x 500 GiB volumes with 4,000 provisioned IOPS each
  * Creates 1000 GiB RAID 0 array with an available bandwidth of 8,000 IOPS and 1,000 MB/s of throughput
* Use cases:
  * An application that needs a lot of IOPS and doesn’t need fault-tolerance
  * A database that has replication already built-in

### RAID 1 (Increase Fault Tolerance)
* RAID 1 = disk mirroring
* If 1 disk fails the other has a copy of the data
* We have to send the data to two EBS volume at the same time (2x network)
* Utalises 2 or more volumes 
  * 2 x 500 GiB volumes with 4,000 provisioned IOPS each 
  * Creates 1500 GiB RAID 1 array with an available bandwidth of 4,000 IOPS and 500 MB/s of throughput
* Use cases:
  * Application that need increase volume fault tolerance
  * Application where you need to service disks
  
## EFS – Elastic File System
* Can be mounted on many EC2
* Multi-AZ
* Highly available
* Expensive (3xEBS-GP2)
* Uses NFSv4.1 protocol to transport data
* POSIX file system - Linux only
* Uses security group to control access
* Pay-per-use, not pay for capacity like EBS

### EFS – Performance & Storage Classes
* EFS Scale
  * 1000s of concurrent NFS clients, 10 GB+ /s throughput
  * Grow to Petabyte-scale network file system, automatically
* Performance mode (set at EFS creation time)
  * General purpose (default): latency-sensitive use cases (web server, CMS, etc...) 
  * Max I/O – higher latency, throughput, highly parallel (big data, media processing)
* StorageTiers (lifecycle management feature – move file after N days)
  * Standard: for frequently accessed files
  * Infrequent access (EFS-IA): cost to retrieve files, lower price to store
  
### EBS vs EFS – Elastic Block Storage
* EBS volumes...
  * can be attached to only one instance at a time
  * are locked at the Availability Zone (AZ) level 
  * gp2: IO increases if the disk size increases
  * io1: can increase IO independently
* To migrate an EBS volume across AZ
  * Take a snapshot
  * Restore the snapshot to another AZ
  * EBS backups use IO and you shouldn’t run them while your application is handling a lot of traffic
* Root EBS Volumes of instances get terminated by default if the EC2 instance gets terminated. (you can disable that)

### EBS vs EFS – Elastic File System
* Mounting 100s of instances across AZ 
* EFS share website files (WordPress)
* Only for Linux Instances (POSIX)
* EFS has a higher price point than EBS 
* Can leverage EFS-IA for cost savings
* Remember : EFS vs EBS vs Instance Store

PostgreSQL: 5432
MySQL: 3306
Oracle RDS: 1521
MSSQL Server: 1433
MariaDB: 3306 (same as MySQL)
Aurora: 5432 (if PostgreSQL compatible) or 3306 (if MySQL compatible)
