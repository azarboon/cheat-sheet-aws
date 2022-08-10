# Tricky tips for AWS Solutions Architect Associate certificate (concise cheat sheet)

### Security

GuardDuty: continuous threat and anomaly detection for AWS accounts, workloads (EC2, EKS), and data stored in **Amazon S3**. Protects against **cryptocurrency mining**. It detect account and bucket **compromises**. Suspending GuardDuty stops the service from analyzing data but doesn't delete findings nor configs. Disabling GuardDuty deletes its data and configs. Data sources: CloudTrail Events, VPC Flow logs and DNS Logs.

Amazon Macie: identify sensitive data stored on S3

Inspector: automated and continual vulnerability management. Inspector is more about identifying vulnerabilities and evaluating against security best practices. It does **NOT** detect compromise.

Web Applicaiton Firewall: can be deployed on ALB, API Gateway, Cloudfront. It helps against SQL injection and cross site scripting attacks.

AWS Shield protects you from common network and transport layer DDoS attacks free of charge. AWS Shield Advanced does offer protection to resources outside of AWS. This should NOT cause unexpected spike in billing costs.

SSM Parameter Store: can serve as a secrets store, but you must rotate the secrets yourself (no automation)

Secrets Manager: secret rotation with built-in integration for RDS, Redshift, DocmentDB.

CloudHSM: dedicated hardware security module to manage and store your encryption keys. It doesn't help with distribution of keys.

AWS allows penetration for some of resources without prior authorization.


### CloudTrail

In console you can view events since last 90 days.

You cannot use Cloudtrail to debug and trace data  across accounts, though you can consolidate log files from multiple accounts in a single bucket. 

Event types. By default, it logs only management events:
* Management events (**control plane operations**): management operations that are performed on resources in your account (e.g. security config, routing, registering new device, log API operations)
* Data events (**data plane operations**): resource operations performed on or within a resource (e.g. S3 object level API)
* Insight events: capture unusual API call rate or error rate activity in your account. If enabled and detects an unusual activity, then insights events are logged



### Data migration

DataSync: **automates and accelerates** (periodically, **no continous**) large amount of data between HDFS, self-managed object storage, Snowcone, S3, EFS, FSx. DataSync over Direct Connect provides the most reliable hybrid connectivity. DataSync supports **only NFS and SMB protocols and not ISCSI or tapes**. To migrate physical tapes use Storage Gateway - Tape Gateway.

Transfer Family: Supports S3 and EFS. Supports FTP, SFTP, FTPS protocols. It's more expensive than DataSync.

Database Migration Service: **continuous data replication with high availability**. Use it along with Schema Conversion Tool to handle handle **complex database configurations**
* Supported sources: databases as well as S3. 
* Supported target: databases, S3, Redis, Redshift, KDS, OpenSearch


Snowball Edge Storage and Compute Optimized offer storage clustering. They can be managed through AWS OpsHub. Use Snowball devices when you need to quickly and securely transfer dozens of TBs (up to PB) of data.

Fastest way to move large amount of data (in order):
1. Snowball Edge: It's fast, cost effective and provides local processing. Compute Optimized provides 39.5 TB of storage (no SSD). Storage Optimized provides 80 TB of storage. Use Schema Conversion Tool (SCT) **to extract and load data locally** onto edge devices. Then you ship devices to AWS, and AWS automatically loads them into S3. Then, you can use Database Migration Service (DMS) to transfer data into target stores. Also remember that you can't directly copy data from Snowball Edge devices into AWS Glacier; instead Snowball -> S3 life cycle policy -> Glacier.
2. Direct connect: It provides a **private connection** but needs few weeks for provisioning.
3. Internet / VPN: good use case if you have immediate need and low to modest bandwidth requirements. Aas a rule of thumb, remember that transfering 100 TB over 50 Mbit/s connection takes around 180 days.

### Encryption

By default, Key Management Service (KMS) enforces 30 days of waiting period before deleting a customer master key; key's status is "pending" during the time.

#### S3

* In client side encryption: Clients encrypts data and uploads the encrypted data to S3; S3 is not aware of the key.
* In server side encryption, client requests Amazon to encrypt & decrypt the keys. It can be done with:
  * S3-Managed Keys (SSE-S3): uses AES-256 and each object is en-crypted with a unique key. 
  * KMS keys (SSE-KMS): Similar to SSE-S3 but has additional benefits; it separates permissions for KMS keys so enables additional protection and **audit** trail. 
  * Customer-Provided Keys (SSE-C): customer manages the keys and Amazon S3 manages the encryption process.


### AWS Organizations

Service control policies (SCP) affect only member accounts (its IAM users, roles, and root user) and not management account. Any account has only those permissions permitted by every parent above it (either implicit or explicit `Deny` will restrict access). SCPs do not affect service-linked role

Organization trail: create a new trail in CloudTrail from within the management account with the organization trails option enabled. Member accounts can see the trail but can't modify or delete it. Nor they have access to log files in the S3 bucket.

To centrally manage and configure firewall rules use AWS Firewall Manager.

Consolidated billing enables tracking, combined usage and volume discount, at no additional fee and with only one bill through management account.

To migrate an account to another Organization: use console, no need to create a new account in the destination Org. Resources will remain under the control of the migrated account. You must have root or IAM access to both the member and master accounts. Here are the steps to follow: 
1. Remove the member account from the old organization 
2. Send an invite to the member account from the new Organization 
3. Accept the invite to the new organization from the member account


### Other services

WorkSpaces: Desktop-as-a-Service (DaaS) solution

X-Ray: debug and trace data **across accounts** and visualize it in a centralized account

Cost Explorer: identify underutilized instances that may be downsized **within the same instance family**. Understand impact of Savings Plan and Reserved Instances on the bill.

Compute Optimizer: downsizing recommendations **within or across instance families**, upsizing recommendations to remove performance bottlenecks, and recommendations for instances that are parts of an ASG. Recommends optimal instance **type** to reduce costs. It does **NOT** recommend instance purchase options

Step Functions: workflow orchestration. It's preferred over Amazon Simple Workflow Service

Trusted Advisor: helps you to reduce cost, increase performance, and improve security. It offers Service Limits check. To get access to its API (AWS Support API) you need Business or Enterprise support plans.

AWS Batch: Efficiently supports multi-node parallel jobs, running a single jobs across instances. Compatible with IP-based, internode communication frameworks such as Apache MXNet, TensorFlow, Caffe2, or Message Passing Interface.

Systems Manager: centralized end-to-end operational hub for hybrid cloud (viewing & automation). You can create logical groups of resources, view their config changes, API activity, compliance status etc. With Systems Manager’s Run Command you can manage configs and run ad-hoc commands remotely

CloudFormation: maintain the **state** of the overall architecture. StackSet allows you to create stacks in multiple accounts and  regions

