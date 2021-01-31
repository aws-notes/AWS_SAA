# EC2

## Security Groups

* Security groups are acting as a “firewall” on EC2 instances 
* Default setting **block** all inbound **allow** all outbound
* Can allow connections from IP, CIDR block, Other security groups
* Elastic IP(EIP) - static public IP. - Limited to 5, billed for unused EIPS
* Pubic IP - Public IP - Change when a instance reboots
* Private IP - Private IP - Persistent even after reboot 
* EC2 User Data - bash scripts only run once at the instance first start (sudo access)

## EC2 Instance LaunchTypes 

  * EC2 On Demand 
    * Has the highest cost 
    * Recommended for short-term and un-interrupted workloads 
  * EC2 Reserved Instances 
    * Up to 75% discount compared to On-demand
    * Reserve a specific instance type
    * For steady state usage applications (think database) 
    * **Convertible Reserved Instance**
       * Can change the EC2 instance type
       * Up to 54% discount 
    * **Scheduled Reserved Instances**
       * Launch within time window you reserve
       * When you require a fraction of day / week / month 
    * **EC2 Spot Instances**
       * Discount of up to 90% compared to On-demand 
       * Instances that you can “lose” at any point of time
       * Useful for Batch jobs, Data analysis, Image processing 
       * Great combo: Reserved Instances for baseline + On-Demand & Spot for peaks 
    * **EC2 Dedicated Hosts** - Full dedicated server - full control
       * Physical dedicated EC2 server for your use
       * Full control of EC2 Instance placement
       * Visibility into the underlying sockets / physical cores of the hardware    
       * Good fro BYOL
    * **EC2 Dedicated Instances** - Full dedicated server - partial control
       * Instances running on hardware that’s dedicated to you 
       * Share hardware with other instances in same account 
       * No control over instance placement 

* EC2 Spot instances
  * **Spot Instance Requests**
    * Discount of up to 90% compared to On-demand
    * Define max spot price. When spot price > current spot price = instance stop/terminate
  * **Spot Block Requests**
    * Book spot instance during a specified time frame (1 to 6 hours) without interruptions 
  * Cancelling a Spot Request does not terminate instances!!
  * You must first cancel a Spot Request, and then terminate the associated Spot Instances 
  * **Spot Fleets**
    * Spot Fleets = set of Spot Instances + (optional) On-Demand Instances 
      * Spot Fleets allow us to automatically request Spot Instances with the lowest price
      * Define possible launch pools: instance type (m5.large), OS, Availability Zone
      * Can have multiple launch pools, so that the fleet can choose
      * Spot Fleet stops launching instances when reaching capacity or max cost 
    * Strategies to allocate Spot Instances:
      * **LowestPrice:** from the pool with the lowest price (cost optimization, short workload) 
      * **Diversified:** distributed across all pools (great for availability, long workloads)
      * **CapacityOptimized:** pool with the optimal capacity for the number of instances 

* Types of instance
  * **R:** RAM – in-memory caches   * **C:** CPU – compute / databases   * **M:** Medium / balanced – general / web app 
  * **I:** I/O (instance storage / local storage) – databases   * **G:** GPU – video rendering / machine learning 
  * **T2 / T3:** burstable instances (up to a capacity) 
  * **T2 / T3** - unlimited: unlimited burst 

* T2/T3 - Burstable Instances
  * OK CPU performance    
  * CPU is VERY good when spikes occur
  * CPU bursts use burst credits    
  * When bust credits are gone, CPU becomes BAD    
  * When machine stops bursting, credits are accumulated over time 

* T2/T3  - Unlimited    
  * Unlimited burst credit balance   
  * High cost

## AMIs - basics
* Image of a instance
* Advantages
  * Pre-installed packages needed 
  * Faster boot time (no need for ec2 user data at boot time)   
  * Machine comes configured with monitoring / enterprise software   
  * Security concerns – control over the machines in the network   
  * Control of maintenance and updates of AMIs over time   
  * Active Directory Integration out of the box   
  * Installing your app ahead of time (for faster deploys when auto-scaling)   
  * Using someone else’s AMI that is optimised for running an app, DB, etc... 
