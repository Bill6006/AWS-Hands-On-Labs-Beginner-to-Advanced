Below is one way to reformat the Level 02 README so that it visually follows the same structured layout as the Level 01 README—without altering any of the original content. You can save this as your new “02 README.md” file.

---

```markdown
# How to Create a Basic Web Server with AWS CloudFormation Designer

This guide walks you through using AWS CloudFormation Designer to visually create a CloudFormation template that provisions a basic web server running on an EC2 instance in a VPC. You’ll learn to add and connect resources, configure parameters and mappings, specify resource properties, and finally launch a CloudFormation stack to bring your web server online.

---

## AWS Services Used

- **AWS CloudFormation Designer** – To visually design and edit your infrastructure template  
- **Amazon VPC** – To create a virtual network for your resources  
- **Amazon EC2** – To host your web server instance  
- **Security Groups** – To control network traffic for your EC2 instance  
- **Internet Gateway & Route Table** – To enable Internet connectivity for your VPC

---

## AWS CloudFormation Walkthrough

This document is based on the official AWS CloudFormation Getting Started walkthrough. It guides you through creating your first CloudFormation stack using the AWS Management Console. By following these steps, you'll learn how to provision basic AWS resources, monitor stack events, test your web server, and clean up afterward.

*Note:* While CloudFormation itself is free, you will be charged for the underlying AWS resources (e.g., Amazon EC2 and Amazon S3) that you create. If you're new to AWS, you can use the [AWS Free Tier](https://aws.amazon.com/free/) to minimize or eliminate costs during your testing.

---

## Creating Your First Stack

This walkthrough demonstrates the process of creating a CloudFormation stack written in YAML—a human-readable format that is widely used for defining infrastructure as code. For a guided, hands-on workshop, see [Getting Started with AWS CloudFormation](https://catalog.us-east-1.prod.workshops.aws).

### Prerequisites

- **AWS Account:** Ensure you have an AWS account with an IAM user or role that has permissions for Amazon EC2, Amazon S3, and CloudFormation (or administrative access).  
- **VPC:** You must have a Virtual Private Cloud (VPC) with internet access. The default VPC that comes with your account is sufficient for this exercise.

---

### Step-by-Step Instructions

1. **Open the CloudFormation Console:**  
   Visit the [CloudFormation Console](https://console.aws.amazon.com).

2. **Start Stack Creation:**  
   Choose **Create Stack**.

3. **Use Infrastructure Composer:**  
   On the Create Stack page, select **Build from Infrastructure Composer**, then click **Create in Infrastructure Composer**. This takes you to Infrastructure Composer mode where you can upload and validate the template.

4. **Upload and Validate the Template:**  
   - Choose **Template** and copy/paste the following CloudFormation template into the template editor:

```yaml
AWSTemplateFormatVersion: 2010-09-09
Description: CloudFormation Template for WebServer with Security Group and EC2 Instance

Parameters:
  LatestAmiId:
    Description: The latest Amazon Linux 2 AMI from the Parameter Store
    Type: 'AWS::SSM::Parameter::Value<AWS::EC2::Image::Id>'
    Default: '/aws/service/ami-amazon-linux-latest/amzn2-ami-hvm-x86_64-gp2'

  InstanceType:
    Description: WebServer EC2 instance type
    Type: String
    Default: t2.micro
    AllowedValues:
      - t3.micro
      - t2.micro
    ConstraintDescription: must be a valid EC2 instance type.

  MyIP:
    Description: Your IP address in CIDR format (e.g. 203.0.113.1/32).
    Type: String
    MinLength: '9'
    MaxLength: '18'
    Default: 0.0.0.0/0
    AllowedPattern: '^(\d{1,3}\.){3}\d{1,3}\/\d{1,2}$'
    ConstraintDescription: must be a valid IP CIDR range of the form x.x.x.x/x.

Resources:
  WebServerSecurityGroup:
    Type: AWS::EC2::SecurityGroup
    Properties:
      GroupDescription: Allow HTTP access via my IP address
      SecurityGroupIngress:
        - IpProtocol: tcp
          FromPort: '80'
          ToPort: '80'
          CidrIp: !Ref MyIP

  WebServer:
    Type: AWS::EC2::Instance
    Properties:
      ImageId: !Ref LatestAmiId
      InstanceType: !Ref InstanceType
      SecurityGroupIds:
        - !Ref WebServerSecurityGroup
      UserData: !Base64 |
        #!/bin/bash
        yum update -y
        yum install -y httpd
        systemctl start httpd
        systemctl enable httpd
        echo "<html><body><h1>Hello World!</h1></body></html>" > /var/www/html/index.html

Outputs:
  WebsiteURL:
    Value: !Join
      - ''
      - - http://
        - !GetAtt WebServer.PublicDnsName
    Description: Website URL
```

   - Click **Validate** to ensure the YAML syntax is correct.  
   - Next, choose **Create template** to upload it to an S3 bucket. (Note the S3 bucket name for later cleanup.)

5. **Configure the Stack:**  
   - Click **Next** on the Create Stack page.  
   - On the Specify Stack Details page, enter a stack name (for example, `MyTestStack`).  
   - Under Parameters, adjust values as needed:
     - **LatestAmiId:** Defaults to the latest Amazon Linux 2 AMI.  
     - **InstanceType:** Choose `t2.micro` or `t3.micro`.  
     - **MyIP:** Provide your public IP in CIDR format (e.g., `203.0.113.1/32`).

6. **Review and Submit:**  
   Click **Next** twice to review your settings, then click **Submit** to create the stack.

---

## Monitoring Stack Creation

Once you submit your stack, CloudFormation begins creating the resources. The stack (e.g., `MyTestStack`) will appear with a status of `CREATE_IN_PROGRESS`. To monitor progress:

- In the CloudFormation console, select your stack.
- Navigate to the **Events** tab to see real-time status updates such as:  
  - `CREATE_IN_PROGRESS` when resource creation starts.  
  - `CREATE_COMPLETE` when a resource is successfully created.
- When the stack reaches `CREATE_COMPLETE`, your stack and its resources have been provisioned.

---

## Testing the Web Server

After the stack is successfully created:

1. Go to the **Outputs** tab in the CloudFormation console.
2. Find the `WebsiteURL` output, which provides the public URL of your EC2 instance.
3. Open the URL in your web browser; you should see a "Hello World!" message confirming that Apache HTTP Server is running on your EC2 instance.

---

## Clean Up

To avoid any unexpected charges, delete the stack and its resources when you’re finished testing.

### Deleting the CloudFormation Stack

1. Open the CloudFormation console.
2. Select your stack (`MyTestStack`) from the list.
3. Click **Delete**.
4. Confirm the deletion when prompted. The stack status will change to `DELETE_IN_PROGRESS` and then eventually disappear.

### Deleting the S3 Bucket (if applicable)

If you created an S3 bucket to store your template, follow these steps:

1. Open the Amazon S3 console.
2. In the left navigation pane, click **Buckets**.
3. Select your bucket and choose **Empty**. Confirm by typing `permanently delete`.
4. Once emptied, select the bucket again and choose **Delete**.
5. Confirm the bucket deletion.



Delete:
https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/gettingstarted.walkthrough.html