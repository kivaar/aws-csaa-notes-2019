# aws-csaa-notes-2019

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
