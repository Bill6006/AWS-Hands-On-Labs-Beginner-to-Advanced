# How to Deploy a Containerized Application with AWS EKS Workshop

This guide walks you through setting up an Amazon EKS (Elastic Kubernetes Service) cluster using the AWS EKS Workshop. You will learn how to create a cluster, deploy a containerized application, expose it using a Load Balancer, and monitor your application with AWS CloudWatch. These step-by-step instructions are designed for beginners to get real-life experience in managing containerized workloads on AWS.

---

## AWS Services Used

- **Amazon EKS** – Managed Kubernetes service to run containerized applications  
- **Docker** – Containerize your application  
- **AWS EC2** – Underlying compute for EKS worker nodes  
- **Amazon VPC** – Virtual network automatically created for your cluster  
- **AWS CloudWatch** – Monitor logs and performance of your containers  
- **AWS IAM** – Manage permissions and security for cluster operations

---

## AWS Architecture Diagram

![AWS architecture diagram](<Resources/Images/AWS-EKS-Architecture.png>)

---

## Deployment Steps

### Step 1: Prerequisites and Cluster Setup

> **Note:** Before you begin, ensure you have installed the AWS CLI, eksctl, and kubectl. Also, configure your AWS credentials.

1. **Install Prerequisites:**  
   - **AWS CLI:** Follow the [AWS CLI Installation Guide](https://docs.aws.amazon.com/cli/latest/userguide/getting-started-install.html).  
   - **eksctl:** Install by following the instructions on [eksctl’s website](https://eksctl.io/introduction/installation/).  
   - **kubectl:** Get it from the [Kubernetes documentation](https://kubernetes.io/docs/tasks/tools/).

2. **Create Your EKS Cluster:**  
   - Open your terminal and run:
     ```bash
     eksctl create cluster --name demo-cluster --region us-west-2 --nodes 2
     ```
   - **What This Does:**  
     - **Amazon VPC & Subnets:** Automatically creates a new VPC along with necessary subnets.  
     - **EC2 Instances:** Provisions two worker nodes that run your containerized application.  
   - **Verify Your Cluster:**  
     Run:
     ```bash
     kubectl get nodes
     ```
     You should see the nodes listed, confirming your cluster is active.

---

### Step 2: Deploying Your Application

> **Tip:** We’ll deploy a simple web application using a pre-built Docker image available in the workshop.

1. **Create a Deployment Manifest:**  
   - Create a file named `deployment.yaml` with the following content:
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
   - **Explanation:** This file defines a deployment that runs two replicas of a container based on the NGINX image.

2. **Deploy the Application:**  
   - Run:
     ```bash
     kubectl apply -f deployment.yaml
     ```
   - **Verify the Deployment:**  
     Run:
     ```bash
     kubectl get deployments
     ```
     Ensure that the `sample-app` deployment shows up and that the desired number of replicas is running.

---

### Step 3: Exposing the Application

> **Note:** To access your application from the internet, you need to create a service that exposes it externally.

1. **Create a Service Manifest:**  
   - Create a file named `service.yaml` with the following content:
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
   - **Explanation:** This manifest creates a Kubernetes Service of type LoadBalancer that directs incoming traffic on port 80 to your application pods.

2. **Deploy the Service:**  
   - Run:
     ```bash
     kubectl apply -f service.yaml
     ```
   - **Access Your Application:**  
     - Run:
       ```bash
       kubectl get services
       ```
     - Look for the `EXTERNAL-IP` under `sample-app-service`. Once it appears, open your browser and navigate to that IP address to view your running application.

---

### Step 4: Monitoring and Tracing

> **Tip:** Monitoring is crucial to troubleshoot and ensure your application is running smoothly.

1. **View Application Logs:**  
   - To see logs from your pods, run:
     ```bash
     kubectl logs <pod-name>
     ```
   - Replace `<pod-name>` with the name of one of your application pods.

2. **Enable CloudWatch Container Insights:**  
   - Follow the AWS EKS Workshop instructions to enable CloudWatch Container Insights. This feature automatically collects metrics and logs from your cluster, helping you monitor performance and troubleshoot issues.

3. **(Optional) Set Up Distributed Tracing:**  
   - If you wish to trace requests across services, refer to the tracing sections in the AWS EKS Workshop. You can integrate AWS X-Ray or other tracing tools to get deeper insights into your application’s behavior.

---

## Additional Enhancements

- **Auto Scaling:**  
  Configure a Horizontal Pod Autoscaler to automatically adjust the number of pods based on traffic load.
- **Rolling Updates:**  
  Use Kubernetes rolling update strategies to deploy new versions of your application with zero downtime.
- **CI/CD Integration:**  
  Link your deployment with AWS CodePipeline and CodeBuild for continuous integration and deployment.

---

## Estimated Project AWS Costs

- **EKS Cluster:**  
  Costs depend on the instance types and usage. For a demo cluster with two t3.medium instances, costs are generally minimal.
- **Load Balancer:**  
  Charges are based on usage; refer to the AWS pricing calculator for detailed estimates.
- **Monitoring:**  
  CloudWatch costs depend on the volume of logs and metrics collected.

---

## Resources

- [AWS EKS Workshop](https://www.eksworkshop.com/) 0  
- [AWS EKS User Guide](https://docs.aws.amazon.com/eks/latest/userguide/what-is-eks.html) 1  

---

## Author

Created by **Tyree** – Feel free to contribute or ask questions!

---

# End Of Project