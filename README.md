# Bakery Web Application: End-to-End DevOps Deployment
## Cloud Architecture on AWS: EC2, RDS, Docker, and Kubernetes (EKS)

This repository contains a full-stack bakery application deployed on AWS using managed services. The project follows a modern DevOps workflow: containerizing the application with Docker, managing data with AWS RDS, and orchestrating containers with AWS EKS.

---

## Phase 1: Infrastructure Setup (RDS & EC2)
### Section 1: AWS RDS (Database) Setup
#### H4 – What is AWS RDS?
**Amazon Relational Database Service (RDS)** is a managed database service that automates time-consuming tasks like hardware provisioning, database setup, patching, and backups.

#### H4 – Creating the Database
1. Go to **RDS Console** -> **Create Database**.
2. Select **MySQL 8.0** (Free Tier).
3. **DB Instance Identifier**: `bakery-db`.
4. **Credentials**: Username: `admin` | Password: `YourSecurePassword123`.
5. **Public Access**: No.
6. **Initial Database Name**: `bakery_db`.

#### H4 – Database Initialization (SQL Commands)
Connect to RDS from your EC2 and run the following to create the schema:

##### H5 – Create Schema and Tables
```sql
CREATE DATABASE IF NOT EXISTS bakery_db;
USE bakery_db;

CREATE TABLE products (
    id INT AUTO_INCREMENT PRIMARY KEY,
    name VARCHAR(255),
    price DECIMAL(10,2),
    image VARCHAR(255)
);

CREATE TABLE team (
    id INT AUTO_INCREMENT PRIMARY KEY,
    name VARCHAR(255),
    role VARCHAR(255),
    image VARCHAR(255)
);

CREATE TABLE contact_messages (
    id INT AUTO_INCREMENT PRIMARY KEY,
    payload JSON,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
```

##### H5 – Insert Sample Data
```sql
INSERT INTO products (name, price, image) VALUES 
('Chocolate Cake', 49.99, 'img/product-1.jpg'),
('French Bread', 14.99, 'img/product-2.jpg');
```

---

## Phase 2: Management Server Setup (EC2)
### Section 1: Tool Installation Sequence
#### H4 – Preparing the EC2 Environment
Launch an Ubuntu 22.04 instance and run these commands in sequence:

##### H5 – Install Docker
```bash
sudo apt update -y
sudo apt install docker.io -y
sudo systemctl start docker
sudo usermod -aG docker ubuntu
```

##### H5 – Install AWS CLI v2
```bash
sudo apt install unzip curl -y
curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
unzip awscliv2.zip
sudo ./aws/install
aws configure # Region: ca-central-1
```

##### H5 – Install kubectl
```bash
curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
sudo install -o root -g root -m 0755 kubectl /usr/local/bin/kubectl
```

---

## Phase 3: Dockerization & Registry Management
### Section 1: Building and Pushing Images
#### H4 – Docker Login & Build
```bash
docker login -u siddheshsomvanshi27

# Build Frontend
docker build -t siddheshsomvanshi27/bakery-frontend:latest .

# Build Backend
cd backend-java
docker build -t siddheshsomvanshi27/bakery-backend:latest .
```

#### H4 – Tagging & Pushing
```bash
docker tag containerizedbakerywebapplication-frontend siddheshsomvanshi27/bakery-frontend:latest
docker tag containerizedbakerywebapplication-backend siddheshsomvanshi27/bakery-backend:latest
docker push siddheshsomvanshi27/bakery-frontend:latest
docker push siddheshsomvanshi27/bakery-backend:latest
```

---

## Phase 4: Kubernetes Orchestration (EKS)
### Section 1: Cluster Connection
#### H4 – Connecting to Canada Cluster
```bash
aws eks update-kubeconfig --region ca-central-1 --name BakeryProject-EKS
kubectl get nodes # Ensure status is 'Ready'
```

### Section 2: Important Configuration (main.js Fix)
#### H4 – Connecting Frontend to Backend Service
In EKS, the Frontend and Backend use different Load Balancers. You must update the JavaScript to point to the correct Backend URL.

1. Get Backend URL: `kubectl get svc backend-service`.
2. Update `js/main.js`:
```javascript
// Change this line:
url: 'http://<YOUR-BACKEND-EXTERNAL-IP>:8080/api/contact',
```
3. Re-push Frontend:
```bash
docker build -t siddheshsomvanshi27/bakery-frontend:latest .
docker push siddheshsomvanshi27/bakery-frontend:latest
kubectl rollout restart deployment bakery-frontend
```

---

## Phase 5: Final Deployment & Verification
### Section 1: Launching the Project
#### H4 – Applying Manifests
```bash
kubectl apply -f bakery-deployment.yaml
kubectl get pods # Wait for Running
kubectl get svc  # Get your website URL
```

### Section 2: Networking Security
#### H4 – RDS Security Group Rule
To allow the application to save data, you must add an **Inbound Rule** to the **RDS Security Group**:
- **Type**: MySQL (3306)
- **Source**: Select your **EKS Node Security Group ID**.

---

## Output and Results
### Section 1: How to get the final output
1. Access the website via the **frontend-service External-IP**.
2. Verify the **Products** page loads data from RDS.
3. Submit the **Contact Form** and check for the success alert.

###### H6 – Project Deployment Documentation End
