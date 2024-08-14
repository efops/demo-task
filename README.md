# EKS Cluster with Karpenter

This Terraform configuration deploys an Amazon EKS cluster with Karpenter for autoscaling, supporting both x86 and arm64 (Graviton) instances.

## Prerequisites
* AWS CLI configured with appropriate credentials
* Terraform installed (version 1.0.0 or later)
* kubectl installed
* An existing VPC in your AWS account

## Usage
* Clone this repository:
    ```shell
    git clone <repository-url>
    cd <repository-directory>
    ```
* Initialize Terraform:
    ```shell
    terraform init
    ```
* Create a `terraform.tfvars` file with your specific values:
    ```shell
    region       = "us-west-2"
    vpc_id       = "vpc-xxxxxxxxxxxxxxxxx"
    cluster_name = "my-eks-cluster"
    ```
* Review the plan:
    ```shell
    terraform plan
    ```
* Apply the configuration:
    ```shell
    terraform apply
    ```
* After the apply is complete, configure kubectl:
    ```shell
    aws eks update-kubeconfig --region <your-region> --name <your-cluster-name>
    ```
## Running Pods on x86 or Graviton Instances

To run a pod on a specific architecture, you can use node selectors or node affinity. Here are examples for both x86 and arm64:

## For x86 (amd64) instances:
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: x86-app
spec:
  replicas: 1
  selector:
    matchLabels:
      app: x86-app
  template:
    metadata:
      labels:
        app: x86-app
    spec:
      nodeSelector:
        kubernetes.io/arch: amd64
      containers:
      - name: app
        image: nginx:latest
```
## For arm64 (Graviton) instances:
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: arm64-app
spec:
  replicas: 1
  selector:
    matchLabels:
      app: arm64-app
  template:
    metadata:
      labels:
        app: arm64-app
    spec:
      nodeSelector:
        kubernetes.io/arch: arm64
      containers:
      - name: app
        image: nginx:latest
```
Save these YAML definitions to files (e.g., `x86-deployment.yaml` and `arm64-deployment.yaml`) and apply them using kubectl:
```shell
kubectl apply -f x86-deployment.yaml
kubectl apply -f arm64-deployment.yaml
```