AWS Config: evaluates the configurations and compliance of your resources and to remedy non-compliant resources. You can configure it to stream configuration changes an SNS topic. It can maintain a **history of resource configuration changes**

App Mesh: **application-level** networking for microservices applications.

Beanstalk: quick application deployment. Allows access to underlying resources and OS.

CodeDeploy: automates **application (not infrastructure) deployments** into AWS /on-premise. Good fit when code changes need to be managed but not for infrastructure services.

OpsWork: Chef or Puppet solution

EventBridge: event-based service for **third-party SaaS** and AWS resources. Enables asynchronous messaging patterns.

Amazon MQ: Similar to SQS, but used for **existing applications that are being migrated** into AWS. It supports AMQP, STOMP, MQTT and WebSocket.

Amazon Glue: ETL service. It **requires significant coding efforts**. You can write transformed data in a compressed format.

EMR: MapReduce, big data workloads such as Spark, Hive, Hadoop. EMR requires **infra management and significant coding effort**.


### Containers

Fargate can intergate with both EKS and ECS, and provides serverless orchestration. ECS + EC2 is not serverless, though it provides access to underlying OS (unlike Fargate)

ALB's Dynamic Port Mapping redirects traffic to multiple ECS Tasks running on the same ECS Container instance.

Fargate is charged based on vCPU and memory resources used. ECS EC2 charges based on used resources (e.g. EC2 and EBS)

To troubleshoot disconnected ECS container instances: Verify that ECS agent is installed and running, Docker daemon is running, and that the IAM instance profile has the necessary permissions applied. Remember that Amazon ECS optimized AMI already contains ECS container agent.

ECS Container instance roles give permission to all the apps (ECS agent and docker daemon use that). To have fine grain permissions, use IAM Task Role. 


### Elastic Load Balancer

ELB does not have any caching capability. ELB is responsible for distributing traffic and ASG responsible for placing & terminating in-stances. 

Let's say AZ1 has 4 targets and AZ2 has 6 targets. If ELB's cross-zone load balancing is:
* Enabled: each target receives 10% of traffic
* Disabled: Each AZ1’s target receives 12.5% and AZ2’s target receives 8.3% of traffic.

Minimum 2 AZs are required for high availability (not 3)

Supports multiple TLS certificates using Server Name Indication (SNI)

Enable ELB access logs to capture detailed information about requests.

Routing mechanisms for target types:
* Instance id: uses primary private IP address 
* IP address: uses any private IP address 

Target IPs can be instances in peered VPCs, AWS resources that are addressed by IP and port, onpremise resources linked with Direct Connect or VPN.

To attach instances in a private subnet to ELB: create a public subnet in SAME AZ, then associate PUBLIC subnet to the ELB. This also prevents direct connectivity to instances from Internet.

Gateway Load Balancer: supports GENEVE protocol on port 6081

#### Application Load Balancer

Layer 7. Supports HTTP and HTTPS. Provides only static DNS. Weighted target group (weighted routing policy is for Route 53). Supported targets are instance id, IP, Lambda. If target group has no registered target, it returns `HTTP 503: Service unavailable`

Supports authentication from corporate and OIDC compliant IdPs: using authentication action on a listener rule that integrates with Cognito **User Pools**.

Does NOT do SSL passthrough, instead it terminates the first SSL connection on the ALB and then re-encrypt the traffic and connect to the EC2. To encryption in-transit: use HTTP listner, then install SSL certificates on the ALB and EC2 instances.

In host-based routing, the rule `*.example.com` matches `test.example.com` but doesn't match `example.com`

Has Security Group (SG) Associated rules:
* Allows inbound traffic to load balancer’s listener port
* Allows outbound traffic to instance's SG on instance's listener port as well as health check port


#### Network Load Balancer

