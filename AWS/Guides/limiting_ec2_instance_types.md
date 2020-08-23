### Method to limit the type of instances a User can create

(applies to Console as well as Programmatic access)

The limits can be applied by creating the appropriate role. Avoid providing uses with AdministrativeAccess and give them specific roles. New roles can be created and as many policies can be attached as needed. 

* **Policy**: First create a policy that does the following: 
    * Allow the user to use console and API. 
    * Allow the user to start, stop, run and terminate instances (create and delete tags etc.)
    * Deny running instances that do not belong to the allowed list of instances. 

    The policy below does the above in three statements. This can be directly pasted, or created using the Visual Editor. 

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

Parsing the above policy: 
* The "Version" key specifies the year of policy specification (should not be changed).
* The "Statement" key can be a single statement or an array of statements as needed. 
* The "Sid" key is just for readability and has no impact on the actual implementation. 
* The "Action"(s) and "Resource"(s) will vary depending on the type of resource this policy is trying to address. EC2 usually need access to these resources. 
* Resources can be made more specific than specified above. For example, a resource is specified its Amazon Resource Name (ARN). Replacing the current ARN in statement 2 ("Sid": "VisualEditor1") with `"arn:aws:ec2:us-west-2:*:subnet/subnet-xxxxx"` would limit the EC2 instances to only belong to region `us-west-2` and a subnet specified by `subnet-xxxxx` (where xxxxx is a specific subnet in your VPC). Similarly, other `*` can be replaced to make them more specific. 
* Finally, the statement with `"Effect": "Deny"` specifies what is denied, and here the `"Condition"` limits the user to spin up only t2.nano, t2.small, t2.micro and t2.medium. If, for example, all nano instances are allowed, then the above can be modified to `*.nano`. 

* **Role**: Now attach the above created policy to a role. A new role can be created or an existing role can be modified to include the above policy. 

#### Example usage of the above role 

The users with the role above should be able to use the AWS Console to spin-up the said type of instances (not tested). The example below, shows how to apply these limits on an EC2 instance that programmatically spins up EC2 instances. 

* Create the EC2 instance (parent) using the AWS Console. 
* After the parent instance is created, goto Actions -> Instance Settings -> Attach/Replace IAM Role and specify the role created above. 
* This parent instance now, does not need the Access Keys to spin up new instances (child) programmatically. It can perpetually (as long as the IAM role is attached) spin up *only* the specified type of instances. This internally uses the AWS STS to give the parent instance credentials to generate child instances. 


#### References 

* [Limiting Allowed AWS Instance Type With IAM Policy](https://blog.vizuri.com/limiting-allowed-aws-instance-type-with-iam-policy)
* [Amazon EC2: Allows Launching EC2 Instances in a Specific Subnet, Programmatically and in the Console](https://docs.aws.amazon.com/IAM/latest/UserGuide/reference_policies_examples_ec2_instances-subnet.html)

