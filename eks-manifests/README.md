# Two-Tier Flask Application Deployment on AWS EKS

[![AWS EKS](https://img.shields.io/badge/AWS-EKS-orange?style=for-the-badge&logo=amazon-aws&logoColor=white)](https://aws.amazon.com/eks/)
[![Kubernetes](https://img.shields.io/badge/Kubernetes-326CE5?style=for-the-badge&logo=kubernetes&logoColor=white)](https://kubernetes.io/)
[![Flask](https://img.shields.io/badge/Flask-000000?style=for-the-badge&logo=flask&logoColor=white)](https://flask.palletsprojects.com/)
[![MySQL](https://img.shields.io/badge/MySQL-4479A1?style=for-the-badge&logo=mysql&logoColor=white)](https://www.mysql.com/)

## Table of Contents

- [Project Overview](#-project-overview)
- [Architecture](#️-architecture)
- [Prerequisites](#-prerequisites)
- [Step-by-Step Deployment Guide](#️-step-by-step-deployment-guide)
- [Deployment Commands](#-deployment-commands)
- [Verification & Testing](#-verification--testing)
- [Troubleshooting](#-troubleshooting)
- [Cost Optimization](#-cost-optimization)
- [Cleanup](#-cleanup)
- [Common Issues](#-common-issues)

## 🚀 Project Overview

This project demonstrates the deployment of a **two-tier Flask web application** on **Amazon EKS (Elastic Kubernetes Service)**. The application consists of:

- **Frontend/Backend**: Python Flask application with web interface
- **Database**: MySQL database for data persistence
- **Infrastructure**: AWS EKS cluster with managed Kubernetes
- **Load Balancing**: AWS Load Balancer for external access
- **Storage**: ConfigMaps and Secrets for configuration management

### Key Features

✅ **Production-ready** Flask web application  
✅ **MySQL database** with persistent storage  
✅ **AWS EKS** managed Kubernetes cluster  
✅ **LoadBalancer service** for external access  
✅ **Secrets management** for sensitive data  
✅ **ConfigMaps** for database initialization  
✅ **Scalable architecture** with replicas

## 🏗️ Architecture

```
┌─────────────────────────────────────────────────────────────────┐
│                        AWS Cloud                               │
│                                                                 │
│  ┌─────────────────────────────────────────────────────────┐   │
│  │                 EKS Cluster                             │   │
│  │                                                         │   │
│  │  ┌─────────────────┐    ┌─────────────────┐            │   │
│  │  │  Worker Node 1  │    │  Worker Node 2  │            │   │
│  │  │  (t2.medium)    │    │  (t2.medium)    │            │   │
│  │  │                 │    │                 │            │   │
│  │  │  Flask Pod      │    │  MySQL Pod      │            │   │
│  │  │  Port: 5000     │    │  Port: 3306     │            │   │
│  │  └─────────────────┘    └─────────────────┘            │   │
│  │           │                       │                    │   │
│  │  ┌─────────────────┐    ┌─────────────────┐            │   │
│  │  │ Flask Service   │    │ MySQL Service   │            │   │
│  │  │ LoadBalancer    │    │ ClusterIP       │            │   │
│  │  │ Port: 80        │    │ Port: 3306      │            │   │
│  │  └─────────────────┘    └─────────────────┘            │   │
│  └─────────────────────────────────────────────────────────┘   │
│                              │                                 │
│  ┌─────────────────────────────────────────────────────────┐   │
│  │            AWS Load Balancer                            │   │
│  │         (External Access Point)                         │   │
│  └─────────────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────────────┘
                              │
                    ┌──────────────────┐
                    │   Internet       │
                    │   Users          │
                    └──────────────────┘
```

<img width="1908" height="939" alt="Image" src="https://github.com/user-attachments/assets/e9d10248-8f43-4670-b26a-de30c8876abd" />

<img width="1908" height="875" alt="Image" src="https://github.com/user-attachments/assets/3152f2e6-b333-4a8d-b8fa-ce69873fe3b5" />

## 📋 Prerequisites

### AWS Account Requirements

- **AWS Account** with appropriate permissions
- **IAM User** with `AdministratorAccess` policy
- **Region**: `us-east-1` (Any region as per your choice)

### Local Environment

- **Ubuntu EC2 Instance** (t3.micro or higher)
- **Internet Connection** for downloading tools
- **SSH Access** to your EC2 instance

### Estimated Costs

- **EKS Cluster**: ~$0.10/hour
- **EC2 Worker Nodes**: ~$0.0464/hour per t2.medium instance
- **Load Balancer**: ~$0.0225/hour
- **Total**: ~$0.20/hour for testing environment

> ⚠️ **Cost Warning**: Remember to delete resources after testing to avoid charges!

## 🛠️ Step-by-Step Deployment Guide

### Step 1: Create IAM User

1. **Login to AWS Console** → Navigate to IAM
2. **Create User**:
   ```
   Username: eks-admin
   Access Type: Programmatic access
   ```
3. **Attach Policy**: `AdministratorAccess`
4. **Download Credentials**: Save Access Key ID and Secret Access Key

### Step 2: Launch EC2 Instance

1. **Launch Instance**:

   ```
   AMI: Ubuntu Server 22.04 LTS
   Instance Type: t3.micro (or t3.small for better performance)
   Region: us-east-1
   Security Group: Allow SSH (22), HTTP (80), HTTPS (443)
   Key Pair: Create or use existing
   ```

2. **Connect to Instance**:
   ```bash
   ssh -i "your-key.pem" ubuntu@<EC2-PUBLIC-IP>
   ```

### Step 3: Install Required Tools

#### Install AWS CLI v2

```bash
# Download and install AWS CLI
curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
sudo apt install unzip -y
unzip awscliv2.zip
sudo ./aws/install -i /usr/local/aws-cli -b /usr/local/bin --update

# Verify installation
aws --version
```

#### Configure AWS Credentials

```bash
aws configure
# Enter your IAM user credentials:
# AWS Access Key ID: [Your Access Key]
# AWS Secret Access Key: [Your Secret Key]
# Default region name: us-east-1
# Default output format: json
```

#### Install Docker

```bash
# Update system and install Docker
sudo apt-get update
sudo apt install docker.io -y

# Start Docker service
sudo systemctl start docker
sudo systemctl enable docker

# Add user to docker group
sudo chown $USER /var/run/docker.sock

# Verify installation
docker --version
docker ps
```

#### Install kubectl

```bash
# Download kubectl for EKS
curl -o kubectl https://amazon-eks.s3.us-east-1.amazonaws.com/1.19.6/2021-01-05/bin/linux/amd64/kubectl

# Make executable and move to PATH
chmod +x ./kubectl
sudo mv ./kubectl /usr/local/bin

# Verify installation
kubectl version --short --client
```

#### Install eksctl

```bash
# Download and install eksctl
curl --silent --location "https://github.com/weaveworks/eksctl/releases/latest/download/eksctl_$(uname -s)_amd64.tar.gz" | tar xz -C /tmp
sudo mv /tmp/eksctl /usr/local/bin

# Verify installation
eksctl version
```

### Step 4: Create EKS Cluster

```bash
# Create EKS cluster (this takes 15-20 minutes)
eksctl create cluster \
  --name three-tier-cluster \
  --region us-east-1 \
  --node-type t2.medium \
  --nodes-min 2 \
  --nodes-max 2

# Update kubeconfig
aws eks update-kubeconfig --region us-east-1 --name three-tier-cluster

# Verify cluster
kubectl get nodes
```

**Expected Output**:

```
NAME                                          STATUS   ROLES    AGE   VERSION
ip-192-168-xx-xx.us-east-1.compute.internal   Ready    <none>   2m    v1.x.x
ip-192-168-xx-xx.us-east-1.compute.internal   Ready    <none>   2m    v1.x.x
```

### Step 5: Prepare Application Files

#### Clone Repository

```bash
# Clone the application repository
git clone https://github.com/deepanshub9/two-tier-flask-app.git
cd two-tier-flask-app/Eks-manifests
```

### Step 6: Deploy Application

#### Create Namespace

```bash
# Create dedicated namespace
kubectl create namespace two-tier-ns

# Set default namespace (optional)
kubectl config set-context --current --namespace=two-tier-ns
```

#### Deploy All Resources

```bash
# Navigate to EKS manifests directory
cd Eks-manifests

# Apply all manifests
kubectl apply -f . -n two-tier-ns
```

## 🔧 Deployment Commands

### Quick Deployment

```bash
# One-command deployment
kubectl apply -f . -n two-tier-ns
```

### Manual Step-by-Step

```bash
# 1. Deploy secrets and config
kubectl apply -f mysql-secrets.yml
kubectl apply -f mysql-configmap.yml

# 2. Deploy MySQL
kubectl apply -f mysql-deployment.yml
kubectl apply -f mysql-svc.yml

# 3. Wait for MySQL to be ready
kubectl wait --for=condition=ready pod -l app=mysql -n two-tier-ns --timeout=300s

# 4. Deploy Flask app
kubectl apply -f two-tier-app-deployment.yml
kubectl apply -f two-tier-app-svc.yml
```

## ✅ Verification & Testing

### Check Pod Status

```bash
# Check all pods
kubectl get pods -n two-tier-ns

# Expected output:
# NAME                            READY   STATUS    RESTARTS   AGE
# mysql-xxxx-xxxx                 1/1     Running   0          2m
# two-tier-app-xxxx-xxxx          1/1     Running   0          1m
```

### Check Services

```bash
# Check services
kubectl get services -n two-tier-ns

# Expected output:
# NAME                  TYPE           CLUSTER-IP      EXTERNAL-IP                                              PORT(S)        AGE
# mysql                 ClusterIP      10.100.xxx.xxx  <none>                                                   3306/TCP       2m
# two-tier-app-service  LoadBalancer   10.100.xxx.xxx  axxxxx-xxxxx.us-east-1.elb.amazonaws.com               80:xxxxx/TCP   1m
```

### Get Application URL

```bash
# Get LoadBalancer URL
kubectl get service two-tier-app-service -n two-tier-ns -o jsonpath='{.status.loadBalancer.ingress[0].hostname}'
```

### Test Database Connection

```bash
# Connect to MySQL pod
kubectl exec -it deployment/mysql -n two-tier-ns -- mysql -u flaskuser -pflaskpass -D flaskdb -e "SHOW TABLES;"

# Expected output:
# +------------------+
# | Tables_in_flaskdb|
# +------------------+
# | messages         |
# +------------------+
```

### Test Application

```bash
# Get LoadBalancer URL and test
LOAD_BALANCER_URL=$(kubectl get service two-tier-app-service -n two-tier-ns -o jsonpath='{.status.loadBalancer.ingress[0].hostname}')
echo "Application URL: http://$LOAD_BALANCER_URL"

# Test with curl
curl -I http://$LOAD_BALANCER_URL

# Expected: HTTP/200 OK
```

## 🐛 Troubleshooting

### Common Issues and Solutions

#### 1. Pod Stuck in Pending

```bash
# Check node resources
kubectl describe nodes
kubectl get pods -n two-tier-ns -o wide

# Solution: Scale cluster or use smaller instance types
```

#### 2. MySQL Pod CrashLoopBackOff

```bash
# Check logs
kubectl logs deployment/mysql -n two-tier-ns

# Common causes:
# - Incorrect secret values
# - Resource constraints
# - Invalid SQL in ConfigMap
```

#### 3. Flask App Can't Connect to MySQL

```bash
# Check DNS resolution
kubectl exec -it deployment/two-tier-app -n two-tier-ns -- nslookup mysql

# Check environment variables
kubectl describe pod -l app=two-tier-app -n two-tier-ns | grep -A 10 Environment
```

#### 4. LoadBalancer Pending

```bash
# Check service status
kubectl describe service two-tier-app-service -n two-tier-ns

# Wait time: LoadBalancer creation takes 3-5 minutes
```

### Debug Commands

```bash
# View all resources
kubectl get all -n two-tier-ns

# Check events
kubectl get events -n two-tier-ns --sort-by='.lastTimestamp'

# Check logs
kubectl logs -l app=mysql -n two-tier-ns
kubectl logs -l app=two-tier-app -n two-tier-ns

# Describe problematic resources
kubectl describe pod <pod-name> -n two-tier-ns
kubectl describe service <service-name> -n two-tier-ns
```

## 💰 Cost Optimization

### Development Environment

```bash
# Use smaller instances for development
eksctl create cluster \
  --name dev-cluster \
  --region us-east-1 \
  --node-type t3.micro \
  --nodes-min 1 \
  --nodes-max 1
```

### Production Environment

```bash
# Use appropriate instances for production
eksctl create cluster \
  --name prod-cluster \
  --region us-east-1 \
  --node-type t3.medium \
  --nodes-min 2 \
  --nodes-max 5 \
  --nodes-desired 2
```

### Cost Monitoring

```bash
# Monitor costs with AWS CLI
aws ce get-cost-and-usage \
  --time-period Start=2025-01-01,End=2025-01-31 \
  --granularity MONTHLY \
  --metrics BlendedCost \
  --group-by Type=DIMENSION,Key=SERVICE
```

## 🧹 Cleanup

### Delete Application

```bash
# Delete application resources
kubectl delete -f . -n two-tier-ns

# Delete namespace
kubectl delete namespace two-tier-ns
```

### Delete EKS Cluster

```bash
# Delete the entire cluster (this takes 10-15 minutes)
eksctl delete cluster --name three-tier-cluster --region us-east-1
```

### Verify Cleanup

```bash
# Verify no EKS clusters remain
aws eks list-clusters --region us-east-1

# Verify no LoadBalancers remain
aws elbv2 describe-load-balancers --region us-east-1
```

## 🚨 Common Issues

### Issue 1: `aws: command not found`

**Solution**:

```bash
# Reinstall AWS CLI
curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
unzip awscliv2.zip
sudo ./aws/install --update
```

### Issue 2: `kubectl: connection refused`

**Solution**:

```bash
# Update kubeconfig
aws eks update-kubeconfig --region us-east-1 --name three-tier-cluster

# Verify context
kubectl config current-context
```

### Issue 3: `eksctl create cluster fails`

**Solution**:

```bash
# Check IAM permissions
aws sts get-caller-identity

# Check CloudFormation stacks
aws cloudformation list-stacks --region us-east-1
```

### Issue 4: LoadBalancer takes too long

**Expected behavior**: LoadBalancer creation takes 3-5 minutes  
**Solution**: Wait or check AWS Console → EC2 → Load Balancers

## 📊 Monitoring

### Basic Monitoring

```bash
# Check resource usage
kubectl top nodes
kubectl top pods -n two-tier-ns

# Check cluster status
kubectl cluster-info
kubectl get componentstatuses
```

### Application Logs

```bash
# Tail logs in real-time
kubectl logs -f deployment/two-tier-app -n two-tier-ns
kubectl logs -f deployment/mysql -n two-tier-ns
```

## 🎯 Success Metrics

After successful deployment, you should see:

✅ **EKS Cluster**: 2 worker nodes in `Ready` state  
✅ **MySQL Pod**: `1/1 Running` with 0 restarts  
✅ **Flask Pod**: `1/1 Running` with 0 restarts  
✅ **LoadBalancer**: External IP assigned and accessible  
✅ **Application**: Web interface working and database connected

## 📋 Manifest Files in This Directory

| File                          | Purpose                      | Type                   |
| ----------------------------- | ---------------------------- | ---------------------- |
| `mysql-secrets.yml`           | MySQL root password          | Secret                 |
| `mysql-configmap.yml`         | Database initialization SQL  | ConfigMap              |
| `mysql-deployment.yml`        | MySQL database deployment    | Deployment             |
| `mysql-svc.yml`               | MySQL internal service       | Service (ClusterIP)    |
| `two-tier-app-deployment.yml` | Flask application deployment | Deployment             |
| `two-tier-app-svc.yml`        | Flask external service       | Service (LoadBalancer) |

## 📝 Additional Resources

- [AWS EKS Documentation](https://docs.aws.amazon.com/eks/)
- [eksctl Documentation](https://eksctl.io/)
- [Kubernetes Documentation](https://kubernetes.io/docs/)
- [Flask Documentation](https://flask.palletsprojects.com/)

## 🤝 Contributing

Feel free to contribute to this project by:

- Reporting issues
- Suggesting improvements
- Submitting pull requests
- Sharing your deployment experiences

## 📄 License

This project is licensed under the MIT License - see the LICENSE file for details.

---

**Project Status**: ✅ **Successfully Deployed on AWS EKS**

**Application Access**: `http://<LoadBalancer-URL>`

**Last Updated**: July 2025

**Tested Environment**: AWS EKS 1.29 + Ubuntu 22.04

---

_This guide provides a complete walkthrough for deploying a two-tier Flask application on AWS EKS. Every step has been tested and verified to ensure a smooth deployment experience._
