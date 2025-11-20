# 🚀 **Building Production-Grade Azure Infrastructure with Terraform (with Auto-scaling)**

This project demonstrates how to build a **fully automated, production-grade Azure infrastructure** using **Terraform**, complete with:

* Virtual Network & Subnet
* NAT Gateway
* Network Security Groups
* Public IP
* Azure Standard Load Balancer
* Virtual Machine Scale Set (VMSS)
* Apache web server provisioning
* CPU-based Autoscaling

This is the same architecture used in real-world enterprise environments for scalable and resilient web applications.


## 📌 **Architecture Overview**

Below is the full architecture diagram used in this project:

<img width="900" height="550" alt="Project drawio" src="https://github.com/user-attachments/assets/57fa3d1c-6c13-446a-a77e-8fad78070079" />



### 🔷 **Key Components**

#### **1. Terraform Backend (State Storage)**

Terraform uses an **Azure Blob Storage** container to store its state file.
This ensures:

* Safe remote locking
* State consistency
* Team collaboration
* No drift in infrastructure


### **2. Terraform Deployment (Admin → Terraform → Azure)**

Terraform code is executed either:

* Through **Azure DevOps Pipeline** (CI/CD), or
* Locally on your machine (optional)

A single pipeline run deploys all Azure resources automatically.


### **3. Azure Resource Group**

All components of the infrastructure are deployed inside a single resource group for better lifecycle management and cleanup.


## 🌐 **Networking Layer**

### **4. Virtual Network (VNet) + Subnet**

A dedicated Azure VNet isolates the environment.
All VMs inside the VM Scale Set live inside this subnet.


### **5. NAT Gateway**

The NAT Gateway provides:

* Secure outbound connectivity
* Static outbound public IP
* No direct exposure of VMs to the internet

This is the recommended production pattern.


### **6. Network Security Group (NSG)**

The NSG is applied to the subnet and allows only the following inbound traffic:

* HTTP (80)
* HTTPS (443)
* SSH (22)

Everything else is blocked.


## 🎛 **Compute Layer**

### **7. Load Balancer (with Public IP)**

An Azure Standard Load Balancer:

* Distributes incoming traffic across VMSS instances
* Uses an HTTP health probe on port 80
* Routes only to healthy VMs

This ensures high availability.


### **8. Virtual Machine Scale Set (VMSS)**

The VMSS is the core compute engine.
It automatically manages:
* VM provisioning
* Load balancing integration
* Scaling (in/out)
* VM health and lifecycle

Each VM runs a cloud-init script that installs Apache, PHP, and metadata display logic.


## ⚙️ **Autoscaling Configuration**
Autoscaling is CPU-based and configured using Azure Monitor Autoscale settings.

### **Scale Settings**

| Setting               | Value |
| --------------------- | ----- |
| **Default Instances** | 3     |
| **Minimum Instances** | 1     |
| **Maximum Instances** | 10    |

### **Scaling Rules**

#### **1. Scale Out (Add VM)**

* Condition: **CPU > 80%** for 2 minutes
* Action: Add 1 VM

#### **2. Scale In (Remove VM)**

* Condition: **CPU < 10%** for 2 minutes
* Action: Remove 1 VM

This ensures cost optimization and performance efficiency.


## ▶️ **How to Deploy**

### **Option 1 — Using Pipeline**
Just push your code → pipeline triggers → all resources are created.

### **Option 2 — Local Deployment**
```
terraform init
terraform plan
terraform apply
```
Make sure your backend storage is configured.


## 📦 **VM Startup Script (user-data.sh)**
The script automatically:
* Installs Apache2
* Installs PHP
* Fetches Azure instance metadata
* Displays it through a webpage
* Restarts Apache properly for VMSS
* Ensures the LB health probe works


## 📡 **Testing the Deployment**
After deployment, open:
```
http://<LoadBalancerPublicURL>/index.php
```

You should see:
* Metadata-based index page
* Served by Apache
* Behind the Load Balancer and VMSS
