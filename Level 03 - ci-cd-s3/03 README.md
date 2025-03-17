Below is the revised version with corrections to the markdown formatting (notably, the bold/italic markers in the AWS Services list have been fixed):

---

# CI/CD for a Secure Serverless Static Website on AWS

This repository showcases AWS projects and lessons learned while deploying a secure static website using multiple AWS services. In this project, we integrate a CI/CD pipeline to automate the build, test, and deployment processes—ensuring that changes to your website are rapidly and reliably released.

## AWS Services Used

- ***AWS CodePipeline*** – Orchestrates the CI/CD workflow  
- ***AWS CodeBuild*** – Builds and tests the application  
- ***GitHub*** – Hosts the source code repository  
- ***AWS Lambda*** – (Optional) Performs CloudFront cache invalidation post-deployment  
- **Amazon S3** – Stores static website files  
- **Amazon CloudFront** – Distributes content globally with low latency  
- **AWS Certificate Manager (ACM)** – Manages SSL/TLS certificates for HTTPS  
- **Amazon Route 53** – Manages the domain name  

---

## AWS Architecture Diagram

![AWS Architecture Diagram](Resources/Images/AWS_CICD_Architecture_Diagram.png)

*(Diagram includes a pipeline triggering a build in CodeBuild, storing artifacts in S3, optionally invalidating CloudFront caches via Lambda, etc.)*

---

### Prerequisites

- A secure static website hosted on **Amazon S3**, distributed via **CloudFront**, with domain management through **Route 53** and SSL/TLS certificates from **ACM**. See **Level 02** for initial setup details.
- A **GitHub** account and repository to store your website’s source code.

---

## Setting up GitHub

### Step 1: Set Up the GitHub Repository

1. **Create a Repository**:  
   - On GitHub, create a new repository for your project.  
   - Initialize the repository (optionally with a README) and note the clone URL (HTTPS or SSH).

2. **Clone and Populate the Repository**:  
   - Using Git or GitHub Desktop, clone the repo locally.
   - Add your static website files (HTML, CSS, JS, images, etc.) to the repository.
   - Commit and push changes to GitHub.

### Step 2: Configure the CI/CD Pipeline with GitHub

1. **Create a Pipeline** in **AWS CodePipeline**:  
   - In the AWS Console, go to **CodePipeline** → **Create pipeline**.
   - Choose **Build custom Pipeline**.
   - Provide a pipeline name (e.g., `MyStaticSitePipeline`).
   - Use or create a new **Service Role** if prompted.
   - Specify your own the S3 bucket for pipeline artifacts by choosing **"Custom Location"**
   > **Note:** This is the S3 Bucket created in **LEVEL 02** that contains the html file,
   - Click **Next**.

2. **Source Stage**:  
   - **Source provider**: Select **GitHub** (via the GitHub App connection).
   - **Connect to GitHub**: Follow prompts to install/authorize the GitHub App and connect your repository.
   - **Repository Name and Branch**: Choose your repo and the branch (e.g., `main`) that triggers the pipeline.
   - Click **Next**.

3. **Add Build and Deploy Stages**:

   #### 3.1 Build Stage (AWS CodeBuild)

   1. **Action Provider**:  
      - Under “Build provider,” choose **Other build providers**.  
      - Select **AWS CodeBuild**.

   2. **Project Creation**:  
      - Choose your existing CodeBuild project if you have one, or click **Create project** to set one up and name it (ex. `MyStaticSiteBuild`).

   3. **Buildspec Instructions**:  
      - In the **CodeBuild** console, under **Buildspec**, select **Insert Build commands** then choose **Switch to editor**.  
      - **Copy and paste** the below Buildspec code into the Build Commands:
   >**IMPORTANT:** For the code below, if you have multiple html files in one Repository, instead of the line "`- '**/*' `" instead change it to the file path where your html file and resource are located. (ex. `- 'Level 02 - static-website-s3/Resources/**'`)
      ```yaml
      version: 0.2

      phases:
        install:
          commands:
            - echo "No dependencies to install for a plain static site."
        build:
          commands:
            - echo "No build steps needed for a plain static site."
        post_build:
          commands:
            - echo "Build phase complete."

      artifacts:
        files:
          - '**/*' 
        discard-paths: yes  
      ```

      This instructs CodeBuild to simply pass all files through without transformation.

   4. **Save the Project**:  
      - After configuring everything in the pipeline wizard, click **Create build project**.

   5. **Input Artifacts**:  
      - Select the output artifact from the Source stage (typically `SourceArtifact`).

   6. **Click Next** to proceed and you also **Skip** the testing portion if you like.

---

   #### 3.2 Deploy Stage (S3 Deployment)

   1. **Action Provider**: Choose **Amazon S3**.  
   2. **Input Artifact**: Set this to `BuildArtifact` (the output from the Build stage).  
   3. **Bucket**: Select your S3 bucket that hosts the static site.  
   4. **S3 Object Key**: Provide a name ending with `.zip` (ex. `MyStaticSite.zip`).  
      > **Note**: This ensures CodePipeline has a location to upload the build artifact. It will look something like: `s3://mydomain.org/MyStaticSite.zip`.

   5. **Click Next** and proceed.

4. **(Optional) Lambda Cache Invalidation**:  
   - Configure a Lambda function to perform **CloudFront** cache invalidation after deployment.  
   - Add this function as an action in your pipeline to ensure the cache is updated whenever new files are deployed.

5. **Review and Create**:  
   - Confirm your pipeline configuration and click **Create pipeline**.  
   - The pipeline will attempt a first run, so ensure your repo has valid files.

---



FIGURE OUT WHY WHEN I PUSH GITHUB, MY STATIC WEBSITE UPDATE IN PIPELINE, BUT DOES NOT OVERWRITE THE HTML FILE IN THE S3 BUCKET THAT CONTAINS MY STATIC WEBSITE