* AMI are built for a specific AWS region (!) 
* AMI take space and they live in Amazon S3 - You won’t see them 
* By default, your AMIs are private, and locked for your account / region 
* You can also make your AMIs public and share them with other AWS accounts or sell them on the AMI Marketplace 

## AMIs - Cross Account AMI Copy 
* You can share an AMI with another AWS account.  
* Sharing an AMI does not affect the ownership of the AMI.  
* If you copy an AMI that has been shared with your account, you are the owner of the target AMI in your account.  
* To copy an AMI that was shared with you from another account, the owner of the source AMI must grant you read permissions for the storage that backs the AMI, either the associated EBS snapshot (for an Amazon EBS-backed AMI) or an associated S3 bucket (for an instance store-backed AMI).  
* Limits:  
  * You can't copy an encrypted AMI that wasshared with you from another account. Instead,if the underlying snapshot and encryption key were shared with you, you can copy the snapshot while re-encrypting it with a key of your own. You own the copied snapshot, and can register it as a new AMI.
  * You can't copy an AMI with an associated billingProduct code that was shared with you from another account.This includes Windows AMIs and AMIs from the AWS Marketplace. To copy a shared AMI with a billingProduct code, launch an EC2 instance in your account using the shared AMI and then create an AMI from the instance.
  
## Placement Groups 
**Cluster** - (Same AZ) clusters instances into a low-latency group in a single Availability Zone 
  * Pros: Great network (10 Gbps bandwidth between instances) 
  * Cons: If the rack fails, all instances fails at the same time 
  * Use case: 
    * Big Data job that needs to complete fast 
    * Application that needs extremely low latency and high network throughput 
**Spread** - (instance in each AZ) spreads instances across underlying hardware (max 7 instances per group per AZ) 
  * Pros:
    * Can span across Availability 
    * Reduced risk is simultaneous failure 
    * EC2 Instances are on different physical hardware
  * Cons:
    * Limited to 7 instances per AZ per placement group 
  * Use case: 
    * Application that needs to maximize high availability 
    * Critical Applications where each instance must be isolated from failure from each other 
**Partition** - (EC2 group in each AZ) spreads instances across many different partitions (which rely on different sets of racks) within an AZ. Scales to 100s of   * EC2 instances per group (Hadoop, Cassandra, Kafka) 
* Up to 7 partitions per AZ 
* Up to 100s of EC2 instances 
* The instances in a partition do not share racks with the instances in the other partitions 
* A partition failure can affect many EC2 but won’t affect other partitions 
* EC2 instances get access to the partition information as metadata 
* Use cases: HDFS, HBase, Cassandra, Kafka 

## Elastic Network Interfaces (ENI) 
* Logical component in a VPC that represents a virtual network card 
* The ENI can have the following attributes
    * Primary private IPv4, one or more secondary IPv4 
    * One Elastic IP (IPv4) per private IPv4
    * One Public IPv4
    * One or more security groups
    * A MAC address
* Bound to a specific availability zone (AZ) 
* Can be moved from instance 2 instance

## EC2 Hibernate 
* Saves memory to root EBS storage
* Freezes state (like laptop)
* The instance boot is much faster! (the OS is not stopped / restarted) 
* Use cases: 
    * long-running processing 
    * saving the RAM state 
* services that take time to initialize 
* Root Volume: must be EBS encrypted 
* Available for On-Demand and Reserved Instances 

## For the exam
* EC2 instances are billed by the second, t2.micro is free tier 
* On Linux / Mac we use SSH, on Windows we use Putty 
* SSH is on port 22, lock down the security group to your IP 
* Timeout issues => Security groups issues 
* Permission issues on the SSH key => run “chmod 0400” 
* Security Groups can reference other Security Groups instead of IP  ranges (very popular exam question) 
* Know the difference between Private, Public and Elastic IP 
* You can customize an EC2 instance at boot time using EC2 User Data 
* Know the 4 EC2 launch modes: • On demand
  * Reserved
  * Spot instances
  * Dedicated Hosts
* Know the basic instance types: R,C,M,I,G,T2/T3
* You can create AMIs to pre-install software on your EC2 => faster boot • AMI can be copied across regions and accounts
* EC2 instances can be started in placement groups:
  * Cluster
  * Spread
