# Secure Digital Research Environments on the AWS Cloud


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
