---
title: "Create change sets before updating your stacks"
weight: 300
---

_Lab Duration: ~10 minutes_

---

### Overview

This lab will guide you through the process of using Change Sets in AWS CloudFormation before updating a stack. Change Sets allow you to preview modifications before applying them, reducing the risk of unintended changes.

An update can include different types of changes:

+ No Interruption updates
+ Some Interruption updates
+ Replacement updates

### Topics Covered

By the end of this lab, you will be able to:

+ Deploy a CloudFormation stack
+ Modify a CloudFormation template
+ Create a Change Set to preview changes before updating a stack
+ Execute the Change Set and verify the update


### Step 1: Create the Initial CloudFormation Stack

::alert[As you read through each section, there are code samples at the end. Copy these into your own template file.]{type="info"}

1. Go to `code/workspace/` directory.
2. Open the `initial-stack.yaml` file.
3. Copy the code as you go through the topics below.

:::code{language=yaml showLineNumbers=false showCopyAction=true}

AWSTemplateFormatVersion: '2010-09-09'
Description: Demonstrates different CloudFormation update types

Resources:
  MyLogGroup:
    Type: AWS::Logs::LogGroup
    Properties:
      LogGroupName: !Sub "update-demo-log-group-${AWS::AccountId}"
      RetentionInDays: 30

  MyS3Bucket:
    Type: AWS::S3::Bucket
    Properties:
      BucketName: !Sub "update-demo-bucket-${AWS::AccountId}"

  MyEC2Instance:
    Type: AWS::EC2::Instance
    Properties:
      InstanceType: t2.micro
      ImageId: <replace with AMI ID ami-xxxxx>
      Tags:
        - Key: "Environment"
          Value: "Dev"

:::

#### Deploy the Initial Stack

:::code{showLineNumbers=false showCopyAction=true}
aws cloudformation create-stack --stack-name UpdateDemoStack --template-body file://initial-stack.yaml --capabilities CAPABILITY_NAMED_IAM
:::

#### Step 2: Modify the Stack to Trigger Different Update Types
Now, we will modify the stack to include:
	1.	No Interruption Update → Add a new tag to the CloudWatch Log Group
	2.	Interruption Update → Enable versioning on the S3 Bucket (causes temporary interruption)
	3.	Replacement Update → Change the EC2 Instance type to t3.micro (causes instance replacement)

Modify the Template (update-stack.yaml)

```yaml
AWSTemplateFormatVersion: '2010-09-09'
Description: Demonstrates different CloudFormation update types

Resources:
  MyLogGroup:
    Type: AWS::Logs::LogGroup
    Properties:
      LogGroupName: !Sub "update-demo-log-group-${AWS::AccountId}"
      RetentionInDays: 30
      Tags:
        - Key: "Updated"
          Value: "Yes"   # Adding a tag (No Interruption)

  MyS3Bucket:
    Type: AWS::S3::Bucket
    Properties:
      BucketName: !Sub "update-demo-bucket-${AWS::AccountId}"
      VersioningConfiguration:
        Status: Enabled   # Enabling versioning (Interruption)

  MyEC2Instance:
    Type: AWS::EC2::Instance
    Properties:
      InstanceType: t3.micro   # Changing instance type (Replacement)
      ImageId: <replace with AMI ID ami-xxxxx>
      Tags:
        - Key: "Environment"
          Value: "Prod"  # Changing tag value for visibility
```

#### Step 3: Create a Change Set and Preview the Impact


:::code{showLineNumbers=false showCopyAction=true}
aws cloudformation create-change-set --stack-name UpdateDemoStack --change-set-name MixedUpdate --template-body file://update-stack.yaml --capabilities CAPABILITY_NAMED_IAM
:::

#### Step 4: Describe the Change Set


:::code{showLineNumbers=false showCopyAction=true}
aws cloudformation describe-change-set --stack-name UpdateDemoStack --change-set-name MixedUpdate
:::

Expected Output:
+ MyLogGroup (CloudWatch Log Group) → No Interruption (Tag added)
+ MyS3Bucket (S3 Bucket) → Interruption (Versioning enabled)
+ MyEC2Instance (EC2 Instance) → Replacement (New instance type)

#### Step 5: Execute the Change Set

If the changes look correct, execute them:


:::code{showLineNumbers=false showCopyAction=true}
aws cloudformation execute-change-set --stack-name UpdateDemoStack --change-set-name MixedUpdate
:::

#### Step 5: Validate the Changes

+ Check CloudWatch Log Group Tags

:::code{showLineNumbers=false showCopyAction=true}
aws logs list-tags-log-group --log-group-name update-demo-log-group-<YOUR_ACCOUNT_ID>
:::

Expected: Tag Updated: Yes should be present (No Interruption)

+ Check S3 Bucket Versioning

:::code{showLineNumbers=false showCopyAction=true}
aws s3api get-bucket-versioning --bucket update-demo-bucket-<YOUR_ACCOUNT_ID>
:::

Expected: "Status": "Enabled" (Interruption)

+ Check EC2 Instance Type

:::code{showLineNumbers=false showCopyAction=true}
aws ec2 describe-instances --filters "Name=tag:Environment,Values=Prod"
:::

Expected: New EC2 instance with t3.micro (Replacement)

#### Step 6: Clean Up 

You no longer need the stack:

:::code{showLineNumbers=false showCopyAction=true}
aws cloudformation delete-stack --stack-name UpdateDemoStack
:::

### Conclusion
Congratulations! You have successfully learned how to validate CloudFormation update before deploying a stack using Change Sets reducing risks in production environments.