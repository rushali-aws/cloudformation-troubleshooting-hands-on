---
title: "Avoid Making Out-of-band changes to Cloudformation Stack Resources"
weight: 300
---

_Lab Duration: ~10 minutes_

---

### Overview

This lab will guide you on the impact of making out-of-band changes to the Cloudformation stack resources and the effect they can have on Cloudformation stack workflow.

An out-of-band change can include:

+ Modifying resource configuration via AWS Console
+ Updating the resource configuration via AWS CLI
+ Making SDK calls to modify/update the resource
+ Deleting the complete resource via AWS CLI/Console/SDK

### Topics Covered

By the end of this lab, you will be able to:

+ Deploy a CloudFormation stack
+ Understand the effects of changing the resource configuration, manually
+ Understand why out-of-band changes should be avoided 


### Step 1: Create the Initial CloudFormation Stack

::alert[As you read through each section, there are code samples at the end. Copy these into your own template file.]{type="info"}

1. Go to `code/workspace/` directory.
2. Open the `initial-stack.yaml` file.
3. Copy the code as you go through the topics below.

:::code{language=yaml showLineNumbers=false showCopyAction=true}

AWSTemplateFormatVersion: '2010-09-09'
Description: Demonstrates different CloudFormation update types

Resources:
  MyS3Bucket:
    Type: AWS::S3::Bucket
    Properties:
        BucketName: !Sub "update-demo-bucket-${AWS::AccountId}"
        VersioningConfiguration:
          Status: Enabled

:::

#### Deploy the Initial Stack

:::code{showLineNumbers=false showCopyAction=true}
aws cloudformation create-stack --stack-name UpdateDemoStack --template-body file://initial-stack.yaml --capabilities CAPABILITY_NAMED_IAM
:::

#### Step 2: Modify the S3 bucket manually from the S3 console
Now, we will modify the S3 bucket configuration:
    1.  Select the Cloudformation stack created in the Step 1
    2.  Select the Resources section of the cloudformation step and open the S3 bucket created through the hyperlink
    3.  Click on the Properties tab and under Bucket Versioning, click on Edit
  4.  Choose 'Suspend' and save the changes

::alert[By following the above steps, we disabled the bucket versioning manually whereas according to the Cloudformation stack, the bucket versioning is 'Enabled'. When you detect the drift on the Cloudformation, you will find this S3 bucket resource drifted]{type="info"}

#### Step 3: Update the Cloudformation stack using the modified template

:::code{language=yaml showLineNumbers=false showCopyAction=true}

AWSTemplateFormatVersion: '2010-09-09'
Description: Demonstrates different CloudFormation update types

Resources:
  MyS3Bucket:
    Type: AWS::S3::Bucket
    Properties:
      BucketName: !Sub "update-demo-bucket-${AWS::AccountId}"
      VersioningConfiguration:
        Status: Enabled
      ObjectLockEnabled: true
      ObjectLockConfiguration:
        ObjectLockEnabled: Enabled
        Rule:
          DefaultRetention:
            Mode: COMPLIANCE
            Days: 5  

:::

::alert[Stack update will fail with an error message]{type="info"}

#### Step 4: Observe the error message

+ Below is the error message observed:
:::code{showLineNumbers=false showCopyAction=true}
Resource handler returned message: "Versioning must be 'Enabled' on the bucket to apply a Object Lock configuration"
:::
+ The above error message states that the versioning must be enabled on the S3 bucket. As per the cloudformation stack template, we can observe that the bucket versioning is already enabled. However we are still getting the error
+ This is because we have made out-of-band changes on the bucket due to which there is mismatch in the resource configuration. 


#### Step 5: Clean Up 

You no longer need the stack:

:::code{showLineNumbers=false showCopyAction=true}
aws cloudformation delete-stack --stack-name UpdateDemoStack
:::

### Conclusion
Congratulations! You have successfully learned the impact of making out-of-band changes to the Cloudformation stack resources and the effect they can have on Cloudformation stack workflow. Thus, out-of-band changes must be avoided. 
