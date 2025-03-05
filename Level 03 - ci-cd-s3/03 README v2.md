# Add a CI/CD Pipeline to an Amazon S3 Bucket

This project sets up an automated continuous integration and continuous delivery (CI/CD) pipeline that deploys updates to a static website hosted on Amazon S3. Every time you check in code to GitHub, the pipeline automatically deploys your website changes.

## Overview

Modern development practices rely on automation to reduce manual steps and the risk of errors. In this project, you will create a pipeline that:
- Monitors your GitHub repository for changes.
- Automatically deploys updated website code to an existing S3 bucket configured for static website hosting.
- (Optionally) Uses AWS CodeStar for enhanced project management.

## Services Used

- **Amazon S3:** Hosts your static website.
- **AWS CodePipeline:** Orchestrates the CI/CD process.
- **AWS CodeStar:** (Optional) Provides an integrated interface for project management.
- **GitHub:** Source repository for your website code.

## Prerequisites

Before you begin, ensure you have:
- A **static website** (HTML, CSS, JavaScript, etc.) ready.
- Your website code checked into a **GitHub repository**.
- An **Amazon S3 bucket** set up and configured for static website hosting.
- An AWS account with permissions to create and manage CodePipeline, S3, and (optionally) CodeStar resources.

## Setup Instructions

Follow these steps to create your CI/CD pipeline:

### 1. Prepare Your Static Website
- Ensure your website files are complete.
- Push your website code to your GitHub repository.

### 2. Configure Your Amazon S3 Bucket
- Create or use an existing S3 bucket.
- Enable static website hosting on the bucket.
- Set up the appropriate bucket policy to allow public read access (if needed).

### 3. Create the CI/CD Pipeline in AWS CodePipeline
1. **Open AWS CodePipeline in the AWS Management Console.**
2. **Create a New Pipeline:**
   - **Pipeline Name:** Choose a descriptive name (e.g., `S3StaticWebsitePipeline`).
   - **Service Role:** Create a new service role or use an existing role with the necessary permissions.
3. **Add the Source Stage:**
   - **Source Provider:** Select **GitHub**.
   - **Repository & Branch:** Connect to your GitHub repository and choose the branch you want to monitor.
4. **Add the Build Stage (Optional):**
   - If your static website requires a build step (for example, if using a framework that needs compiling), add a CodeBuild action.  
   - If not, simply choose to skip the build stage.
5. **Add the Deploy Stage:**
   - **Deploy Provider:** Select **Amazon S3**.
   - **Bucket:** Enter the name of your S3 bucket.
   - **Extract File Before Deploy:** Enable this option if your source artifact is a ZIP file.
6. **Review and Create:** Verify your pipeline settings and create the pipeline.

### 4. Test Your Pipeline
- Make a change to your website code in GitHub.
- Commit and push the change.
- Monitor CodePipeline to confirm that it picks up the change and successfully deploys the update to your S3 bucket.
- Verify the live website by accessing the S3 website endpoint.

## References

- [AWS Prescriptive Guidance Patterns](https://docs.aws.amazon.com/prescriptive-guidance/latest/patterns/welcome.html) :contentReference[oaicite:1]{index=1}
- [Automating CI/CD With AWS CodePipeline](https://www.pluralsight.com/resources/blog/cloud/automating-ci-cd-with-aws-codepipeline) :contentReference[oaicite:2]{index=2}


