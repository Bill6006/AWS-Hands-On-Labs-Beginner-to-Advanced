# CI/CD for a Secure Serverless Static Website on AWS

This repository showcases my AWS projects and lessons learned while deploying a secure static website using multiple AWS services. In this project, we integrate a CI/CD pipeline to automate the build, test, and deployment processes, ensuring that changes to your website are rapidly and reliably released.

## AWS Services Used

- ***AWS CodePipeline** – Orchestrates the CI/CD workflow
- ***AWS CodeBuild** – Builds and tests the application
- ***AWS CodeCommit / GitHub** – Hosts the source code repository
- ***AWS Lambda** – Performs CloudFront cache invalidation post-deployment
- **Amazon S3** – Stores static website files
- **Amazon CloudFront** – Distributes content globally with low latency
- **AWS Certificate Manager (ACM)** – Manages SSL/TLS certificates for HTTPS
- **Amazon Route 53** – Manages the domain name

---

## AWS Architecture Diagram

![AWS Architecture Diagram](Resources/Images/AWS_CICD_Architecture_Diagram.png)

---

## Deployment Steps

### Prerequisites

- A secure static website hosted on Amazon S3, distributed via CloudFront, with domain management through Route 53 and SSL/TLS certificates from ACM. See **Level 02** for initial setup details.

---

## Option 1: Using AWS CodeCommit

### Step 1: Set Up the CodeCommit Repository

1. **Create a Repository**:  
   - Navigate to the AWS CodeCommit console and click **Create repository**.
   - Provide a unique repository name and description.
   - Note the repository’s clone URL (HTTPS or SSH) for connecting your local environment.

2. **Configure Your Local Environment**:  
   - **Git Credentials**: Configure your Git credentials for HTTPS (via the IAM console) or set up SSH keys.
   - **Clone the Repository**:
     ```bash
     git clone https://git-codecommit.<region>.amazonaws.com/v1/repos/YourRepositoryName
     ```
     Replace `<region>` and `YourRepositoryName` with your specific details.

3. **Add Website Code and Configuration**:  
   - Place your static website files and `buildspec.yml` file at the root of the repository.
   - Commit and push the changes:
     ```bash
     git add .
     git commit -m "Initial commit of static website and CI/CD configuration"
     git push origin main
     ```

### Step 2: Configure the CI/CD Pipeline with CodeCommit

1. **Create a Pipeline**:
   - In AWS CodePipeline, click **Create pipeline**.
   - Configure your pipeline name, service role, and artifact store (S3 bucket).

2. **Add the Source Stage**:  
   - **Source Provider**: Select **AWS CodeCommit**.
   - **Repository Name**: Choose your CodeCommit repository.
   - **Branch**: Specify the branch (e.g., `main`) that triggers the pipeline.

3. **Add Build and Deploy Stages**:  
   - **Build Stage**: Use AWS CodeBuild with your project configured to use the `buildspec.yml`.
   - **Deploy Stage**: Configure deployment to your S3 bucket hosting the static website.

4. **(Optional) Lambda Cache Invalidation**:  
   - Create and configure a Lambda function to perform CloudFront cache invalidation post-deployment.
   - Integrate this function into your pipeline as a post-deployment action.

---

## Option 2: Using GitHub

### Step 1: Set Up the GitHub Repository

1. **Create a Repository**:  
   - On GitHub, create a new repository for your project.
   - Initialize the repository (optionally with a README) and note the clone URL (HTTPS or SSH).

2. **Clone and Populate the Repository**:  
   - Following the instructions within Git to install and configure GitHub Desktop application. 

### Step 2: Configure the CI/CD Pipeline with GitHub

1. **Create a Pipeline**:
   - In AWS CodePipeline, click **Create pipeline**.
   - Set up your pipeline with a name, service role, and S3 artifact store.

2. **Add the Source Stage**:  
   - **Source Provider**: Select **GitHub**.
   - **Repository Name and Branch**: Connect your GitHub account and choose the repository and branch (e.g., `main`) that will trigger the pipeline.

