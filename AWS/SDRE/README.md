# Secure Digital Research Environments on the AWS Cloud
## [Extended Documentation (Source)](http://cloudmaven.github.io/documentation/aws_hipaa.html)


These notes concern management of Private Health Information (PHI) under HIPAA regulations.
The original version is (as of March 2022) still available 
[at this link](http://cloudmaven.github.io/documentation/aws_hipaa.html) including
may circa-2017 screencaps. On this page: Screencaps are removed the notes 
have been partially updated. The potential
value I would summarize as follows: 


* Reading these notes will provide some exposure to the jargon, ideas and considerations
* A fair amount is relevant in a topical way; but it is not a "builder's guide"
* Regardless of what value this may have, please refer to current AWS documentation and tutorials



The term introduced above, **SDRE** for Secure Digital Research Environment, may in the ensuing notes
be referred to as a Secure Computing 
Environment or **SCE**. 


## Links


- [AWS features that are HIPAA-aligned](https://aws.amazon.com/compliance/hipaa-eligible-services-reference/)
- [AWS talk (YouTube) on HIPAA compliance](https://www.youtube.com/watch?v=g4XI4IIYVrw)
- [AWS HIPAA compliance blog Part 1 Automation](https://aws.amazon.com/blogs/security/how-to-automate-hipaa-compliance-part-1-use-the-cloud-to-protect-the-cloud/)
- [AWS HIPAA compliance blog Part 2 Deployment](https://aws.amazon.com/blogs/security/how-to-use-aws-service-catalog-for-code-deployments-part-2-of-the-automating-hipaa-compliance-series/)
- [AWS HIPAA compliance blog Part 3 CloudFormation](https://aws.amazon.com/blogs/security/how-to-translate-hipaa-controls-to-aws-cloudformation-templates-part-3-of-the-automating-hipaa-compliance-series/)
- [AWS HIPAA compliance blog Part 4 Config](https://aws.amazon.com/blogs/security/how-to-use-aws-config-to-help-with-required-hipaa-audit-controls-part-4-of-the-automating-hipaa-compliance-series/)
- [AWS HIPAA compliance architecture](https://medium.com/aws-activate-startup-blog/architecting-your-healthcare-application-for-hipaa-compliance-part-2-ea841a6f62a7)
- [AWS HIPAA compliance white paper](http://d0.awsstatic.com/whitepapers/compliance/AWS_HIPAA_Compliance_Whitepaper.pdf)
- The [NIST 800-66 HIPAA Compliance](http://nvlpubs.nist.gov/nistpubs/Legacy/SP/nistspecialpublication800-66r1.pdf) documents
  - Note that this cross-links to NIST 800-53 in Appendix D: A supporting document
- [HIPAA home page](https://www.hhs.gov/hipaa/index.html/)
- [HIPAA on Wikipedia](https://en.wikipedia.org/wiki/Health_Insurance_Portability_and_Accountability_Act)
- [HIPAA Privacy Rule Summary](https://www.hhs.gov/hipaa/for-professionals/privacy/laws-regulations/index.html)
- [AWS Accelerator: NIST broad spectrum](https://aws.amazon.com/quickstart/architecture/accelerator-nist/)
    - [Related Excel 'security matrix'](https://s3.amazonaws.com/quickstart-reference/enterprise-accelerator/nist/latest/assets/NIST-800-53-Security-Controls-Mapping.xlsx)
- [Observational Medical Outcomes Partnership (OMOP)](http://omop.org/)




## SDRE / SCE Introduction


The material presented here is for learning purposes. It is centered around the concept
of compliance with HIPAA for research using medical (clinical) data. These notes while
technical in nature do not comprise *training material*. They describe how one might 
go about building a SDRE on the AWS cloud.


- Elements of SDRE building and use
    - Building corresponding cyberinfrastructure (see Azure task above)
    - Legal process considerations
    - Project phases
        - Initialize
        - Training
        - Operation 
        - Incident Response
        - Exit

## References and important notes

* [University of Washington IT on ip address assignment](https://wiki.cac.washington.edu/pages/viewpage.action?pageId=53181947)
* R-Studio creates **`.rhistory`** files
    * These files capture user activity; which might include entering access key credentials 
    * If such a file is committed to a public repository (GitHub): The keys can be found
    * If the commit is subsequently deleted: It can still persiste because GitHub is all about version control
    * **ACTION: In all cases DELETE THE ACCESS KEY so it can not be used**  (We can always generate new ones)
* TB Scale: Object storage for fast access costs approximately $100 / TB / month
* [NIST 800-66 Rev 1 HIPAA Security Rule](http://csrc.nist.gov/publications/nistpubs/800-66-Rev1/SP-800-66-Revision1.pdf)
* [Sherlock](https://sherlock.sdsc.edu/) SDRE at the UCSD San Diego Supercomputing Center
* Cloud resources can be tagged with a multitude of key-value pairs
    * These absolutely must be used extensively to make clear the purpose and context of the resource
    * Example: A VM has attached block storage, equivalent to an attached disk drive
        * The VM is tagged with key-value pair 'Name': 'Surgery Study'
        * This is insufficient. There are many other questions that additional tagging can answer at a glance:
            * When was the VM started?
            * For what role in the research is the VM intended? 
            * Who is the cognizant person associated?
            * When should the VM be re-evaluated?
            * What is the VMs intended on/off cycle?
            * Who has access to the VM on the research team?
            * Is the VM internet facing?
            * Does the VM have attached storage drives?
        * In addition the attached block storage (and other ancillary resources) should be tagged
* In what follows we often use hexadecimal strings as IDs. Here we typically use **`aaaaaaaa`** to indicate these.
    * Similarly 12.23.34.45 for ip addresses, 12.23.34.45/16 for CIDR blocks



### Notional SDRE build


This is a framing description of the constituent resources needed.


* Establish two Virtual Machines (VMs) running on AWS
    * First is a publicly visible guardian or gatekeeper VM called a **Bastion Server**
    * Second is a private VM for computation; called the **Worker**
    * To support them we need the following terminology
        * Virtual Private Cloud (VPC) is a hermetic networking construct inside of the AWS cloud
            * The VPC acts as a sub-cloud within the cloud with carefully controlled access
        * Sub-net or sub-network is a routing network within a VPC
            * Each VPC automatically has a private sub-net
            * A VPC may also implement a public sub-net
* Starting point: Open an AWS account and inform AWS the account will be used for sensitive data
    * Follow all recommendations on log-in: MFA, use only an *admin* sub-account (not root), etcetera
* Create a dedicated Virtual Private Cloud (VPC)
    * ID **vpc-aaaaaaaa** in a proximal region e.g. US-West-2
    * Note the Availability Zone as well; let's suppose it is 'C' from A, B or C
    * Note that an IPv4 CIDR block of `10.0.0.0/16` will have more than adequate: 2**(32-16) addresses available
    * Note the network ACL acl-aaaaaaaa
    * Create a route table rtb-aaaaaaaa named something like **sce Main** 
    * Tenancy: Default
    * DNS resolution: Enabled
    * DNS hostnames: Enabled
    * Owner: account number
    * No flow logs
* Establish subnets and other related infrastructure
    * Public subnet **subnet-aaaaaaaa** IPv4 10.0.0.0/24 (256) Route table rtb-aaaaaaaa **sce Second**
    * Private subnet **subnet-aaaaaaab** IPv4 10.0.1.0/24 (256) Route table rtb-aaaaaaab **sce Main**
    * Route tables
        * **sce Main** rtb-aaaaaaaa 
            * Destinations, targets, S3 connectivity
            * Target NAT Gateway **nat-aaaaaaaaaaaaaaaaa** 
        * **sce Second** rtb-aaaaaaab
            * Destinations, targets, S3 connectivity...
    * S3 object storage
        * encryption
    * Internet Gateway
        * **igw-aaaaaaaa** 
    * NAT Gateway for secure transactions to internet from worker ('network address translation')
        * **nat-aaaaaaaaaaa** 
        * Network Interface eni-aaaaaaaa
        * Subnet **subnet-aaaaaaaa** (sce Public)
        * CloudWatch enabled; see Monitoring tab
    * Elastic IPs
        * **eipalloc-aaaaaaaa** 
    * Network Interface
        * **eni-aaaaaaaa** Subnet ID **subnet-aaaaaaaa** Zone us-west-2c
        * Description: **Interface for NAT Gateway Interface for NAT Gateway nat-02b636be6988c3f83**
        * IPv4 Public IP: 12.23.34.45
        * Primary private IPv4 IP 10.0.0.104
  * Flow log, CloudWatch, CloudTrail, Roles, Security Groups, AWS config
  

> Note: The private subnet with CIDR block 10.0.1.0/24 is home to the EC2 Worker; firewalled behind a NAT Gateway
that blocks traffic: You can't **`ssh`** to the Worker from the internet. 
The public subnet is for external access to the EC2 Bastion server.
This subnet has a CIDR block equal to 10.0.0.0/24 so it supports two-raised-to-the-(32-24) = 256 network addresses.
The public subnet connects to the internet via an Internet Gateway. It also hosts the 
NAT Gateway; so the NAT Gateway is not sequestered on the private subnet. Both the Bastion and the NAT Gateway are 
on the public subnet but also have private subnet IP addresses. This is true of *any* resource on the public subnet: 
It always has a private ip address in the VPC as well. Public names resolve to private addresses within the VPC as needed. 


> From this framework: Build **bastion** and **worker**


### Notes on the EC2 VMs: Worker and Bastion


Rather than build these Virtual Machines and 'be done with it': They should be built, 
stored as an AWS Machine Image (AMI) and then terminated (deleted). This sets up the
idea of reproducibility: The actual VMs will be built from the stored AMIs. If they 
undergo significant future changes: Create new reference AMIs. Keeping older AMIs 
for a period of time would be insurance should there be a need to look back at the 
project timeline. 

* Ubuntu Linux or a version of Amazon Linux are fine
* Instance types should match needed compute power
* Wizard process for EC2 instance Create:
    * Place on the already-built secure VPC
        * Subnet = **sce Private** subnet-aaaaaaaa
    * Auto-assign Public IP: **Use subnet setting (Disable)**
    * Capacity Reservation: **Open**
    * IAM Role **EMR_EC2_DefaultRole**
    * Shutdown Behavior **Stop**
    * Monitoring **Checked** Enable CloudWatch detailed monitoring
    * Tenancy: **Shared**
    * Elastic Inference: **Not Checked**
    * Storage
        * 32GB Root /dev/sda1 provides some headroom above the restrictive default of 8GB for the OS
        * Encryption: Encrypting the drive is the good choice; but may incur a performance hit
        * Attach drive: Attaching an encrypted EBS volume creates working space
            * There is additional training needed here to understand encryption key options
    * Security group
        * Defaults are: Type SSH, Protocol TCP, Port Range 22, Source 0.0.0.0/0 (open to world)
        * This requires additional training and effort...
            * ...to restrict inbound ip address space to the bastion
            * ...to anticipate internal private subnet access from bastion to worker
    * Launch: obtain separate key pairs for bastion and worker
        * Bastion keypair will be used by a local machine on the internet, 'my laptop'
        * Worker keypair will be used only from the bastion machine 











There are three primary considerations: Data security
(e.g. encryption considerations), reporting (e.g. logging access) and human use factors (e.g. access key
best practices).



#### Cloud Services

* AWS EC2 instance: A working Virtual Machine
* AWS S3 object storage: Unlimited data storage capacity
* AWS RDS relational database as a service


#### Issues 


- Logging/reporting: How the Researcher can report to a CRO 'these data were used' at the table row level
- Access constrained to certain IP addresses only (CIDR block)
- Authentication via confederated service like CILogon or NetID
- If a data breech: What is the response plan, how does the vendor participate?
- Synthetic data: Simple Python example, OMOP as a deeper resource
    - OMOP = Observational Medical Outcomes Partnership
    - Specifically there is an unrestricted-use synthetic SynPUF dataset. 


In a simple view HIPAA regulations require that data be encrypted both at rest (on a storage device) 
and in transit (moving from one location to another).  In addition all transactions around PHI data
should be logged in a traceable manner to validate the assertion that only authorized individuals
have had access to PHI. 


HIPAA regulations may be seen through the lens of NIST; we find for example the 
[NIST HIPAA Security Rule](https://www.nist.gov/healthcare/security/hipaa-security-rule) 
which is detailed in NIST Special Publication 800-66 Revision 1. This document discusses 
HIPAA=compliant implementations in practice. 


Putting a conforming technical system in place is fairly straightforward though 
detailed (Part 1).  Satisying conditions for HIPAA compliance in a 
real implementation also involves sociological elements.  Real implementations 
operate under a *Shared Responsibility* model: AWS provides 
necessary security measures while practitioners build and operate the environment 
according to established practices; including documentation and logging access. 
HIPAA compliance is *more* than using HIPAA-aligned technologies from AWS.


### Admonitions


- ***Notify cloud provider of intent to use PHI***
- ***The acronym SCE is used extensively on this page to denote a Secure Computing Environment.***
- ***AWS has many HIPAA-aligned technologies. Only these technologies may come into contact 
with PHI.  Other technologies can be used in an SCE provided there is no such contact.***
- ***HIPAA compliance is an obligation of the data system builder, the medical researcher
and the parent organization(s).***
- ***The public cloud is very secure: physically and technologically. 
Compromise is most likely to be caused by human error.***
- ***File names may not include PHI.***
- ***AWS VM types have associated connectivity rules; be aware before including a particular type in an SCE***


### Extended User Story


- A scientist receives approval from the IRB to work with PHI data. 
    - Intent: Analyze these data in a Secure Compute Environment (SCE) 
- scientist initiates a cloud solution project that includes a billing mechanism
- scientist contacts IT professional to build an SCE as a working environment
- Data are uploaded to storage on AWS from a secure device
- scientist is provided with access to the SCE 
    - A pem file and ip address of a bastion server
    - A login and password to a private subnet EC2 instance
    - Notice there is no IAM User account involved: Console access not needed
- IT pro logs in; does configuration
- scientist carries out data analysis over time; system activity is automatically logged
- issue to consider: Patient-held devices could contribute data streams e.g. via phone app using the API Gateway 
- study concludes, data and results are preserved, log files preserved, the SCE is deleted, no trace remains
- issue to consider: AMIs are preserved for future SCE work


### Questions / Discussion Topics

- A VPC costs nothing per day. EC2, EBS, networking egress (9 cents per Gig EC2 to internet), S3 egress is 9 cents per Gig up to 10 TB (then it drops a bit for more) 
    - Other charges: NAT Gateway is 4.5 cents per hour plus 4.5 cents per Gigabyte
    - S3 also costs... now 2.3 cents per GB-month
    - RDS costs based on chosen instance size
    - Stopped EC2: Still pay for its EBS. Snapshot rate is $0.05 per GB month (but unused volume is compressed to zero)
- A code base on GitHub is an open topic
- Changing instance type
- Create a VPC
    - 'Namespace': Does the VPC need a CIDR that doesn't overlap other VPCs in my account? 10.0.0.0 is very popular...
        - VPCs can have overlapping CIDR blocks. However not if they are going to be peered. 
        - VPN also can create conflicts. So yes: Overlap is fine when they are independent bubbles. 
        - Peered VPCs are to be avoided and must not have collisions; otherwise fine
    - Stipulate 'Default Tenancy': 
        - Shared versus dedicated: Shared is now allowed; so do not use Dedicated Tenancy for HIPAA: This is now in the BAA as unnecessary for HIPAA compliance.
    - Creating subnets: AZ: VPC is regional
        - Subnets are associated with AZs and should be intentionally designated
        - Using multiple AZs (multiple subnets across AZs) will make the VPC "present" in those AZs
        - There is a charge for inter-AZ bandwidth: 1 cent per Gig which is one ninth the egress to internet rate
        - for SCE it is more controlled to be in just one AZ
        - Going multi-AZ would be a high availability strategy which is a compute-heavy idea 
- Coffee shop risk: What happens with that laptop?
- Verify correct: There is one private address space per VPC: All VPC resources map to this
  - ...and Public elements also have world-facing ip addresses
  - ...and all private addresses are stable; no bounce problem
  - 10.0.0.x as the address space of **Spublic** is its private VPC address space
      - That is: 10.0.0.x is actually a private ip address for a public-facing instance
      - We get both public ip address x.y.z.w and a DNS entry ec2-etcetera
        - Inside the VPC a public ip always resolves to a private '10.0.0.9' to keep traffic inside the VPC
        - ec2-etcetera DNS entries change when the machine bounces; as do ip addresses
  - How does one verify that auto-generated resources are not public?
    - Equivalent: How are auto-generated resources assigned to subnets? 
    - How are private subnet addresses determined? "Pick one"?
    - This is tremendously important so that a new **Wi** is not publicly visible
    - Look at the VPC Routing Table "default subnet" column (verify this is correct)
- Key management story
  - **K_Bastion** must reside on **L**. How is this considered secure? Chain of custody from origination?
  - **K_EC2private** same; and must be carried via scp to **B**
  - **B** is a single point of failure compromising **EC2private** 
    - Password-protection on **B**
  - Group access: one login is not an option. Now we have Users.
  - For that matter should ec2-user be logging on to **EC2private** to do work?
  - How do keys work in view of an AMI source?
- Is the public **B** ip (Elastic or not) address vulnerable?  Risk++ How to use SG and/or CIDR? 
- EBS Encryption Keys: Risk++
  - EBS encryption select has an account default key: Details are listed below
- Risk: Check DLT T&C; "who is responsible for the risk?" 
- Risk: Boot volume is not encrypted
- Risk: EC2 swap space and PHI
- Scale problem: Manage **Wi** keys and roles
- If *K* and her team are not IAM Users how do they cut cost by stopping machines?
  - CLI from **L** works but requires authentication
- For the learner we must differentiate SQS and SNS
  - [S3 event notification link](https://aws.amazon.com/blogs/aws/s3-event-notification/)
  - [AWS notification how-to](http://docs.aws.amazon.com/AmazonS3/latest/dev/NotificationHowTo.html)
  - Describe the PubSub concept
  - Try this: SQS is a queue where *things* sit; polling it triggers processes
    - CANVAS LMS is apparently a good example since students are interacting with it frequently
- What is Ansible and what does it get us?
  - Configuration management
  - cf Chef and Puppet
- In what sense does the NAT Gateway prevent or not prevent us from poisoning ourselves? 
- The RT / subnet section needs review
- What are the tradeoffs in building **B**, **M** and **Wi** from AMIs?
  - Config management tool: Option 1 (Ansible Chef Puppet)
  - How does 'sudo yum update -y' happen?
  - Manually take a stale AMI and update it and re-save it (and update the hot machine)
    - "You still have to manage the OS" is part of Shared Responsibility
- Cost: Shut down instances over the weekend when not IAM Users...
- Logging: CloudWatch and CloudTrail are AWS logging services; frequently parsed using Splunk
  - Intrusion detection! 
    - Jon Skelton (Berkeley AWS Working Group) reviewed use of Siricata (mentions 'Snort' also) 
- Filenames may not include PHI. 
- 'IT, Admin and Research approvals.'
- Figure out 'Default subnet' for new resources: Not accidentally public when auto-generated
- Explain how small CIDR block ranges do not conflict with the internet
- Bastion server inbound ip range should be modified to match UW / UW Med / etcetera
  - Unless you want people able to wfh
  - Also differentiate the UW VPN CIDR block 
- Bastion and **Sprivate** **W**: More details on the configuration steps!
  - Enable cloudwatch checkbox? Yes
- Missing instructions on setting up S3 buckets: For FlowLog and for DataIn
- Be clear that subnet CIDR blocks are *always* for the private subnet component. 
  = Subnet *public* means a second set of (public: on the internet) ip addresses 
  - These map to the private subnet addresses. 
- Glossary
  - Ansible
  - Regions and Availability Zones on AWS
  - Bastion Server 
  - Siricata / Snort
  - Direct tools like **ssh** and third party apps like Cloudberry 
  - Dedicated Instance 
  - Lambda Service: A micro-service which is code that runs under certain trigger conditions; see the dedicated cloudmaven page.
  - NAT Gateway: A Diode that permits a private sub-net instance to access the internet without the internet having access back.
    - Notice that an EC2 can use a NAT Gateway to transparently install software; but if this is malware then you have done it to yourself.
  - [HIPAA-aligned components of AWS](https://aws.amazon.com/compliance/hipaa-eligible-services-reference/)
- "How can you be using Lambda? It is not on the list...?"
  - Tools that do not come in contact with PHI can be thought of as 'triggers and orchestration'.
  - Services that may come in contact with PHI can be described as 'data and compute'
- This block of text was formerly the close-out plan for the POC; please review
  - Set up Ansible-assisted process for configuring and running jobs on EC2 instances
  - Pushing data to S3
    - Console does not seem like a good mechanism
    - Third party apps such as Cloudberry are possible...
    - AWS CLI with scripts: Probably the most direct method
- How does the SCE fire up arbitrary EC2 instances as needed?
  - Launch **W** x 5 Dedicated instances, call these **Wi**
  - Assign S3 access role 
  - Encrypted EBS volumes 
  - S/w pre-installed (e.g. genomics pipelines)
  - Update issue: Pipeline changes, etcetera; 
- **Wi** can be pre-populated with reference data: Sheena Todhunter operational scenario
    - Assumes that a SCE exists in perpetuity to perform some perfunctory pipeline processing
    - On **B**
      - Create SQS queue of objects in S3
      - Start a **Wi** for each message in queue...
    - Go
      - Latest pipeline... EBS Genomes... chew
      - If last instance running: Consolidate / clean-up
      - SNS topic notifies me when last instance shuts down.
      - Run Ansible script to configure **Wi** (patch, get data file names from DynamoDB table, etcetera)
      - Get Ws the Key from E
      - The Ws send an Alert through the NAT gateway to Simple Notification Service (SNS) 
      - Which uses something called SES to send an email to the effect that the system is working with PHI data 
        - Ws pull data from S3 using VPC Endpoint; thanks to the Route table
        - Ws decrypt data using HomerKey
        - Ws process their data into result files: Encrypted EBS volume. 
        - Optionally the result files are encrypted in place in the EBS volume.
    - Through **S3EP_O** the results are moved to S3.
    - **Wi** sends an Alert through the NAT gateway to SNS 
      - which uses something called SES to send another email: Done
    - **Wi** evaporates completely leaving no trace




## Part 1. Building an SCE


We use **boldface** abbreviations for system components.
We also present *tagging* and *naming* resources using a short identifier
string unique to each project: The Project Identifier Tag (abbreviated PIT). 


Our main objective is to use a Laptop **L** or other cloud-external data source
to feed data into an **SCE** wherein we operate on that data.  The data are assumed 
to be Private Health Information (PHI) / Personally Identifiable Information (PII).


- Write down or obtain a Project Identifier Tag (PIT) to use in naming/tagging everything
  - This is a handy string of characters
  - In our example here PIT = 'hipaa'. Short, easy to read = better


- Identify our source computer as **L**, a Laptop sitting in a coffee shop 
  - This is intentionally a *non-secure* source
  - **L** does *not* have a PIT
  - **L** could also be a secure resource operated by Med Research / IRB / data warehouse


We are starting with a data source and will return to encryption later. The first burst 
of activity will be the creation of a Virtual Private Cloud (VPC) on AWS per the diagram
above. We assume you have done Step 0 above and are acting in the capacity of a
system builder; but you may not be an experienced IT professional. That is: We assume
that you are building this environment and that you may or may not be doing research
once it is built; but someone will.


#### VPC via Wizard versus manual build


The easiest way to create a VPC is using the console Wizard. That method is covered in a 
section below and it can automate many of the steps we describe manually.  We describe 
the manual method to illuminate the components.


### 1A: Start


#### Initial Required Action


As SCE builder your absolute First Priority Step 0 Must Do is to designate to AWS and DLT 
that the account you are using involves PII/PHI/HIPAA data. Contact DLT (the AWS account
provider) with your account number using email address *cloud@dlt.com*. Your message 
should be to the effect that this account will work with PHI. 


#### Cost tracking and cost reduction


- This will be a multi-day effort. Shut down instances to save money at the end of the day.


#### CIDR block specification


The CIDR block syntax uses a specification like **10.0.0.0/16**. This has two 
components: A 'low end of range' ip address **w.x.y.z** and a width parameter **/N**. 
w, x, y and z are integers from 0 to 255, in total providing 32 bits of address space.  


**N** determines an addressable space of size s = 2^(32 - N). For example 
N = 24 produces s = 2^8 or s = 256 available addresses, starting at w.x.y.z.
Hence z (and possibly y) would increase to span the available address space.


Another example: Suppose we specify 10.0.0.0/16.  Then s = 2^16 so 65536 addresses 
are available: 10.0.0.0, 10.0.0.1, 10.0.0.2, ..., 10.0.1.0, 10.0.1.0, ..., 
10.0.255.255. **y** and **z** together span the address space.


These ip addresses are defined in the VPC, contextually *local* within the VPC.


Any subnets we place within the VPC will be limited by this address space.  
In fact we proceed  by defining subnets within the VPC with respective 
CIDR ranges, subranges of the VPC CIDR block.  In our case the first subnet will 
have CIDR = 10.0.0.0/24 with 256 addresses available: 10.0.0.0, 10.0.0.1, ..., 
10.0.0.255. Five of these are *appropriated* by AWS machinery.
The second subnet will be non-overlapping with CIDR range = 10.0.1.0/24.


Since AWS appropriates five ip addresses for internal use
(.0, .1, .2, .3, and .255) we should look for ways of making 
ip address assignment automatic.  


#### Build the VPC 


Here we abbreviate elements with boldface type. In most cases the entity we create
can be named so to remind you: For consistency we have come up with a Project
Identifier Tag like 'hipaa' so that each entity can be given a PIT name: 'hipaa_vpc'
and so on. In naming associated S3 buckets: The name may be harder to produce because
it must be an allowed DNS name that does not conflict with any existing S3 buckets
across the entire AWS cloud.


- From the console create a new VPC **V**

  - Give **V** a PIT name 


  - **V** will not use IPv6v.  


  - **V** will have a CIDR block defining an ip address space
    - We use 10.0.0.0/16: 65536 (minus a few) available addresses


  - **V** automatically has a routing table **RT**
    - Select Routing Tables, sort by VPC and give **RT** a PIT name
      - 'hipaa_routingtable'
      - The routing table is a logical mapping of ip addresses to destinations


  - **V** is automatically given a security group **SG**
    - Select Security Groups, sort by VPC and give **SG** a PIT name
      - 'hipaa_securitygroup'


  - Create an associated Flow Log **FL**
    - In March 2017 the AWS console UI was a little tetchy so be prepared to go around twice
    - On the console view of the VPC: Click the Create Flow Log button
      - Assuming permissions are not set: Click on the Hypertext to **Set up permissions**
        - Because: We need to define the proper Role
        - On the Role creation page: Give the Role a PIT name; Create new; Allow
        - You now have an IAM Role for FlowLogs
          - This gives the account the necessary AWS permissions to work with Flow Logs
          - In so doing we fell out of the Create Flow Log dialog so... back around
      - Return to the VPC in the console
      - Click on Create Flow Log
        - Filter = All is required (not "accepted" and not "rejected" traffic)
        - Role: Select the role we just created above
        - Destination log group: Give it a PIT name 
          - Example: hipaa_loggroup


  - Create subnets **Spublic** and **Sprivate**


    - The private subnet **Sprivate** is where work on PHI proceeds
      - CIDR block 10.0.1.0/24
      - **Sprivate** will be firewalled behind a NAT gateway
        - This prevents traffic in (such as ssh)
    - The public subnet **Spublic** is for internet access
      - CIDR block 10.0.0.0/24
      - **Spublic** connects with the internet via an Internet Gateway
      - **Spublic** will be home to a Bastion server **B** 
      - **Spublic** will be home to the NAT Gateway **NG** mentioned above
        - **B** and **NG** are on the public subnet but also have private subnet ip addresses 
          - That is: Everthing on the public subnet also has a private ip address in the VPC. 
          - This will use the private ip address space 
          - Public names will resolve to private addresses within the VPC at need.


  - Create an an Internet Gateway **IG**



    - Give a PIT name as in 'hipaa_internetgateway'
    - Attach hipaa_internetgateway to **V**


  - Create a NAT Gateway **NG**


    - Give it a PIT name
    - Elastic IP assignment may come into play here


  - Create a route table **RTpublic** 



    - Give it a PIT name: 'hipaa_publicroutes'
    - This will supersede the **V** routing table **RT**
    - Select the Subnet Associations tab 
      - Edit subnet association to be **Spublic**
    - Select the Routes tab 
      - Edit (under Routes) and add 0.0.0.0/0 pointing to **IG**


  - public route table **RTpublic** 


Note: The console column for subnets shows "Auto-assign Public IP" and this should be set to
*Yes* for **Spublic**. Note the column title includes the term *Public IP*. The Private subnet 
should have this set to *No*. If necessary change these entries using the *Subnet Actions* 
button. 


Note: In the subnet table find a "Default Subnet" column. In this work-through both **Spublic** 
and **Sprivate**  have this set to *No*:  There is no default subnet.  This will be modified
later via the route table **RT** in **V**.


**RT** reads:  
```
10.0.0.0/16         VPC "local" 
0.0.0.0/0           NAT gateway 
```


**RTpublic** (hipaa_publicroutes) has
```
10.0.0.0/16 
0.0.0.0/0      Internet Gateway
```

We now have two RTs. hipaa_routingtable (default for the VPC in general) and for 
Spublic (hipaa_publicroutes). Notice that **RT** operates by default and **RTpublic**
supersedes this on **Spublic**. This means that new resources on **Sprivate** will
by default use **NG** which is what we want. 


The **V** **RT** 0-entry points at **NG**: All internet-traffic will route through **NG**. 
**NG** does not accept inbound traffic. This is by default. (Analogy: Home router)


**Spublic** has the custom **RTpublic** which routes non-local traffic to the IG, i.e. 
the internet. This *does* accept inbound traffic allowing us to ssh in. This is an exception
to the default.


### 1C S3 buckets and EC2 

#### S3 Encryption policy

We create new S3 buckets associated with projects and assign them a Policy to ensure that
Server-side encryption is requested by anyone attempting to upload data. This ensures the 
data will be encrypted when it comes to rest in the bucket. 


[AWS link for S3 server-side encryption policy for copy-paste](http://docs.aws.amazon.com/AmazonS3/latest/dev/UsingServerSideEncryption.html)



#### S3 Endpoints


An S3 Endpoint is routing information associated with the VPC.  S3 access from the VPC should not go through 
the public internet; and this routing information ensures that. The S3 Endpoint is not subsequently invoked; 
it is simply infrastructure. For example an EC2 instance might access an S3 bucket via the AWS CLI as in

```
% aws s3 cp s3://any-bucket-name/content local_filename 
```

  

### 1D. Building a Bastion Server **B**


- On **V** create a public-facing Bastion Server **B**
  - **B** has only port 22 open (to support ssh) 
  - **B** uses Secure Groups on AWS to limit access to only a subset of URLs
  - **B** can be a modest general-purpose machine such as a m4.large
    - In this example we choose an m4.large running AWS Linux 
    - DO NOT USE T instances 
      - They will not connect to our Dedicated Tenancy VPC: This is not supported.
  - Go through all config steps: Memory, tags; 
    - Security group is important
      - Create a new security group 
      - PIT name: hipaa_bastion_ssh_securitygroup
      - Description = allow ssh from anywhere
      - Notice that in the config table "Source" is 0.0.0.0/0 which is "anywhere"
        - Best practice is to restrict the inbound range
        - Consider differentiating UW from the UW-VPN 
          - This would allow someone to log in from anywhere VIA the UW VPN


    - Key pair use a PIT as 'hipaa_bastion_keypair' 
      - Generate new, download to someplace safe on **L** 
    - Launch instance


#### Building a work environment EC2 instance on **Sprivate**

- On **Sprivate** install a small Dedicated EC2 instance **E**

#### Encrypting the key to the private subnet instance

New Linux instances should have Gnupg installed, otherwise use:

```bash
sudo apt install gpg 
```

To encrypt

```bash
$ gpg -c filename
```


Output: 


> Enter passphrase:
> Enter passphrase again:


To decrypt 
```bash
$ gpg filename.gpg
```

Now you have a mechanism for encrypting access to your private subnet instance on the bastion. There 
are two considerations that follow: 

- The private subnet instance key should be set aside in a secure location by the admin building 
the system so that corruption of the key does not make the instance completely inaccessible.
- The private subnet instance key on the bastion should be regularly encrypted so that a breach
of the bastion does not grant the black hat access to the private instance.


### 1E. Server side encryption S3 bucket


- The file must be encrypted on **L**
- We upload this file to S3 and stipulate "encrypt this when it comes to rest in S3"
- S3 manages this
- We create an associated policy that *only* allows this type of upload
  - Therefore a not-encrypted-at-rest request will be denied


As a guide see the [S3 AWS encryption link](http://docs.aws.amazon.com/AmazonS3/latest/dev/UsingEncryption.html).


- Click Server-Side Encryption
- Click on Amazon S3-Managed Encryption keys in the left sidebar to arrive at
[this documentation](http://docs.aws.amazon.com/AmazonS3/latest/dev/UsingServerSideEncryption.html).


Copy the code box contents
Go to console
S3 bucket
Permissions tab
Bucket Policy button
Paste
Replace in two places: actual bucket name

PutObject command: must have server-side-encryption set to AES256 
(that is the name of the encryption algorithm) AND must have server-side-encryption set to true.


"Encrypted in transit" is done via the https endpoint to S3.




We are now configuring the S3 Endpoint. 
  In so doing we selected the VPC RT (not the public subnet RT; could use that)
  We are concerned about S3 traffic
  S3 Endpoint is a new type of Gateway that gives S3 access from Private subnet direct into S3 without any traverse of the internet.
  

WARNING: Kilroy: After I added the S3 Endpoint the NAT gateway entry had vanished from my VPC routing table. 
This is bad. Right now the procedure is going to be: After S3 EP go examine RT and re-add NAT gateway if it 
is not properly present. holy cow!!!!!!!!!!!!!!!


Created S3 bucket

### 1E. Ancillary components

This section describes the use of automated services, data bases tables, IOT endpoints and other 
AWS features to augment the SCE. Such ancillary components may or may not touch directly on PHI,
an important differentiator as only HIPAA-aligned technologies are permitted to do so. 


- Set up a DynamoDB table to track names of uploaded files
- Set up a Lambda service 
  - Triggered by new object in bucket in the S3 input bucket
  - This Lambda service is managed using a role
- Set up an SQS Simple Queue Service **Q** 
- Create an SNS to notify me when interesting things happen


### 1F. Encryption


Our EC2 instance will maintain encrypted PHI on an Elastic Block Storage (EBS) 
volume by requiring that this EBS volume be encrypted.  That is, this volume must be 
created with the 'encryption' check-box checked. Once this is done we mount a file 
partition on the EBS. This procedure is described on the [EC2 page](aws_ec2.html).


In creating the EBS: Volume type can be *General Pupose*.
For EBS costing and performance refer to 
[this link](http://docs.aws.amazon.com/AWSEC2/latest/UserGuide/EBSVolumeTypes.html).
Notice the EBS volume has a Region and an Availability Zone which **must match** the target 
EC2 where it will be attached.


The Master Key issue is ignored here; we assert that the default Key is simplest and secure. 
Using the default Key encrypts this data at rest. So that's done. 
(Working with your own set of keys would be part of risk management: Should the keys be compromised?
etcetera.  There is some associated hassle.)
We intentionally do not encrypt the root volume. This is another risk mitigation choice.


ok done: hipaa_ebs_ec2_private 


Next log in to EC2 on the private subnet:
- log in to the bastion server
- used copy-paste to edit a file on the bastion called ec2_private_keypair.pem
  - I did a bad paste and was missing the -----BEGI
  - on this file we ran chmod 600 on the pem file
  - ssh -i <this file> 10.0.1.248
    - notice this ip address is in the console Description tab when highlighting the private subnet EC2 instance
  - Notice that the pem file on the bastion would immediately compromise the EC2 on the private subnet
    - That sure sucks for you


Using (sudo) fdisk to create the partition on the raw block device (fast: writing the partition table) will blow 
away anything already there. 


```
% aws s3 cp s3://bucket-name/keyname .
```

Kilroy that should probably be compared to using 'scp'.


#### S3 key names versus file names


In the above copy command I knew in advance the *keyname* that I wanted to copy to the EBS volume. 
This is because it was identical to the filename that I uploaded to this S3 bucket from **L**. 
That file was pushed from **L** using the AWS browser console but it could also be pushed using CLI
or using a utility program. The important point is that *filename* and *keyname* are the same thing.

 


Transferring to the instance: scp 


Last encryption note: SSL is used by the CLI be default; so our EC2-private command 'aws s3 ... etc': 
Look at cli/latest/reference for the link on this. This means that S3 to EC2private is encrypted in transit. 
The EBS /hipaa is encrypted at rest. Done. 


### 1G. Auditing

SCE activity must be logged in such a manner as to permit subsequent tracing of PHI data 
access and ongoing monitoring of the security state of the system. 

#### CloudTrail and CloudWatch

CloudTrail is logging the API calls to my account. Whether through the console, the CLI, the APIs directly: All logged in 
cloud trail once it is enabled. They all use the same APIs; so we log on the API calls. Logs to S3. 

**Here is the key thing: Enable cloud trail which creates a destination S3 bucket where all of the logging will go.**

Best practice is to turn on Cloud Trail in all of your regions; so you are not blind. "Either you are logging or it is gone."

Cloud Watch is more for monitoring: Performance metrics. 

In both cases there is never PHI data in the log: PRovided ss#s are not in the filenames (for example) 


And S3 has its own internal logging mechanism as well. 
- Create a new bucket hipaa-s3-access-logs with vanilla settings
- Locate the existing inbound data bucket > Properties > Logging > enable > stipulate the access logs bucket; add a tag...

This will have http traffic details; where stuff was coming from for example

#### AWS Config

Cloud trail tells you what's happening in terms of the API
AWS Config tells you what changed

Use them together to see if something is happening / happened of concern

High level set up and then more detailed is possible

Console > MAnagement > Config

Think of Config (like Cloud Trail) as account-wide, not "per resource" so tagging with the PIT is not correct.

My process was pretty default.


So now we have Cloud Trail, S3 logging and Config operating on this account. 
So as we get more sophisticated we could dig in to Cloud Watch. 


### 1H. Disaster Recovery


Indicate awareness; up to CISO to provide hurdles


### 1I. Operating within the environment


I begin by starting a windows laptop connected to the internet and run PuTTYGen. I use this to convert
my .pem access file to the bastion server to a .ppk file. This is just a necessary formality. I then
start PuTTY and use the Connection / SSH / Auth panel to Browse to my .ppk key file. 


I look on the AWS Console at my EC2 instance, the bastion server for this VPC, and note the current
ip address. If this is an elastic ip then it will not change over time; which is useful. I enter 
into the PuTTY Session main window the Host Name ec2-user@'ip-address' (Note that ec2-user is the
verbatim string; ip-address you must substitute, for example 35.27.93.105. Port is 22. 


Before clicking the Open button in PuTTY I first give this session a name and Save it. This can be 
loaded next time to make the process faster. Now click Open and after confirming in a dialog if all
went well I see a Linux command prompt.  I immediately update my instance using 


```
% sudo yum update 
```


I have obtained a .pem keypair file for access to my private subnet EC2 instance.  I use scp (actually
the application **WinSCP**) to copy this pem file (no need to translate to .ppk) to the bastion server. 
It should appear in my ec2-user home directory. Now I log in to the private EC2 instance using


```
ssh -i ec2_private_keypair.pem ec2-user@10.0.1.248 
```


Notice that this uses the private subnet ip address 10.0.1.248. This ip address can be seen in the 
AWS console by highlighting the private EC2 instance and reading the Description tab found below the 
instance table.  Again I have a command prompt and I issue


```
% sudo yum update 
```


This is obtaining updates via the NAT Gateway; so while this machine is able to obtain and use PHI
it is also able to safely get updates and install other software from the web. Being a Python programmer
I also issue


```
% sudo yum install anaconda
```


Now I can turn my attention to obtaining a dataset from an S3 bucket to my local block storage (disk drive). 
This narrative is continued below in Part 2 under 'operating within the POC'.



## Part 2. POC A, a SCE with a small synthetic dataset


### 2A. Documenting the creation of the environment


### 2B. Creating the synthetic dataset 


[This code](https://github.com/robfatland/cloudsecurity/blob/master/syntheticdata/createsynthetic.py) generates 
a simple synthetic dataset.



### 2C. Installing software and operating POC A


### 2D. Concluding remarks on POC A


## Part 3 POC B including [OMOP](http://omop.org) unrestricted dataset ported to AWS Redshift


### 3A: New SCE


Follow the instructions using a new PIT. I will use **omop**.


### 3B: OMOP to Redshift

This POC is a good direction for a number of reasons; including the fact that 
UW Med has an initiative underway to create an OMOP warehouse from clinical data. 

From the website is a Quick Link to an Unrestricted Vocabulary.
From the README here are the steps to load the vocabulary data.



```
- Download vocabulary files to local drive and unzip them or (Redshift)
- Upload gzipped files to S3 storage
- Create target tables
- Transfer data from files to tables
  - Redshift
    - copy vocabulary.vocabulary from 's3://.../vocabulary.txt.gz' credentials 'aws_access_key_id=;aws_secret_access_key=' delimiter '\362' ACCEPTINVCHARS GZIP emptyasnull dateformat 'auto'; 
```

### 3C: Visualization


OHDSI has several open source tools that have data visualization, specifically Atlas. 
Challenge then becomes: Configure a website for us to view the Atlas output operating on the 
synpuf dataset (OMOP; must agree this is apples/apples). This gets us to a dataset in omop, 
a data viz tool, hosted on the cloud with a web GUI. 


(Looking ahead: What other data viz and analytic tools does the cloud environment give us access to and can 
we prototype some use case there? From there a notionally short jump to get live data and a 'real' demo (not
synthetic.)) 


#### Installing ATLAS


See [this page](http://www.ohdsi.org/web/wiki/doku.php?id=documentation:software:atlas:setup).


The challenge is going to be installing the DB and then the web server components on the right parts of the VPC. 


Step 1 is Install the OHDSI WebAPI and configure the appropriate sources. 


... and so on ...


### Risk


This section identifies points of risk and their severity. Severity is described both
'when protocol is properly observed' and 'when protocol is not observed'. That is: We 
provide examples of how failure to follow protocols could result in the compromise of 
PHI. There is in all of this a notion of diminishing returns: A tremendous amount of 
additional effort might be incorporated in building an SCE that provides only small 
reduction of risk.


#### VPC creation: Manual versus Wizard


#### Dedicated instances


We drive cost up using dedicated instances on **Sprivate**. It is technically feasible
to not do this but there is an attendant cost in time and risk.


#### Extended key management strategy


Encryption keys here are taken to be default keys associated with the AWS account. 
It is possible on setting up the SCE to create an entire structure around management 
of newly-generated keys. This is a diminishing-returns risk mitigation procedure:
It may create a profusion of complexity that is itself a risk. 


One open question is whether a single AWS account should / could / will be used to
provision multiple independent research projects, each with one or more respective 
data systems. 


- Log in to **B** and move **K** to **E** 
  - Observe that material encrypted 
  - Maybe instead we should be tunneling directly to **E** from **L**
- Use an AMI to create a processing EC2 **W**


  - Also with **ENC**
- Create S3 buckets 
  - **S3D** for data
  - **S3O** for output
  - **S3L** for logging
  - **S3A** for ancillary purposes (non-PHI is the intention)


  - Such S3 buckets only accept http PUT; not GET or LIST
- Create an S3 bucket **S3O** for output
- Create an S3 bucke **S3L** for logging
- Create an S3 Endpoint **S3EP_D** in **V**
- Create an S3 Endpoint **S3EP_O** in **V**
- Create an S3 Endpoint **S3EP_L** in **V**
  - "S3 buckets have a VPC Endpoint included... ensure this terminates inside the VPC"
- Create role **R** allowing **W** to read data at **S3EP** from **S3**


