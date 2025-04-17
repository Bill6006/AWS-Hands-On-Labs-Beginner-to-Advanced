# How to Deploy a Containerized Application with AWS EKS (Kubernetes-Focused)

This guide helps you create an AWS EKS cluster and deploy a containerized application using Kubernetes. You’ll learn how to create a cluster, deploy your application with a Kubernetes deployment, expose it via a LoadBalancer service, and monitor it using AWS CloudWatch. This version is designed for beginners to get hands-on experience with Kubernetes on AWS without the additional complexity of Docker image creation.

---

## AWS Services Used

- **Amazon EKS** – Managed Kubernetes service to run containerized applications  
- **AWS EC2** – Underlying compute for EKS worker nodes  
- **Amazon VPC** – Virtual network automatically created for your cluster  
- **AWS CloudWatch** – Monitor logs and performance of your application  
- **AWS IAM** – Manage permissions and security for cluster operations

---

## AWS Architecture Diagram

*(Put a AWS-EKS architecture image here.)*

---

## Deployment Steps

### Step 1: Prerequisites and Cluster Setup

> **Note:** Before you begin, ensure you have installed the following:
>
> - **AWS CLI:** 
> - **eksctl:** 
> **Note:** eksctl is a command-line tool for creating and managing clusters on Amazon EKS
> - **kubectl:**.

1. **Create Your EKS Cluster:**  
   Open your terminal and run:
   ```bash
   eksctl create cluster --name demo-cluster --region us-east-1 --nodes 2
   ```
   - **What This Does:**  
     - **Amazon VPC & Subnets:** Automatically creates a new VPC along with necessary subnets.
     - **EC2 Instances:** Provisions two worker nodes (Instances) that will run your containerized application.

2. **Verify Your Cluster:**  
   Run:
   ```bash
   kubectl get nodes
   ```
   You should see the nodes listed, confirming that your cluster is up and running.

---

### Step 2: Deploying Your Application

> **Tip:** We’re deploying a simple web application using a pre-built container image (NGINX) that’s publicly available. This avoids Docker image creation, letting you focus on the Kubernetes side.

1. **Create a Deployment Manifest:**  
   Create a file named `deployment.yaml` with the following content:
   ```yaml
   apiVersion: apps/v1
   kind: Deployment
   metadata:
     name: sample-app
   spec:
     replicas: 2
     selector:
       matchLabels:
         app: sample-app
     template:
       metadata:
         labels:
           app: sample-app
       spec:
         containers:
         - name: sample-app
           image: public.ecr.aws/nginx/nginx:latest
           ports:
           - containerPort: 80
   ```
   - **Explanation:**  
     This manifest defines a deployment that runs two replicas of a container using the official NGINX image, exposing port 80 in each pod.

2. **Deploy the Application:**  
   Run:
   ```bash
   kubectl apply -f deployment.yaml
   ```

3. **Verify the Deployment:**  
   Check the status by running:
   ```bash
   kubectl get deployments
   ```
   Confirm that the `sample-app` deployment is running with the desired number of replicas by running:
      ```bash
   kubectl get deployment sample-app
      ```
---

### Step 3: Exposing the Application

> **Note:** To access your application from the internet, create a Kubernetes Service that exposes it externally.

1. **Create a Service Manifest:**  
   Create a file named `service.yaml` with the following content:
   ```yaml
   apiVersion: v1
   kind: Service
   metadata:
     name: sample-app-service
   spec:
     type: LoadBalancer
     selector:
       app: sample-app
     ports:
     - port: 80
       targetPort: 80
   ```
   - **Explanation:**  
     This manifest creates a Kubernetes Service of type LoadBalancer that routes incoming traffic on port 80 to your application pods.

2. **Deploy the Service:**  
   Run:
   ```bash
   kubectl apply -f service.yaml
   ```

3. **Access Your Application:**  
   Check the service details with:
   ```bash
   kubectl get services
   ```
   When an `EXTERNAL-IP` appears for `sample-app-service`, open your browser and navigate to that IP address.

---

### Step 4: Monitoring and Tracing

> **Tip:** Effective monitoring helps you quickly troubleshoot and maintain your application.

1. **View Application Logs:**  
   Access logs from your pods by running:
   ```bash
   kubectl logs <pod-name>
   ```
   Replace `<pod-name>` with the actual pod name.

2. **Enable CloudWatch Container Insights:**  
   Follow the [AWS EKS Workshop instructions](https://eksworkshop.com/) to enable CloudWatch Container Insights, which collects metrics and logs from your cluster.

3. **(Optional) Set Up Distributed Tracing:**  
   If you want to trace requests across services, consider integrating AWS X-Ray or a similar tool for deeper insights into your application’s behavior.

---

## Additional Enhancements

- **Auto Scaling:**  
  Configure a Horizontal Pod Autoscaler (HPA) to automatically adjust the number of pods based on your traffic load.
- **Rolling Updates:**  
  Leverage Kubernetes rolling updates to deploy new versions of your application with zero downtime.
- **CI/CD Integration:**  
  Integrate with AWS CodePipeline and CodeBuild for continuous deployment and integration.

---

## Estimated Project AWS Costs

- **EKS Cluster:**  
  Cost depends on the instance types and usage. A demo cluster with two t3.medium instances typically incurs minimal costs.
- **Load Balancer:**  
  Charges are based on usage; consult the AWS pricing calculator for details.
- **Monitoring:**  
  CloudWatch costs depend on the volume of logs and metrics collected.

---

## Summary

This revised project concentrates on delivering a thorough Kubernetes experience using AWS EKS. By using pre-built container images, you can dedicate your efforts to understanding cluster setup, deployments, service exposure, and monitoring. Once you’ve mastered these concepts, additional Docker-focused projects can help you learn how to build and manage your own container images.

---

## Key Terms

| Term                          | Definition                                                                                  |
|-------------------------------|---------------------------------------------------------------------------------------------|
| **Amazon EKS**                | Managed Kubernetes service on AWS for running containerized workloads.                      |
| **AWS EC2**                   | Elastic compute instances used as worker nodes in an EKS cluster.                           |
| **Amazon VPC**                | Virtual private cloud that provides isolated networking for your cluster.                  |
| **AWS CloudWatch**            | Monitoring service for collecting logs and metrics from AWS resources and applications.     |
| **AWS IAM**                   | Identity and Access Management for configuring permissions and security.                   |
| **eksctl**                    | CLI tool for creating and managing EKS clusters with simple commands.                       |
| **kubectl**                   | Kubernetes CLI for deploying and managing resources in a cluster.                          |
| **Deployment**                | Kubernetes object that ensures a specified number of pod replicas are running.              |
| **Pod**                       | Smallest deployable unit in Kubernetes, encapsulating one or more containers.               |
| **Service (LoadBalancer)**    | Kubernetes abstraction that exposes pods externally via an AWS ELB.                         |
| **Replica**                   | Copy of a pod managed by a Deployment to provide high availability.                         |
| **Manifest**                  | YAML file defining Kubernetes resources (e.g., Deployment, Service).                        |
| **Horizontal Pod Autoscaler** | Kubernetes feature to scale pod replicas automatically based on observed metrics.           |
| **Rolling Update**            | Strategy to update application versions with zero downtime by incrementally replacing pods. |
| **Distributed Tracing**       | Technique for tracking requests across microservices, e.g., via AWS X‑Ray.                  |
| **Container**                 | Lightweight, standalone unit of software that packages code and its dependencies to run consistently across environments.                  |
| **Image**                     | Read-only template used to create containers, including application code, runtime, and dependencies.                  |



