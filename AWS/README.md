# AWS Cloud Security

Here we go through some security procedures. 

## Glossary (move common terms to the main README)

* **IAM**: Identity and Access Management, a generic term for the access mechanisms relative to a particular cloud
  * __User__: A person with access to a cloud account. User-level access is controlled by assigned Roles which act as privileges.  
  * __Role__: A logical construct on the cloud that grants privilege to an entity to perform actions. Entities may be Users or Services.
    * Example: A Lambda service is unable to execute its own code unless it is granted a Lambda execution Role
      * This decouples permission to act from the action itself as a means of making actions externally managed
  * __Policy__: A text (JSON) document commonly associated with a Role that spells out the logic of granted permissions

* **Authentication**: 'logging in' to some service, i.e. the process of providing credentials to gain access to a resource like a Virtual Machine. COmmon authentication strategies include: 
  * __Access Keys__ (Access Key ID / Secret Access Key): On AWS, a pair of character strings, respectively medium and long, automatically generated, used in authentication at the IAM User level.
    * Example: The AWS CLI can be configured with these Keys to authenticate and carry out actions like launching a VM.
  * __PEM file__: An access key file for `ssh` access to a Linux operating system. Bypasses User login password. Safer than passwords usually as no one can see you type the password. 
  * __AWS STS__: AWS Security Token Service enables you to request temporary, limited-privileged credentials for authentication. This is the recommended way to use the AWS CLI / SDK on AWS resources for longer term deployments. 
  * **SSO**: Single Sign-On, refers to confederating authentication where a single identity can be used to authenticate to more than one service.
  * **MFA / TFA**: Multi-factor authentication / Two-factor authentication. Means of using multiple authentication actions to reduce intrusion risk.

* **Security Group**: A list of rules specifying the type of access to your AWS resource. These rules can be updated on the fly without stopping the resource, and apply immediately. 
  * Example: For EC2 instance, we usually open the port 22 for access (Inbound rules), and specify either anyone (least recommended, use with care) can access it or only your IP can access it (safest, best to use if this is your first time) or something in between.  

* **Intrusion**: Unuathorized use of a resource.

### Relocate this link to section on scale

* [ML at scale: Kubernetes on AWS](https://aws.amazon.com/blogs/storage/using-high-performance-persistent-storage-for-machine-learning-workloads-on-kubernetes/


## Resources as of July 2020

* [AWS Support video on MFA CLI](https://aws.amazon.com/premiumsupport/knowledge-center/authenticate-mfa-cli/)


* [AWS CLI Role configuration](https://docs.aws.amazon.com/cli/latest/userguide/cli-configure-role.html#cli-configure-role-mfa)
* [IAM Roles and permissions](https://docs.aws.amazon.com/IAM/latest/UserGuide/id_roles_use_permissions-to-switch.html
* [IAM Roles for Users](https://docs.aws.amazon.com/IAM/latest/UserGuide/id_roles_create_for-user.html
* [IAM Roles: Assuming Roles](https://awscli.amazonaws.com/v2/documentation/api/latest/reference/sts/assume-role.html)


## Very Secure method outline provided by an AWS Solutions Architect

* Create a Role for use by an IAM user. Optional: Require MFA.
  * In Permissions for this Role, select permissions to grant to the IAM user.
* Create an IAM User, configure MFA, download the combined AccessKey/SecretKey as a (human-readable) text file
* Create an IAM Policy which only allows: `sts:AssumeRole` and attach this to the IAM User. 
  * Alternative: Attach it to a Group and place the User in the Group.
* In the AWS CLI configure with the AccessKey/SecretKey.
* Create a profile in the AWS CLI per [these instructions](https://docs.aws.amazon.com/cli/latest/userguide/cli-configure-role.html#cli-configure-role-mfa)
* For the first command run from the CLI for your session...
  * ... i.e. until the temporary credentials expire in 12 hours or you close your shell, whichever comes first
  * ... add `–profile` 
  * After one command is run specifying the profile (and thereby asking for MFA)...
    * ... export the temporary credentials to environment variables so further adding of `–profile` (is? is not? necessary???)
    * ... from with every CLI command.
    * ... `export AWS_PROFILE=profilename`

## Alternative example python script

* Allows you to assume a role and prompt for credentials:
* [Link to guide](https://docs.aws.amazon.com/IAM/latest/UserGuide/id_credentials_mfa_sample-code.html#MFAProtectedAPI-example-assumerole)
* Modify this script to simply add the temporary credentials received to the OS’ environment variables instead of explicitly running an S3 action

## Alternative cheap EC2 method

* Skip all of the above and use an inexpensive EC2 instance with a role attached. 
* The instance itself receives and rotates session tokens automatically so you won’t need to enter any Acceskey/Secretkey in the CLI config.
* Example: An `a1.medium` instance (ARM based) is $0.0255 per Hour. 
  * This comes to $300 / year for a simple secure access path
  * Reduce this cost by shuting the instance down when not in use

## Guides

* [Limiting the type of EC2 instances that the user can launch](Guides/limiting_ec2_instance_types.md). 
* [Programmatically launching an EC2 instance using AWS JavaScript SDK with an IAM role authorizing AWS CloudWatch Agent to send logs to AWS CloudWatch without using Access Keys](Guides/programatic_ec2_cw.md)
