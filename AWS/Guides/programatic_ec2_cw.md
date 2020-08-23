### How to launch an EC2 instance that sends logs to AWS CloudWatch without Access Keys

Rather than storing Access Keys on an EC2 instance (launched programmtically) to allow it to send logs to AWS CloudWatch, attach the required IAM role.

* Create a new role with policy "CloudWatchAgentServerPolicy". This policy exists for this usecase. 

```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": [
                "cloudwatch:PutMetricData",
                "ec2:DescribeVolumes",
                "ec2:DescribeTags",
                "logs:PutLogEvents",
                "logs:DescribeLogStreams",
                "logs:DescribeLogGroups",
                "logs:CreateLogStream",
                "logs:CreateLogGroup"
            ],
            "Resource": "*"
        },
        {
            "Effect": "Allow",
            "Action": [
                "ssm:GetParameter"
            ],
            "Resource": "arn:aws:ssm:*:*:parameter/AmazonCloudWatch-*"
        }
    ]
}
```

* A role is specified by several strings. **We will use the "Instance Profile ARN"**. This is of the type: `arn:aws:iam:<account-id>:instance-profile/<role-name>`. I will call this string IPA for the rest of the guide. 

* The code snippet using AWS-SDK in NodeJS below shows how to attach this IPA to an EC2 instance created programmatically. 

```js
const AWS = require('aws-sdk');

const ec2 = new AWS.EC2({ apiVersion: '2016-11-15' });

const ec2Params = {
    ImageId: 'YOUR_AMI_ID',
    InstanceType: 'YOUR_INSTANCE_TYPE',
    KeyName: 'YOUR_KEYPAIR_NAME',
    MinCount: 1,
    MaxCount: 1,
    SecurityGroupIds: [
        'YOUR_SECURITY_GROUP_NAME'
    ],
    UserData: '',
    IamInstanceProfile: {
        Arn: 'IPA' // Created above
    },
    TagSpecifications: [
        {
            ResourceType: "instance",
            Tags: [
                {
                    Key: "Name",
                    Value: "YOUR_NAME"
                },
                // All tags are optional. 
                // Add as many additional tags as necessary
        }
    ]
};

ec2.runInstances(ec2Params, function (err, data) {
    if (err) {
        console.log(err, err.stack);
    } else {
        console.log(data);
    }
});

```

Several of the parameters in the above example are optional. In case they are not specified, AWS will make the best decision. Many other parameters can be specified. 

#### References

* [Using an IAM Role to Grant Permissions to Applications Running on Amazon EC2 Instances](https://docs.aws.amazon.com/IAM/latest/UserGuide/id_roles_use_switch-role-ec2.html)

* [Create IAM Roles and Users for Use With CloudWatch Agent](https://docs.aws.amazon.com/AmazonCloudWatch/latest/monitoring/create-iam-roles-for-cloudwatch-agent-commandline.html)

* [AWS JavaScript SDK runInstances](https://docs.aws.amazon.com/AWSJavaScriptSDK/latest/AWS/EC2.html#runInstances-property)

