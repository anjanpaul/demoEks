# AWS CLI, eksctl, and kubectl Installation and EKS Cluster Setup Guide

## 📌 Prerequisites
- AWS account with necessary IAM permissions (AdministratorAccess recommended for initial setup)
- macOS or Linux terminal access
- Internet connectivity

---

## 1. AWS CLI Installation and Configuration

### ✅ Install AWS CLI on macOS:
```bash
curl "https://awscli.amazonaws.com/AWSCLIV2.pkg" -o "AWSCLIV2.pkg"
sudo installer -pkg AWSCLIV2.pkg -target /
```

### ✅ Verify AWS CLI installation:
```bash
which aws
aws --version
```

### ✅ Configure AWS CLI:
```bash
aws configure
```
You will be prompted for:
- AWS Access Key ID
- AWS Secret Access Key
- Default region name (e.g., `us-east-1`)
- Default output format (e.g., `json`)

**Official AWS CLI installation guide:**  
https://docs.aws.amazon.com/cli/latest/userguide/getting-started-install.html

---

## 2. Install eksctl (EKS Cluster Management Tool)

### ✅ Download and install eksctl (Linux or macOS - AMD64 systems):
```bash
ARCH=amd64
PLATFORM=$(uname -s)_$ARCH

curl -sLO "https://github.com/eksctl-io/eksctl/releases/latest/download/eksctl_$PLATFORM.tar.gz"

# (Optional) Verify checksum
curl -sL "https://github.com/eksctl-io/eksctl/releases/latest/download/eksctl_checksums.txt" | grep $PLATFORM | sha256sum --check

# Extract and move binary
tar -xzf eksctl_$PLATFORM.tar.gz -C /tmp && rm eksctl_$PLATFORM.tar.gz
sudo mv /tmp/eksctl /usr/local/bin
```

### ✅ Verify eksctl installation:
```bash
eksctl version
```

**Official eksctl installation guide:**  
https://github.com/eksctl-io/eksctl/releases

---

## 3. Install kubectl (Kubernetes CLI)

### ✅ Follow AWS official guide:  
https://docs.aws.amazon.com/eks/latest/userguide/install-kubectl.html

Example for macOS (using Homebrew):
```bash
brew install kubectl
```

### ✅ Verify kubectl installation:
```bash
kubectl version --client
```

---

## 4. AWS VPC Setup for EKS Cluster

For high availability, create a VPC with:

✅ 3 Public Subnets (across 3 Availability Zones)  
✅ 3 Private Subnets (across 3 Availability Zones)  
✅ NAT Gateway for internet access from private subnets

You can create the VPC using AWS Console or use the **EKS VPC Quick Start CloudFormation template**.

---

## 5. EKS Cluster Configuration Using eksctl

### ✅ Create a `cluster.yaml` file:
```bash
touch cluster.yaml
```

### ✅ Example `cluster.yaml` Configuration:
```yaml
apiVersion: eksctl.io/v1alpha5
kind: ClusterConfig

metadata:
  name: my-cluster
  region: us-east-1
  version: "1.28"

iam:
  withOIDC: true

vpc:
  id: "vpc-12345678"
  subnets:
      us-east-1a:
        id: "subnet-11111111"
      us-east-1b:
        id: "subnet-22222222"
      us-east-1c:
        id: "subnet-33333333"

managedNodeGroups:
  - name: ng-main
    amiFamily: AmazonLinux2
    instanceType: t3.medium
    minSize: 2
    maxSize: 6
    desiredCapacity: 2
    volumeSize: 60
    privateNetworking: true
    availabilityZones: ["us-east-1a", "us-east-1b", "us-east-1c"]
    ssh:
      allow: true
    labels:
      environment: production
    iam:
      withAddonPolicies:
        autoScaler: true
        awsLoadBalancerController: true
        ebs: true
        efs: true

addons:
  - name: vpc-cni
  - name: eks-pod-identity-agent
```

---

## 6. Create the EKS Cluster

```bash
eksctl create cluster -f cluster.yaml
```

This command will provision the EKS cluster with the specified VPC, subnets, node groups, IAM roles, and EKS addons.

---

## ✅ Final Validation:

### Check EKS Cluster Nodes:
```bash
kubectl get nodes
```

If nodes are in `Ready` state, your EKS cluster is successfully deployed!

---

## ✅ Summary:
This guide covers:

- ✅ Installing AWS CLI
- ✅ Installing eksctl
- ✅ Installing kubectl
- ✅ Setting up AWS IAM and VPC
- ✅ Creating an EKS cluster using eksctl and your custom config

---

## 📚 Useful Links:
- [AWS CLI Getting Started](https://docs.aws.amazon.com/cli/latest/userguide/getting-started-install.html)
- [eksctl Official Releases](https://github.com/eksctl-io/eksctl/releases)
- [kubectl Installation Guide](https://docs.aws.amazon.com/eks/latest/userguide/install-kubectl.html)
- [Amazon EKS User Guide](https://docs.aws.amazon.com/eks/latest/userguide/what-is-eks.html)
