## Method for limiting User choice of instance types

* Applies to Console as well as Programmatic access

The limits are based on creating an appropriate Role. 
This avoids providing Users with blanket *AdministrativeAccess*. 
As many Policies as needed can be attached to a Role.
The Role can be assigned to a User *or* to another entity such as an EC2 instance. 
The latter case supports spinning up additional Instances programmatically.


* Underlying **Policy**: The idea is to create a policy that 
    * Allows the User to use console and API
    * Allows the User to start, stop, run and terminate instances, create and delete tags, ...
    * Denies the User acess to Instance types that do not belong to the allowed list of types 


The policy below does this in three statements. This can be directly pasted; or created using the Visual Editor.


> ***Note: This Policy has not been recently tested (March 2022). Be prepared to test/verify!!!***


```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Sid": "VisualEditor0",
            "Effect": "Allow",
            "Action": [
                "ec2:Describe*",
                "ec2:GetConsole*"
            ],
            "Resource": "*"
        },
        {
            "Sid": "VisualEditor1",
            "Effect": "Allow",
            "Action": [
                "ec2:TerminateInstances",
                "ec2:DeleteTags",
                "ec2:CreateTags",
                "ec2:RunInstances",
                "ec2:StopInstances"
            ],
            "Resource": [
                "arn:aws:ec2:*:*:key-pair/*",
                "arn:aws:ec2:*:*:instance/*",
                "arn:aws:ec2:*:*:subnet/*",
                "arn:aws:ec2:*:*:volume/*",
                "arn:aws:ec2:*:*:security-group/*",
                "arn:aws:ec2:*:*:network-interface/*",
                "arn:aws:ec2:*::image/*"
            ]
        },
        {
            "Sid": "VisualEditor2",
            "Effect": "Deny",
            "Action": "ec2:RunInstances",
            "Resource": [
                "arn:aws:ec2:*:*:subnet/*",
                "arn:aws:ec2:*:*:instance/*",
                "arn:aws:ec2:*:*:volume/*",
                "arn:aws:ec2:*:*:security-group/*",
                "arn:aws:ec2:*:*:network-interface/*",
                "arn:aws:ec2:*::image/*"
            ],
            "Condition": {
                "ForAnyValue:StringNotLike": {
                    "ec2:InstanceType": [
                        "t2.nano",
                        "t2.small",
                        "t2.micro",
                        "t2.medium"
                    ]
                }
            }
        }
    ]
}
```

Parsing this JSON:

* The "Version" key specifies the year of policy specification (should not be changed).
* The "Statement" key can be a single statement or an array of statements as needed. 
* The "Sid" key is just for readability and has no impact on the actual implementation. 
* The "Action"(s) and "Resource"(s) will vary depending on the type of resource this policy is trying to address. EC2 usually need access to these resources. 
* Resources can be made more specific than specified above. For example, a resource is specified its Amazon Resource Name (ARN). Replacing the current ARN in statement 2 ("Sid": "VisualEditor1") with `"arn:aws:ec2:us-west-2:*:subnet/subnet-xxxxx"` would limit the EC2 instances to only belong to region `us-west-2` and a subnet specified by `subnet-xxxxx` (where xxxxx is a specific subnet in your VPC). Similarly, other `*` can be replaced to make them more specific. 
* The statement with `"Effect": "Deny"` specifies what is denied, and here the `"Condition"` limits the user to spin up only t2.nano, t2.small, t2.micro and t2.medium. If, for example, all nano instances are allowed, then the above can be modified to `*.nano`. 

* **Role**: Now attach the above created policy to a role. A new role can be created or an existing role can be modified to include the above policy. 


The first idea is to assign the Role to an IAM User. The second idea is to assign the Role to a Group and place the User in that Group. 


> ***Note: As mentioned above this needs testing/verification circa March 2022*** 


### Example use


Users on the Console with the Role above should be ready to go. Below we have the continuation of the idea: Assigning the Role to an EC2 instance
so that it can be used to spin up other instances. 


* Create an EC2 parent Instance using the AWS Console. 
* On the Console with this Instance selected > Actions > Instance Settings > Attach/Replace IAM Role and specify the above Role
* Now the parent Instance does not need Access Keys to spin up a child Instances (child) programmatically.
As long as the IAM role is attached it can spin up the specified types of Instances. Internally this uses the AWS STS to give the parent instance credentials to generate child instances.


#### References 

* [Limiting Allowed AWS Instance Type With IAM Policy](https://blog.vizuri.com/limiting-allowed-aws-instance-type-with-iam-policy)
* [Amazon EC2: Allows Launching EC2 Instances in a Specific Subnet, Programmatically and in the Console](https://docs.aws.amazon.com/IAM/latest/UserGuide/reference_policies_examples_ec2_instances-subnet.html)

