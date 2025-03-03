How to Deploy a Sample Application on Amazon EKS

This guide walks you through deploying a sample application on Amazon Elastic Kubernetes Service (EKS), providing hands-on experience with Docker, Kubernetes, and SQL databases. By following these steps, you'll enhance your qualifications for an AWS Cloud Engineer role through practical, real-world scenarios.


---

AWS Services Used

Amazon EKS (Elastic Kubernetes Service): Managed Kubernetes service to run Kubernetes without needing to install and operate your own control plane or nodes.

Amazon RDS (Relational Database Service): Managed relational database service for SQL databases.

Amazon EC2 (Elastic Compute Cloud): Scalable virtual servers for running applications.

Amazon VPC (Virtual Private Cloud): Isolated network environment to launch AWS resources.

AWS IAM (Identity and Access Management): Manage access to AWS services and resources securely.



---

AWS Architecture Diagram




---

Deployment Steps

Step 1: Set Up Your AWS Environment

1. Create an AWS Account:

If you don't have one, sign up at AWS.



2. Install AWS CLI:

Download and install the AWS Command Line Interface from the AWS CLI User Guide.



3. Configure AWS CLI:

Run aws configure and provide your AWS Access Key, Secret Key, region, and output format.



4. Install kubectl:

Follow the instructions in the Amazon EKS User Guide to install the Kubernetes command-line tool.



5. Install eksctl:

Install eksctl, a simple command-line utility for creating and managing Kubernetes clusters on EKS, as described in the eksctl documentation.




Step 2: Create an Amazon EKS Cluster

1. Create a Cluster:

Run the following command to create a cluster named my-cluster:

eksctl create cluster --name my-cluster --region us-west-2 --nodegroup-name linux-nodes --node-type t2.micro --nodes 3

This command sets up a VPC, subnets, and the EKS control plane.



2. Verify Cluster:

Update your kubeconfig:

aws eks --region us-west-2 update-kubeconfig --name my-cluster

Verify the cluster is running:

kubectl get nodes




Step 3: Deploy a Sample Application

1. Clone the Sample Application Repository:

Clone the AWS Containers Retail Sample repository:

git clone https://github.com/aws-samples/aws-retail-store-sample

Navigate to the directory:

cd aws-retail-store-sample



2. Deploy the Application:

Apply the Kubernetes manifests:

kubectl apply -f kubernetes-manifests/

Verify the pods are running:

kubectl get pods



3. Expose the Application:

Create a LoadBalancer service to expose the application:

kubectl expose deployment retail-store --type=LoadBalancer --name=retail-store-service

Retrieve the external IP:

kubectl get services retail-store-service

Access the application using the external IP in your browser.




Step 4: Set Up Amazon RDS for SQL Database

1. Create an RDS Instance:

Navigate to the RDS console and create a new PostgreSQL instance.

Configure the instance with appropriate settings (e.g., db.t2.micro, publicly accessible).



2. Configure Security Groups:

Ensure the RDS security group allows inbound traffic from your EKS cluster's security group on the PostgreSQL port (default 5432).



3. Initialize the Database:

Connect to the RDS instance using a SQL client and run the provided SQL scripts to set up the database schema and seed data.




Step 5: Integrate the Application with RDS

1. Update Application Configuration:

Modify the application's Kubernetes ConfigMap or Secrets to include the RDS endpoint, database name, username, and password.



2. Redeploy the Application:

Apply the updated configuration:

kubectl apply -f kubernetes-manifests/

Verify the application connects to the RDS database successfully.




Step 6: Monitor and Trace SQL Queries

1. Enable Logging:

Configure PostgreSQL to log queries by modifying the log_statement parameter to all.



2. Set Up CloudWatch Logs:

Install the CloudWatch Agent on your EKS nodes to collect application and database logs.

Configure the agent to stream logs to Amazon CloudWatch.



3. Analyze Logs:

Use CloudWatch Logs Insights to query and analyze SQL logs for performance issues or errors.





---

Additional Enhancements

Implement CI/CD Pipelines:

Use AWS CodePipeline and CodeBuild to automate application deployments.


Set Up Auto Scaling:

Configure Kubernetes Horizontal Pod Autoscaler (HPA) to scale application pods based on CPU or memory usage.


Enhance Security:

Implement AWS IAM Roles for Service Accounts (IRSA) to manage permissions for your pods securely.




---

Estimated Project AWS Costs

Amazon EKS: ~$0.10 per hour for the EKS control plane.

Amazon EC2: Costs vary based on instance types and usage.

Amazon RDS: Costs depend on the instance class and storage.

Additional Services: Costs for CloudWatch, VPC, and data transfer may apply.


For detailed pricing, refer to the AWS Pricing Calculator.


---

Resources

Amazon EKS User Guide

Amazon RDS Documentation

Kubernetes Documentation

AWS Containers Retail Sample



---

Author

Created by [Your Name] – Feel free to contribute or ask questions!


---

End Of Project

