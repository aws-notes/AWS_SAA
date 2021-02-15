# Route 53

## Overview
* Route53 is a Managed DNS (Domain Name System)
* DNS is a collection of rules and records which helps clients understand how to reach a server through URLs.
* In AWS, the most common records are: 
* A: hostname to IPv4
* AAAA: hostname to IPv6
* CNAME: hostname to hostname
* Alias: hostname to AWS resource.

## Public vs Private
* **Public** domain names you own (or buy) - **application1.mypublicdomain.com**
* **Private** domain names that can be resolved by your instances in your VPCs. - **application1.company.internal**

$0.50 per month per hosted zone

## TTL (Time to Live)
* High TTL: (e.g. 24hr)
  * Less traffic on DNS
  * Possibly outdated records
* Low TTL: (e.g 60 s)
  * More traffic on DNS
  * Records are outdated for less time
  * Easy to change records
  
## CNAME vs Alias
* CNAME:
  * Points a hostname to any other hostname. (app.mydomain.com => blabla.anything.com) 
  * **ONLY FOR NON ROOT DOMAIN** (aka. something.mydomain.com)
* Alias:
  * Points a hostname to an AWS Resource (app.mydomain.com => blabla.amazonaws.com)
  * Works for ROOT DOMAIN and NON ROOT DOMAIN (aka mydomain.com)
  * Free of charge
  * Native health check

## Simple Routing Policy (1:1)
* Maps a hostname to another hostname
* Use when you need to redirect to a single resource
* You can’t attach health checks to simple routing policy
* If multiple values are returned, a random one is chosen by the client

## Weighted Routing Policy (1:m %)
* Control the % of the requests that go to specific endpoint
* Helpful to test 1% of traffic on new app version for example
* Helpful to split traffic between two regions
* Can be associated with Health Checks

## Latency Routing Policy (performance)
* Redirect to the server that has the least latency close to us
* Super helpful when latency of users is a priority
* Latency is evaluated in terms of user to designated AWS Region
* Germany may be directed to the US (if that’s the lowest latency)

## Health Checks
* Have X health checks failed => unhealthy (default 3)
* After X health checks passed => health (default 3)
* Default Health Check Interval: 30s (can set to 10s – higher cost)
* About 15 health checkers will check the endpoint health
* => one request every 2 seconds on average
* Can have HTTP,TCP and HTTPS health checks (no SSL verification) 
* Possibility of integrating the health check with CloudWatch
* Health checks can be linked to Route53 DNS queries!

## Failover Routing Policy
* Health check are **manditory**
* **Primary** - main
* **Secondary** - failover / disaster recovery

## Geo Location Routing Policy
* Different from Latency based!
* This is routing based on user location
* Here we specify: traffic from the UK should go to this specific IP
* Should create a “default” policy (in case there’s no match on location)

## Multi Value Routing Policy(client based load balancing)
* Use when routing traffic to multiple resources
* Want to associate a Route 53 health checks with records
* Up to 8 healthy records are returned for each MultiValue query 
* Multi Value is **not** a substitute for having an ELB
