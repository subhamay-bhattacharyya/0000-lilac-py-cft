![](https://img.shields.io/github/commit-activity/t/subhamay-bhattacharyya/0000-lilac-py-cft)&nbsp;![](https://img.shields.io/github/last-commit/subhamay-bhattacharyya/0000-lilac-py-cft)&nbsp;![](https://img.shields.io/github/release-date/subhamay-bhattacharyya/0000-lilac-py-cft)&nbsp;![](https://img.shields.io/github/repo-size/subhamay-bhattacharyya/0000-lilac-py-cft)&nbsp;![](https://img.shields.io/github/directory-file-count/subhamay-bhattacharyya/0000-lilac-py-cft
)&nbsp;![](https://img.shields.io/github/actions/workflow/status/subhamay-bhattacharyya/0000-lilac-py-cft/deploy-stack.yaml)&nbsp;![](https://img.shields.io/bitbucket/issues/subhamay-bhattacharyya/0000-lilac-py-cft)&nbsp;![](https://img.shields.io/github/languages/top/subhamay-bhattacharyya/0000-lilac-py-cft)&nbsp;![](https://img.shields.io/github/commit-activity/m/subhamay-bhattacharyya/0000-lilac-py-cft)&nbsp;![](https://img.shields.io/badge/project_status-green-green)



# Project Lilac: Custom Resource Lambdas

This repository creates a stack set with two Custom Resource Lambda Functions.

## Description

This repository creates two Lambda fucntion which can be used a custom resource lambda to create:

1. **lilac-s3-folder-devl-us-east-1**: Creates / Deletes a custom S3 folder using creation / deleteion of the stack.
1. **lilac-s3-notify-devl-us-east-1**: Creates a S3 event totification.

Sample S3 Bucket YAML
```yaml
  S3LandingZoneBucket:
    Type: AWS::S3::Bucket
    Properties:
      BucketName: !Sub '${ProjectName}-${S3LandingZoneBucketBaseName}-${Environment}-${AWS::Region}${CiBuild}'
      BucketEncryption:
          ServerSideEncryptionConfiguration: 
          - BucketKeyEnabled: True
            ServerSideEncryptionByDefault: 
              SSEAlgorithm: 'aws:kms'
              KMSMasterKeyID: !Sub 'alias/${S3KmsMasterKeyAlias}'
      PublicAccessBlockConfiguration:
        BlockPublicAcls: !Ref S3BucketBlockPublicAcls
        BlockPublicPolicy: !Ref S3BucketBlockPublicPolicy
        IgnorePublicAcls: !Ref S3BucketIgnorePublicAcls
        RestrictPublicBuckets: !Ref S3BucketRestrictPublicBuckets
      Tags:
        - Key: ProjectName
          Value: !Ref ProjectName
        - Key:  Environment
          Value: !Ref Environment
        - Key: GitHubRef
          Value: !Ref GitHubRef
        - Key: GitHubURL
          Value: !Ref GitHubURL
        - Key: GitHubWFRunNumber
          Value: !Ref GitHubWFRunNumber
        - Key: GitHubSHA
          Value: !Ref GitHubSHA
        - Key: StackName
          Value: !Ref StackName
  S3DataBucketCustomResource:
    Type: Custom::S3CustomResource
    Properties:
      ServiceToken: 
        Fn::ImportValue: !Sub '${S3CustomResourceStackName}-S3CreateFolderCustomResourceLambdaFunctionArn' 
      bucket_name: !Ref S3LandingZoneBucket
      folders_to_create: !Ref S3DataBucketFolder
```

### Getting Started

* This repository is configured to deploy the stack in Development, Staging and Production AWS Accounts. To use the pipeline you need to have 
three AWS Accounts created in an AWS Org under a Management Account (which is the best practice). The Org structure will be as follows:

```
Root
├─ Management
├─ Development
├─ Test
└─ Production
```

* Create KMS Key in each of the AWS Accounts which will be used to encrypt the resources.

* Create an OpenID Connect Identity Provider

* Create an IAM Role for OIDC and use the sample Trust Policy in each of the three AWS accounts
```
{
    "Version": "2008-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Principal": {
                "Federated": "arn:aws:iam::<Account Id>:oidc-provider/token.actions.githubusercontent.com"
            },
            "Action": "sts:AssumeRoleWithWebIdentity",
            "Condition": {
                "StringLike": {
                    "token.actions.githubusercontent.com:sub": [
                        "repo:<GitHub User>/<GitHub Repository>:ref:refs/head/main",
                    ]
                }
            }
        }
    ]
}
```

  * Create an IAM Policy to allow CloudFormation access and attach it to the OIDC Role, using the sample policy document:
```
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": [
                "cloudformation:CreateStack",
                "cloudformation:DescribeStacks",
                "cloudformation:CreateChangeSet",
                "cloudformation:DescribeChangeSet",
                "cloudformation:DeleteChangeSet",
                "cloudformation:ExecuteChangeSet",
                "cloudformation:DeleteStack"
            ],
            "Resource": "*"
        }
    ]
}
```

* Create an IAM Policy to allow creation and deletion of resources  and attach it to the OIDC Role, using the following sample policy document:
```
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Sid": "AllowS3Access",
            "Effect": "Allow",
            "Action": [
                "s3:PutObject",
                "s3:GetObject",
                "s3:CreateBucket",
                "s3:PutObjectTagging",
                "s3:PutBucketPublicAccessBlock",
                "s3:PutBucketAcl",
                "s3:DeleteObject",
                "s3:DeleteBucket",
                "s3:ListBucket",
                "s3:PutBucketTagging",
                "s3:GetBucketTagging",
                "s3:GetBucketPolicy",
                "s3:GetBucketAcl",
                "s3:GetObjectAcl",
                "s3:GetObjectVersionAcl",
                "s3:PutObjectAcl",
                "s3:PutObjectVersionAcl",
                "s3:GetBucketCORS",
                "s3:PutBucketCORS",
                "s3:GetBucketWebsite",
                "s3:GetBucketVersioning",
                "s3:GetAccelerateConfiguration",
                "s3:GetBucketRequestPayment",
                "s3:GetBucketLogging",
                "s3:GetLifecycleConfiguration",
                "s3:GetReplicationConfiguration",
                "s3:GetEncryptionConfiguration",
                "s3:GetBucketObjectLockConfiguration",
                "s3:GetBucketOwnershipControls",
                "s3:GetObjectTagging",
                "s3:GetBucketNotification",
                "s3:PutEncryptionConfiguration",
                "s3:PutBucketOwnershipControls",
                "s3:PutBucketNotification"
            ],
            "Resource": "*"
        },
        {
            "Sid": "IAMAccess",
            "Effect": "Allow",
            "Action": [
                "iam:GetRole",
                "iam:CreateRole",
                "iam:DeleteRolePolicy",
                "iam:PutRolePolicy",
                "iam:DeleteRole",
                "iam:PassRole",
                "iam:TagRole",
                "iam:CreatePolicy",
                "iam:ListRolePolicies",
                "iam:GetPolicy",
                "iam:ListAttachedRolePolicies",
                "iam:GetPolicyVersion",
                "iam:ListInstanceProfilesForRole",
                "iam:AttachRolePolicy",
                "iam:DetachRolePolicy",
                "iam:ListPolicyVersions",
                "iam:DeletePolicy",
                "iam:UntagRole"
            ],
            "Resource": "*"
        },
        {
            "Sid": "LambdaCreateAccess",
            "Effect": "Allow",
            "Action": [
                "lambda:GetFunction",
                "lambda:DeleteFunction",
                "lambda:CreateFunction",
                "lambda:TagResource",
                "lambda:InvokeFunction",
                "lambda:PutFunctionConcurrency",
                "lambda:CreateFunction",
                "lambda:ListVersionsByFunction",
                "lambda:GetFunctionCodeSigningConfig",
                "lambda:PutFunctionEventInvokeConfig",
                "lambda:AddPermission",
                "lambda:GetFunctionEventInvokeConfig",
                "lambda:GetPolicy",
                "lambda:DeleteFunctionEventInvokeConfig",
                "lambda:RemovePermission",
                "lambda:ListTags"
            ],
            "Resource": "*"
        },
        {
            "Sid": "SQSCreateAccess",
            "Effect": "Allow",
            "Action": [
                "sqs:GetQueueAttributes",
                "sqs:ListQueueTags",
                "sqs:CreateQueue",
                "sqs:TagQueue",
                "sqs:SetQueueAttributes",
                "sqs:SendMessage",
                "sqs:DeleteQueue"
            ],
            "Resource": "*"
        },
        {
            "Sid": "SNSCreateAccess",
            "Effect": "Allow",
            "Action": [
                "SNS:GetTopicAttributes",
                "SNS:ListTagsForResource",
                "SNS:GetSubscriptionAttributes",
                "SNS:CreateTopic",
                "SNS:TagResource",
                "SNS:SetTopicAttributes",
                "SNS:DeleteTopic",
                "SNS:Subscribe",
                "SNS:Unsubscribe"
            ],
            "Resource": "*"
        },
        {
            "Sid": "DynamoDBAccess",
            "Effect": "Allow",
            "Action": [
                "dynamodb:DescribeTable",
                "dynamodb:DescribeContinuousBackups",
                "dynamodb:DescribeTimeToLive",
                "dynamodb:ListTagsOfResource",
                "dynamodb:CreateTable",
                "dynamodb:TagResource",
                "dynamodb:DeleteTable"
            ],
            "Resource": "*"
        },
        {
            "Sid": "KMSKeyAccess",
            "Effect": "Allow",
            "Action": [
                "kms:Encrypt",
                "kms:Decrypt",
                "kms:DescribeKey"
            ],
            "Resource": [
                "arn:aws:kms:<AWS Region>:<AWS Account Id>:key/<KMS Key Id>"
            ]
        },
        {
            "Sid": "CloudWatchAccess",
            "Effect": "Allow",
            "Action": [
                "cloudwatch:PutMetricAlarm",
                "cloudwatch:DescribeAlarms",
                "cloudwatch:ListTagsForResource",
                "cloudwatch:DeleteAlarms",
                "cloudwatch:TagResource"
            ],
            "Resource": "*"
        }
    ]
}
```

### Installing

* Clone the repository.
* Create a S3 bucket to used a code repository.
* Modify the params/cfn-parameter.json with you Key Id
```
{
    "stack-prefix": "<Stack Prefix>",
    "stack-suffix": "<Stack Suffix>",
    "template-path": "/cft/s3-custom-resource-lambda.yaml",
    "parameters": {
        "devl": [
            {
                "Environment": "devl",
                "KmsMasterKeyId": "<KMS Key Id in Dev Account>",
                "ProjectName": "<Your project name>"
            }
        ],
        "test": [
            {
                "Environment": "test",
                "KmsMasterKeyId": "<KMS Key Id in Test Account>",
                "ProjectName": "<Your project name>"
            }
        ],
        "prod": [
            {
                "Environment": "prod",
                "KmsMasterKeyId": "<KMS Key Id in Prod Account>",
                "ProjectName": "<Your project name>"
            }
        ]
    }
}
```

* Create three repository environments in GitHub (devl, test, prod)

* Create the following GitHub repository Secrets:


|Secret Name|Secret Value|
|-|-|
|AWS_REGION|```us-east-1```|
|DEVL_AWS_KMS_KEY_ARN|```arn:aws:kms:<AWS Region>:<Development Account Id>:key/<KMS Key Id in Development>```|
|TEST_AWS_KMS_KEY_ARN|```arn:aws:kms:<AWS Region>:<Test Account Id>:key/<KMS Key Id in Test>```|
|PROD_AWS_KMS_KEY_ARN|```arn:aws:kms:<AWS Region>:<Production Account Id>:key/<KMS Key Id in Production>```|
|DEVL_AWS_ROLE_ARN|```arn:aws:iam::<Development Account Id>:role/<OIDC IAM Role Name>```|
|TEST_AWS_ROLE_ARN|```arn:aws:iam::<Test Account Id>:role/<OIDC IAM Role Name>```|
|PROD_AWS_ROLE_ARN|```arn:aws:iam::<Production Account Id>:role/<OIDC IAM Role Name>```|
|DEVL_CODE_REPOSITORY_S3_BUCKET|```<Repository S3 Bucket in Development>```|
|TEST_CODE_REPOSITORY_S3_BUCKET|```<Repository S3 Bucket in Test>```|
|PROD_CODE_REPOSITORY_S3_BUCKET|```<Repository S3 Bucket in Production>```|

### Executing the CI/CD Pipeline

* Create Create a feature branch and push the code.
* The CI/CD pipeline will create a build and then will deploy the stack to devlopment.
* Once the Stage and Prod deployment are approved (If you have configured with protection rule ) the stack will be reployed in the respective environments

## Help

[:email:] Subhamay Bhattacharyya  - [subhamay.aws@gmail.com]


## Authors

Contributors names and contact info

Subhamay Bhattacharyya  - [subhamay.aws@gmail.com]

## Version History

* 0.1
    * Initial Release

## License

This project is licensed under Subhamay Bhattacharyya. All Rights Reserved.

## Acknowledgments
