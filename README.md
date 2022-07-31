# Tricky tips for AWS Solutions Architect Associate certification exam (concise cheat sheet)

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
* System status check: requires AWS to fix it (due to hardware failure etc). Alternatively, you can automatically recover the instance: simplified automatic recovery based on instance configuration or by creating an CloudWatch alarm action. You cannot use CloudWatch events to directly trigger the recovery of the instance. A recovered instance is identical to the original instance; **everything** remains the same except that instance store and in-memory data get lost. Terminated or stopped instances cannot be recovered. The recover action is supported only on instances that use EBS volumes only (not instance store volumes)

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

