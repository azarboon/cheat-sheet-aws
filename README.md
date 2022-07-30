# cheat-sheet-aws


 
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

Ways to control access on API Gateway:
* IAM permissions: great for users/roles that are already within in your account. Does both authentication and authorization. Leverages Signature Version 4.
* Lambda/custom authorizer: Returns IAM policy so its flexible. Good for 3rd party auth (OAuth, SAML, etc). Does both authentication and authorization.
* Cognito User Pool: no need to write any custom code. Only authentication, no authorization


ElastiCache: IAM-auth not supported. Redis uses Auth command + SG, Memcached uses SASL-based auth.

Only ALB supports authentication (authentication action on a listener rule + Cognito user pool)

For fine grained permissions on ECS, use IAM Task Role. ECS Container instance roles give permission to all the apps (ECS agent and docker daemon use that)

To authenticate at CloudFront edge use Lambda@edge

Services that enable enable federation:
* AWS SSO: define federated access permissions for your users based on their group memberships in a single centralized directory
* AWS IAM: If you use multiple directories, or want to manage the permissions based on user attributes, consider AWS IAM as your design alternative

Onprem Federation (SSO): You cannot assume a role, instead you must use federation (e.g. SAML 2.0 + AWS STS gives users temporary access without using IAM credentials). If your onprem IdP doesn’t support SAML 2.0, you need to use a Custom Identity Broker, which authenticates users, calls STS to obtain temporary credentials, then calls AWS federation endpoint and supply temporary credentials to request a sign-in token. Then you you can construct a URL with the token and give the URL to users, with which they can access AWS resources and console.

To sync data across mobile and webapps in real time: AppSync / Cognito Sync (which requires Cognito federated identity pool (not User Pools) to work)

Cognito is used for authenticating users to web and mobile apps NOT for providing hybrid SSO. 
Cognito user pool: authentication (including social sign-in), user management
Cognito identity pool (federated identities): verifies token, exchanges it with temporary credentials (through STS) and returns temp credentials to client. Client directly call AWS services using temporary records.

IAM group is not an identity and can’t be identified as a principal in an IAM policy, can’t be used to group EC2 instances, can’t assume a role (only users and services can), can’t be nested.