3. **Add Build and Deploy Stages**:  
   - **Build Stage**: Use AWS CodeBuild configured to use the `buildspec.yml` file from your repository.
   - **Deploy Stage**: Configure deployment to your S3 bucket hosting the static website.

4. **(Optional) Lambda Cache Invalidation**:  
   - Configure a Lambda function to perform CloudFront cache invalidation after deployment.
   - Add this function as an action in your pipeline to ensure cache is updated.

---

## Additional Deployment Steps (Common for Both Options)

### Step 1: Set Up a Source Code Repository

1. **Choose and Initialize a Repository**:  
   - Create a new repository on your preferred platform.
   - Push your website’s source files to the repository.
2. **Define Trigger Conditions**:  
   - The CI/CD pipeline will automatically trigger on commits to a specific branch (e.g., `main`).

### Step 2: Configure AWS CodeBuild

1. **Create a Build Specification File** (`buildspec.yml`):  
   Define your build phases and commands. For example:
   ```yaml
   version: 0.2
   phases:
     install:
       commands:
         - echo Installing dependencies...
         # Install dependencies here if required
     build:
       commands:
         - echo Build started on `date`
         # Add any build commands if necessary
     post_build:
       commands:
         - echo Build completed on `date`
   artifacts:
     files:
       - '**/*'
   ```
   - Place the `buildspec.yml` at the root of your repository.

2. **Create a CodeBuild Project**:  
   - Navigate to the AWS CodeBuild console and click **Create build project**.
   - Configure your project:
     - **Project Name**: Enter a unique name.
     - **Source Provider**: Select your repository.
     - **Environment**: Choose the runtime and build image.
     - **Buildspec**: Ensure the project uses your `buildspec.yml`.

### Step 3: Set Up AWS CodePipeline

1. **Create a New Pipeline**:  
   - In the AWS CodePipeline console, click **Create pipeline**.
   - Configure:
     - **Pipeline Name**: Provide a unique name.
     - **Service Role**: Use an existing role or create a new one.
     - **Artifact Store**: Use the default S3 bucket for artifacts.

2. **Add Source Stage**:  
   - **Source Provider**: Select your repository.
   - **Repository Name and Branch**: Specify the branch (e.g., `main`) that triggers the pipeline.

3. **Add Build Stage**:  
   - **Build Provider**: Choose AWS CodeBuild.
   - **Project Name**: Select your CodeBuild project.

4. **Add Deploy Stage**:  
   - **Deploy Provider**: Choose Amazon S3.
   - **Bucket Name**: Select the S3 bucket hosting your static website.
   - **Extract File Before Deploying**: Ensure this option is selected.

5. **(Optional) Add Manual Approval**:  
   - If desired, insert a manual approval action before the deploy stage to review builds before deployment.

6. **Review and Create Pipeline**:  
   - Confirm all configurations and click **Create pipeline**.

### Step 4: Configure CloudFront Cache Invalidation

1. **Create a Lambda Function** for Invalidation:
   - In the AWS Lambda console, create a new function.
   - **Function Name**: Choose a unique name.
   - **Runtime**: Use Python 3.x (or your preferred language).
   - **Permissions**: Ensure the Lambda’s IAM role has permissions for CloudFront invalidation (i.e., the `cloudfront:CreateInvalidation` action).

2. **Add Invalidation Logic**:  
   Example in Python:
   ```python
   import boto3
   import os

   def lambda_handler(event, context):
       cloudfront = boto3.client('cloudfront')
       distribution_id = os.environ['DISTRIBUTION_ID']
       response = cloudfront.create_invalidation(
           DistributionId=distribution_id,
           InvalidationBatch={
               'Paths': {
                   'Quantity': 1,
                   'Items': ['/*']
               },
               'CallerReference': str(context.aws_request_id)
           }
       )
       return response
   ```
   - Set an environment variable `DISTRIBUTION_ID` with your CloudFront distribution ID.

3. **Integrate Lambda in CodePipeline**:  
   - Edit your CodePipeline and add a new Lambda action after the deploy stage.
   - **Action Name**: e.g., `InvalidateCache`.
   - **Function Name**: Choose your Lambda function.
   - **Input Artifacts**: Link to the build artifact if needed.

