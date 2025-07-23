# Helm Deployment Troubleshooting Guide: Two-Tier Flask Application

[![Helm](https://img.shields.io/badge/Helm-0F1689?style=for-the-badge&logo=helm&logoColor=white)](https://helm.sh/)
[![Kubernetes](https://img.shields.io/badge/Kubernetes-326CE5?style=for-the-badge&logo=kubernetes&logoColor=white)](https://kubernetes.io/)
[![Flask](https://img.shields.io/badge/Flask-000000?style=for-the-badge&logo=flask&logoColor=white)](https://flask.palletsprojects.com/)
[![MySQL](https://img.shields.io/badge/MySQL-4479A1?style=for-the-badge&logo=mysql&logoColor=white)](https://www.mysql.com/)

## 📋 Table of Contents

- [Quick Start Guide](#-quick-start-guide)
- [Overview](#-overview)
- [Initial Problem](#-initial-problem)
- [Environment Setup](#-environment-setup)
- [Troubleshooting Journey](#-troubleshooting-journey)
- [Root Cause Analysis](#-root-cause-analysis)
- [Solution Implementation](#-solution-implementation)
- [Verification Steps](#-verification-steps)
- [Lessons Learned](#-lessons-learned)
- [Command Reference](#-command-reference)

## 🚀 Quick Start Guide

**Want to deploy this application on your Kubernetes cluster? Follow these simple steps!**

### Prerequisites ✅

Before you begin, ensure you have:

- ✅ **Kubernetes cluster** running (master + worker nodes)
- ✅ **kubectl** configured and connected to your cluster
- ✅ **Helm 3.x** installed
- ✅ **Docker** (if you want to build custom images)

### Step 1: Verify Your Environment

```bash
# Check if kubectl is working
kubectl get nodes
# Expected: Shows your master and worker nodes in 'Ready' state

# Check if Helm is installed
helm version
# Expected: Shows Helm v3.x.x

# Check cluster info
kubectl cluster-info
# Expected: Shows your cluster endpoints
```

### Step 2: Clone/Download This Repository

```bash
# Clone the repository
git clone https://github.com/deepanshub9/two-tier-flask-app.git
cd two-tier-flask-app

# Or create the Helm charts manually (structure shown below)
```

### Step 3: Create Helm Charts Structure

If you don't have the charts, create them:

```bash
# Create MySQL chart
helm create mysql-chart
cd mysql-chart

# Create Flask chart
helm create flask-front-chart
cd flask-front-chart
```
---
### Just Ensure all the files of your Flask and mysql app is properly mention everthings(Variable, Value.yaml, Comment livenessProbe and ReadnessProb) and then last package all of them.
---
### Step 4: Deploy the Application

**Deploy MySQL first (databases should start before applications):**

```bash
# Navigate to your project directory
cd /path/to/your/charts

# Deploy MySQL chart
helm install mysql-chart ./mysql-chart

# Wait for MySQL to be ready (this is important!)
kubectl wait --for=condition=ready pod -l app.kubernetes.io/name=mysql-chart --timeout=300s
```

**Verify MySQL is running:**

```bash
# Check MySQL pod status
kubectl get pods -l app.kubernetes.io/name=mysql-chart

# Expected output:
# NAME                           READY   STATUS    RESTARTS   AGE
# mysql-chart-xxxx-xxxx          1/1     Running   0          2m

# Check MySQL logs
kubectl logs -l app.kubernetes.io/name=mysql-chart --tail=10
# Should show: "ready for connections"
```

**Deploy Flask application:**

```bash
# Deploy Flask chart
helm install flask-front-chart ./flask-front-chart

# Wait for Flask to be ready
kubectl wait --for=condition=ready pod -l app.kubernetes.io/name=flask-front-chart --timeout=300s
```

### Step 5: Verify Deployment

```bash
# Check all resources
kubectl get all

# Expected output:
# NAME                                    READY   STATUS    RESTARTS   AGE
# pod/flask-front-chart-xxxx-xxxx         1/1     Running   0          1m
# pod/mysql-chart-xxxx-xxxx               1/1     Running   0          3m
#
# NAME                        TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)        AGE
# service/flask-front-chart   NodePort    10.xxx.xxx.xxx   <none>        80:30007/TCP   1m
# service/mysql-chart         ClusterIP   10.xxx.xxx.xxx   <none>        3306/TCP       3m

# Check Helm releases
helm list
# Should show both charts as 'deployed'
```

### Step 6: Access Your Application

**Get your node's external IP:**

```bash
# If on cloud (AWS/GCP/Azure):
curl -s http://checkip.amazonaws.com  # For AWS
# Or check your cloud console for external IP

# If on-premises or local:
kubectl get nodes -o wide
# Use the EXTERNAL-IP or INTERNAL-IP of any node
```

**Open in browser:**

```bash
# Replace <NODE-IP> with your actual node IP
http://<NODE-IP>:30007
```

### Step 7: Test Database Functionality

```bash
# Test MySQL connection
kubectl exec -it deployment/mysql-chart -- mysql -u flaskuser -pflaskpass -D flaskdb -e "SELECT 1 as test;"

# Expected output:
# +------+
# | test |
# +------+
# |    1 |
# +------+
```

## 🔧 Troubleshooting Common Issues

### Issue 1: Pods Stuck in Pending

```bash
# Check node resources
kubectl describe nodes
kubectl top nodes  # If metrics-server is installed

# Solution: Ensure nodes have sufficient CPU/Memory
```

### Issue 2: MySQL Pod CrashLoopBackOff

```bash
# Check logs
kubectl logs -l app.kubernetes.io/name=mysql-chart

# Common causes:
# 1. HTTP probes on MySQL (remove them)
# 2. Insufficient resources (add resource limits)
# 3. Wrong environment variables
```

### Issue 3: Flask Can't Connect to MySQL

```bash
# Check service DNS resolution
kubectl exec -it deployment/flask-front-chart -- nslookup mysql-chart

# Check environment variables
kubectl describe pod -l app.kubernetes.io/name=flask-front-chart | grep -A 10 Environment

# Ensure MySQL service name matches MYSQL_HOST env variable
```

### Issue 4: Application Not Accessible

```bash
# Check NodePort service
kubectl get svc flask-front-chart

# Check security groups (if on cloud)
# Ensure port 30007 is open in firewall/security groups

# Test locally first
kubectl port-forward svc/flask-front-chart 8080:80
# Then access: http://localhost:8080
```

## 🎯 Quick Commands Summary

```bash
# One-time setup
kubectl get nodes                           # Verify cluster
helm version                               # Verify Helm

# Deploy application
helm install mysql-chart ./mysql-chart     # Deploy database
helm install flask-front-chart ./flask-front-chart  # Deploy app

# Monitor deployment
kubectl get pods                           # Check pod status
kubectl get svc                           # Check services
helm list                                 # Check Helm releases

# Troubleshoot
kubectl logs <pod-name>                   # Check logs
kubectl describe pod <pod-name>           # Detailed info
kubectl get events --sort-by='.lastTimestamp'  # Recent events

# Clean up
helm uninstall flask-front-chart         # Remove Flask
helm uninstall mysql-chart              # Remove MySQL
kubectl get all                         # Verify cleanup
```



## 🎯 Overview

This is issue i faced when i tried to deploy this application the complete troubleshooting process for deploying a two-tier Flask application using Helm charts on Kubernetes. The application consists of:

**Duration**: ~2 hours of troubleshooting  
**Final Status**: ✅ Successfully Deployed

## 🚨 Initial Problem

### Symptoms Observed

```bash
# Initial check revealed pods in CrashLoopBackOff state
$ kubectl get pods
NAME                                READY   STATUS             RESTARTS        AGE
flask-front-chart-95964856d-2vq5x   0/1     CrashLoopBackOff   15 (2m4s ago)   53m
mysql-chart-75975bdb6c-hzdl5        0/1     CrashLoopBackOff   15 (96s ago)    34m
```

### Error Messages

**Flask Application Error:**

```
MySQLdb.OperationalError: (2005, "Unknown server host 'msql-chart' (-2)")
```

**MySQL Pod Errors:**

```
Warning  Unhealthy  4m28s  kubelet  Liveness probe failed: Get "http://192.168.144.162:3306/": malformed HTTP response "I\x00\x00\x00"
Warning  Unhealthy  4m28s  kubelet  Readiness probe failed: Get "http://192.168.144.162:3306/": malformed HTTP response "I\x00\x00\x00"
```

## 🛠️ Environment Setup

### Infrastructure Details

```bash
# Kubernetes Cluster Status
$ kubectl cluster-info
Kubernetes control plane is running at https://172.31.27.130:6443
CoreDNS is running at https://172.31.27.130:6443/api/v1/namespaces/kube-system/services/kube-dns:dns/proxy

# Node Information
$ kubectl get nodes -o wide
NAME               STATUS   ROLES           AGE   VERSION    INTERNAL-IP     EXTERNAL-IP   OS-IMAGE             KERNEL-VERSION   CONTAINER-RUNTIME
ip-172-31-27-130   Ready    control-plane   37h   v1.29.15   172.31.27.130   <none>        Ubuntu 24.04.2 LTS   6.8.0-1031-aws   containerd://1.7.27
ip-172-31-28-223   Ready    <none>          37h   v1.29.15   172.31.28.223   <none>        Ubuntu 24.04.2 LTS   6.8.0-1031-aws   containerd://1.7.27

# Helm Version
$ helm version
version.BuildInfo{Version:"v3.18.4", GitCommit:"d80839cf37d860c8aa9a0503fe463278f26cd5e2", GitTreeState:"clean", GoVersion:"go1.24.4"}
```

### Project Structure

```bash
$ ls -la flask-app/
total 32
drwxrwxr-x 4 ubuntu ubuntu 4096 Jul 23 21:26 .
drwxr-x--- 8 ubuntu ubuntu 4096 Jul 23 21:45 ..
drwxr-xr-x 4 ubuntu ubuntu 4096 Jul 23 21:09 flask-front-chart
-rw-rw-r-- 1 ubuntu ubuntu 4340 Jul 23 21:26 flask-front-chart-0.1.0.tgz
drwxr-xr-x 4 ubuntu ubuntu 4096 Jul 23 21:45 mysql-chart
-rw-rw-r-- 1 ubuntu ubuntu 4309 Jul 23 20:39 mysql-chart-0.1.0.tgz
```

## 🔍 Troubleshooting Journey

### Step 1: Initial Assessment

#### Check Helm Deployments

```bash
$ helm list --all-namespaces
NAME                    NAMESPACE       REVISION        UPDATED                                         STATUS    CHART                   APP VERSION
mysql-chart             default         1               2025-07-23 21:45:58.333123379 +0000 UTC        deployed  mysql-chart-0.1.0      1.16.0
```

#### Check Pod Status

```bash
$ kubectl get pods
NAME                                READY   STATUS             RESTARTS        AGE
flask-front-chart-95964856d-2vq5x   0/1     CrashLoopBackOff   15 (2m4s ago)   53m
mysql-chart-75975bdb6c-hzdl5        0/1     CrashLoopBackOff   15 (96s ago)    34m
```

**🔍 Analysis**: Both applications are failing to start properly.

### Step 2: Log Analysis

#### Flask Application Logs

```bash
$ kubectl logs flask-front-chart-95964856d-2vq5x
Traceback (most recent call last):
  File "/app/app.py", line 46, in <module>
    init_db()
  File "/app/app.py", line 18, in init_db
    cur = mysql.connection.cursor()
  File "/usr/local/lib/python3.9/site-packages/flask_mysqldb/__init__.py", line 94, in connection
    ctx.mysql_db = self.connect
  File "/usr/local/lib/python3.9/site-packages/flask_mysqldb/__init__.py", line 81, in connect
    return MySQLdb.connect(**kwargs)
  File "/usr/local/lib/python3.9/site-packages/MySQLdb/__init__.py", line 121, in Connect
    return Connection(*args, **kwargs)
  File "/usr/local/lib/python3.9/site-packages/MySQLdb/connections.py", line 200, in __init__
    super().__init__(*args, **kwargs2)
MySQLdb.OperationalError: (2005, "Unknown server host 'msql-chart' (-2)")
```

**🔍 Analysis**: Flask app can't resolve MySQL hostname. Initially suspected typo in hostname.

#### MySQL Application Logs

```bash
$ kubectl logs mysql-chart-75975bdb6c-hzdl5
2025-07-23 22:18:20+00:00 [Note] [Entrypoint]: Entrypoint script for MySQL Server 9.4.0-1.el9 started.
2025-07-23 22:18:20+00:00 [Note] [Entrypoint]: Switching to dedicated user 'mysql'
2025-07-23 22:18:20+00:00 [Note] [Entrypoint]: Initializing database files
2025-07-23T22:18:20.654295Z 0 [System] [MY-015017] [Server] MySQL Server Initialization - start.
...
2025-07-23T22:18:32.461235Z 0 [Warning] [MY-010068] [Server] CA certificate ca.pem is self signed.
2025-07-23T22:18:32.461281Z 0 [System] [MY-013602] [Server] Channel mysql_main configured to support TLS.
2025-07-23T22:18:49.831650Z 0 [System] [MY-015016] [Server] MySQL Server - end.
```

**🔍 Analysis**: MySQL starts but then terminates unexpectedly.

### Step 3: Configuration Analysis

#### Check Service Names

```bash
$ kubectl get services
NAME                TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)        AGE
flask-front-chart   NodePort    10.98.60.249     <none>        80:30007/TCP   92s
kubernetes          ClusterIP   10.96.0.1        <none>        443/TCP        23h
mysql-chart         ClusterIP   10.108.93.170    <none>        3306/TCP       78s
```

**🔍 Analysis**: Service names are correct. The issue is not hostname resolution.

#### Check Flask Chart Configuration

```bash
$ cat flask-front-chart/values.yaml
# Key configuration sections:
env:
  mysqlhost: mysql-chart    # ✅ Correct hostname
  mysqlpw: flaskpass       # Password configuration
  mysqluser: flaskuser     # User configuration
  mysqldb: flaskdb         # Database name
```

#### Check MySQL Chart Configuration

```bash
$ cat mysql-chart/values.yaml
# Key configuration sections:
env:
  mysqlrootpw: rootpassword
  mysqldb: flaskdb         # ✅ Matches Flask config
  mysqluser: flaskuser     # ✅ Matches Flask config
  mysqlpass: flaskpass     # ✅ Matches Flask config
```

**🔍 Analysis**: Configuration looks correct. The problem must be elsewhere.

### Step 4: Deep Dive - Pod Description

#### MySQL Pod Deep Analysis

```bash
$ kubectl describe pod mysql-chart-75975bdb6c-hzdl5
Name:             mysql-chart-75975bdb6c-hzdl5
Namespace:        default
...
Events:
  Type     Reason     Age                    From               Message
  ----     ------     ----                   ----               -------
  Warning  Unhealthy  4m28s                  kubelet            Liveness probe failed: Get "http://192.168.144.162:3306/": malformed HTTP response "I\x00\x00\x00"
  Warning  Unhealthy  4m28s                  kubelet            Readiness probe failed: Get "http://192.168.144.162:3306/": malformed HTTP response "I\x00\x00\x00"
  Normal   Killing    4m28s                  kubelet            Container mysql-chart failed liveness probe, will be restarted
```

#### Flask Pod Deep Analysis

```bash
$ kubectl describe pod flask-front-chart-df4d8cc69-hw74j
Name:             flask-front-chart-df4d8cc69-hw74j
...
Environment:
  MYSQL_HOST:      mysql-chart     # ✅ Correct
  MYSQL_PASSWORD:  flaskpass       # ✅ Correct
  MYSQL_USER:      flaskuser       # ✅ Correct
  MYSQL_DB:        flaskdb         # ✅ Correct
Events:
  Warning  Unhealthy  2m16s (x2 over 2m40s)  kubelet  Readiness probe failed: Get "http://192.168.144.163:5000/": dial tcp 192.168.144.163:5000: connect: connection refused
```

## 🎯 Root Cause Analysis

### The Critical Discovery

The key breakthrough came when analyzing the MySQL pod events:

```bash
Warning  Unhealthy  4m28s  kubelet  Liveness probe failed: Get "http://192.168.144.162:3306/": malformed HTTP response "I\x00\x00\x00"
```

**🚨 ROOT CAUSE IDENTIFIED**:

The MySQL Helm chart was configured with **HTTP health probes** trying to connect to port 3306, but **MySQL only supports TCP connections**. The Kubernetes liveness and readiness probes were continuously failing, causing the container to restart in an endless loop.

### Why This Happened

1. **Default Helm Template**: The MySQL chart was generated with default HTTP probe templates
2. **Protocol Mismatch**: HTTP probes don't work with MySQL's TCP protocol
3. **Cascading Failure**: MySQL instability prevented Flask app from connecting

### Error Pattern Analysis

```bash
# MySQL was receiving HTTP requests on TCP port 3306
# This caused malformed HTTP response errors:
"malformed HTTP response 'I\x00\x00\x00'"

# The 'I\x00\x00\x00' is the beginning of MySQL's TCP handshake
# being interpreted as an invalid HTTP response
```

## ✅ Solution Implementation

### Step 1: Uninstall Broken Deployments

```bash
# Remove both failing deployments
$ helm uninstall mysql-chart
release "mysql-chart" uninstalled

$ helm uninstall flask-front-chart
release "flask-front-chart" uninstalled

# Verify cleanup
$ kubectl get pods
No resources found in default namespace.
```

### Step 2: Fix MySQL Chart Configuration

#### Create Backup

```bash
$ cp mysql-chart/values.yaml mysql-chart/values.yaml.backup
```

#### Remove Problematic Probes

```bash
# Comment out HTTP probes that don't work with MySQL
$ sed -i '/livenessProbe:/,/port: http/s/^/#/' mysql-chart/values.yaml
$ sed -i '/readinessProbe:/,/port: http/s/^/#/' mysql-chart/values.yaml
```

#### Verify Changes

```bash
$ grep -A 5 -B 2 "livenessProbe\|readinessProbe" mysql-chart/values.yaml
# This is to setup the liveness and readiness probes more information can be found here: https://kubernetes.io/docs/tasks/configure-pod-container/configure-liveness-readiness-startup-probes/
#livenessProbe:
#  httpGet:
#    path: /
#    port: http
#readinessProbe:
#  httpGet:
#    path: /
#    port: http
```

### Step 3: Deploy Fixed MySQL Chart

```bash
$ helm install mysql-chart ./mysql-chart
NAME: mysql-chart
LAST DEPLOYED: Wed Jul 23 22:31:16 2025
NAMESPACE: default
STATUS: deployed
REVISION: 1
```

#### Verify MySQL Deployment

```bash
$ kubectl get pods -l app.kubernetes.io/name=mysql-chart
NAME                           READY   STATUS    RESTARTS   AGE
mysql-chart-5b58cb87dc-jvx4z   1/1     Running   0          8s
```

**🎉 SUCCESS**: MySQL pod is now `1/1 Running` with no restarts!

#### Check MySQL Logs

```bash
$ kubectl logs mysql-chart-5b58cb87dc-jvx4z --tail=10
2025-07-23T22:31:28.898496Z 0 [System] [MY-010931] [Server] /usr/sbin/mysqld: ready for connections. Version: '9.4.0'  socket: '/var/run/mysqld/mysqld.sock'  port: 3306  MySQL Community Server - GPL.
```

**🎉 SUCCESS**: MySQL is stable and "ready for connections"!

### Step 4: Test Database Connectivity

```bash
$ kubectl exec -it mysql-chart-5b58cb87dc-jvx4z -- mysql -u flaskuser -pflaskpass -D flaskdb -e "SELECT 1 as test;"
mysql: [Warning] Using a password on the command line interface can be insecure.
+------+
| test |
+------+
|    1 |
+------+
```

**🎉 SUCCESS**: Database authentication and connectivity working perfectly!

### Step 5: Deploy Flask Application

```bash
$ helm install flask-front-chart ./flask-front-chart
NAME: flask-front-chart
LAST DEPLOYED: Wed Jul 23 22:31:53 2025
NAMESPACE: default
STATUS: deployed
REVISION: 1
```

#### Verify Flask Deployment

```bash
$ kubectl get pods -l app.kubernetes.io/name=flask-front-chart
NAME                                READY   STATUS    RESTARTS   AGE
flask-front-chart-df4d8cc69-bbvpf   1/1     Running   0          7s
```

**🎉 SUCCESS**: Flask pod is also `1/1 Running`!

## ✅ Verification Steps

### Check All Resources

```bash
$ kubectl get all
NAME                                    READY   STATUS    RESTARTS   AGE
pod/flask-front-chart-df4d8cc69-bbvpf   1/1     Running   0          15s
pod/mysql-chart-5b58cb87dc-jvx4z        1/1     Running   0          53s

NAME                        TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)        AGE
service/flask-front-chart   NodePort    10.103.231.213   <none>        80:30007/TCP   15s
service/kubernetes          ClusterIP   10.96.0.1        <none>        443/TCP        23h
service/mysql-chart         ClusterIP   10.101.98.253    <none>        3306/TCP       53s

NAME                                READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/flask-front-chart   1/1     1            1           15s
deployment.apps/mysql-chart         1/1     1            1           53s

NAME                                          DESIRED   CURRENT   READY   AGE
replicaset.apps/flask-front-chart-df4d8cc69   1         1         1       15s
replicaset.apps/mysql-chart-5b58cb87dc        1         1         1       53s
```

### Check Helm Releases

```bash
$ helm list --all-namespaces
NAME                    NAMESPACE       REVISION        UPDATED                                 STATUS      CHART                    APP VERSION
flask-front-chart       default         1               2025-07-23 22:31:53.822201278 +0000 UTC deployed    flask-front-chart-0.1.0  1.16.0
mysql-chart             default         1               2025-07-23 22:31:16.361373849 +0000 UTC deployed    mysql-chart-0.1.0        1.16.0
```

### Test External Access

```bash
# Get external IP
$ curl -s http://checkip.amazonaws.com
54.164.122.182

# Test application access (this worked for the user)
$ curl -I http://54.164.122.182:30007
# Application responded successfully! ✅
```


## 🎓 Lessons Learned

### Technical Lessons

1. **Protocol Awareness**: Always match health check protocols with application protocols

   - ❌ HTTP probes on MySQL (TCP-based)
   - ✅ TCP probes on MySQL or no probes
   - ✅ HTTP probes on web applications

2. **Helm Chart Generation**: Default Helm templates may not suit all applications

   - Review generated templates carefully
   - Customize probes based on application type

3. **Progressive Debugging**: Follow a systematic approach

   - Check pod status first
   - Analyze logs second
   - Examine configuration third
   - Deep dive into events and descriptions

4. **Service Dependencies**: Deploy database services before application services
   - MySQL must be stable before Flask can connect
   - Use `helm install` order: database → application

### Operational Lessons

1. **Log Analysis is Critical**: Detailed error messages revealed the exact issue
2. **Events Tell the Story**: Kubernetes events showed the probe failures clearly
3. **Backup Configurations**: Always backup before making changes
4. **Test Each Layer**: Database connectivity, service discovery, application logic

### Helm-Specific Lessons

1. **Chart Dependencies**: Understand how your charts interact
2. **Values Validation**: Ensure environment variables match between charts
3. **Template Review**: Don't trust generated templates blindly
4. **Incremental Deployment**: Deploy one chart at a time for easier debugging

## 📚 Command Reference

### Diagnostic Commands Used

```bash
# Basic status checks
kubectl get nodes
kubectl get pods
kubectl get services
kubectl get all
helm list --all-namespaces

# Detailed analysis
kubectl describe pod <pod-name>
kubectl logs <pod-name>
kubectl logs <pod-name> --tail=20
kubectl logs <pod-name> --previous

# Configuration checks
cat <chart>/values.yaml
kubectl get pods -o wide
kubectl get services -o wide

# Testing connectivity
kubectl exec -it <pod-name> -- <command>
curl -I http://<ip>:<port>
curl -s http://checkip.amazonaws.com

# Helm operations
helm install <release-name> <chart-path>
helm uninstall <release-name>
helm list
```

### File Operations Used

```bash
# Backup files
cp mysql-chart/values.yaml mysql-chart/values.yaml.backup

# Edit configurations
sed -i '/livenessProbe:/,/port: http/s/^/#/' mysql-chart/values.yaml
sed -i '/readinessProbe:/,/port: http/s/^/#/' mysql-chart/values.yaml

# Verify changes
grep -A 5 -B 2 "livenessProbe\|readinessProbe" mysql-chart/values.yaml
```

## 🚀 Success Metrics

### Before Fix

- ❌ MySQL: `CrashLoopBackOff` (15 restarts in 34 minutes)
- ❌ Flask: `CrashLoopBackOff` (15 restarts in 53 minutes)
- ❌ Application: Inaccessible
- ❌ Database: Connection failures

### After Fix

- ✅ MySQL: `1/1 Running` (0 restarts)
- ✅ Flask: `1/1 Running` (0 restarts)
- ✅ Application: Accessible via browser on port 30007
- ✅ Database: Full connectivity and authentication working
- ✅ Helm: Both charts deployed successfully

### Time to Resolution

- **Total Duration**: ~2 hours
- **Root Cause Identification**: ~1.5 hours
- **Fix Implementation**: ~30 minutes
- **Verification**: ~5 minutes

## 🎯 Final Status

```bash
# Final verification - All systems operational
$ kubectl get all
NAME                                    READY   STATUS    RESTARTS   AGE
pod/flask-front-chart-df4d8cc69-bbvpf   1/1     Running   0          ✅
pod/mysql-chart-5b58cb87dc-jvx4z        1/1     Running   0          ✅

NAME                        TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)        AGE
service/flask-front-chart   NodePort    10.103.231.213   <none>        80:30007/TCP   ✅
service/mysql-chart         ClusterIP   10.101.98.253    <none>        3306/TCP       ✅

$ helm list
NAME              NAMESPACE  REVISION  UPDATED                                STATUS    CHART                    APP VERSION
flask-front-chart default    1         2025-07-23 22:31:53.822201278 +0000   deployed  flask-front-chart-0.1.0  1.16.0    ✅
mysql-chart       default    1         2025-07-23 22:31:16.361373849 +0000   deployed  mysql-chart-0.1.0        1.16.0    ✅

# User confirmation: "Yes thankyou so much my application is working and visible on port." ✅
```

---

**Troubleshooting completed successfully on July 23, 2025**  
**Application Status**: 🟢 FULLY OPERATIONAL  
**Access URL**: `http://<worker-node-ip>:30007`

_This guide serves as a complete reference for troubleshooting similar Helm deployment issues with multi-tier applications._
