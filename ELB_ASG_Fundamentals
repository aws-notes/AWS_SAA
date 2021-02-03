
# ELB_ASG_Fundamentals

## Why use an EC2 Load Balancer?
* An ELB (EC2 Load Balancer) is a managed load balancer
* AWS guarantees that it will be working
* AWS takes care of upgrades, maintenance, high availability
* AWS provides only a few configuration knobs
* Health checks

### AWS has 3 kinds of managed Load Balancers
Classic Load Balancer 
* HTTP, HTTPS, TCP
Application Load Balancer 
* HTTP, HTTPS, WebSocket
Network Load Balancer 
* TCP, TLS (secure TCP) & UDP

### Good to Know
LBs can scale but not instantaneously – contact AWS for a “warm-up”
* Troubleshooting
  * 4xx errors are client induced errors
  * 5xx errors are application induced errors
  * Load Balancer Errors 503 means at capacity or no registered target
  * If the LB can’t connect to your application, check your security groups!
* Monitoring
  * ELB access logs will log all access requests (so you can debug per request)
  * CloudWatch Metrics will give you aggregate statistics (ex: connections count)
  
## Classic Load Balancers 
Fixed hostname - "XXX.region.elb.amazonaws.com"
TCP or HTTP health checks

## Application Load Balancer
* Application load balancers is Layer 7 (HTTP)
* Load balancing to multiple HTTP applications across machines (target groups)
* Load balancing to multiple applications on the same machine (ex: containers)
* *Dynamic IP* behind DNS
* Support for HTTP/2 and WebSocket
* Support redirects (from HTTP to HTTPS for example)
* Routing tables to different target groups:
  * Routing based on *path* in URL (example.com/users & example.com/posts)
  * Routing based on *hostname* in URL (one.example.com & other.example.com)
  * Routing based on *Query String*, Headers (example.com/users?id=123&order=false)
* Has a port mapping feature to redirect to a dynamic port in ECS(would need multiple CLBs for that)

### ALB - Target Groups
* EC2 instances 
* ECS tasks (containers)
* Lambda functions
* IP Addresses (internal IPs only)

* ALB can route to multiple target groups (1:n)
* Health checks are at the target group level
* The clients true IP from (*X-Forwarded-For*) header
* The clients true Port from (*X-Forwarded-Port*) header
* The clients true protocal from (*X-Forwarded-Proto*) header

## Network Load Balancer
* Layer 4 - Forward TCP & UDP traffic
* High performance - Handle millions of request per seconds
* Less latency ~100 ms (vs 400 ms for ALB)
* NLB has one *Static IP* per AZ, and supports assigning Elastic IP (helpful for whitelisting specific IP)
* NLB are used for extreme performance,TCP or UDP traffic

*Load Balancer Stickiness*
* Sends a cookie that fixs a session to a prticulae target group (ec2, ip, lambda, ecs)
* Enabling stickiness may bring imbalance to the load over the backend EC2 instances

### Cross-Zone Load Balancing
* Each load balancer instance distributes evenly across all registered instances in all AZ
* Otherwise, each load balancer node distributes requests evenly across the registered instances in its Availability Zone only.

* Classic Load Balancer
  * Disabled by default
  * No charges for inter AZ data if enabled
* Application Load Balancer
  * Always on (can’t be disabled) • No charges for inter AZ data
* Network Load Balancer
  * Disabled by default
  * You pay charges ($) for inter AZ data if enabled
  
### SSL/TLS

* You can manage certificates using ACM (AWS Certificate Manager)
* You can create upload your own certificates alternatively
* HTTPS listener:
  * You must specify a default certificate
  * You can add an optional list of certs to support multiple domains
  * Clients can use SNI (Server Name Indication) to specify the hostname they reach
  * Ability to specify a security policy to support older versions of SSL /TLS (legacy clients)
  
*Server Name Indication (SNI)*
* SNI solves the problem of loading multiple SSL certificates onto one web server (to serve multiple websites)
* Requires the client to indicate the hostname of the target server in the initial SSL handshake
* The server will then find the correct certificate, or return the default one
* *Only* works for ALB & NLB
* Does not work for CLB 

