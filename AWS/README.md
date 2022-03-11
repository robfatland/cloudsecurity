# AWS Cloud Security Notes

Authors: Chintan Pathak, Rob Fatland

## Terminology in progressive order

* **Intrusion**: Unuathorized use of a resource.
* **CLI**: Command line interface, a means of manipulating AWS resources across a secure connection
* **SDK**: Software Development Kit, a toolbox for connecting a computing environment to the AWS cloud; see CLI above
* **Security Group**: A logical construct the defines / restrics access to AWS resources 
    * Can be updated / applied on the fly without stopping the resources 
    * Example: Commonly and by default an AWS EC2 instance opens port 22 for **`ssh`** access
        * This port rule can be modified within the Security Group
* **IAM**: Identity and Access Management, a generic term for the access mechanisms relative to a particular cloud
    * __User__: A person with access to a cloud account. User-level access is controlled by assigned Roles which have attached Policy JSON documents  
    * __Role__: A logical construct on the cloud that grants privilege to an entity to perform actions. Entities may be Users or Services.
        * Example: A Lambda service is unable to execute its own code unless it is granted a Lambda execution Role
            * This decouples permission to act from the action itself: Makes actions externally manageable
    * __Policy__: A text (JSON) document commonly associated with a Role that spells out the logic of granted permissions
* **Authentication**: gaining access to a cloud service or resource via credentials
    * __Access Keys__ (two components: public and secret): On AWS: Two auto-generated character strings
        * Example: The AWS CLI can be configured with these Keys to authenticate and carry out actions like launching a VM
        * ***DANGER***: Publishing access keys on GitHub will incur thousands of dollars of cloud spend in a matter of minutes
            * To learn more, search on "AWS access key github danger"
            * To avoid this fate: Treat access keys like you would the password to your bank account
    * __Keypair file__: An access key file -- commonly with a **`.pem`** file extension -- for **`ssh`** access to a Linux VM
        * Used in lieu of user passwords as more secure
    * __AWS STS__: AWS Security Token Service: Temporary, limited-privileged authentication credentials
        * Recommended for authentication for the AWS CLI / SDK
    * **SSO**: Single Sign-On, refers to confederated authentication: A single identity like a NetID is used to authenticate to more than one service
    * **MFA / TFA**: Multi-factor authentication / Two-factor authentication: A security measure to reduce intrusion risk



## Security Resources


* [AWS Support video on MFA CLI](https://aws.amazon.com/premiumsupport/knowledge-center/authenticate-mfa-cli/)
* [AWS CLI Role configuration](https://docs.aws.amazon.com/cli/latest/userguide/cli-configure-role.html#cli-configure-role-mfa)
* [IAM Roles and Permissions](https://docs.aws.amazon.com/IAM/latest/UserGuide/id_roles_use_permissions-to-switch.html
* [IAM Roles for Users](https://docs.aws.amazon.com/IAM/latest/UserGuide/id_roles_create_for-user.html
* [IAM Roles: Assuming Roles](https://awscli.amazonaws.com/v2/documentation/api/latest/reference/sts/assume-role.html)


## Very Secure method outline


* Create a Role for use by an IAM user
    * Optional: Require MFA
    * In Permissions select permissions to grant to the IAM user
* Create an IAM User, configure MFA, download the combined AccessKey/SecretKey as a (human-readable) text file to a safe non-repo location
* Create an IAM Policy which only allows: `sts:AssumeRole` and attach this to the IAM User 
    * Alternative: Attach the Policy to a Group and place the User in the Group
* In the AWS CLI use **`configure`** with this Access Key
    * [Instructions](https://docs.aws.amazon.com/cli/latest/userguide/cli-configure-role.html#cli-configure-role-mfa)
* For the first command run from the CLI for your session...
    * Temporary credentials expire in 12 hours or when the shell closes
    * ... add `–profile` (note: MFA transaction is included)
    * Thereafter...
        * Export the temporary credentials to environment variable **`profilename`**
        * Subsequent CLI calls include **`export AWS_PROFILE=profilename`**


## Alternative: Python


* Allows you to assume a Role and prompt for Credentials:
* [Instructions](https://docs.aws.amazon.com/IAM/latest/UserGuide/id_credentials_mfa_sample-code.html#MFAProtectedAPI-example-assumerole)
* Modify this script to add the temporary credentials as environment variables


## Alternative cheap EC2 method


* Use a cheap EC2 instance with a role attached. 
* Instance receives and rotates session tokens automatically
    * You do not enter Access keys into AWS CLI `config`
    * Example: `a1.medium` (ARM) at $0.0255 per hour runs about $300 / year (less with duty cycle start/stop) 
    * Provides a simple and secure access path


## Guides


* [Limit User EC2 type access](Guides/limiting_ec2_instance_types.md). 
* [Programmatic EC2 instance launching](Guides/programatic_ec2_cw.md)