Layer 4. Supports TCP, UDP and TLS. Provides static DNS and static IP. Doesn't have security group. Doesn't do connection termination (i.e. EC2 can access client's IP). Supported targets are instance id, IP and ALB. 

Does SSL passthrough. To encrypt intransit: use TCP listener then terminate SSL on EC2 instances.

Provides better performance than ALB.


### Auto Scaling Group

ELB is responsible for distributing traffic wheras ASG is responsible for placing / terminating instances. ASG uses EC2's status checks by default. You should configure it to use ELB's health checks. Then if any check fails, ASG replaces the instance. Moreover, you can attach one or more load balancer's target groups to ASG within same region. Then, any existing or added ASG's instance will be automatically registered with the ELB (if the process fails, most likely ASG's max capacity is exceeded). 

You can attach a running instance to an ASG.

If connection draining is enabled, ASG waits for in-flight requests to complete or timeout before terminating instances.

`Sticky Session`: binds a user's session to a specific instance. This ensures that all requests from the user during the session are sent to the same instance.

SQS + ASG (using target tracking policy): enables dynamically scaling based on backlog per instance.

#### Launch template vs Launch configuration

Launch configuration (LC) is legacy, can **NOT** be modified, though can be used with multiple ASGs. Changing launch configuration of an ASG does **NOT affect existing instances** (until the instance gets terminated/replaced)

Launch templates offer more options: mixing Spot instances with on-demand, enables you to specify multiple instance types, it can be modified.

#### Scaling policies

Simple scaling policy is discouraged: After your ASG launches or terminates instances, it waits for a cooldown period (300s by default) to end before any further scaling activities. You can reduce cooldown period by leveraging AMI. If there is a relevant metric for scaling, AWS recommends using target tracking policy, otherwise step scaling policy (Enable `default instance warmup` to wait before collecting metrics). However both those policies entail a lag at the beginning, so if the question mentions a recurring schedule set, most likely it asks for Scheduled policy. 

If there are multiple policies in force, ASG chooses the one that provides largest capacity (whether its scale out or in)

#### Troubleshooting

For updating or troubleshooting:
* You can suspend & resume ASG’s processes (scaling activities). For example, suspend `ReplaceUnhealthy` process, do the update, then set instance health status back to healthy, then reactivate the process.
* Put the instance into Standby state, do the update, exit Standby state. Standby instances count toward available instances and don’t get replaced (by default), ASG doesn’t perform health check on them.


#### Termination

Default termination policy (in order): AZ with most instances -> allocation strategy for On-Demand vs Spot instances -> instance with oldest launch configuration -> instance with oldest launch template -> instance that is closest to the next billing hour

Reasons why ASG doesn't terminate an instance:
* Health check grace period not expired yet
* ASG's process suspended: HealthCheck, ReplaceUnhealthy, Terminate
* ASG doesn't immediately terminate instances with an Impaired status
* ASG doesn't perform health checks on instances in the Standby state (set it back to In-Service)
* ASG uses EC2's status check by default, set it to use ELB health check.


#### Tenancy value

Instance's tenancy values:
* `default`: shared tenancy hardware
* `host`: dedicated host, runs on an isolated server
* `dedicated`: dedicated instance, runs on a single-tenant hardware

VPC's tenancy values: `default` or `dedicated`. By default, instances get VPC’s tenancy value unless otherwise specified.

You can change the instance’s tenancy from dedicated to host and vice versa. Also, you can change it from shared tenancy to dedicated host and vice versa.

if Launch configuration's instance placement tenancy value is 
* `Null` or `default`: instances will get VPC's tenancy value.
* `Dedicated`: instances will overide VPC's tenancy value and become Dedicated Instances.

### Virtual Private Cloud and networking

To uniquely identify an Availability Zone (AZ) across the two accounts use AZ ID (e.g. use1-az2) instead of AZ code (e.g. us-east-1a)

A subnet must reside within A SINGLE AZ, unlike VPC that can span across multiple AZs.

To enable private DNS routing using Route 53, enable DNS resolution + DNS host name in VPC.

Default VPC's instances have public and private hostname. Non-default VPC's instances have private but not public hostname (depending on settings)

VPC console options (to create a new VPC): single public subnet, public and private subnets (using NAT gateway), public and private subnets using Site-to-Site (S2S) VPN, private subnet with S2S VPN

Internet Gateway is highly available, and performs network address translation (NAT) for instances in public subnet. NAT instance/gateway are located in public subnet and perform NAT for instances in private subnet.

#### connectivity and sharing

Connectivity from outside to onprem through VPC: public subnet route table needs to route onprem-destined-traffic to Virtual Private Gateway (VPG) and subsequently to onprem

Private connection from on-premise to API Gateway avoiding Internet: create private VIF across Direct Connect and use API Gateway Private Endpoint 

* VPC Peering: simplest way to connect two VPCs. Cross-region. **No transitive routing**. Can be use along with security groups (SG) to restrict access but its difficult to manage peering connections and SGs at scale (it should be used ideally for < 10 VPCs). Lowest cost. Peering enables you to route traffic between VPCs using private IP addresses (VPCs' CIDRs must not overlap). Instances in either VPC can communicate with each other as if they are within the same network. VPC peering does **NOT** facilitate centrally managed VPCs. An  account owner can **NOT share the VPC itself** with another AWS accounts. Peering exposes **the whole network**; its recommended to expose an app to other VPCs using AWS PrivateLink (VPC Endpoint Services); which is a scalable and secure alternative.
* Transit VPC: hub VPC connects to spoke VPCs through VPN. Customer has to manage EC2-based VPN which costs more and limits throughout.
* Transit Gateway: hub-spoke model, fully managed service, highly available, No VPN overlay re-quired. Simlplifies management and reduces operational costs. Regional service so connecting to VPCs within **same region**
* Shared services VPC: centralized VPC endpoints with either Transit Gateway / VPC peering. reduces cost and administrative overhead. Communications between VPC and public AWS services won't egress AWS.
* VPC sharing: leverages Resource Access Manager (RAM) service to share one or more **subnets** (from owner VPC) with other AWS accounts (participants) **belonging to the same parent organization** from AWS Organizations. The owner account can **NOT share VPC itself**. VPC sharing **leverages implicit routing** within a VPC for applications that require a **high degree of interconnectivity**. 

Cost of sharing services centrally between accounts within same region: Resource Access Manager is cheaper than Transit Gateway

To achieve transitive routing between private subents in VPCs, you can use Transit Gateway + Resource Access Manager

#### endpoints
VPC endpoints are highly available and enables you to privately connect your VPC to supported services (traffic won't egress AWS). 
Only consumer VPCs initiate connections to the service provider VPC.

Endpoint types:
* Interface: It’s an ENI with private IP within service consumer VPC, and connects to NLB within service provider VPC.
* Gateway Load Balancer: Use it to send traffic to a fleet of virtual appliances.
* Gateway: It's a gateway that you specify as a target in your route table for traffic destined to S3 or DynamoDB. It's free of charge.



### Route 53 

To setup DNS resolution for hybrid cloud using conditional forwarding rules and DNS endpoints (which are created on Route 53 Resolver):
* Inbound endpoint: Forward from DNS resolvers on your network to Route 53 Resolver (in your VPC)
* Outbound endpoint: Forward from Route 53 Resolver (in your VPC) to resolvers on your network

DNS failover configuration types using Route 53 health checks:
* Active-active: returns one or more resource. In case of failure, fails back to a healthy resource. Configured using any routing policy except “failover routing policy”
* Active-passive: Route 53 actively returns a primary resource. In case of failure, Route 53 returns the backup resource. Configured using only "failover routing policy".
* Combination


To setup a health check for private resources: create a Cloudwatch metric and associate an alarm. Then create a health check that checks the alarm itself.

To use 3rd party registrar with Route 53: buy domain from 3rd party registrar. Create a hosted zone in Route 53. Update Name Server records on 3rd party website to use route 53 name servers.

To have private DNS using Route 53 enable DNS resolution + DNS host names in VPC.

Route 53 is prone to DNS caching. Alternatively, you can use Global Accelerator (GA). Route 53 doesn’t reduce internet latency as GA does. GA will direct users to the closest edge location and then use the AWS global network.

#### routing policies

* Failover: only for active-passive failover
* Geo-location: routing based on user location. You cannot use it to reduce latency. It can exclude certain regions and localize the content.
* Geo proximity: routing based on the geographic location of users and resources. You can use positive/negative bias to expand/shrink the size of the geographic region from which traffic is routed to a resource.
* Latency: Use when you have resources in multiple AWS Regions and you want to lowest latency. It’s not used for failover. How it works: DNS routes the query to a Route 53 name server. Route 53 refers to its data on latency between regions choose the one with lowest and responds with IP address

Unlike simple routing policy, multi-value routing policy returns only healthy resources

When simple routing policy returns multiple values, a random value is chosen by the CLIENT.

Unlike Elastic Load Balancer, Route 53's multi value routing policy is a client side load balancing


#### DNS reocrd types:

`Domain1.com` is zone apex, `www.domain1.com` and `sub.domain.com` are subdomains.

A/AAAA reocrd: maps a hostname to ipv4/ipv6

CNAME record doesn’t work on zone apex but can redirect query to ANY record. Followings are allowed:
* `sub.domain1.com` to `sub.domain2.com`
* `sub.domain1.com` to `domain2.com`
* `www.domain1.com` to `www.domain2.com`
* `www.domain1.com` to `sub.domain2.com`

Followings are not allowed:
* `Domain1.com` to anywhere else

Alias record can apply to zone apex and subdomains but can redirect queries only to select AWS services. Alias record can’t redirect queries to another domain. Following is allowed:
* `Domain1.com` to `www.domain1.com`

Alias doesn't work to any other domain. Following is not allowed:
* `domain1.com` to `sub.domain2.com`

BTW, following is allowed too:
`Domain1.com Alias(A) service.amazonaws.com`

Route 53 doesn't charge for Alias queries to AWS resources, but charges for CNAME queries.

After updating a record, user may be directed to the old target because of TTL (time to live)



### Logging and Monitoring

CloudTrail, VPC Flow Logs, CloudWatch Events can't be used debug and trace data across accounts (wheras X-Ray can do cross-account)

Enable VPC Flow logs (at VPC, Subnet or ENI levels) to capture info about traffic inside VPC. To troubleshoot why a conneciton might not work: If inbound and outbound are different (i.e. one is ACCEPT and other is REJECT) problem likely lies in Network ACL. Otherwise, the problem likely lies in either security group or in Network ACL.

Enable ELB's access logs to capture detailed information about requests.

EC2's basic monitoring has 5-min interval (default). Enable detailed monitoring for 1-min interval (can be done after launch), then you can use Cloudwatch to perform analysis.

Cloudwatch services: Events, Logs

Cloudwatch Logs: Monitor, store, and access your log files (Cloudwatch Subscriptions: for real time processing)

Cloudwatch Events: Near real time stream of system events from AWS resources

Cloudwatch Container Insights: Collects, aggregate, and summarize metrics and logs from container-ized apps

Unified CloudWatch agent: collect internal system-level metrics (e.g. Swaputilization, memory usage) as custom metrics

Standard resolution (default) has 1 min interval. Custom metrics can go to high resolution (1s), and their alarm can be triggered every 10s, 30s, or any multiple of 60s.

Cloudwatch instance recovery: Cloudwatch **alarm** for system status check (StatusCheckFailed_System metric), and set it to do an action named EC2 Instance Recovery. Everything (ebs, ips, placement group etc) remains same but instance store gets wiped out.



### Lambda

Lambda by default has access to Internet. VPC-enabled lambda needs a NAT gateway (and its subject to routing rules). To connect to VPC-resources, Lambda needs subnet IDs and security group IDs

Cloudwatch metrics track number of requests, latency per request, requests resulting in error.

Supported runtimes: C#/.NET , Go, Java, Node.js, Python, Ruby

To optimize performance: use provisioned capacity, optimize static initialization, change memory-settings (memory is proportional to vCPU). Reuse code / dependency with Lambda Layers.

Lambda IAM role to access S3 bucket:
* Same account: grant permission to role, verify there is no explicit deny in bucket policy
* Different accounts: grant permissions on both role and bucket policy

Limits:
* Env variable size: 4 KB. 
* Deployment through container image or zip file, max 50 or 250 MB respectivcely 
* Disk (/tmp): 512 MB 
* Max duration: 15 min (900s)

Invocation:
* synchronous: retry / error handling by invoker (nothing by default). e.g. API, ALB
* Asynchronous: two retries (3 in total). Destination or DLQ for failed attempts. e.g. S3, SNS, SES
* Poll-based: Sync invocation, retry based on data expiration in data source. e.g. Kinesis, SQS, DynamoDB Streams
In async invocation, DLQ has to be on Lambda side. In (Lambda + SQS), (Lambda + SNS), (Lambda + S3 event) scenarios, DLQ must be set on SQS, Lambda, Lambda respectively.


### Simple Storage Service


S3 is a regional service, buckets are defined at regional level, but console is global.

S3 Select: retrieve only subset of data using SQL

By default, an S3 object is owned by the AWS account that uploaded it. So the S3 bucket owner will not implicitly have access to the objects written by another account.

S3 storage class analysis only provides recommendations for Standard to Standard IA classes.

All storage classes offer (11 9's) durability, but S3 One Zone-IA can get destroyed in case of AZ destruction. 

Replication:
* Both source and destination buckets must have versioning enabled. 
* Once enabled, replication copies only newly created objects. To copy existing objects, use S3 sync command. If the command fails, you can rerun it without duplication. 

Logs:
* CloudTrail logs: detailed API tracking for bucket-level and object-level operations
* S3 Server access logs: object-level operations, detailed records of requests for audit purpose, help you learn your customer base and understand your S3 bill

Optimize performance
* To improve performance **within** datacenter, you can use ElastiCache + S3; it enables sub-millisecond latency, high throughput, lower retrieval cost from S3. But to get media closer to user, use Cloudfront.
* Multi-part upload for files > 100 MB and up to 5TB. 
* S3 Transfer Acceleration (S3TA) for files more than 1 GB; for less than 1 GB, use Cloudfront PUT/POST. S3TA charges only for accelerated transfers (not for failed attempts)
* s3 byte-range fetche: parallelize GETs and transferg only the specified portion of the object
* Cloudfront + S3: cheaper, faster and more secure than delivering directly form S3
* You can scale read & write performance N times by using N prefixes in parallel.

Locking:
* S3 Object Lock: prevents objects from being deleted or overwritten for a fixed amount of time or indefinitely. In two ways: retention period and legal hold. An object version can have both, either or neither of them. You can define retention period either explicitly ("Retain Until Date" parameter) or through a bucket default setting (need to specify duration). Different versions of an object can have different retention modes and periods. Legal Hold provides same protection but remains in effect until removed. 
* Vault Lock is only for Glacier and not for S3


Versioning
* In a versioned bucket, DELETE operation only inserts a delete marker (the marker becomes current version). S3 Replication can replicate the markers. To automate recovery of the objects, you can transition "noncurrent versions" of objects. 
* To permanently delete versioned object, you must use DELETE Object versionId. Similarly, life cycle rules can expire current object versions or permanently remove noncurrent object versions.
* S3 always returns the latest version of an object.
* If a bucket already has files, after enabling versioning, the existing files will have version Null.
* In a versioned bucket, sync command copies only the current version of the object. 
* Once versioning is enabled, you can never reverse it, you can just suspend it.

Access control:
* Bucket policy: cross-account access, when IAM Policy size limit is a constraint, to enforce encryption at upload, to grant public access, to enforce Multi-factor authentication. It provides user and account level access, and applies to all of the objects within a single bucket.
  - To enforce encryption at upload time: create a policy that denies any S3 Put request that does not include the x-amz-server-side-encryption header ("AES256" value for S3-managed keys, "aws:kms" value for AWS KMS–managed keys)
  - To restrict access to a specific URL use "aws:referer". This is preferred over IP-based restriction.
* Access Control List: legacy. 
* IAM: It's centralized and easier to manage. Grants access only to users within your aws account (not account level).
  - To grant folder-level permission to many users: create an IAM policy that grants folder-level permission (using policy variables), attach the policy to a group, then add users to the group.

Latency:
* Glacier Instant Retrieval: milliseconds,
* Glacier Flexible Retrieval: minutes or hours,
* Glacier Deep Archive: hours (and no expedited mode)

Designed availability:
* 99.5: One Zone IA
* 99.9: Intelligent-Tiering, Standard IA, Glacier Instant
* 99.99: Standard, Glacier Deep Archive, Glacier Flexible

Availability SLA:
* 99.9: Standard, Glacier Deep Archive
* Other classes: 99%

Minimum storage:
* Standard IA, One Zone IA charge for min 30 days
* Glacier (instant and flexible retrieval) charges for min 90 days
* Glacier Deep Archive charges for 180 days
* Intelligent-Tiering has no min storage duration, but monitors access pattern for at least 30 days to move objects. Also it charges extra for monitoring.

Transition between classes:
* Standard -> Standard IA -> Intelligent -> One Zone IA -> Glacier Instant -> Glacier Flexible -> Deep Archive
* You can NOT transition into S3 Standard (only out of)
* You can NOT transition from One Zone-IA to the Intelligent-Tiering, Standard-IA, or Glacier Instant Retrieval.
* Before you transition objects from the S3 Standard or S3 Standard-IA storage classes to S3 Standard-IA or S3 One Zone-IA, you must store them at least 30 days in the S3 Standard storage class. However, you can transfer to other classes e.g. Deep Archive after one day being in Standard.

### Simple Queue Service

Apps poll at their comfort. Good to **decouple app components**. At least once delivery without ordering, wheras, FIFO does exactly once delivery with ordering (To scale, use group ID). Provides automatic scaling at read time. Supports batching and buffering (SNS + SQS fan-out pattern). Stores message for 14 days. There can be multiple consumers per queue, but message can be consumed by one consumer at a time. SQS doesn't integrate directly with 3rd party SaaS.

Options to set delay:
* Delay Queues (max 15 min): message is hidden when it is first added to the queue (i.e. it's invisible to consumers) - DelaySeconds parameter
* Individual message delay: up to 15 min
* Visibility timeout (max 12 hours): message is hidden only after it is consumed from the queue (i.e. it prevents other consumers from receiving and processing the message again) Increasing the timeout, gives more time to consumers to process the message and prevents duplicate reading of the message. If a job is processed within the visibility timeout the message will be deleted. Otherwise, the message will become visible again (could be delivered twice). 
* To set delay seconds on individual messages, use message timer.

SQS Request-Response pattern requires SQS Temporary Queue Client

SQS Dead Letter Queue used for debugging. SQS-Lambda integration is poll-based and synchronous invocation; and DLQ must be set on SQS side.

To prioritize SQS messages, use separate queues and configure polling prioritization at app layer.

To migrate from standard to FIFO queue: recreate queue, queue's name must end with .fifo suffix and throughput must be < 3000 msg/s. FIFO by default supports 300 msg/s without batching and 3000 msg/s with batching (i.e. batching 10 msg/operation)

Polling controls API call time and retry behaviour from consumer’s side:
* Short polling (default): queries only subet of servers, so it may not return all messages at first.
* Long polling: queries all servers, SQS responds as soon as a msg is available. If polling wait time expires, SQS sends empty response. It reduces (false) empty responses, reduces latency, increases efficiency, reduces cost and returns msg as soon as they become available.

SQS + Auto Scaling Group (with target tracking policy): dynamical scaling based on backlog per instance.

### Simple Notification Service: 

pub-sub model, passively pushing message. Ordered only in FIFO. No buffer (SNS + SQS fan-out pattern allows buffering). SNS doesn't directly integrate with 3rd party SaaS. 

Supported:
* transport protocols: HTTP(S), Email/Email-JSON, SQS, SMS
*subscribers: above list, mobile push, Lambda, Kinesis Data Firehose (not KDS)

S3 cannot directly write data into SNS (can send only events). To stream existing files and updates from S3 to KDS, you cna use DMS.


### Kinesis

Kinesis Data Firehose is a near real-time streaming ETL and **the easiest way to load streaming data** into data stores and analytics tools, eg. S3, Redshift, Elasticsearch/Opensearch, HTTP endpoint. Kinesis Data Stream (KDS) can't directly send data to the mentioned destinations; it needs Firehose in between. Firehose is a fully managed scaling, no administration is needed. It does't support replay but supports batching and buffering (KDS is opposite).  Retention period max 24h.

Kinesis Data Streams is **real-time processing** of streaming data. It supports replay, orderly set of records, and multiple consumers in parallel. A stream has +1 shards. KDS segregates stream’s data records into shards, using partition key associated with each data record. Producer push and consumer pulls data. There is at least once delivery. KDS requires manual scaling of shards. It doesn't support batching nor buffering. It can store data for up to 365 days. Enhanced-fanout provides dedicated read throughput for each consumer.

Remember that KDS can only be a source to Firehose but not a destinaiton. In KDS-Firehose model, Firehose’s PutRecord(Batch) operations are disabled and Kinesis Agent cannot write to Firehose directly. Instead, it needs to use KDS's PutRecord(s) operations, then KDS sends data to Firehose.


### Elastic Block Store and Instance store

Instance store: can be root volume, offers very low latency, supports millions of IOPS (>256,000), offers temporary storage for frequently changed data or data that is  replicated across a fleet of instances, such as a load-balanced pool of web servers. It's included as part of instance usage cost (so it's more cost effective than Proviisoned IOPS volumes)

SSD-backed EBS volumes:
* All of them can be used as boot volume.
* gp2 / gp3: 3 IOPS/GB, 16,000 IOPS/volume
* io1 / io2 (Provisioned IOPS): 50 IOPS/GB, 64,000 IOPS/volume. Supports multi-attach. To maximize performance, use them with EBS-optimized EC2 instances.
* io2 Block Express: Max iops 256,000, supports multi-attach, sub-millisecond latency.

In PIOPS and gp3 volumes, size and performance are independent (can be changed independently). In gp2, they are linked. 

HDD-backed EBS volumes:
* Can't be used as boot volume.
* low cost.
* st1 (throuput optimized): 500 IOPS/volume but no SLA. 
* sc1 (cold storage): 250 IOPS/volume but no SLA

Unlike instance stores, EBS volumes can be added to running instance.

Block device mapping supports EBS volumes and instance store. 

RAID 0 to aggregate performance, RAID 1 to increase redundancy (can apply to both EBS volumes and instance store).

Root EBS volumes are by default deleted upon EC2's termination (unlike non-root volumes). Instance store gets deleted upon failure or termination. 

Encryption is supported by all EBS volume types (at rest and in transit). Encrypted and unencrypted volumes have same IOPS performance. All instance families support encryption, but not all instance types.

You can have encrypted and unencrypted EBS volumes attached to an instance at the same time.

You can't directly share an encrypted Amazon EBS volume with another AWS account (need to use snapshot). Snapshots are constrained to the Region in which they were created. To share a snapshot with another Region, copy the snapshot to that Region and then share the copy. 

To share an encrypted snapshot with another account: share the custom key and encrypted snapshot (by modifying the permissions). The receiving account must copy the snapshot before they can then create volumes from the snapshot.

You can share snapshot that are encrypted with customer managed key (not with default AWS keys).

Encryption keys used by an Amazon EBS volume can't be changed. However, you can create a snapshot of the volume and then use the snapshot to create a new, encrypted copy of the volume. While creating the new volume, specify the new encryption key.


### Cloudfront and Global Accelerator

Cloudfront (CF) can serve both static and dynamic content (e.g. video stream, APIs), but it may not be a good fit for highly dynamic and frequently changing content. Remember that S3 is not a good fit for dynamic content. Cloudfront + S3 can be cheaper, faster and more secure than delivering directly form S3. And to serve S3 static webiste thorugh HTTPS, you need to use CF.

Cloudfront custom origin can be S3 static website, ALB, Lambda, or any other HTTP server (e.g. onprem server). Custom origins must be publicly accessible. CloudFront can route to multiple origins based on the content type. For HA and failover, use an origin group with primary and secondary origins.

Points of presence (POP) skips regional edge case for:
* Proxy HTTP methods (PUT, POST, PATCH, OPTIONS, and DELETE)
* Dynamic requests, as determined at request time
* when origin S3 bucket and optimal regional edge cache are in the same region

Signed url serve individual files (e.g. paid content, dynamic generated url), while signed cookie serve multiple files.

With Lambd@Edge, you can authenticate users at edge location or compress files to reduce data transfer cost.

Field-level encryption: encrypt at edge, decrypted only by target app

Cloudfront supports HTTP(s) and WebSocket. It can't expose static public IP. Cloudfront is better for spiked traffic than Global Accelerator.

Global Accelerator (GA): For TCP, UDP (gaming), IoT (MQTT), VoIP, and HTTP use cases that require static IP addresses or deterministic, fast regional failover. It's more expensive than CF. It uses same network as CF so it provides the same latency.

### Direct Connect and VPN

Direct Connect (DX) provides private but not encrypted connection. It can connect to all AZs within same region.  An IPSec VPN connection using the same BGP prefix can be a low-cost backup connection for the DX.

Virtual Private Gateway (VPG) and Transit gateway are components of Site-to-Site VPN as well as DX gateway; whereas, virtual interface (VIF) is component of DX . Public VIFs are for public resources as well as IPSec VPN. Private VIFs are for private resources. A hosted virtual interface is used to allow another account to access your Direct Connect link. 

On customer side, DX has router/firewall, wheras, Site-to-Site VPN has **customer gateway device**.

DX gateway is a grouping of VPGs and private VIFs. It's a globally available resource, can be created in one region and be accessed from all regions. DX gateway can connect to either:
* Transit gateway; for VPCs in same region
* VPG; for VPCs in different regions. 

In architectural diagrams, the line between DX location and DX Gateway is a virtual interface. And after DX Gateway, its either VPG associations (to VPC) or Transit Gateway association.

VPN CloudHub: provides secure connection between multiple sites and optionally with VPC. As it relies on Internet, the conneciton is not very reliable.

### API Gateway

API Gateway can directly access serveral services (e.g. DynamoDB) using proxy and mapping (no need for Lambda etc. in between)

Access controls:
* IAM permissions: great for users/roles that are already within in your account. Does both authentication and authorization. Leverages Signature Version 4.
* Lambda/custom authorizer: Returns IAM policy so its flexible. Good for 3rd party auth (OAuth, SAML, etc). Does both authentication and authorization.
* Cognito User Pool: no need to write any custom code. Only authentication, no authorization

Supported API types:
* RESTful: stateless client-server communication
* WebSocket APIs: stateful full-duplex communication

Throttling:
* AWS throttling limits: Applies to all accounts and clients in a region.
* Per-account limits: Applies to all APIs in an account in a Region.
* Per-API, per-stage: applied at the API method level for a stage, and applies to **all** customers using the method.
* Per-client throttling limits: using API keys

API Gateway edge-optimized leverages Cloudfront, and is good for global clients. However, the gateway itself stays in one reigon.

API cache can be applied for a stage not for a method.

To enable private connection from on-premise to API Gateway (avoiding Internet): create private VIF across Direct Connect and use API Gateway Private Endpoint.

### Security Group vs Network ACL

SG supports only allow rules (you can't explicitly deny any a traffic). All rules evaluated. To enable pinging, allow ICMP protocol. In peered VPCs, you can reference SGs which are across different accounts but within same region.

SGs are stateful, that is, if you allow e.g. inbound traffic on port 80, you don't need to configure the outbound traffic (and vice versa); because, security group automatically allows the return traffic.

* Default SG's default rules: allows all outbound, allows inbound only from SG itself
* Custom SG's default rules: allows all outbound, no inbound

NACL supports allow and deny rules. Rules evaluated in order.
* Default NACL’s default rules: allows all outbound, allows all inbound
* Custom NACL’s default rules: denies all outbound, denies all inbound

NACL are stateless, that is, if you allow e.g. inbound port on port 80, you need to explicitly allow outbound traffic on **ephermal portS** associated with the client (vice versa). 

You can't use Internet Gateway as source/destination for SG.

How to secure following architectures and block certain IPs:
* Cloudfront + ALB + EC2: ALB has SG, so restrict ALB's SG to CF's public IPs. And restrict EC2's SG to ALB's SG. You can block IPs using WAF on CF. Remember that in this architecture, NACL is useless.
* NLB + EC2:  NLB does't have SG nor terminates connection, and EC2 gets client's IP. SG can't block (it can only allow), so use NACL to block IPs.


### Storage

Storage gateway: unlimited storage to on-prem:
* S3 File Gateway: NFS and SMB protocols, integrates with AD.
* FSx File Gateway: SMB protocl, stores on Windows File server Server (not S3), integrates with AD
* Tape Gateway: iSCSI virtual tape library (VTL) interface. Storage on S3, can be transitioned to Glacier/Deep Archive
* Volume Gateway: block storage (EBS snapshot), iSCSI protocol. Stores on S3 but data is not directly accessible. Two modes:
  1. Cached: Primary data stored on s3, frequently access data cached locally
  2. Stored: Primary data stored locally, low-latency access to entire dataset, async backup to AWS.


S3 is the cheapest storage solution (after Glacier). EFS costs 3x more than EBS per GB, but you pay for what you use and it can be used by multiple instances at a time. Whereas in EBS you pay for provisioned capacity (even if you don't use it) and a volume can be accessed only by an instance at a time. Meawhile, EFS costs way less than FSx Lustre.

Sub-millisecond latency: FSx for Lustre

Concurrent access: EFS, S3, FSx for Lustre

Parallel file system: FSx for Lustre

EFS and FSx for Lustre can be mounted only on Linux instances.

FSx Windows File shares can be mounted on both Windows and Linux instances (you need to install cifs-utils package)

FSx Lustre integrates only with ECS EC2. Wheras, EFS integrates with both ECS EC2 and Fargate.

S3 integrates with FSx for Lustre  but not with EFS, EBS and FSx Windows.

Multi-AZ: EFS and FSx Windows

* NAS is a **file system**, works with NFS and SMB protocols. Possible destinaitons are FSx and EFS. 
* SAN is a **block-storage**, works with ISCSI and Fibre Channel protocols. Possible desitnaiton is EBS.
* S3 is a **object storage**, exposes data through a RESTfull API that can be accessed from anywhere. S3 is not a destination for file systems nor can be mounted on EC2 instances. 

### Elastic File System

POSIX compliant, Supports only NFS protocol and Linux. Supports multi-AZ. 

Supports concurrent access; you can mount EFS file system onto many instances, concurrently read & write into it as you would on your local file system. To achieve this, create a subdirectory for each user and grant read-write-execute permissions to the users. Then mount the subdirectory to the users' home directory (don't modify permission on root directory). To lower latency, you can create EFS mount targets in each AZ.

For cross-region collaboration on EFS with least operational overhead: use EC2 instances through cross-region VPC peering. Remember that cross-region copying EFS doesn't enable collaboration, everybody works on its own files.

EFS integrates with both ECS EC2 and Fargate. Use DataSync to integrate EFS-S3.

Performance modes: General purpose (low latency good for web server) and Max I/O

Throughput modes: 
* Bursting (default): throughput and size are linked
* Provisioned: for constant throughput

Storage modes: Standard and One Zone. There can be one life cycle policy to transition least-used-files to the correpsonding IA storage class, and another policy to transition back.

Access control: 
* IAM: to control who can administer file system 
* EFS Access Point: App level access to files and directories using POSIX compliant user & group permission.
* Security Group: acts like a firewall to contorl trafifc flow


### Elastic Compute Cloud

To reduce deployment time:
* Create a Golden AMI with the static installation components already setup
* Use EC2 user data to customize the dynamic installation parts at boot time

AMI represents files written on disk, not memory. Use Hibernation to save instance memory (RAM) content to EBS root volume. AMI can help with dependencies, but won't help speeding up apps start time (instead use Hibernate).

Cross-region copying of an AMI automatically creates a snapshot in the destination region.

EC2 user data: By default, scripts have root user privileges. By default, runs only during the boot cycle when you first launch an instance

Spot Fleet: Spot and On-Demand Instances to meet the target capacity specified in Spot Fleet request. Spot Fleet request select pools, subsequently Spot Instances come from the pools. Instance types in a capacity pool are the same, while Spot Fleet request can have various instance type and specs. Spot Fleet with *lowestPrice* strategy can be the most cost optimal solution. The request for Spot Instances is fulfilled if there is available capacity and the maximum price you specified in the request exceeds the current Spot price.

You can cancel Spot Instance requests that are open, active or disabled. Cancelling an active Spot Instance request, does NOT terminate associated running instances. First you must cancel a persistent Spot Instance request, then manually terminate associated Spot Instances.

DR strategies:
RPO (data loss) – disaster- RTO (downtime)
* pilot lights (RTO/RPO: 10s of minute): core minimum of services running
* Warm standby (RTO/RPO: minutes): scaled down but fully functional system
* Multi-site active/active (RTO/RPO real time)

Scale up/down: vertical scaling

Scale out/in: horizontal scaling

Dedicated host: more visibility and control. Per host billing, more expensive. 

Dedicated instance: per instance billing, can NOT be used for existing server-bound software licenses

Instance's tenancy values:
* default: shared tenancy hardware
* host: dedicated host, runs on an isolated server
* dedicated: dedicated instance, runs on a single-tenant hardware


VPC tenancy values: 
* default
* dedicated. 

Instances get VPC’s tenancy value by default, unless otherwise specified.

If Launch configuration (LC)'s instance placement tenancy value is 
* Null/default: instances will get VPC tenancy value.
* Dedicated: instances will override VPC tenancy value and become Dedicated Instances.

You can change the instance’s tenancy from dedicated to host and vice versa. Also, you can change it from shared tenancy to dedicated host and vice versa.

Status check types:
* System status check: requires AWS to fix it (due to hardware failure etc). Alternatively, you can automatically recover the instance: simplified automatic recovery based on instance configuration or by creating an CloudWatch alarm action. A recovered instance is identical to the original instance; **everything** remains the same except that instance store and in-memory data get lost. Terminated or stopped instances cannot be recovered. The recover action is supported only on instances that use EBS volumes only (not instance store volumes)

* Instance status check: requires customer to fix it. Cloudwatch recovery option works only for system status check failures, not for instance status check failures.

In order to improve instances network performance, use:
* Cluster placement group
* EC2 Enhanced Networking (SR-OV)
* Elastic Fabric Adapter (EFA). Improved ENA for HPC (provides all functionalities of ENA). Only works for Linux. Greate for tightly coupled workloads. Bypasses underlying Linux OS to provide low latency. This option provides lowest latency.

Spread placement group: For apps that have a small number of critical instances that should be kept separate from each other. Supports Multi-AZ. Not suited for distributed and replicated workloads such as Hadoop.

Partition placement group: For large distributed and replicated workloads, such as HDFS, HBase, and Cassandra. Supports Multi-AZ

* Stopping an instance: private IP persists but public IP and data in instance store get lost
* Rebooting an instance: keeps all IPs and data in instance store

To get metadata: Download and run the Instance Metadata Query Tool (ec2 metadata command from SDKs), or “curl http://169.254.169.254/latest/meta-data/"

You can manage configs and run ad-hoc commands rmeotely using System's Manager Run Command.

Reservaiton options:
* Capacity reservation: no commitment, capacity reserved in a specific AZ, no billing discount.
* Zonal reserved instance: Fixed 1 or 3 year commitment, capacity reserved in a specific AZ, provides billing discount
* Regional reserved instance: Fixed 1 or 3 year commitment, no capacity reserved, provides billing discount
* Savings plan: fixed 1 or 3 year commitment, no capacity reserved, provides billings discount


### Database Types

* key-value : Only Dynamodb, Millisecond latency. For high-traffic web applications
* In-memory: ElastiCache, sub-millisecond latency. Caching. For session management, gaming leaderboards, geospatial applications
* Graph: Amazon Neptune, for fraud detection, social networking, recommendation engines. 
* Relational: Redshift (columnar storage), latency in seconds. Analyze all your data and get insights across operational databases, data lakes, data warehouses.

### Redshift

Both Redshift and RDS run relational databases. Redshift is specifically designed for online analytic processing (OLAP) and (concurrent) business intelligence (BI) applications, while RDS is for OLTP workloads.

Redshift uses parallelism and caching to ipmrove performance. It's a columnar storage. Latency in seconds.

DMS + Redshift: Enables resource-efficient petabyte-scale data warehouse with minimal effort.

Redshift enhanced VPC routing forces all COPY and UNLOAD traffic moving between cluster and data repositories through your VPC, subsequently you can use VPC features.

Redshift Spectrum is NOT serverless: it requires SQL client and an available Redshift cluster on an EC2 instance.

Redshift Serverless doesn't require infra management. 

Redshift HA: You can enable cross-Region snapshot copy on your cluster. In the event of a DR event, the snapshots in the replica Region can be restored to create a new cluster.


### ElastiCache

ElastiCache should not be used as the main database. Can be used to store aggregation results (e.g. retrieving data, do some computation and storing that). It's NOT serverless, requires provisioning & maintenance. Provides sub-millisecond latency. 

* MemCached: For simple models. Supports multi-thread/core. SASL-based authentication.
* Redis: Supports everything execpt multi-thread, that is: encryption, HA (opt for Multi-AZ and automatic fail-over), compliances, pub/sub etc. Authentication using AUTH command and SG

Both supports partitioning.

### DynamoDB:

It's serverless, and good for unpredictable / unstructured data (e.g eCommerce apps). Millisecond latency. Automated scaling with minimal overhead.

It can query only primary keys (i.e. partition and sort key) and indexes. Global secondary index and local secondary index enable you to query attributes other than primary keys. For partial match, use ElasticSearch.

DynamoDB transactions: writes to multiple tables (at the same time) or none.

DynamoDB Accelerator (DAX): for individual object cache, query & scan cache. **No need to modify app logic**

Creating Global table requires DynamoDb Streams. DynamoDB Streams + Lambda: Poll-based and synchronous invocation for new entries. Retry based on data model in source.

Capacity modes:
* Provisioned: Cost effective. Auto scaling for RCU & WCU (they can change independently). Good for smooth sustained usage.
* On-demand: For unpredictable workloads, more expensive

Best practices: keep item size small (max 400 KB. Compress bigger objects or use S3 Object ID pointers). Keep number of tables minimum (unless for time series data). Separate more and less frequently access data.


### Relational Database Service  & Aurora


Aurora Global Database: RPO of seconds, RTO of minutes. For fail-over, read replicas have to be manually promoted.

Read replicas have async replication. RDS Multi-AZ has sync replication.

You have to pay for cross-region data transfers.

Aurora Replicas connect to the same storage volume as the primary DB instance, but support read operations only. Read replica's priority in case of a failover: replica with the lowest numbered tier will be promoted. If two or more replicas share the same priority, then replica that is largest in size will be rpomoted.
 
Migrating RDS to Aurora requires significant effort. To scale, you can enable storage auto-scaling for RDS MySQL.

Aurora supports multi-master (MySQL compatible), but not RDS. In a multi-master cluster, all DB instances have read/write capability (no failover mechanism, has continuous availability). It scales out write performance across multiple AZs.

To encrypt and un-encrypted instance: Take a snapshot of the database, copy it as an encrypted snapshot, and restore a database from the encrypted snapshot. Terminate the previous database

To have in-transit encrypted connection with an already encrypted RDS/Aurora database, you can download and use existing AWS-provided root certificates.

In encrypted instances, all logs, backups, and snapshots are encrypted. 
 

### Access Control

IAM service supports only one type of resource-based policy called *trust policy*, which is attached to an IAM role. Trust policies define which principal entities can assume the role.

Amazon Cloud Directory: hierarchical data store. Doesn’t support trust relationship between on-prem and other domains

AWS Directory Services:
* Managed Microsoft AD: Supports MFA. Users are both on-prem and AWS. Allows directory-aware workloads, e.g. SharePoint, .NET or SQL Server-based apps. Provides trust relationship between on-prem and cloud. Good for +5000 users
* AD Connector: proxies users to onprem. Users are managed only onprem. Allows onprem users to login to AWS using their AD credentials. Can’t run directory aware workloads
* Simple AD: AD-compatible managed directory on AWS. Can NOT be joined with onprem AD. Doesn’t support trust relationship. Cheapest option, good for 5000 users or less

IAM policy conditions:
* aws:RequestedRegion: restrict target of the API call, not the API call requester
* aws:SourceIp: restricts IP where the API call originates from

Use a Null condition operator to check if a condition key is present at the time of authorization. In the policy statement, if the value is "true", it means the key doesn't exist (because it's null), and if the value is "false" it means the key exists (its value is not null).

Use instance profile (i.e. IAM role) to grant permissions (using temporary crednetials) to applications running on EC2 instances.

Permission boundary: max permissions users can grant to principals they create. Can only be applied to roles or users, not IAM groups.

S3 permissions: least privilege applies. Default permission is DENY unless there is an ALLOW. An explicit DENY trumps all ALLOWs.

RDS/Aurora IAM authentication: short-lived token, more secure. Handled by AWSAuthenticationPlugin. Supports MariaDB, MySQL and PostgreSQL.

Direct access to IAM through HTTPS: using IAM Query API along with access key ID and secret access key

ElastiCache: IAM-auth not supported. Redis uses Auth command + SG, Memcached uses SASL-based auth.

Only ALB supports authentication (authentication action on a listener rule + Cognito user pool)

For fine grained permissions on ECS, use IAM Task Role. ECS Container instance roles give permission to all the apps (ECS agent and docker daemon use that)

To authenticate at CloudFront edge use Lambda@edge

Remember that federation is different from assume role.

Services that enable enable federation:
* AWS SSO: define federated access permissions for your users based on their group memberships in a single centralized directory
* AWS IAM: If you use multiple directories, or want to manage the permissions based on user attributes, consider AWS IAM as your design alternative

Onprem Federation (SSO): SAML 2.0 Identity provider + AWS STS gives users temporary access WITHOUT using IAM credentials. If your onprem IdP doesn’t support SAML 2.0, you need to use a Custom Identity Broker, which authenticates users, calls STS to obtain temporary credentials, then calls AWS federation endpoint and supply temporary credentials to request a sign-in token. Then you you can construct a URL with the token and give the URL to users, with which they can access AWS resources and console. 

To sync data across mobile and webapps in real time: AppSync / Cognito Sync (which requires Cognito federated identity pool (not User Pools) to work)

Cognito is used for authenticating users to web and mobile apps NOT for providing hybrid SSO. 
Cognito user pool: authentication (including social sign-in), user management
Cognito identity pool (federated identities): verifies token, exchanges it with temporary credentials (through STS) and returns temp credentials to client. Client directly call AWS services using temporary records.

IAM group is not an identity and can’t be identified as a principal in an IAM policy, can’t be used to group EC2 instances, can’t assume a role (only users and services can), can’t be nested.