* *Classic Load Balancer (v1)*
  * Support only one SSL certificate
  * Must use multiple CLB for multiple hostname with multiple SSL certificates
* *Application Load Balancer (v2)*
  * Supports multiple listeners with multiple SSL certificates • Uses Server Name Indication (SNI) to make it work
* *Network Load Balancer (v2)*
  * Supports multiple listeners with multiple SSL certificates
  * Uses Server Name Indication (SNI) to make it work

### ELB – Connection Draining
* Called **Connection Draining** with CLB
* Called **Deregistration Delay** with ALB & NLB (referances target groups)
* A delay on a terminating a instance to allow current connection to finish
* When a instance enters Draining mode all new connections are diverted to other instances by the ELB/NLB
* Between 1 to 3600 seconds, default is 300 seconds
* Can be disabled (set value to 0)
* Set to a low value if your requests are short

## Auto Scaling Group (ASG)
* *Auto Scaling Alarms*
  * An Alarm monitors a metric (such as Average CPU)
  * Metrics are computed for the overall ASG instances
  * Based on the alarm:
    * We can create scale-out policies (increase the number of instances)
    * We can create scale-in policies (decrease the number of instances)
* *Auto Scaling New Rules*
  * ”Better” auto scaling rules that are directly managed by EC2
    * Target Average CPU Usage
    * Number of requests on the ELB per instance
    * Average Network In
    * Average Network Out
* *Auto Scaling Custom Metric*
  * Based on a custom metric (ex: number of connected users)
    * Send custom metric from application on EC2 to CloudWatch (PutMetric API)
    * Create CloudWatch alarm to react to low / high values
    * Use the CloudWatch alarm as the scaling policy for ASG
* *ASG Notes*
  * Scaling policies can be on CPU, Network... and can even be on custom metrics or based on a schedule (if you know your visitors patterns)
  * ASGs use Launch configurations or Launch Templates (newer)
  * To update an ASG, you must provide a new launch configuration / launch template
  * IAM roles attached to an ASG will get assigned to EC2 instances
  * ASG are free.You pay for the underlying resources being launched
  * Having instances under an ASG means that if they get terminated for whatever reason, the ASG will automatically create new ones as a replacement. Extra safety!
  * ASG can terminate instances marked as unhealthy by an LB (and hence replace them)
  
* *Auto Scaling Groups – Scaling Policies*
* Target Tracking Scaling
  * Most simple and easy to set-up
  * Example: I want the average ASG CPU to stay at around 40%
* Simple / Step Scaling
  * When a CloudWatch alarm is triggered (example CPU > 70%), then add 2 units
  * When a CloudWatch alarm is triggered (example CPU < 30%), then remove 1
* Scheduled Actions
  * Anticipate a scaling based on known usage patterns
  * Example: increase the min capacity to 10 at 5 pm on Fridays
  
* *Auto Scaling Groups - Scaling Cooldowns*
* Cool down period is a buffer to stop erratic scalling
* We can create cooldowns that apply to a specific *simple scaling* policy
* Overrides the default cooldown period

* *ASG Default Termination Policy*
* ASG tries the balance the number of instances across AZ by default
* The az with the most instances will have one removed wheh scaling in
* The instance with the oldest launch configuration will be deleated

* *ASG - Lifecycle Hooks*
* By default as soon as an instance is launched in an ASG it’s in service
* You have the ability to perform extra steps before the instance goes in service (Pending state)
* Scan it for vulns, install agents - whatever
* You can do the same for terminated

* *LaunchTemplate vs Launch Configuration*
*  Both:
  * ID of the Amazon Machine Image (AMI), the instance type, a key pair, security groups,and the other parameters that you use to launch EC2 instances (tags, EC2 user-data...) 
* Launch Configuration (legacy):
  * Must be re-created every time • 
* LaunchTemplate (newer):
  * Can have multiple versions
  * Create parameters subsets (partial configuration for re-use and inheritance)
  * Provision using both On-Demand and Spot instances (or a mix)
  * Can use T2 unlimited burst feature
  * Recommended by AWS going forward
