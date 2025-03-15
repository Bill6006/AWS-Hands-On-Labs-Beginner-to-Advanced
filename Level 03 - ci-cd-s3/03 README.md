# Deploying a Static Website to Amazon S3 with AWS CodeStar and CodePipeline

This project demonstrates how to set up a continuous integration and continuous delivery (CI/CD) pipeline for a static website hosted on Amazon S3, utilizing AWS CodeStar for project management and AWS CodePipeline for automation.

## Overview

By integrating AWS CodeStar, you gain a unified project management interface that simplifies the setup and monitoring of your CI/CD pipeline. The workflow includes:

- **AWS CodeStar:** Provides a centralized dashboard to manage your project's resources, including CodePipeline, CodeCommit, and S3.
- **AWS CodePipeline:** Automates the process of fetching code from the repository and deploying it to the S3 bucket.
- **Amazon S3:** Serves as the hosting platform for your static website.
- **AWS CodeCommit or GitHub:** Acts as the source repository for your website's code.

## Prerequisites

Before starting, ensure you have:

- An **AWS account** with permissions to create and manage CodeStar, CodePipeline, S3, and CodeCommit resources.
- A **static website** (HTML, CSS, JavaScript files) ready for deployment.
- If using GitHub, a **GitHub account** with a repository containing your website's code.
- If using CodeCommit, your website's code should be in a **local Git repository** ready for pushing to CodeCommit.

## Setup Instructions

Follow these steps to set up your CI/CD pipeline with AWS CodeStar:

### 1. Create an S3 Bucket for Static Website Hosting

1. **Sign in to the AWS Management Console** and open the S3 console.
2. **Create a new S3 bucket**:
   - Click on **"Create bucket"**.
   - Enter a unique bucket name (e.g., `my-static-website-bucket`).
   - Choose the AWS Region where you want to create the bucket.
   - Click **"Create bucket"**.
3. **Enable static website hosting**:
   - Select your newly created bucket.
   - Go to the **"Properties"** tab.
   - Scroll down to **"Static website hosting"** and click **"Edit"**.
   - Choose **"Enable"**.
   - Specify the **index document** (e.g., `index.html`).
   - Specify the **error document** if applicable.
   - Click **"Save changes"**.

### 2. Set Up AWS CodeStar Project

1. **Navigate to the AWS CodeStar console**.
2. Click on **"Create project"**.
3. **Select a project template**:
   - Choose **"Web application"**.
   - Select **"Static Website"** as the project type.
4. **Configure the project**:
   - **Project Name:** Enter a unique name for your project.
   - **Repository:** Choose between **AWS CodeCommit** or **GitHub**:
     - For **CodeCommit**:
       - AWS CodeStar will create a new CodeCommit repository for you.
     - For **GitHub**:
       - You'll need to connect your GitHub account and select the repository containing your website's code.
5. **Set Up the Project**:
   - AWS CodeStar will automatically set up the necessary resources, including CodePipeline and the repository.
   - It will also configure IAM roles and permissions required for the services to interact.

### 3. Configure the CI/CD Pipeline

1. **Access the CodePipeline setup**:
   - Within your CodeStar project dashboard, locate the **"CI/CD pipeline"** section and click on the provided link to open CodePipeline.
2. **Verify Source Stage**:
   - Ensure that the source stage is correctly set to your chosen repository (CodeCommit or GitHub) and the appropriate branch.
3. **Configure Build Stage (Optional)**:
   - If your static website requires a build process (e.g., using a static site generator or bundler), add a build stage using AWS CodeBuild.
   - If no build is necessary, you can skip this stage.
4. **Set Up Deploy Stage**:
   - **Deploy Provider:** Select **"Amazon S3"**.
   - **Bucket Name:** Enter the name of the S3 bucket you created for static website hosting.
   - **Extract File Before Deploy:** Ensure this option is checked if your source artifacts are zipped.

### 4. Deploy Your Website

1. **Push Code to Repository**:
   - If using CodeCommit:
     - Clone the repository to your local machine.
     - Add your static website files to the repository.
     - Commit and push the changes.
   - If using GitHub:
     - Ensure your static website files are committed and pushed to the selected repository.
2. **Trigger the Pipeline**:
   - Pushing changes to the repository will automatically trigger CodePipeline.
   - Monitor the pipeline's progress through the AWS CodeStar dashboard or the CodePipeline console.
3. **Verify Deployment**:
   - Once the pipeline completes successfully, your static website should be live.
   - Access your website using the S3 static website endpoint (e.g., `http://my-static-website-bucket.s3-website-us-east-1.amazonaws.com`).

## References

- [AWS CodeStar Documentation](https://docs.aws.amazon.com/codestar/latest/userguide/welcome.html)
- [AWS CodePipeline User Guide](https://docs.aws.amazon.com/codepipeline/latest/userguide/welcome.html)
- [Hosting a Static Website on Amazon S3](https://docs.aws.amazon.com/AmazonS3/latest/dev/WebsiteHosting.html)

## License

This project is licensed under the MIT License.
