# How to Create a Basic Web Server with AWS CloudFormation Designer

This guide walks you through using AWS CloudFormation Designer to visually create a CloudFormation template that provisions a basic web server running on an EC2 instance in a VPC. You’ll learn to add and connect resources, configure parameters and mappings, specify resource properties, and finally launch a CloudFormation stack to bring your web server online.

---

## AWS Services Used

- **AWS CloudFormation Designer*** – To visually design and edit your infrastructure template  
- **Amazon VPC*** – To create a virtual network for your resources  
- **Amazon EC2*** – To host your web server instance  
- **Security Groups*** – To control network traffic for your EC2 instance  
- **Internet Gateway & Route Table*** – To enable Internet connectivity for your VPC

---

## AWS Architecture Diagram

![AWS architecture diagram](<Resources/Images/AWS architecture diagram.png>)

---

## Deployment Steps

### Step 1: Add and Connect Resources

> **Note:** This step uses the drag-and-drop interface of CloudFormation Designer. All resources are added from the EC2 category.

1. **Open CloudFormation Designer:**  
   - Navigate to [AWS CloudFormation Designer](https://console.aws.amazon.com/cloudformation/designer).

2. **Set the Template Name:**  
   - In the integrated editor (lower half), click the Edit (pencil) icon and change the template name to `BasicWebServerInVPC`.

3. **Add a VPC:**  
   - From the Resource types pane, drag a **VPC** resource onto the canvas.  
   - Rename it to `VPC`.

4. **Add a Subnet:**  
   - Drag a **Subnet** resource into the VPC container and rename it to `PublicSubnet`.  
   - *This subnet will host your EC2 instance.*

5. **Add an EC2 Instance:**  
   - Drag an **Instance** resource inside the `PublicSubnet` and rename it to `WebServerInstance`.

6. **Add a Security Group:**  
   - Drag a **SecurityGroup** resource into the VPC and rename it to `WebServerSecurityGroup`.

7. **Add an Internet Gateway:**  
   - Drag an **InternetGateway** resource onto the canvas (outside the VPC) and rename it to `InternetGateway`.

8. **Connect the Internet Gateway:**  
   - Hover over the Internet gateway attachment (created as `AWS::EC2::VPCGatewayAttachment`) on the **InternetGateway** resource and drag a connection to the `VPC`.  
   - This creates an attachment associating the Internet gateway with the VPC.

9. **Add a Route Table and Route:**  
   - Drag a **RouteTable** resource inside the VPC and rename it to `PublicRouteTable`.  
   - Then, add a **Route** resource inside `PublicRouteTable` and rename it to `PublicRoute`.  
   - Configure the route to direct all traffic (`0.0.0.0/0`) to the Internet gateway by connecting its `GatewayId` to `InternetGateway`.

10. **Establish Dependencies:**  
    - Create an explicit dependency from `PublicRoute` to the gateway attachment by dragging a connection from the `DependsOn` dot on `PublicRoute` to the `AWS::EC2::VPCGatewayAttachment`.  
    - Drag another connection from the `WebServerInstance` resource to the `PublicRoute` resource, ensuring the instance depends on the public route for Internet access.

11. **Associate the Route Table:**  
    - Connect `PublicRouteTable` to `PublicSubnet` so that the subnet uses the public route table.

12. **Save Your Template:**  
    - From the File menu (file icon) in CloudFormation Designer, save the template locally to avoid losing your work.

---

### Step 2: Add Parameters, Mappings, and Outputs

> **Tip:** Using parameters and mappings makes your template flexible and reusable across environments.

1. **Add Parameters:**  
   - Click on an open area of the canvas to access template-level components, then switch to the **Parameters** tab in the integrated editor.  
   - Add the following parameters (available in both JSON and YAML):

   **YAML Example:**
   ```yaml
   Parameters:
     InstanceType:
       Description: WebServer EC2 instance type
       Type: String
       Default: t2.small
       AllowedValues:
         - t1.micro
         - t2.nano
         - t2.micro
         - t2.small
         - t2.medium
         - t2.large
         - m1.small
         - m1.medium
         - m1.large
         - m1.xlarge
         - m2.xlarge
         - m2.2xlarge
         - m2.4xlarge
         - m3.medium
         - m3.large
         - m3.xlarge
         - m3.2xlarge
         - m4.large
         - m4.xlarge
         - m4.2xlarge
         - m4.4xlarge
         - m4.10xlarge
         - c1.medium
         - c1.xlarge
         - c3.large
         - c3.xlarge
         - c3.2xlarge
         - c3.4xlarge
         - c3.8xlarge
         - c4.large
         - c4.xlarge
         - c4.2xlarge
         - c4.4xlarge
         - c4.8xlarge
         - g2.2xlarge
         - g2.8xlarge
         - r3.large
         - r3.xlarge
         - r3.2xlarge
         - r3.4xlarge
         - r3.8xlarge
         - i2.xlarge
         - i2.2xlarge
         - i2.4xlarge
         - i2.8xlarge
         - d2.xlarge
         - d2.2xlarge
         - d2.4xlarge
         - d2.8xlarge
         - hi1.4xlarge
         - hs1.8xlarge
         - cr1.8xlarge
         - cc2.8xlarge
         - cg1.4xlarge
       ConstraintDescription: must be a valid EC2 instance type.
     KeyName:
       Description: Name of an EC2 KeyPair for SSH access
       Type: AWS::EC2::KeyPair::KeyName
       ConstraintDescription: must be the name of an existing EC2 KeyPair.
     SSHLocation:
       Description: The IP address range to allow SSH access
       Type: String
       MinLength: '9'
       MaxLength: '18'
       Default: 0.0.0.0/0
       AllowedPattern: '(\d{1,3}\.){3}\d{1,3}/(\d{1,2})'
       ConstraintDescription: must be a valid IP CIDR range.
   ```

2. **Add Mappings:**  
   - Switch to the **Mappings** tab and add a mapping to determine the AMI ID based on instance type and region:

   **YAML Example:**
   ```yaml
   Mappings:
     AWSInstanceType2Arch:
       t2.micro:
         Arch: HVM64
       t2.small:
         Arch: HVM64
       t2.medium:
         Arch: HVM64
       # (Additional mappings as needed)
     AWSRegionArch2AMI:
       us-east-1:
         HVM64: ami-0ff8a91507f77f867
       us-west-2:
         HVM64: ami-a0cfeed8
       # (Add mappings for other regions)
   ```

3. **Add Outputs:**  
   - Switch to the **Outputs** tab and add an output to display the web server’s URL using the instance’s public IP:

   **YAML Example:**
   ```yaml
   Outputs:
     WebsiteURL:
       Value: !Join 
         - ''
         - - 'http://'
           - !GetAtt WebServerInstance.PublicIp
       Description: URL of the web server
   ```

4. **Save Your Changes:**  
   - Save the updated template to preserve parameters, mappings, and outputs.

---

### Step 3: Specify Resource Properties

> **Note:** Now you will define configuration details for each resource using the integrated JSON/YAML editor.

1. **Configure the VPC:**  
   - Select the `VPC` resource and add the following properties:
     - `CidrBlock: 10.0.0.0/16`
     - `EnableDnsSupport: true`
     - `EnableDnsHostnames: true`

2. **Configure the Public Subnet:**  
   - Select `PublicSubnet` and add:
     - `VpcId: !Ref VPC`
     - `CidrBlock: 10.0.0.0/24`

3. **Configure the Public Route:**  
   - Select `PublicRoute` and add:
     - `DestinationCidrBlock: 0.0.0.0/0`
     - `RouteTableId: !Ref PublicRouteTable`
     - `GatewayId: !Ref InternetGateway`

4. **Configure the Security Group:**  
   - Select `WebServerSecurityGroup` and add:
     - `VpcId: !Ref VPC`
     - `GroupDescription: Allow HTTP and SSH access`
     - **SecurityGroupIngress:**  
       - Allow HTTP (port 80) from `0.0.0.0/0`  
       - Allow SSH (port 22) from the `SSHLocation` parameter

5. **Configure the EC2 Instance:**  
   - Select `WebServerInstance` and set the following properties:
     - `InstanceType: !Ref InstanceType`
     - `ImageId: !FindInMap [ AWSRegionArch2AMI, !Ref 'AWS::Region', !FindInMap [ AWSInstanceType2Arch, !Ref InstanceType, Arch ] ]`
     - `KeyName: !Ref KeyName`
     - **NetworkInterfaces:**  
       - Associate with the `PublicSubnet`  
       - Set `AssociatePublicIpAddress` to true  
       - Reference `WebServerSecurityGroup` in `GroupSet`
     - **UserData:**  
       - Base64-encode a script that installs and starts Apache HTTP Server. For example:
         ```bash
         #!/bin/bash -xe
         yum update -y
         yum install -y httpd
         systemctl start httpd
         systemctl enable httpd
         echo "<h1>Congratulations, your web server is running!</h1>" > /var/www/html/index.html
         ```
6. **Add EC2 Metadata for Initialization:**  
   - Under the `Metadata` tab for `WebServerInstance`, add an `AWS::CloudFormation::Init` section that instructs the instance (via cfn-init) to configure the web server and create a simple web page.

7. **Validate and Save:**  
   - Use the **Validate Template** button (check icon) to ensure there are no syntax errors. Save your changes.

---

### Step 4: Provision Resources

> **Note:** Your template is now complete. It’s time to launch the CloudFormation stack and provision your web server.

1. **Create the Stack:**  
   - From the CloudFormation Designer toolbar, click the **Create Stack** (cloud icon).  
   - CloudFormation Designer will save your template to an S3 bucket and open the Create Stack Wizard.

2. **Specify Stack Details:**  
   - Enter a stack name, for example, `BasicWebServerStack`.
   - In the Parameters section, provide values for `InstanceType`, `KeyName`, and `SSHLocation` (ensure you enter a valid EC2 key pair and your CIDR for SSH).

3. **Review and Create:**  
   - Review the stack details and click **Create**.
   - Monitor the stack creation progress in the CloudFormation console’s Events tab.

4. **Test the Deployment:**  
   - Once the stack status is `CREATE_COMPLETE`, go to the **Outputs** tab.
   - Copy the URL from the `WebsiteURL` output and open it in your browser to verify that your web server is running.

---

## Additional Enhancements

- **Enable Logging & Monitoring:**  
  Use AWS CloudWatch to monitor EC2 logs and performance.
- **Implement Auto Scaling:**  
  Modify the template to replace the single instance with an Auto Scaling group for higher availability.
- **Improve Security:**  
  Refine security group rules and restrict SSH access further.

---

## Estimated Project AWS Costs

For a small-scale deployment (using a single t2.small instance in the free tier region), monthly costs should be minimal – generally under a few dollars per month if you exceed free tier limits. Always review pricing details on [aws.amazon.com](http://aws.amazon.com).

---

## Resources

- [AWS CloudFormation User Guide](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/) cite1†AWS CloudFormation  
- [AWS CloudFormation Designer Walkthrough](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/working-with-templates-cfn-designer-walkthrough-createbasicwebserver.html) cite0†Documentation  
- [Amazon EC2 Key Pairs Documentation](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/ec2-key-pairs.html) cite10†Amazon EC2 key pairs  

---

## Author

Created by **Tyree** – Feel free to contribute or ask questions!

---

# End Of Project

