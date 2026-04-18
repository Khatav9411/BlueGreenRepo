# 🚀 Blue-Green Deployment using Jenkins & AWS ALB

## 📌 Project Overview

This project demonstrates a **production-ready Blue-Green Deployment strategy** using **Jenkins and AWS Application Load Balancer (ALB)** to achieve **zero-downtime deployments**.

---

## 🎯 Objective

* Eliminate downtime during deployments
* Maintain two environments (**Blue & Green**)
* Deploy to inactive environment
* Perform validation before traffic switch
* Automatically rollback on failure

---

## 🏗️ Architecture

```
GitHub Repo
     ↓
Jenkins Pipeline
     ↓
Deploy to Inactive Environment (Blue/Green)
     ↓
Health Check
     ↓
AWS ALB Traffic Switch
     ↓
Users access updated version
```
<img width="700" height="467" alt="image" src="https://github.com/user-attachments/assets/c34ad5d9-c58a-40e6-bbc7-82e5339d16d3" />

---

## 🧰 Technologies Used

* AWS EC2
* AWS Application Load Balancer (ALB)
* Jenkins
* GitHub

---

## 🌐 Infrastructure Setup

### 🔹 EC2 Instances

* **Blue Server** → Version 1
* **Green Server** → Version 2

Both servers:

* Installed with Apache/Nginx
* Serve static HTML pages

---

### 🔹 Application Files

* `blue.html` → Blue environment (Version 1)
* `green.html` → Green environment (Version 2)
* `index1.html` → Not used (ignored)

---

### 🔹 Load Balancer Setup

* Application Load Balancer created
* Listener configured on port **80 (HTTP)**
* Two Target Groups:

  * `BLUE-TG`
  * `GREEN-TG`

---

## ⚙️ Jenkins Pipeline Workflow

### 1️⃣ Select Target Environment

* Identifies current active environment
* Selects inactive environment for deployment

---

### 2️⃣ Deploy Application

* Copies correct HTML file (`blue.html` or `green.html`)
* Uses SCP to transfer file
* Replaces `/var/www/html/index.html` on target server

---

### 3️⃣ Health Check

* Uses `curl` to validate deployment
* Ensures application is responding correctly

---

### 4️⃣ Switch Traffic

* Uses AWS CLI
* Updates ALB listener to point to new target group

---

### 5️⃣ Rollback Mechanism

* If health check fails:

  * Deployment stops
  * Traffic remains on previous environment
  * No downtime occurs

---

## 🔄 Deployment Flow

### 🟦 First Run

* Deploys to **Green environment**
* Traffic switches to Green

---

### 🟢 Second Run

* Deploys to **Blue environment**
* Traffic switches back to Blue

---

## 🧪 Rollback Scenario

* Introduce failure in deployment
* Health check fails
* ALB does NOT switch traffic
* Previous environment continues serving users

---

## 📸 Deliverables

* ✅ Jenkins pipeline execution logs
* ✅ Target group switching proof
* ✅ ALB output verification
* ✅ Deployment demo

---

## 📊 Key Features

* Zero downtime deployment
* Automated traffic switching
* Safe rollback mechanism
* Production-ready CI/CD pipeline

---

## 💡 Conclusion

This project successfully implements a **Blue-Green deployment strategy** ensuring **high availability, zero downtime, and safe releases** using Jenkins and AWS services.

---

## 👨‍💻 Author

**Kamlesh Khatavkar**
