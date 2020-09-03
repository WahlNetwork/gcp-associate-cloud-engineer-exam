# Google Associate Cloud Engineer Exam

- [Google Associate Cloud Engineer Exam](#google-associate-cloud-engineer-exam)
  - [Design Principles](#design-principles)
    - [Global System](#global-system)
    - [Separation of Duties](#separation-of-duties)
    - [Physical Presence](#physical-presence)
    - [Cost](#cost)
    - [Projects](#projects)
    - [References](#references)
    - [Marketplace](#marketplace)
  - [Management](#management)
    - [Cloud Shell](#cloud-shell)
    - [Billing](#billing)
    - [API](#api)
    - [gcloud](#gcloud)
  - [Storage](#storage)
    - [Cloud Storage](#cloud-storage)
    - [Cloud Bigtable](#cloud-bigtable)
  - [Database](#database)
    - [BigQuery](#bigquery)
  - [Compute](#compute)
    - [Google Compute Engine (GCE)](#google-compute-engine-gce)
    - [Cloud Functions](#cloud-functions)
  - [Security](#security)
    - [Concepts](#concepts)
    - [Data Flow](#data-flow)
    - [Identity and Access Management (IAM)](#identity-and-access-management-iam)
    - [References](#references-1)
  - [Network](#network)
    - [Routing Tiers](#routing-tiers)
    - [Cloud Load Balancer](#cloud-load-balancer)
    - [Virtual Private Cloud (VPC)](#virtual-private-cloud-vpc)
    - [Cloud DNS](#cloud-dns)
  - [Kubernetes](#kubernetes)
    - [Cluster](#cluster)
    - [Networking](#networking)
    - [Services](#services)
    - [Storage](#storage-1)
    - [Security](#security-1)
    - [Logging](#logging)
    - [Secrets / Variables](#secrets--variables)
    - [Applications](#applications)
    - [kubectl](#kubectl)
  - [App Engine](#app-engine)
    - [Application](#application)
    - [Environments](#environments)
    - [Deployment](#deployment)
    - [Testing](#testing)
    - [Security](#security-2)

## Design Principles

### Global System

- Focused on delivering a global set of services
- Simpler handling of latency and regional failures

### Separation of Duties

- Data Center personnel have no access to services, and the reverse is true

### Physical Presence

- Data Centers
- Zones
- Regions
- Multi-Regions

	- 100+ miles between regions

- Points of Presence (PoP)

### Cost

- Provisioned resources
- Usage of resources
- Network Traffic

	- Free to come in
	- Costs money to leave

- Pricing Calculator

	- https://cloud.google.com/products/calculator/

### Projects

- Info

	- Used to manage and group resources
	- Console generates a unique ID suffix upon creation
	- Each project has a unique dashboard
	- Supported by a billing account
	- Most services require the context of a project

		- Some resources tie directly to the billing account

- Permissions

	- Linked billing account changes

		- Project owner
		- Project billing administrator

- Best Practices

	- Best to separate projects rather than dumping all resources into one or a few

- Sharing

	- Can share a VPC among multiple projects
	- Host Project owns the shared VPC
	- Service Projects have access to the VPC

- gcloud

	- gcloud projects create [PROJECT-ID] --name "[PROJECT NAME]"
	- Must link projects made in gcloud to a billing account

### References

- https://github.com/gregsramblings/google-cloud-4-words
- https://raw.githubusercontent.com/gregsramblings/google-cloud-4-words/master/DarkPoster-medres.png

### Marketplace

- Known as GCP Marketplace

	- FKA Cloud Launcher
	- https://cloud.google.com/marketplace

## Management

### Cloud Shell

- Web based shell

	- Session remains even across browser disconnects / interruptions
	- Boost mode launches a new, more powerful Cloud Shell machine
	- 5GB home directory will persist across sessions, but the VM is ephemeral and will be reset after 20 minutes from session end

- Number of pre-installed tools

	- gcloud
	- kubectl
	- docker
	- pip/python
	- emacs
	- Can download files using "dl" command

- Web Preview

	- Ability to connect to Cloud Shell over range of pre-defined ports, e.g. 8080
	- Can use simple node/nodemon scripts to host web page(s)

- Editor

	- View files in the home directory
	- Modify files and folders

- References

	- https://cloud.google.com/shell/docs/quickstart
	- https://cloud.google.com/shell/docs/examples

### Billing

- Billing Accounts

	- Pays for the GCP service usage
	- Outside of a project scope, but linked to a project (to pay)

		- Projects created in the Console have a billing account attached by default
		- Projects created via CLI (gcloud) may require using "gcloud beta billing" command to link a billing account

	- Owned by the GCP Organization

- Setup a billing export into a BigQuery data set to easily query billing data

	- Only feeds in net-new billing data once configured

- Budgets

	- Setup a budget to projects or billing accounts

		- Can also specify Google services to associate with the budget

	- Budgets trigger alerts at thresholds
	- https://cloud.google.com/billing/docs/how-to/budgets

- Billing IAM

	- Billing Account User role

		- Links projects to billing accounts

			- One billing account per project

		- Practice least privledge by splitting roles out from primary account

	- Roles

		- Creator

			- Create new self-serve (online) billing accounts

				- Organization level assignment

		- Administrator

			- Manage (but not create) billing accounts

		- User

			- Link projects to billing accounts

				- Often combined with "Project Creator" to allow devs to create a new project and link a billing account

		- Viewer

			- View billing account cost information and transactions

		- Manager

			- Link/unlink the project to/from a billing account

### API

- Configuration

	- Some APIs are enabled by default, such as Cloud Storage
	- Other APIs require being enabled

		- Compute, as an example
		- gcloud services enable compute.googleapis.com

	- Search for APIs

		- gcloud services list --enabled (default)
		- gcloud services list --available | grep compute

### gcloud

- Syntax

	- gcloud <global flags> <service/product> <group/area> <command> <flags> <params>
	- Interpreted from left to right
	- Global Flags

		- --help (-h)
		- --project <ProjectID>
		- --account <Account>
		- --filter

			- Alternative to grep
			- Not always available

		- --format

			- JSON, YAML, CSV, etc.

		- --quiet (-q)

	- Configuration Properties

		- Specify once, re-use by commands that require the value
		- Can be overided by local parameters in a specific command
		- Can be saved for "workspace" style profiles

			- gcloud config configurations list

		- gcloud config list
		- gcloud config get-value <property>

	- SSH

		- Can setup SSH keys on your behalf

			- gcloud compute ssh <VM>
			- Prompts SSH keygen

		- SSH keys are stored in ~\ by default
		- Also appear in Console, such as GCE keys in GCE > Metadata > SSH Keys

- References

	- https://cloud.google.com/sdk/gcloud/

- .gcloudignore

	- File specifies files that are *not* uploaded to Google Cloud Platform
	- gcloud topic gcloudignore

## Storage

### Cloud Storage

- General

	- Objects are private by default

		- Can make public by adding "Public" entity with Reader permission.

	- Bucket names must be unique all of Cloud Storage
	- Maximum size limit of 5 TB for individual objects stored in Cloud Storage

- Design

	- Per-project rate limit to bucket creation and deletion of approximately 1 request every 2 seconds
	- Plan on fewer buckets and more objects in most cases
	- Pre-create all the buckets necessary

		- Bucket names are part of a centralized and global namespace
		- Dependency on this namespace creates a single point of failure for your application

- Tiers

	- Standard

		- No min storage duration
		- Expensive storage, cheaper operations

	- Nearline

		- 30 day min storage duration
		- Cheaper storage (about half of Standard), slightly more expensive operations

	- Coldline

		- 90 day min storage duration
		- Cheaper storage (about 1/5 of Standard), more expensive operations

	- Archive

		- 365 day min storage duration
		- Cheapest storage (about 1/20 of Standard), most expensive operations

- Pricing

	- Data storage: the amount of data stored in your buckets. Storage rates vary depending on the storage class of your data and location of your buckets.
	- Network usage: the amount of data read from or moved between your buckets.

		- Free

			- Data moves within the same location
			- Data moves from a Cloud Storage bucket located in a region to a different Google Cloud service located in a multi-region, and both locations are on the same continent
			- Data moves from a Cloud Storage bucket located in a multi-region to a different Google Cloud service located in a region, and both locations are on the same continent

	- Operations usage: the actions you take in Cloud Storage, such as listing the objects in your buckets.
	- Retrieval and early deletion fees: applicable for data stored in the Nearline Storage, Coldline Storage, and Archive Storage classes.
	- Minimum storage duration

		- You can delete the file before it has been stored for this duration, but at the time of deletion you are charged as if the file was stored for the minimum duration

- Roles

	- Storage Object Creator

		- Allows users to create objects. Does not give permission to view, delete, or overwrite objects

	- Storage Object Viewer

		- Grants access to view objects and their metadata, excluding ACLs.
		- Can also list the objects in a bucket.

	- Storage Object Admin

		- Grants full control over objects, including listing, creating, viewing, and deleting objects.

	- Storage HMAC Key Admin

		- Full control over HMAC keys in a project. This role can only be applied to a project

	- Storage Admin

		- Grants full control of buckets and objects.
		- When applied to an individual bucket, control applies only to the specified bucket and objects within the bucket.

	- https://cloud.google.com/storage/docs/access-control/iam-roles

- gsutil

	- Google Cloud Storage command line utility
	- Provides gs://bucket-url browsing of storage buckets

		- gs://wahlnetwork-storage-1/

	- Get, set, and ch(ange) labels

		- gsutil label ch -l labelkey:labelvalue gs://bucket
		- Otherwise, must use a json file with "set"

	- Access Control List (ACL)

		- This command can make a file public to the Internet
		- gsutil acl ch -u AllUsers:R gs://bucket-url/filename.png

	- Versioning

		- gsutil versioning verb argument

### Cloud Bigtable

- IoT

	- Ingest and analyze large volumes of time series data from sensors in real time, matching the high speeds of IoT data to track normal and abnormal behavior. Enable customers to build dashboards and drive analytics on their data in real time.

- Consistent sub-10ms latency—handle millions of requests per second
- Ideal for use cases such as personalization, ad tech, fintech, digital media, and IoT
- Seamlessly scale to match your storage needs; no downtime during reconfiguration
- Designed with a storage engine for machine learning applications leading to better predictions
- Easily connect to Google Cloud services such as BigQuery or the Apache ecosystem

## Database

### BigQuery

- bq

	- BigQuery command line utility

- BigQuery jobs

	- Jobs are actions that BigQuery runs on your behalf to load data, export data, query data, or copy data.
	- When you use the Cloud Console, the classic BigQuery web UI, or the bq command-line tool to load, export, query, or copy data, a job resource is automatically created, scheduled, and run. You can also programmatically create a load, export, query, or copy job. When you create a job programmatically, BigQuery schedules and runs the job for you.
	- Because jobs can potentially take a long time to complete, they execute asynchronously and can be polled for their status. Shorter actions, such as listing resources or getting metadata, are not managed by a job resource.

- Cost

	- Dry Run

		- When you run a query in the bq command-line tool, you can use the --dry_run flag to estimate the number of bytes read by the query.
		- You can use the estimate returned by the dry run to calculate query costs in the pricing calculator.

			- You are not charged for performing the dry run.

## Compute

### Google Compute Engine (GCE)

- Instance Types

	- N1

		- Intel Skylake
		- "standard" instances are roughly x vCPU and 4x RAM
		- "highmem" instances are roughly x vCPU and 7-8x RAM
		- "highcpu" instances are roughly x vCPU and 0.9x RAM

	- M1

		- Intel Skylake with massive memory
		- Example: "megamem-96" has 96 vCPU and 1.4 TB of RAM

	- C2

		- Intel Cascade Lake
		- "standard-8" has 8 vCPU and 32 GB RAM

	- General

		- highcpu

			- Each vCPU has the minimum amount of RAM allocated
			- Cheapest option

		- standard

			- Standard ratio of vCPU to RAM

				- 3.75 GB per vCPU, on average

		- highmem

			- Max amount of RAM per vCPU allocated
			- Most expensive option

	- Custom

		- gcloud compute instances create --custom-memory --custom-cpu

			- A whole number value indicating how much memory is desired in the custom machine type
			- A whole number value indicating how many cores are desired in the custom machine type

		- https://cloud.google.com/sdk/gcloud/reference/compute/instances/create

- Metadata

	- Metadata is available while in the VM (or Cloud Shell via SSH)
	- curl -H "Metadata-Flavor:Google" metadata.google.internal/computeMetadata/v1/

		- Must use the header information

	- Add custom metadata for use in scripts or other automation

		- Such as bucket name for storing reports or logs
		- Similar idea to environmental variables

- Preemptible VM

	- Cheap instances that can be interrupted by Google
	- Last for 24 hours before self-terminating
	- Will not live migrate for maintenance

- Automation

	- Startup Script

		- Runs on startup or after a reboot

	- Metadata

		- Custom key/value pairs visible to the instance

- Logging

	- Install the Logging Agent into an instance to capture logs

		- The Logging agent google-fluentd is a modified version of the fluentd log data collector
		- A minimum of 250 MiB memory is required, but 1 GiB is recommended
		- https://cloud.google.com/logging/docs/agent/installation

	- Can also configure fluentd to capture / tail custom log files

		- https://medium.com/google-cloud/how-to-log-your-application-on-google-compute-engine-6600d81e70e3
		- https://cloud.google.com/logging/docs/agent/configuration

- Managed Instance Groups

	- Uses instance templates

		- Templates are immutable

	- Scope

		- Single zone
		- Multiple Zones

	- Autoscaling

		- Options

			- On
			- Off
			- Up only

		- Metrics

			- CPU Utilization
			- HTTP load balancing
			- Stackdriver metric

				- See: Operations

		- Cool Down Period

			- Metrics not collected from instance until time (in seconds) expires
			- Prevents boot / startup script from triggering false-positive

	- Auto Healing

		- Health checks for service health within the instance(s)

	- Restart / Replace

		- Restart instances in the group
		- Replace instances in the group with fresh instance template copies

- Lifecycle

	- PROVISIONING. Resources are being allocated for the instance. The instance is not running yet.
	- STAGING. Resources have been acquired and the instance is being prepared for first boot.
	- RUNNING. The instance is booting up or running. You should be able to ssh into the instance soon, but not immediately, after it enters this state.
	- STOPPING: the instance is being stopped. This can be because a user has made a request to stop the instance or there was a failure. This is a temporary status and the instance will move to TERMINATED.
	- TERMINATED. A user shut down the instance, or the instance encountered a failure. You can choose to restart the instance or delete it.
	- SUSPENDING. The instance is being suspended. A user has suspended the instance.
	- SUSPENDED. The instance is suspended. You can choose to resume or delete it.

### Cloud Functions

- Serverless
- Deploy code to handle singular tasks in a microservices architecture
- Pay only for what is used in per-100ms increments

## Security

### Concepts

- CIA Triad

	- Confidentiality

		- Permissions dictate data access

	- Integrity

		- Data cannot be changed without proper permissions

	- Availability

		- Data can be read when needed, when the scope and permissions grant this

- AAA

	- Authentication

		- Basic
		- 2FA

	- Authorization

		- Permissions
		- API Scope

	- Accounting

		- Logs showing activity
		- Reading, viewing, writing, deleting, etc.

- Defense in Depth

	- Include layers of defense when designing a solution
	- Edge, user education, backups

### Data Flow

- User supplies credentials to authentication source
- If successful, credential token returned
- User hands over token to application
- Application verifies the token with the authentication source
- Application validates user has access with authorization source
- User granted access to resource and desired resource(s) returned

### Identity and Access Management (IAM)

- Flow

	- Members are the users, groups, service accounts, and other entities
	- Permissions determine what operations are allowed on a resource
	- A role is a collection fo permissions
	- The resource heirarchy contains resources arranged in a permission inheriting structure that allows policies to be assigned
	- Policies are represented by policy objects that bind Member(s) to Role(s), thus granting permissions to the resource

- Members

	- user

		- Avoid assigning permissions to users directly
		- Sources

			- G Suite
			- Cloud Identity
			- Gmail
			- Validated Email

	- serviceAccount

		- Used for applications and services
		- Default maximum of 100 per project
		- Google Cloud-managed keys are rotated daily
		- Service Account Keys

			- Public/private key pairs provide a secure way of providing access to a service account outside of Google Cloud
			- A service account can have up to 10 keys
			- gcloud iam service-accounts keys create ~/key.json \
			- gcloud iam service-accounts keys create ~/key.json --iam-account sa-name@project-id.iam.gserviceaccount.com

	- group

		- Default method for handling multi-user permissions
		- Group of users and service accounts
		- Can own a project

	- domain

		- All accounts managed by G Suite or Cloud Identity

	- allAuthenticatedUsers

		- Any Google account or service account
		- Essentially "public" while requiring any user account

	- allUsers

		- Public

- Roles

	- Collection of permissions

		- Permissions

			- service.resource.verb

		- Don't apply roles to users in most cases; apply to groups

	- Primitive Roles

		- Assigned at the project level as grand roles
		- Role types

			- Viewer
			- Editor
			- Owner

	- Predefined Roles

		- Templates containing commonly used permissions for a service
		- Admin, Service Admin, Data Editor, Subscriber, etc. for each service
		- Can be combined to make new roles

			- Select multiple roles in the console
			- Choose "Create role from selection"
			- Will comment with source roles

		- BigQuery Data Editor

			- roles/bigquery.dataEditor

		- Pub/Sub Subscriber

			- roles/pubsub.subscriber

	- Custom Roles

		- Build your own!

	- gcloud

		- gcloud projects add-iam-policy-binding

- Resource Heirarchy

	- Organization

		- Tied to G Suite or Cloud Identity domain

	- Folders

		- Contains projects or subfolders
		- Commonly used for departments or department groups

	- Projects

		- Container for related resources
		- Commonly used for stages, tiers, and versions of code / applications
		- Trust services in the same project by default

- Policies

	- General

		- Policies bind Members and Roles for a resource
		- One policy per resource
		- There is no "deny"

			- Use least privledge - don't assign highly permissive roles at the org or parent project levels
			- Grant more permissions to more granular resource heirarchy objects

	- Attached to the Resource Hierarchy

		- Roles and Members listed in policy
		- Max 1500 member bindings per policy
		- Max 250 group bindings per policy

	- gcloud

		- Use add-iam-policy-binding and remove-iam-policy-binding instead of get-iam-policy and set-iam-policy
		- gcloud [GROUP] add-iam-policy-binding [RESOURCE-NAME] --role [ROLE-ID-TO-GRANT] --member user:[USER-EMAIL]

- Resources

	- https://cloud.google.com/iam/docs/faq
	- https://cloud.google.com/iam/docs/using-iam-securely

### References

- https://cloud.google.com/security/infrastructure/design

## Network

### Routing Tiers

- Standard

	- Standard Internet routing of traffic to POP

- Premium

	- Traffic source enters POP early, GCP handles routing to resource(s)

### Cloud Load Balancer

- Features

	- Single IP address to serve as the frontend
	- Automatic intelligent autoscaling of your backends
	- External load balancing for when your users reach your applications from the internet
	- Internal load balancing for when your clients are inside of Google Cloud
	- Regional load balancing for when your applications are available in a single region
	- Global load balancing for when your applications are available across the world
	- Pass-through load balancing (see also direct server return (DSR) or direct routing)
	- Proxy-based load balancing (as an alternative to pass-through)
	- Layer 4-based load balancing to direct traffic based on data from network and transport layer protocols, such as IP address and TCP or UDP port
	- Layer 7-based load balancing to add content-based routing decisions based on attributes, such as the HTTP header and the uniform resource identifier
	- Integration with Cloud CDN for cached content delivery
	- https://cloud.google.com/load-balancing/docs/load-balancing-overview

- Traffic Types

	- For HTTP and HTTPS traffic, use:

		- External HTTP(S) Load Balancing
		- Internal HTTP(S) Load Balancing

	- For TCP traffic, use:

		- TCP Proxy Load Balancing
		- Network Load Balancing
		- Internal TCP/UDP Load Balancing

	- For UDP traffic, use:

		- Network Load Balancing
		- Internal TCP/UDP Load Balancing

### Virtual Private Cloud (VPC)

- Subnets

	- Regional scope
	- Connected globally via GCP backbone
	- Creation Modes

		- Custom
		- Automatic

			- Creates a subnet in each region to act as an IP pool

	- Subnet range can be modified later

		- New value must be larger; cannot shrink a subnet
		- New value must include existing addresses

- Routes

	- Global scope
	- Tag-based routing available

		- e.g. If network tag "public" assigned, add route to the Internet

- Firewall Rules

	- Global scope
	- Configuration

		- Logs
		- Direction of Traffic

			- Ingress
			- Egress

		- Action on Match

			- Deny
			- Allow

		- Targets

			- All
			- Network tag(s)
			- Service account

		- Source Filter

			- Based on target type

		- Protocols and Ports

			- tcp
			- udp
			- other

### Cloud DNS

- https://cloud.google.com/dns/docs/tutorials/create-domain-tutorial
- Flow

	- Create a managed public or private DNS zone
	- Create a new set of records for that zone

		- A record
		- CNAME
		- etc.

	- Set DNS servers to use Cloud DNS name servers

		- https://cloud.google.com/dns/docs/update-name-servers

## Kubernetes

### Cluster

- API

	- RESTful design
	- Maps to CRUD
	- Groups

		- Core

			- Represented as "" in some places

		- Apps
		- Authorization
		- Storage

	- Clients

		- kubectl

			- Tool for making requests to the API server

		- Nodes
		- Kubernetes services

- Resource Quotas

	- A resource quota, defined by a ResourceQuota object, provides constraints that limit aggregate resource consumption per namespace.
	- It can limit the quantity of objects that can be created in a namespace by type, as well as the total amount of compute resources that may be consumed by resources in that project.

- Autoscaling

	- Cluster considers resource request values, not real-time utilization values, when deciding when to autoscale

- Manual Scaling

	- To resize a cluster's node pools, run the following command:
	- gcloud container clusters resize cluster-name --node-pool pool-name --num-nodes num-nodes

- https://cloud.google.com/solutions/best-practices-for-operating-containers

### Networking

- Design

	- All nodes can talk
	- All pods can talk (without NAT)
	- Every pod gets a unique IP address

- Pod Network
- Node Network
- Services Network

	- Rewrites egress traffic destinated to a service network endpoint with a pod network IP address
	- Proxy Modes

		- IPTABLES

			- Since 1.2
			- Poor scalability

		- IPVS Mode

			- Since 1.11
			- Linux IP Virtual Server (IPVS)
			- L4 load balancer

### Services

- Types

	- ClusterIP

		- Exposes the Service on a cluster-internal IP. Choosing this value makes the Service only reachable from within the cluster.
		- This is the default ServiceType.

	- NodePort

		- Exposes the Service on each Node's IP at a static port (the NodePort).
		- A ClusterIP Service, to which the NodePort Service routes, is automatically created.
		- You'll be able to contact the NodePort Service, from outside the cluster, by requesting <NodeIP>:<NodePort>.

	- LoadBalancer

		- Exposes the Service externally using a cloud provider's load balancer.
		- NodePort and ClusterIP Services, to which the external load balancer routes, are automatically created.

	- ExternalName

		- Maps the Service to the contents of the externalName field (e.g. foo.bar.example.com), by returning a CNAME record with its value.
		- No proxying of any kind is set up.

- Reference

	- https://kubernetes.io/docs/concepts/services-networking/service/

### Storage

- Container Storage Interface (CSI)

	- Defines a standard interface for container orchestration systems (like Kubernetes) to expose arbitrary storage systems to their container workloads.
	- CSI support was introduced as alpha in Kubernetes v1.9, moved to beta in Kubernetes v1.10, and is GA in Kubernetes v1.13.

- Persistent Volumes

	- A piece of storage in the cluster that has been provisioned by an administrator or dynamically provisioned using Storage Classes.
	- Persistent Volume Claim (PVC)

		- Request for storage by a user.
		- Claims can request specific size and access modes
		- Access modes

			- ReadWriteOnce

				- single mount with one writer

			- ReadWriteMany

				- multiple mounts with multiple writers

			- ReadOnlyMany

				- multiple mounts with multiple readers

	- Binding

		- A PVC to PV binding is a one-to-one mapping, using a ClaimRef which is a bi-directional binding between the PersistentVolume and the PersistentVolumeClaim
		- Claims will remain unbound indefinitely if a matching volume does not exist.

	- Mount Propogation

		- Mount propagation of a volume is controlled by mountPropagation field in Container.volumeMounts

			- None
			- HostToContainer
			- Bidirectional

- Storage Classes

	- Labels for storage resources to describe their characteristics.
	- Default

		- A default storage class is used when a pvc does not specify the storage class
		- Can be handy when a single default services all pod volumes

	- Provisioner

		- Determines the volume plugin to use for provisioning PVs.
		- Example: gke-pd, azure-disk

	- Reclaim Policy

		- PersistentVolumes that are dynamically created by a StorageClass will have the reclaim policy specified in the reclaimPolicy field of the class

			- Delete

				- When pvc is deleted, also deletes pv and underlying storage object

			- Retain

				- When pvc is deleted, pv remains and volume is "released"

### Security

- Authentication

	- AuthN
	- Prove your identity

		- Bearer tokens
		- Certificates
		- External systems

	- Users

		- Not a native part of Kubernetes
		- Active Directory, IAM, and others supply user objects

	- Service Accounts

		- For system components

- Authorization

	- AuthZ
	- Prove your permissions

		- Subject
		- Verb
		- Resource

	- Objects

		- Role

			- Permissions for Kubernetes
			- Rules

				- API Groups

					- Core, Apps, etc.

				- Resources

					- Pods, Nodes, Deployments, etc.

				- Verbs

					- Get, Describe, Set, etc.

			- ClusterRole

				- Similar to role, except no namespace requirement; applies to the entire cluster

		- RoleBinding

			- Binds rules to users and groups
			- Includes a namespace

	- Authorization Mode

		- Node
		- RBAC

	- Resources

		- https://kubernetes.io/docs/reference/access-authn-authz/rbac/
		- https://kubernetes.io/docs/reference/access-authn-authz/authorization/

- Admission Control

	- Admission controllers are plugins that govern and enforce how the cluster is used
	- Gatekeeper that intercept (authenticated) API requests and may change the request object or deny the request altogether

		- Require pulling images from specific registries
		- Enforce security standards
		- Governance

			- Labels

		- Configuration management

	- Web hooks

		- Used for custom logic without having to merge code into Kubernetes
		- ValidatingAdmissionWebhooks 

			- Validates the request

		- MutatingAdmissionWebhooks

			- Can mutate (change) the request

				- Example: Change a flag in the request to meet corporate governance requirement

- Schema Validation

### Logging

- Write logs to stdout and stderr

	- Docker captures these log lines and allows you to access them by using the docker logs command
	- Use native logging mechanisms
	- Logging provided by fluentd and Cloud Logging

		- Fluentd is an open source data collector for unified logging layer
		- https://www.fluentd.org/
		- A single log line is stored as a document, along with some metadata, in JSON format

	- Elasticsearch, Fluentd, Kibana combination (EFK) is also popular

		- Elastisearch

			- Search engine based on the Lucene library
			- Distributed, multitenant-capable full-text search engine
			- HTTP web interface 
			- Schema-free JSON documents

		- Kibana

			- Open source data visualization dashboard for Elasticsearch
			- Visualization capabilities on top of the content indexed on an Elasticsearch cluster

- Some apps do not natively (or easily) support stdout and stderr

	- Deploy a sidecar for logging

		- Share an empty volume between two containers
		- App writes to shared volume
		- Logging agent reads and forwards logs where they need to go
		- https://cloud.google.com/solutions/best-practices-for-operating-containers#log_aggregator_sidecar_pattern

### Secrets / Variables

- ConfigMap

	- Allow you to decouple configuration artifacts from image content to keep containerized applications portable

		- kubectl create configmap <map-name> <data-source>

	- https://kubernetes.io/docs/tasks/configure-pod-container/configure-pod-configmap/

### Applications

- Lifecycle

	- Pods are only scheduled once in their lifetime.

		- Once a Pod is scheduled (assigned) to a Node, the Pod runs on that Node until it stops or is terminated.

	- If a Node dies, the Pods scheduled to that node are scheduled for deletion after a timeout period.

		- A given Pod (as defined by a UID) is never "rescheduled" to a different node; instead, that Pod can be replaced by a new, near-identical Pod, with even the same name i desired, but with a different UID.

	- https://kubernetes.io/docs/concepts/workloads/pods/pod-lifecycle/

- Container States

	- Waiting

		- A container in the Waiting state is still running the operations it requires in order to complete start up

	- Running

		- The Running status indicates that a container is executing without issues.

	- Terminated

		- A container in the Terminated state has begin execution and has then either run to completion or has failed for some reason.

- Deployment

	- Replicate Set

		- Managed by the deployment

	- Rolling Update

		- Max Surge

			- The quantity of additional replicas allowed during an update
			- Can be quantity or percentage

		- Max Unavailable

			- The quantity of replicas allowed to be unavailable during an update

		- Updates in-flight can be superceded by more recent updates

	- MinReadySeconds

		- Waiting period between pod deployments during a rollout

	- Resource requests

		- Requests for CPU and other resources must be made if using HPAs or cluster autoscaling
		- Use limits and requests

	- Lifecycle

		- Pending

			- The Pod has been accepted by the Kubernetes cluster, but one or more of the containers has not been set up and made ready to run.

				- If no resources available on cluster, Cluster Autoscaling will increased node count if enabled
				- Once node count satisfied, pods in Pending status will be deployed

		- Running

			- The Pod has been bound to a node, and all of the containers have been created.

		- Succeeded

			- All containers in the Pod have terminated in success, and will not be restarted.

		- Failed

			- All containers in the Pod have terminated, and at least one container has terminated in failure.

		- Unknown

			- For some reason the state of the Pod could not be obtained.

- Horizontal Pod Autoscaler

	- Automatically scales the number of pods in a replication controller, deployment, replica set or stateful set based on observed metrics.

		- Implemented as a Kubernetes API resource and a controller

	- Works on utilized resource values, not requested values
	- Control loop default value of 15 seconds
	- https://kubernetes.io/docs/tasks/run-application/horizontal-pod-autoscale/

- Custom Resource Definition (CRD)

	- An extension of the Kubernetes API that is not necessarily available in a default Kubernetes installation.
	- A customization of a particular Kubernetes installation.
	- CRDs allow users to create new types of resources without adding another API server.

		- You do not need to understand API Aggregation to use CRDs.

- Others

	- DaemonSet

		- A DaemonSet ensures that all (or some) Nodes run a copy of a Pod
		- As nodes are added to the cluster, Pods are added to them.
		- As nodes are removed from the cluster, those Pods are garbage collected.

	- Statefulset

		- Manages the deployment and scaling of a set of Pods, and provides guarantees about the ordering and uniqueness of these Pods.

			- StatefulSet maintains a sticky identity for each of their Pods.
			- These pods are created from the same spec, but are not interchangeable: each has a persistent identifier that it maintains across any rescheduling.

		- Use Cases

			- Stable, unique network identifiers.
			- Stable, persistent storage.
			- Ordered, graceful deployment and scaling.
			- Ordered, automated rolling updates.

	- Job

		- A Job creates one or more Pods and ensures that a specified number of them successfully terminate.

			- As pods successfully complete, the Job tracks the successful completions.
			- When a specified number of successful completions is reached, the task (ie, Job) is complete.
			- Deleting a Job will clean up the Pods it created.

	- CronJob

		- One CronJob object is like one line of a crontab (cron table) file.
		- It runs a job periodically on a given schedule, written in Cron format.

### kubectl

- apply

	- kubectl apply -f [FILE].yml

- misc

	- kubectl get svc --watch
	- kubectl get pods -o wide

- exec

	- kubectl exec [POD] -it -- [CMD]

- pv

	- Persistent Volume
	- kubectl get pv
	- kubectl describe pv

- pvc

	- Persistent Volume Claim
	- kubectl get pvc
	- kubectl describe pvc

- rs

	- Replica Set
	- kubectl get rs

- hpa

	- Horizontal Pod Autoscaler
	- kubectl get hpa

## App Engine

### Application

- Service

	- Version

		- Instance

	- Logical components
	- Similar to microservices

- Top-level container that includes the service, version, and instance resources
- All resources are created in the chosen region

	- You cannot change an app's region once it has been set.

- Python 3

	- app.yaml

		- Defines runtimes, handlers, scaling, and other general settings including environment variables
		- You must first create the app.yaml file for the default service of your app before you can create and deploy app.yaml files for additional services.
		- Requires a runtime element

			- runtime: python38

		- Requires a service element (if not default)
		- https://cloud.google.com/appengine/docs/standard/python3/config/appref

	- requirements.txt

		- Dependencies specified in the requirements.txt file will be installed automatically with your deployed app

	- The Google Cloud SDK includes a local development server named dev_appserver that you can run locally to simulate your application running in production App Engine

		- dev_appserver.py

### Environments

- Standard

	- Specific software verisons

		- Python 2.7, Python 3.7, Python 3.8
		- Java 8, Java 11
		- Node.js 8, Node.js 10, and Node.js 12
		- PHP 5.5, PHP 7.2, PHP 7.3, and PHP 7.4 (beta)
		- Ruby 2.5, Ruby 2.6 (beta), and Ruby 2.7 (beta)
		- Go 1.11, Go 1.12, Go 1.13, and Go 1.14 (beta)

	- Sandbox
	- Free / low cost execution

		- Scale to 0 when no activity

	- Spikes in traffic

- Flexible

	- Flexible software version support
	- Runs in Docker container
	- Code deployments can take longer as container images are built by using the Cloud Build service
	- SSH access to the VM instances in the flexible environment is disabled

- https://cloud.google.com/appengine/docs/the-appengine-environments

### Deployment

- Deploy versions of your application's services and each of their configuration files
- Specifying no files with the command deploys only the app.yaml file in your current directory.

	- gcloud app deploy

- Deployment options

	- To specify a custom version ID, use the --version flag.
	- To prevent traffic from being automatically routed to the new version, use the --no-promote flag.
	- To deploy to a specific Google Cloud project, use the --project flag.

### Testing

- Prevent traffic from being automatically routed to the new version

	- gcloud app deploy --no-promote

- When you want to send traffic to the new version, use the Cloud Console to migrate traffic
- Many Google Cloud client libraries depend on environment variable with Cloud project ID

	- GOOGLE_CLOUD_PROJECT
	- dev_appserver.py --application=PROJECT_ID

### Security

- App Engine Admin

	- /roles/appengine.appAdmin
	- Read/Write/Modify access to all application configuration and settings.

- App Engine Deployer

	- /roles/appengine.deployer
	- Read-only access to all application configuration and settings.
	- Write access only to create a new version; cannot modify existing versions other than deleting versions that are not receiving traffic. Cannot configure traffic to a version.

- App Engine Service Admin

	- /roles/appengine.serviceAdmin
	- Read-only access to all application configuration and settings.
	- Write access to service-level and version-level settings, including traffic configuration. Cannot deploy versions of apps.

- App Engine Viewer

	- /roles/appengine.appViewer
	- Read-only access to all application configuration and settings.

- App Engine Code Viewer

	- /roles/appengine.codeViewer
	- Read-only access to all application configuration and settings, and to deployed source code.

