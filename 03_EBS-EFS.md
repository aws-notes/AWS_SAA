#EBS & EFS

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
2. **IO1**I (SSD): Highest-performance SSD volume for mission-critical low-latency or high - throughput workloads
3. **IST1**I (HDD): Low cost HDD volume designed for frequently accessed, throughput - intensive workloads
4. **ISC1**I (HDD): Lowest cost HDD volume designed for less frequently accessed workloads

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
