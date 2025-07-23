# Two-Tier Flask Application on Kubernetes

[![Kubernetes](https://img.shields.io/badge/Kubernetes-326CE5?style=for-the-badge&logo=kubernetes&logoColor=white)](https://kubernetes.io/)
[![Flask](https://img.shields.io/badge/Flask-000000?style=for-the-badge&logo=flask&logoColor=white)](https://flask.palletsprojects.com/)
[![MySQL](https://img.shields.io/badge/MySQL-4479A1?style=for-the-badge&logo=mysql&logoColor=white)](https://www.mysql.com/)
[![AWS](https://img.shields.io/badge/AWS-232F3E?style=for-the-badge&logo=amazon-aws&logoColor=white)](https://aws.amazon.com/)

## 📋 Table of Contents

- [Project Overview](#-project-overview)
- [Architecture](#-architecture)
- [Prerequisites](#-prerequisites)
- [Infrastructure Setup](#-infrastructure-setup)
- [Application Components](#-application-components)
- [Deployment Process](#-deployment-process)
- [Challenges & Solutions](#-challenges--solutions)
- [Troubleshooting Guide](#%EF%B8%8F-troubleshooting-guide)
- [Testing & Validation](#-testing--validation)
- [Monitoring & Maintenance](#-monitoring--maintenance)
- [Lessons Learned](#-lessons-learned)
- [Future Improvements](#-future-improvements)

## 🎯 Project Overview

This project demonstrates the deployment of a **two-tier Flask web application** on **Kubernetes** cluster running on **AWS EC2 instances**. The application consists of a Flask frontend/backend and a MySQL database, showcasing modern containerized application deployment practices.

### What We Built

- **Frontend/Backend**: Python Flask application
- **Database**: MySQL 8.0
- **Container Orchestration**: Kubernetes (K8s)
- **Infrastructure**: AWS EC2 instances
- **Networking**: Kubernetes Services with NodePort
- **Storage**: Persistent Volumes for database data
- **Service Discovery**: Kubernetes native DNS

### Key Achievements

✅ Successfully deployed a production-ready two-tier application  
✅ Implemented persistent storage for database  
✅ Set up service discovery between application tiers  
✅ Configured external access via NodePort services  
✅ Established monitoring and troubleshooting workflows  
✅ Documented comprehensive deployment process

## 🏗️ Architecture

```
┌─────────────────────────────────────────────────────────────┐
│                     AWS Cloud                              │
│  ┌─────────────────────┐    ┌─────────────────────┐       │
│  │   Master Node       │    │   Worker Node       │       │
│  │   (EC2 Instance)    │    │   (EC2 Instance)    │       │
│  │                     │    │                     │       │
│  │  ┌──────────────┐   │    │  ┌──────────────┐   │       │
│  │  │   kubectl    │   │    │  │   kubelet    │   │       │
│  │  │   kube-api   │   │    │  │   kube-proxy │   │       │
│  │  │   etcd       │   │    │  │   container  │   │       │
│  │  │   scheduler  │   │    │  │   runtime    │   │       │
│  │  └──────────────┘   │    │  └──────────────┘   │       │
│  └─────────────────────┘    └─────────────────────┘       │
└─────────────────────────────────────────────────────────────┘
                              │
┌─────────────────────────────────────────────────────────────┐
│              Kubernetes Cluster                            │
│                                                             │
│  ┌──────────────────┐              ┌──────────────────┐    │
│  │   Flask App Pod  │◄────────────►│   MySQL Pod      │    │
│  │                  │              │                  │    │
│  │  ┌─────────────┐ │              │ ┌─────────────┐  │    │
│  │  │   Flask     │ │              │ │   MySQL     │  │    │
│  │  │   App       │ │              │ │   Database  │  │    │
│  │  │   Port:5000 │ │              │ │   Port:3306 │  │    │
│  │  └─────────────┘ │              │ └─────────────┘  │    │
│  └──────────────────┘              └──────────────────┘    │
│           │                                   │             │
│  ┌──────────────────┐              ┌──────────────────┐    │
│  │  App Service     │              │  MySQL Service   │    │
│  │  NodePort:30007  │              │  ClusterIP:3306  │    │
│  └──────────────────┘              └──────────────────┘    │
│                                              │             │
│                                    ┌──────────────────┐    │
│                                    │  Persistent      │    │
│                                    │  Volume (10Gi)   │    │
│                                    └──────────────────┘    │
└─────────────────────────────────────────────────────────────┘
                              │
                    ┌──────────────────┐
                    │   External       │
                    │   Users          │
                    │   Port: 30007    │
                    └──────────────────┘
```

<img width="1911" height="874" alt="Image" src="https://github.com/user-attachments/assets/190c06e6-9b6a-4723-adbf-50a85ec1900b" />

<img width="1315" height="612" alt="Image" src="https://github.com/user-attachments/assets/90333f6e-641a-43b3-afc5-195afbac89b8" />

### Master node

<img width="1323" height="692" alt="Image" src="https://github.com/user-attachments/assets/a1da4600-6926-41d3-95c7-639bb7be810d" />

### Worker node

<img width="1902" height="878" alt="Image" src="https://github.com/user-attachments/assets/0e3a3ca1-24f7-454b-8a90-859a681b0f41" />

## 📋 Prerequisites

### System Requirements

- **AWS Account** with EC2 access
- **2 EC2 Instances** (t3.medium or higher recommended)
  - 1 Master Node (2 vCPU, 4GB RAM)
  - 1 Worker Node (2 vCPU, 4GB RAM)
- **Ubuntu 20.04 LTS** or later
- **Minimum 20GB storage** per instance

### Required Software

- **Docker** (20.10+)
- **Kubernetes** (1.25+)
- **kubectl** CLI tool
- **Git** for version control
- **VS Code** with Remote-SSH extension (optional)

### Network Requirements

- **Security Groups** configured for:
  - SSH (22) - For remote access
  - HTTP (80) - For web traffic
  - Custom TCP (30007) - For NodePort service
  - Custom TCP (6443) - For Kubernetes API
  - Custom TCP (2379-2380) - For etcd
  - Custom TCP (10250-10252) - For kubelet

## 🚀 Infrastructure Setup

### Step 1: AWS EC2 Instance Creation

```bash
# Launch EC2 instances with following specifications:
# - AMI: Ubuntu 20.04 LTS
# - Instance Type: t3.medium
# - Storage: 20GB gp3
# - Security Group: kubernetes-cluster-sg
# - Key Pair: two-tier.pem

# Instance Details:
# Master Node: ip-172-31-27-130 (Private IP)
# Worker Node: ip-172-31-28-223 (Private IP)
```

### Step 2: Connect to EC2 Instances

```bash
# Connect to Master Node
ssh -i "two tier.pem" ubuntu@<MASTER_PUBLIC_IP>

# Connect to Worker Node
ssh -i "two tier.pem" ubuntu@<WORKER_PUBLIC_IP>
```

### Step 3: Install Docker (On Both Nodes)

```bash
# Update system packages
sudo apt update && sudo apt upgrade -y

# Install Docker dependencies
sudo apt install -y apt-transport-https ca-certificates curl software-properties-common

# Add Docker GPG key
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg

# Add Docker repository
echo "deb [arch=amd64 signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

# Install Docker
sudo apt update
sudo apt install -y docker-ce docker-ce-cli containerd.io

# Add user to docker group
sudo usermod -aG docker $USER

# Start and enable Docker
sudo systemctl start docker
sudo systemctl enable docker

# Verify installation
docker --version
```

### Step 4: Install Kubernetes (On Both Nodes)

```bash
# Add Kubernetes GPG key
curl -fsSL https://packages.cloud.google.com/apt/doc/apt-key.gpg | sudo gpg --dearmor -o /usr/share/keyrings/kubernetes-archive-keyring.gpg

# Add Kubernetes repository
echo "deb [signed-by=/usr/share/keyrings/kubernetes-archive-keyring.gpg] https://apt.kubernetes.io/ kubernetes-xenial main" | sudo tee /etc/apt/sources.list.d/kubernetes.list

# Install Kubernetes components
sudo apt update
sudo apt install -y kubelet kubeadm kubectl

# Hold packages to prevent automatic updates
sudo apt-mark hold kubelet kubeadm kubectl

# Configure containerd
sudo mkdir -p /etc/containerd
sudo containerd config default | sudo tee /etc/containerd/config.toml

# Restart containerd
sudo systemctl restart containerd
sudo systemctl enable containerd

# Disable swap (required for Kubernetes)
sudo swapoff -a
sudo sed -i '/ swap / s/^\(.*\)$/#\1/g' /etc/fstab
```

### Step 5: Initialize Kubernetes Cluster (Master Node Only)

```bash
# Initialize the cluster
sudo kubeadm init --pod-network-cidr=192.168.0.0/16

# Configure kubectl for regular user
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config

# Install Calico network plugin
kubectl apply -f https://docs.projectcalico.org/manifests/calico.yaml

# Get join command for worker nodes
kubeadm token create --print-join-command
```

### Step 6: Join Worker Node to Cluster

```bash
# On Worker Node, run the join command from master
sudo kubeadm join <MASTER_IP>:6443 --token <TOKEN> --discovery-token-ca-cert-hash sha256:<HASH>
```

### Step 7: Verify Cluster Setup

```bash
# Check nodes status
kubectl get nodes

# Expected output:
NAME               STATUS   ROLES           AGE   VERSION
ip-172-31-27-130   Ready    control-plane   5m    v1.25.0
ip-172-31-28-223   Ready    <none>          2m    v1.25.0
```

## 🏗️ Application Components

### Flask Application Structure

```
two-tier-flask-app/
├── app.py                 # Main Flask application
├── requirements.txt       # Python dependencies
├── templates/
│   └── index.html        # Frontend template
├── static/
│   ├── style.css         # Styling
│   └── script.js         # JavaScript
├── Dockerfile            # Container definition
└── k8s/                  # Kubernetes manifests
    ├── mysql-deployment.yml
    ├── mysql-svc.yml
    ├── mysql-pv.yml
    ├── mysql-pvc.yml
    ├── two-tier-app-deployment.yml
    ├── two-tier-app-pod.yml
    └── two-tier-app-svc.yml
```

### Flask Application Code

```python
# app.py - Main Application
from flask import Flask, render_template, request, redirect, url_for
import mysql.connector
import os

app = Flask(__name__)

# Database configuration
db_config = {
    'host': os.getenv('MYSQL_HOST', 'mysql-service'),
    'user': os.getenv('MYSQL_USER', 'root'),
    'password': os.getenv('MYSQL_PASSWORD', 'rootpassword'),
    'database': os.getenv('MYSQL_DATABASE', 'two_tier_db')
}

def get_db_connection():
    try:
        conn = mysql.connector.connect(**db_config)
        return conn
    except mysql.connector.Error as e:
        print(f"Database connection error: {e}")
        return None

@app.route('/')
def index():
    conn = get_db_connection()
    if conn:
        cursor = conn.cursor()
        cursor.execute("SELECT * FROM users")
        users = cursor.fetchall()
        conn.close()
        return render_template('index.html', users=users)
    return "Database connection failed"

@app.route('/add_user', methods=['POST'])
def add_user():
    name = request.form['name']
    email = request.form['email']

    conn = get_db_connection()
    if conn:
        cursor = conn.cursor()
        cursor.execute("INSERT INTO users (name, email) VALUES (%s, %s)", (name, email))
        conn.commit()
        conn.close()

    return redirect(url_for('index'))

if __name__ == '__main__':
    app.run(host='0.0.0.0', port=5000, debug=True)
```

### Dockerfile

```dockerfile
FROM python:3.9-slim

WORKDIR /app

COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

COPY . .

EXPOSE 5000

CMD ["python", "app.py"]
```

```yaml
# two-tier-app-pod.yml
apiVersion: v1
kind: Pod
metadata:
  name: two-tier-app-pod
  labels:
    app: two-tier-app
spec:
  containers:
    - name: two-tier-app
      image: your-dockerhub-username/two-tier-flask-app:latest
      ports:
        - containerPort: 5000
      env:
        - name: MYSQL_HOST
          value: "mysql-service"
        - name: MYSQL_USER
          value: "root"
        - name: MYSQL_PASSWORD
          value: "rootpassword"
        - name: MYSQL_DATABASE
          value: "two_tier_db"
```

## 🚀 Deployment Process

### Step 1: Prepare Application Code

```bash
# Clone or create your application directory
mkdir ~/two-tier-flask-app
cd ~/two-tier-flask-app

# Create application files (app.py, requirements.txt, etc.)
# Create Dockerfile
# Create k8s directory with manifests
mkdir k8s
```

### Step 2: Build and Push Docker Image

```bash
# Build Docker image
docker build -t your-dockerhub-username/two-tier-flask-app:latest .

# Login to Docker Hub
docker login

# Push image to registry
docker push your-dockerhub-username/two-tier-flask-app:latest
```

### Step 3: Deploy MySQL Components

```bash
# Navigate to k8s directory
cd ~/two-tier-flask-app/k8s

# Create persistent volume and claim
kubectl apply -f mysql-pv.yml
kubectl apply -f mysql-pvc.yml

# Verify PV and PVC
kubectl get pv
kubectl get pvc

# Deploy MySQL
kubectl apply -f mysql-deployment.yml
kubectl apply -f mysql-svc.yml

# Verify MySQL deployment
kubectl get pods -l app=mysql
kubectl get services mysql-service
```

### Step 4: Initialize Database Schema

```bash
# Connect to MySQL pod to set up database
kubectl exec -it <mysql-pod-name> -- mysql -u root -p

# Create database schema
CREATE DATABASE IF NOT EXISTS two_tier_db;
USE two_tier_db;

CREATE TABLE users (
    id INT AUTO_INCREMENT PRIMARY KEY,
    name VARCHAR(100) NOT NULL,
    email VARCHAR(100) NOT NULL UNIQUE,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

# Insert sample data
INSERT INTO users (name, email) VALUES
('John Doe', 'john@example.com'),
('Jane Smith', 'jane@example.com');
```

### Step 5: Deploy Flask Application

```bash
# Deploy Flask application (using Pod approach)
kubectl apply -f two-tier-app-pod.yml
kubectl apply -f two-tier-app-svc.yml

# Alternative: Deploy using Deployment
# kubectl apply -f two-tier-app-deployment.yml
# kubectl apply -f two-tier-app-svc.yml

# Verify deployment
kubectl get pods -l app=two-tier-app
kubectl get services two-tier-app-service
```

### Step 6: Verify Complete Deployment

```bash
# Check all resources
kubectl get all

# Expected output:
NAME                     READY   STATUS    RESTARTS   AGE
pod/mysql-deployment-xxx Ready   1/1       Running    0          5m
pod/two-tier-app-pod     Ready   1/1       Running    0          2m

NAME                           TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)        AGE
service/kubernetes             ClusterIP   10.96.0.1       <none>        443/TCP        1h
service/mysql-service          ClusterIP   10.96.100.200   <none>        3306/TCP       5m
service/two-tier-app-service   NodePort    10.96.150.100   <none>        80:30007/TCP   2m

NAME                               READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/mysql-deployment   1/1     1            1           5m
```

## 🚨 Challenges & Solutions

### Challenge 1: Pod Stuck in Pending State

**Problem**: MySQL pod remained in `Pending` state after deployment.

**Root Cause**:

- Persistent Volume not created properly
- Node didn't have sufficient storage
- Storage class mismatch

**Solution**:

```bash
# Check pod events
kubectl describe pod <mysql-pod-name>

# Verify PV status
kubectl get pv

# Check node storage
kubectl describe node <node-name>

# Fix: Recreate PV with correct path
kubectl delete -f mysql-pv.yml
# Edit mysql-pv.yml to fix hostPath
kubectl apply -f mysql-pv.yml
```

**Prevention**:

- Always verify storage prerequisites
- Use `kubectl describe` for detailed error information
- Ensure node has sufficient disk space

### Challenge 2: Service Not Accessible Externally

**Problem**: Flask application deployed successfully but not accessible via NodePort.

**Root Cause**:

- AWS Security Group blocking port 30007
- Service selector not matching pod labels
- Wrong targetPort configuration

**Solution**:

```bash
# Check service configuration
kubectl describe service two-tier-app-service

# Verify endpoints
kubectl get endpoints two-tier-app-service

# Check pod labels
kubectl get pods --show-labels

# Fix AWS Security Group
# Add inbound rule: Custom TCP, Port 30007, Source 0.0.0.0/0

# Test connectivity
curl http://<worker-node-public-ip>:30007
```

**Prevention**:

- Always configure security groups before deployment
- Verify label selectors match between services and pods
- Test connectivity after each deployment step

### Challenge 3: Database Connection Issues

**Problem**: Flask app couldn't connect to MySQL database.

**Root Cause**:

- Service name resolution failing
- Wrong database credentials
- MySQL not ready when Flask app started

**Solution**:

```bash
# Check MySQL service DNS resolution
kubectl exec -it two-tier-app-pod -- nslookup mysql-service

# Verify MySQL is running
kubectl get pods -l app=mysql
kubectl logs <mysql-pod-name>

# Check environment variables
kubectl describe pod two-tier-app-pod

# Test database connection
kubectl exec -it <mysql-pod-name> -- mysql -u root -p -e "SHOW DATABASES;"
```

**Prevention**:

- Use Kubernetes service DNS names
- Implement readiness probes
- Add connection retry logic in application

### Challenge 4: Image Pull Errors

**Problem**: `ImagePullBackOff` errors when deploying Flask application.

**Root Cause**:

- Docker image not pushed to registry
- Wrong image name/tag in manifest
- Registry authentication issues

**Solution**:

```bash
# Check pod events
kubectl describe pod two-tier-app-pod

# Verify image exists
docker pull your-dockerhub-username/two-tier-flask-app:latest

# Check image name in manifest
kubectl get pod two-tier-app-pod -o yaml | grep image

# Rebuild and push image
docker build -t your-dockerhub-username/two-tier-flask-app:latest .
docker push your-dockerhub-username/two-tier-flask-app:latest
```

**Prevention**:

- Always verify image push before deployment
- Use specific tags instead of `latest` in production
- Implement image scanning and validation

### Challenge 5: Resource Constraints

**Problem**: Pods getting evicted or stuck in `Pending` state due to resource limits.

**Root Cause**:

- Insufficient CPU/Memory on nodes
- No resource requests/limits defined
- Node pressure conditions

**Solution**:

```bash
# Check node resources
kubectl top nodes
kubectl describe nodes

# Check pod resource usage
kubectl top pods

# Add resource requests/limits to manifests
spec:
  containers:
  - name: mysql
    resources:
      requests:
        memory: "512Mi"
        cpu: "250m"
      limits:
        memory: "1Gi"
        cpu: "500m"
```

**Prevention**:

- Always define resource requests and limits
- Monitor node resource utilization
- Scale cluster when needed

## 🛠️ Troubleshooting Guide

### Level 1: Quick Health Checks

```bash
# 1. Check cluster status
kubectl cluster-info

# 2. Check all nodes
kubectl get nodes -o wide

# 3. Check all pods
kubectl get pods --all-namespaces

# 4. Check services
kubectl get services --all-namespaces
```

### Level 2: Pod Diagnostics

```bash
# 1. Pod details
kubectl describe pod <pod-name>

# 2. Pod logs (current)
kubectl logs <pod-name>

# 3. Pod logs (previous)
kubectl logs <pod-name> --previous

# 4. Interactive shell
kubectl exec -it <pod-name> -- /bin/bash

# 5. Port forwarding for testing
kubectl port-forward pod/<pod-name> 5000:5000
```

### Level 3: Service & Network Issues

```bash
# 1. Service details
kubectl describe service <service-name>

# 2. Check endpoints
kubectl get endpoints <service-name>

# 3. Test DNS resolution
kubectl exec -it <pod-name> -- nslookup <service-name>

# 4. Network connectivity test
kubectl exec -it <pod-name> -- curl http://<service-name>:<port>
```

### Level 4: Storage Issues

```bash
# 1. Check PV status
kubectl get pv

# 2. Check PVC status
kubectl get pvc

# 3. PV details
kubectl describe pv <pv-name>

# 4. PVC details
kubectl describe pvc <pvc-name>
```

### Level 5: System Events & Logs

```bash
# 1. Recent events
kubectl get events --sort-by='.lastTimestamp'

# 2. Node events
kubectl get events --field-selector source.host=<node-name>

# 3. System pod logs
kubectl logs -n kube-system <system-pod-name>

# 4. Kubelet logs (on node)
sudo journalctl -u kubelet -f
```

### Common Error Patterns & Solutions

| Error                | Cause                            | Solution                           |
| -------------------- | -------------------------------- | ---------------------------------- |
| `ImagePullBackOff`   | Wrong image name/registry issues | Check image name, push to registry |
| `CrashLoopBackOff`   | App crashes on startup           | Check logs, fix application code   |
| `Pending`            | Resource/scheduling issues       | Check node resources, taints       |
| `ContainerCreating`  | Storage/network issues           | Check PV/PVC, network policies     |
| `NodeNotReady`       | Node issues                      | Check kubelet, docker daemon       |
| `ServiceUnavailable` | Service/endpoint issues          | Check selectors, pod labels        |

## ✅ Testing & Validation

### Functional Testing

```bash
# 1. Test database connectivity
kubectl exec -it <mysql-pod> -- mysql -u root -p -e "SELECT 1;"

# 2. Test Flask app response
curl http://<worker-node-ip>:30007

# 3. Test complete workflow
# Add user via web interface
# Verify data in database
```

### Performance Testing

```bash
# 1. Load testing with ab (Apache Bench)
ab -n 1000 -c 10 http://<worker-node-ip>:30007/

# 2. Monitor resource usage during load
kubectl top pods
kubectl top nodes

# 3. Check application metrics
kubectl logs <flask-pod> | grep -i error
```

### Integration Testing

```bash
# 1. Test service discovery
kubectl exec -it <flask-pod> -- nslookup mysql-service

# 2. Test database operations
# Create, Read, Update, Delete operations through web interface

# 3. Test failover scenarios
kubectl delete pod <mysql-pod>
# Verify automatic restart and data persistence
```

## 📊 Monitoring & Maintenance

### Daily Monitoring Tasks

```bash
# 1. Check cluster health
kubectl get nodes
kubectl get pods --all-namespaces

# 2. Check resource utilization
kubectl top nodes
kubectl top pods --all-namespaces

# 3. Check for events/issues
kubectl get events --sort-by='.lastTimestamp' | tail -20
```

### Weekly Maintenance Tasks

```bash
# 1. Update application images
kubectl set image deployment/two-tier-app-deployment two-tier-app=<new-image>

# 2. Check persistent storage usage
kubectl exec -it <mysql-pod> -- df -h

# 3. Review and clean up old resources
kubectl get pods --all-namespaces | grep -v Running
```

### Backup Procedures

```bash
# 1. Database backup
kubectl exec -it <mysql-pod> -- mysqldump -u root -p two_tier_db > backup.sql

# 2. Kubernetes configuration backup
kubectl get all --all-namespaces -o yaml > cluster-backup.yaml

# 3. Persistent volume backup (depends on storage provider)
# For hostPath volumes, backup the node directory
```

## 🎓 Lessons Learned

### Technical Insights

1. **Always Use Resource Limits**: Define CPU and memory requests/limits to prevent resource starvation.

2. **Implement Health Checks**: Add readiness and liveness probes for better reliability.

3. **Use ConfigMaps/Secrets**: Don't hardcode configuration in manifests.

4. **Plan Storage Strategy**: Understand persistent volume types and their implications.

5. **Network Security**: Properly configure security groups and network policies.

### Operational Best Practices

1. **Infrastructure as Code**: Version control all Kubernetes manifests.

2. **Monitoring First**: Set up monitoring before production deployment.

3. **Staged Deployments**: Use development → staging → production pipeline.

4. **Documentation**: Maintain updated deployment and troubleshooting guides.

5. **Backup Strategy**: Regular backups of data and configurations.

### Development Workflow

1. **Local Development**: Use Docker Compose for local development.

2. **CI/CD Pipeline**: Automate build, test, and deployment processes.

3. **Testing Strategy**: Unit tests → Integration tests → End-to-end tests.

4. **Version Management**: Use semantic versioning for applications and infrastructure.

## 📚 Additional Resources

### Documentation

- [Kubernetes Official Documentation](https://kubernetes.io/docs/)
- [Flask Documentation](https://flask.palletsprojects.com/)
- [MySQL Documentation](https://dev.mysql.com/doc/)
- [AWS EKS User Guide](https://docs.aws.amazon.com/eks/)

### Tools & Utilities

- [kubectl Cheat Sheet](https://kubernetes.io/docs/reference/kubectl/cheatsheet/)
- [Helm Package Manager](https://helm.sh/)
- [K9s - Kubernetes CLI](https://k9scli.io/)
- [Lens - Kubernetes IDE](https://k8slens.dev/)

### Learning Resources

- [Kubernetes the Hard Way](https://github.com/kelseyhightower/kubernetes-the-hard-way)
- [Play with Kubernetes](https://labs.play-with-k8s.com/)
- [Kubernetes Academy](https://kubernetes.academy/)

## 🤝 Contributing

Feel free to contribute to this project by:

- Reporting issues
- Suggesting improvements
- Submitting pull requests
- Sharing your deployment experiences

## 📄 License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.

---

**Project Status**: ✅ **Successfully Deployed and Running**

**Last Updated**: July 23, 2025

**Deployment Environment**: AWS EC2 + Kubernetes

**Application Access**: `http://<worker-node-ip>:30007`

---

_This documentation represents our journey from concept to production deployment of a two-tier Flask application on Kubernetes. Every challenge faced and solution implemented has been documented to help others in their Kubernetes journey._
