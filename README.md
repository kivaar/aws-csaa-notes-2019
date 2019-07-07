# aws-csaa-notes-2019

Table of Contents
-----------------
- [IAM](#iam)
- [S3](#s3)
    - [S3 Storage Classes](#s3-storage-classes)
    - [S3 Encryption](#s3-encryption)
    - [S3 Versioning](#s3-versioning)
    - [S3 Cross Region Replication](#s3-cross-region-replication)
- [CloudFront](#cloudfront)
- [Snowball](#snowball)
- [Storage Gateway](#storage-gateway)
- [EC2](#ec2)
    - [EC2 Pricing Models](#ec2-pricing-models)
    - [EC2 Instance Types](#ec2-instance-types)
    - [EC2 Security Groups](#ec2-security-groups)
    - [AMI Types](#ami-types)
    - [Encrypted Root Device Volumes & Snapshots](#encrypted-root-device-volumes--snapshots)
    - [AWS Command Line](#aws-command-line)
    - [Instance Metadata](#instance-metadata)
    - [EC2 Placement Groups](#ec2-placement-groups)
    - [EFS](#efs)
- [EBS](#ebs)
    - [EBS Types](#ebs-types)
    - [EBS Volumes & Snapshots](#ebs-volumes--snapshots)
- [CloudWatch](#cloudwatch)
- [Databases](#databases)
    - [Relational Databases](#relational-databases)
    - [DyanamoDB](#dyanamodb)
    - [Redshift](#redshift)
    - [Aurora](#aurora)
    - [Elasticache](#elasticache)

## IAM
- AWS managed policies are default for common permissions
- Access Key ID and Secret Access Key for programmatic access

## S3
- Object based storage (key, value, version ID, and metadata), files only
- Subresources include access control lists (ACLs) and torrents
- Universal namespace, unlimited storage with files up to 5TB
- Data consistency
    - Read after Write consistency for PUTS of new objects
    - Eventual Consistency for overwrite PUTS and DELETES
- Cross Region Replication pricing
- Transfer Acceleration uses edge locations and Amazon’s backbone network (CloudFront)
- Successful uploads generate HTTP 200
- MFA Delete
- Control access using bucket ACLs or bucket policies
- Configure access logs to record all requests made to the bucket

### S3 Storage Classes
- S3 Standard guaranteed 99.9% availability and 99.999999999% (11 x 9s) durability
- S3 IA (Infrequently Accessed) for data accessed less frequently with retrieval fee
- S3 One Zone IA or RRS (Reduced Redundancy Storage) does not have multiple availability zone data resilience
- S3 Intelligent Tiering uses ML to move objects around storage classes
- S3 Glacier for cheap archiving and configurable retrieval times
- S3 Glacier Deep Archive for cheapest storage with a 12 hour retrieval time

### S3 Encryption
- Encryption in transit using SSL/TLS
- Encryption at rest, server side
    - S3 Managed Keys (SSE-S3), managed by S3
    - AWS Key Management Service, Managed Keys (SSE-KMS), managed by KMS and the user
    - Server Side Encryption with Customer Provided Keys (SSE-C)
- Encryption at rest, client side

### S3 Versioning
- Once enabled, versioning can only be suspended
- Have to manually set new file versions to public
- Deleting an object only places a delete marker
- Lifecycle management to move or expire current and previous object versions

### S3 Cross Region Replication
- Versioning must be enabled on both buckets
- Regions must be unique
- Files in an existing bucket are not replicated automatically
- Delete markers, deleting versions, and deleting delete markers are not replicated

## CloudFront
- A distribution (content delivery network or CDN) made up of distributed servers (network)
- Deliver web content to users based on geographic location
- An edge location is where content is cached for the TTL (time-to-live)
- Web distribution typically for websites
- RTMP (real-time message protocol) distribution used for Adobe Flash media streaming
- You can clear old cached objects for a fee (invalidating the cache)
- Restrict access using signed URLs or cookies (e.g. premium Netflix users)

## Snowball
- Petabyte-scale (50TB or 80TB) data transport solution to transfer to and from AWS
- Snowball Edge has 100TB storage and compute capabilities for offline
- Snowmobile is exabyte-scale (up to 100PB) for libraries, repositories, and migrations
- Can import to and export from S3

## Storage Gateway
- Connects on-prem software appliance with cloud-based storage
- Available as physical hardware or a VM image for ESXi or Hyper-V
- File Gateway (NFS), flat files are stored as objects in S3 buckets and accessed through a network file system (NFS) mount point
- Volume Gateway (iSCSI), stored or cached disk volumes that can be backed up as point-in-time snapshots in EBS
    - Stored volumes let you store primary data locally which is asynchronously backed up to S3 as EBS snapshots from 1GB - 16TB
    - Cached volumes (1GB - 32TB in size) use S3 for primary data storage but frequently accessed data is retained locally in the storage gateway
- Tape Gateway (VTL), virtual tape library for archiving

## EC2
- Termination Protection is turned off by default
- The root EBS volume is deleted upon termination by default on EBS-backed instances
- EBS root volumes on default AMIs cannot be encrypted
    - Use third party tools (e.g. BitLocker) or custom AMIs for root volume encryption
- Additional volumes can be encrypted

### EC2 Pricing Models
- On Demand, by the hour or second with no commitment
- Reserved (Standard, Convertible, or Scheduled), capacity reservation on 1 or 3 year contract terms
- Spot, bid on instance capacity, useful for flexible start and end times (no charge if terminated by Amazon)
- Dedicated Hosts, physical EC2 server, useful for server-bound software licenses

### EC2 Instance Types
```
F - FPGA
I - IOPS
G - Graphics
H - High Disk Throughput
T - Cheap general purpose (T2 Micro)
D - Density
R - RAM
M - Main choice for general purpose apps
C - Compute
P - GPGPU
X - Extreme Memory
Z - Extreme Memory and CPU
A - Arm-based workloads
U - Bare Metal
```

### EC2 Security Groups
- By default, all inbound traffic is blocked and all outbound traffic is allowed
- Changes to Security Groups take effect immediately
- You can have many EC2s within a security group and multiple security groups attached to EC2s
- Security Groups are stateful (traffic allowed in is automatically allowed back out)
- You cannot block sepcific IP addresses (use network ACLs instead)
- You cannot specify deny rules

### AMI Types
- Selected based on region, OS, architecture, launch permissions, or storage for root device
- Storage is either Instance Store (Ephermeral Storage) or EBS Backed Volumes
- Instance Store volumes cannot be stopped (will lose data)
- EBS backed instances can be stopped (will not lose data)
- Both types can be rebooted without losing data
- As root volumes, both types are deleted on termination (EBS volumes can be kept as an option)

### Encrypted Root Device Volumes & Snapshots
- Snapshots of encrypted volumes are also encrypted
- Volumes restored from encrypted snapshots are encrypted
- Only unencrypted snapshots can be shared (other AWS accounts or public)
- How to encrypt the root device volume
    - Create a snapshot of the root device volume
    - Create a copy of the snapshot with encryption enabled
    - Create an AMI from the snapshot
    - Launch a new instance using the AMI

### AWS Command Line
- Access is configured in IAM
- The CLI is global
- Roles are universal, easier to manage, and more secure than storing keys on EC2s

### Instance Metadata
- curl http://169.254.169.254/latest/meta-data/
- curl http://169.254.169.254/latest/user-data/

### EC2 Placement Groups
- Placement group names must be unique within your AWS account
- Placement groups cannot be merged
- Existing instances cannot be moved into a placement group (create an AMI instead)
- Cluster Placement Group
    - Grouping of certain instances within a single AZ (cannot span multiple AZs)
    - Recommended for low network latency and/or high network throughput
- Spread Placement Group
    - Group of instances that are each placed on distinct underlying hardware
    - Recommended for a small number of critical instances that should be kept separate
    - Can span multiple AZs

### EFS
- 2+ instances can share an EFS volume
- Grows and shrinks automatically as files are added or removed
- Petabyte-scale storage that supports thousands of NFSv4 connections
- Pay for the storage you use (no pre-provisioning required)
- Data is stored across multiple AZs within a region
- Read after Write consistency

## EBS
- Persistent block storage volumes for use with EC2s
- Automatically replicated within its Availability Zone (AZ)

### EBS Types
API Name | Volume Type
--- | ---
gp2 | General Purpose SSD
io1 | Provisioned IOPS SSD
st1 | Throughput Optimized HDD
sc1 | Cold HDD
Standard | EBS Magnetic

### EBS Volumes & Snapshots
- Snapshots exist on S3 and are incremental (they store the delta between versions)
- Best to stop the instance before taking a snapshot of a root EBS volume
- AMIs can be created from both Volumes and Snapshots
- EBS volume size and storage type can be changed on the fly
- Volumes will always be in the same AZ as the EC2 instance
- EC2 volumes can be moved to different AZs and regions

## CloudWatch
- Monitors AWS resources and applications running on AWS
- Host level metrics consist of CPU, network, disk, and status check (e.g. underlying hypervisor)
- Will monitor EC2 events every 5 minutes (1 minute intervals using detailed monitoring)
- Create alarms which trigger notifications (e.g. for billing)
- Create dashboards, events, and logs
- CloudTrail increases visibility on user resource activity by recording console actions and API calls

## Databases
- Relational databases have tables (worksheets) with rows and fields (columns)
    - SQL, MySQL, PostgreSQL, Oracle, Amazon Aurora, MariaDB
    - Multi-AZ for disaster recovery (DR) and read repliacas for performance
- Non relational databases have collections (tables) with documents (rows) and key value pairs (fields or columns)
    - DynamoDB (NoSQL)
- Data warehousing is for business intelligence (BI), used to pull in large and complex data sets for querying
    - Amazon Redshift for online analytics processing (OLAP)
- ElastiCache improves the performance of web apps by caching the most common web queries (as opposed to retrieving from disk-based databases)
    - Memcached
    - Redis

### Relational Databases
- RDS runs on virtual machines (cannot log in; patching is Amazon's responsibility)
- RDS is not serverless
    - Aurora Serverless is serverless
- Automated backups allow you to recover within a retention period between one and 35 days
    - Takes daily snapshots and stores transaction logs which allows for recovery down to a second
    - Enabled by default and stored in S3
- Database snapshots are manual and are still stored if the original RDS has been deleted (unlike automated backups)
- Restored database versions have a new DNS endpoint
- Encryption uses the KMS service for encryption at rest (includes backups, read replicas, and snapshots)
- Read replicas are available for MySQL, PostgreSQL, MariaDB, and Aurora
    - Backups must be enabled
    - Can be in different regions
- Multi-AZ failover can be forced by rebooting the active RDS instance

### DyanamoDB
- For applications that need consistent, single-digit millisecond latency at any scale
- Supports both document and key-value data models
- Stored on SSDs and spread across 3 geographically distinct data centres
- Eventual consistent reads (default) is when consistency across all copies is usually reached within a second (best read performance)
- Strongly consistent reads return a result that reflects all writes that received a successful response prior to the read

### Redshift
- Petabyte-scale data warehouse service
- Single node (160Gb)
- Multi-node
    - Leader node (manages client connections and receives queries)
    - Up to 128 compute nodes that store data and perform queries and computations
- Advanced compression performed on columns
- Backups are enabled by default with a 1 day retention period (max 35 days)
    - Always attempts to maintain 3 copies (original, replica, and S3 backup)
    - Can asynchronously replicate snapshots to S3 in another region for DR
- Encryption in transit using SSL and at rest using AES-256
- Multi-AZ currently not available

### Aurora
- MySQL-compatible relational database engine
- Start with 10GB and scales in 10GB increments to 64TB
- 2 copies are contained in each AZ with a minimum of 3 AZs (6 copies)
- Designed to handle losing 2 copies of data without affecting write, and 3 copies without affecting read
- Self-healing (disks are continuously scanned for errors and repaired automatically)
- Aurora Replicas (up to 15) or MySQL Replicas (up to 5)
- Automated backups are always enabled and snapshots are available

### Elasticache
- Web service to deploy, operate, and scale an in-memory cache in the cloud
- Memcached for scaling horizontally
- Redis for advanced data types, Multi-AZ, and backup/restore