### Step 5: Rollback, Error Handling & Resource Cleanup

1. **Rollback Procedures**:  
   - Document procedures to revert to a previous commit if a deployment fails.
   - Use CodePipeline’s history and AWS CloudWatch logs to identify and troubleshoot issues.

2. **Error Handling in the Pipeline**:  
   - Configure notifications (using AWS SNS or similar) for pipeline failures.
   - Monitor pipeline stages and set up alarms for build or deploy failures.

3. **Environment Variables and Secrets Management**:  
   - Use AWS Systems Manager Parameter Store or AWS Secrets Manager to securely manage environment variables and sensitive data.
   - Ensure these values are not hard-coded in your repository or buildspec.

4. **Resource Cleanup**:  
   - For development or testing environments, include steps to clean up or disable unused resources (CodePipeline, CodeBuild projects, etc.) to prevent unexpected AWS charges.

### Step 6: Test the CI/CD Pipeline

1. **Commit Changes**:  
   - Make an update to your website's source code.
   - Push the changes to the monitored branch (e.g., `main`).

2. **Monitor Pipeline Execution**:  
   - In the CodePipeline console, track the progress of each stage.
   - Verify that the build, deploy, and Lambda invalidation stages complete successfully.

3. **Validate Deployment**:  
   - Open your domain in a browser to confirm that changes are live.
   - Verify HTTPS functionality and inspect CloudFront caching behavior.
   - Remember that DNS and CloudFront cache propagation may take a few minutes to 48 hours.

---

## Additional Enhancements

- **Automated Testing**:  
  Integrate automated tests into the CodeBuild phase to ensure code quality before deployment.

- **Detailed Notifications**:  
  Set up detailed notifications for pipeline events to keep the team informed about deployments, failures, or rollbacks.

- **Monitoring and Logging**:  
  Leverage AWS CloudWatch to monitor pipeline performance and logs for troubleshooting issues.

- **Propagation Considerations**:  
  Clarify that some changes (e.g., CloudFront or DNS updates) might take time to propagate. Inform users to be patient if updates are not immediately visible.

---

## Estimated Project AWS Costs

Implementing this CI/CD pipeline will add costs beyond static website hosting. Below is a rough cost breakdown:

- **AWS CodePipeline**:  
  - Pricing: Approximately $1 per active pipeline per month.
  
- **AWS CodeBuild**:  
  - Pricing: Costs depend on build time and compute resources. For low-traffic static websites, expect around $1–$3 per month.
  
- **Lambda for Invalidation**:  
  - Minimal cost for infrequent CloudFront cache invalidations.
  
- **Amazon S3, CloudFront, ACM & Route 53**:  
  - Costs remain similar to the static website hosting setup (roughly $1–$3 per month, depending on usage).

Using the AWS Free Tier can help further reduce these costs during initial development.

---

## Resources

- [AWS S3 Static Website Hosting Documentation](https://docs.aws.amazon.com/AmazonS3/latest/userguide/WebsiteHosting.html)
- [Amazon CloudFront User Guide](https://docs.aws.amazon.com/AmazonCloudFront/latest/DeveloperGuide/Introduction.html)
- [AWS Route 53 Documentation](https://docs.aws.amazon.com/Route53/latest/DeveloperGuide/Welcome.html)
- [AWS CodePipeline Documentation](https://docs.aws.amazon.com/codepipeline/latest/userguide/welcome.html)
- [AWS CodeBuild Documentation](https://docs.aws.amazon.com/codebuild/latest/userguide/welcome.html)
- [AWS Lambda Permissions Documentation](https://docs.aws.amazon.com/lambda/latest/dg/lambda-intro-permission-model.html)
- [AWS Secrets Manager Documentation](https://docs.aws.amazon.com/secretsmanager/latest/userguide/intro.html)

---

# End Of Project

This enhanced CI/CD guide is designed to offer a robust and secure method to deploy and manage your serverless static website. With automated builds, thorough testing, cache management, and rollback procedures, you can confidently push updates knowing that your deployment process is resilient and well-monitored.

Feel free to contribute additional improvements or modifications as needed.

---