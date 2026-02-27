# BoardgameListingWebApp

## Description 

**Board Game Database Full-Stack Web Application.**
This web application displays lists of board games and their reviews. While anyone can view the board game lists and reviews, they are required to log in to add/ edit the board games and their reviews. The 'users' have the authority to add board games to the list and add reviews, and the 'managers' have the authority to edit/ delete the reviews on top of the authorities of users.  

## Technologies

- Java
- Spring Boot
- Amazon Web Services(AWS) EC2
- Thymeleaf
- Thymeleaf Fragments
- HTML5
- CSS
- JavaScript
- Spring MVC
- JDBC
- H2 Database Engine (In-memory)
- JUnit test framework
- Spring Security
- Twitter Bootstrap
- Maven

## Features

- Full-Stack Application
- UI components created with Thymeleaf and styled with Twitter Bootstrap
- Authentication and authorization using Spring Security
  - Authentication by allowing the users to authenticate with a username and password
  - Authorization by granting different permissions based on the roles (non-members, users, and managers)
- Different roles (non-members, users, and managers) with varying levels of permissions
  - Non-members only can see the boardgame lists and reviews
  - Users can add board games and write reviews
  - Managers can edit and delete the reviews
- Deployed the application on AWS EC2
- JUnit test framework for unit testing
- Spring MVC best practices to segregate views, controllers, and database packages
- JDBC for database connectivity and interaction
- CRUD (Create, Read, Update, Delete) operations for managing data in the database
- Schema.sql file to customize the schema and input initial data
- Thymeleaf Fragments to reduce redundancy of repeating HTML elements (head, footer, navigation)

## How to Run

1. Clone the repository
2. Open the project in your IDE of choice
3. Run the application
4. To use initial user data, use the following credentials.
  - username: bugs    |     password: bunny (user role)
  - username: daffy   |     password: duck  (manager role)
5. You can also sign-up as a new user and customize your role to play with the application! 😊














#  Phase 1 – Infrastructure Setup (AWS)

## 1️ EC2 Instances Launched

I launched 6 EC2 instances in AWS to build a complete DevOps pipeline environment:

| Server Name | Purpose                                                    |
|-------------|----------------------------------------------------------- |
| Master      | Kubernetes Control Plane (Cluster management & API server) |
| Slave-1     | Kubernetes Worker Node (Runs application pods)             |
| Jenkins     | CI/CD automation server (Build & Deploy pipeline)          |
| SonarQube   | Static code analysis & Quality Gate checks                 |
| Nexus       | Artifact repository (Stores JAR/Docker images)             |
| Monitor     | Monitoring server (Prometheus, Grafana, Blackbox Exporter) |


<img width="1916" height="568" alt="image" src="https://github.com/user-attachments/assets/b330ecc9-0161-4bd0-a119-7d7b740e8d44" />


---

## 2️ Security Group Configuration

Created a primary Security Group and attached it to all instances.

###  Inbound Rules Configured

| Port | Purpose                                                                                |
|------       |---------------------------------------------------------------------------------|                                                                    
| 22          | SSH access                                                                      |
| 80          | HTTP access                                                                     |
| 443         | HTTPS access                                                                    |
| 6443        | Kubernetes API Server                                                           |
| 10250       | Kubelet API                                                                     |
| 30000-32767 | Kubernetes NodePort services                                                    |
| 3000-10000  | Application & Monitoring tools (Grafana, SonarQube, Nexus, Jenkins etc.)        |
| 25          | SMTP                                                                            |
| 465         | SMTPS                                                                           |
| 179         | BGP (Internal cluster communication if required)                                |



## 3️ Terminal Access Setup

Downloaded **MobaXterm** for remote server access.

- Used MobaXterm for SSH connection to all EC2 instances.
- Connected using `.pem` key authentication.
- Verified connectivity to all servers.

 Note:  
MobaXterm is optional. We can also use:
- Native Linux terminal
- Git Bash (Windows)
- AWS EC2 Instance Connect


<img width="1919" height="810" alt="image" src="https://github.com/user-attachments/assets/dcdb0f55-68cd-49b9-b601-804a93c190c2" />

---

#  Phase 2 – Kubernetes Cluster Setup

## 1️ Install Prerequisites (Master & Worker Node)

Executed the following steps on both Master and Slave-1:

###  Update Packages
```
sudo apt-get update
```

###  Install Docker
```
sudo apt install docker.io -y
sudo chmod 666 /var/run/docker.sock
```

###  Install Required Dependencies
```
sudo apt-get install -y apt-transport-https ca-certificates curl gnupg
sudo mkdir -p -m 755 /etc/apt/keyrings
```

###  Add Kubernetes Repository & GPG Key
```
curl -fsSL https://pkgs.k8s.io/core:/stable:/v1.28/deb/Release.key | sudo gpg --dearmor -o /etc/apt/keyrings/kubernetes-apt-keyring.gpg
echo 'deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] https://pkgs.k8s.io/core:/stable:/v1.28/deb/ /' | sudo tee /etc/apt/sources.list.d/kubernetes.list
```

###  Update Repository
```
sudo apt update
```

###  Install Kubernetes Components
```
sudo apt install -y kubeadm=1.28.1-1.1 kubelet=1.28.1-1.1 kubectl=1.28.1-1.1
```
---

## 2️ Initialize Kubernetes Cluster (Master Node Only)

###  Initialize Control Plane
```
sudo kubeadm init --pod-network-cidr=10.244.0.0/16
```

###  Configure kubectl Access
```
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```
---

## 3️ Install Networking (Master Node)

###  Deploy Calico Network Plugin
```
kubectl apply -f https://docs.projectcalico.org/manifests/calico.yaml
Calico provides pod-to-pod networking inside the cluster.
```
---

## 4️ Install Ingress Controller

###  Deploy NGINX Ingress Controller
```
kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/controller-v0.49.0/deploy/static/provider/baremetal/deploy.yaml
Ingress Controller manages external HTTP/HTTPS access to services inside the cluster.
```
---

## 5️ Join Worker Node (Slave-1)

After Master initialization, executed the kubeadm join command on Slave-1 to connect it to the cluster.

This allowed Slave-1 to run application pods.

---

## 6️ Kubernetes Security Scanning (kubeaudit)

Installed kubeaudit on Master Node:
```
wget https://github.com/Shopify/kubeaudit/releases/download/v0.22.2/kubeaudit_0.22.2_linux_amd64.tar.gz
tar -xvzf kubeaudit_0.22.2_linux_amd64.tar.gz
```

###  Why kubeaudit?

kubeaudit is used to:

its checks your k8s cluster & tells you:
- Scan Kubernetes cluster for security misconfigurations
- if containers are running as root
- if privileged mode is enabled
- if network policy is missing

How to Run it 
```
kubeaudit all

```
this command scan the entire cluster and show the any vulnerabilities
This improves cluster security posture and helps maintain compliance.


After Kubernetes cluster setup, I configured Code Quality and Artifact Management tools.

---

## 7 SonarQube Setup (On SonarQube Server)

###  Step 1 – Install Docker

Installed Docker using official Docker repository.

Purpose:
- To run SonarQube as a containerized service
- Ensures portability and simplified management

---

### Step 2 – Run SonarQube Container

```
docker run -d --name sonar -p 9000:9000 sonarqube:lts-community
```

Port Used:
- 9000 → SonarQube Web UI

Access URL:
```
http://<SonarQube-Server-IP>:9000
```

Default Credentials:
- Username: admin
- Password: admin

Purpose of SonarQube:
- Static Code Analysis
- Code Quality Check
- Detect Bugs, Vulnerabilities, Code Smells
- Enforce Quality Gates in CI/CD Pipeline

<img width="1919" height="958" alt="image" src="https://github.com/user-attachments/assets/430c28d7-1ce5-4846-97bc-44d59a112ad1" />

---

## 8 Nexus Setup (On Nexus Server)

###  Step 1 – Install Docker

Installed Docker using same process as SonarQube server.

---

###  Step 2 – Run Nexus Container

```
docker run -d --name Nexus -p 8081:8081 sonatype/nexus3
```

Port Used:
- 8081 → Nexus Web UI

Access URL:
```
http://<Nexus-Server-IP>:8081
```

---

###  Step 3 – Retrieve Nexus Admin Password

To get initial admin password:

```
docker exec -it <container_id> /bin/bash
cd sonatype-work/nexus3
cat admin.password
```

Purpose of Nexus:
- Artifact Repository Manager
- Stores:
  - Maven JAR files
  - Build artifacts
  - Docker images
  - Acts as centralized artifact storage in CI/CD pipeline

<img width="1917" height="624" alt="image" src="https://github.com/user-attachments/assets/7838f3ec-25d8-4d49-a83d-c6fae330fac8" />

## 9 Nexus Configuration – Storing Build Artifacts

After installing Nexus, I configured my Maven project to store build artifacts (.jar files) in Nexus repository.

###  Step 1 – Configure pom.xml

Added the following configuration inside / following `pom.xml`:

```
<distributionManagement>
    <repository>
        <id>nexus-releases</id>
        <url>http://<NEXUS-IP>:8081/repository/maven-releases/</url>
    </repository>
    <snapshotRepository>
        <id>nexus-snapshots</id>
        <url>http://<NEXUS-IP>:8081/repository/maven-snapshots/</url>
    </snapshotRepository>
</distributionManagement>
```

---

###  Step 2 – Build & Deploy Artifact

Used the following Maven command in Jenkins pipeline:
```
mvn clean deploy
```
Purpose:
- `package` → Creates .jar file
- `deploy` → Uploads .jar file to Nexus repository

---

###  Maven Releases vs Snapshots

 Maven Releases:
- Used for stable, production-ready versions
- Version format: 1.0.0
- Immutable (cannot be overwritten)

Maven Snapshots:
- Used for development/testing versions
- Version format: 1.0.0-SNAPSHOT
- Can be updated multiple times

---

### Why We Use Both?

- Snapshots → For continuous development builds
- Releases → For stable production deployments
- Enables version control and artifact promotion strategy
- Ensures the same tested artifact moves from Dev → QA → Prod

---

### Where Artifacts Are Stored in Nexus

Nexus UI → Browse → maven-releases or maven-snapshots  
Path structure:

groupId / artifactId / version / .jar file

---

# 1️0 Jenkins Setup (On Jenkins Server)

---

##  Step 1 – Install Docker (docker.sh)

```bash
# Update system
sudo apt update

# Install required packages
sudo apt install ca-certificates curl -y
sudo install -m 0755 -d /etc/apt/keyrings

# Add Docker GPG key
sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg -o /etc/apt/keyrings/docker.asc
sudo chmod a+r /etc/apt/keyrings/docker.asc

# Add Docker repository
sudo tee /etc/apt/sources.list.d/docker.sources <<EOF
Types: deb
URIs: https://download.docker.com/linux/ubuntu
Suites: $(. /etc/os-release && echo "${UBUNTU_CODENAME:-$VERSION_CODENAME}")
Components: stable
Signed-By: /etc/apt/keyrings/docker.asc
EOF

# Update and install Docker
sudo apt update
sudo apt install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin -y

# Start Docker
sudo systemctl start docker

# Allow ubuntu user to run Docker without sudo
sudo chmod 666 /var/run/docker.sock

# Test Docker
docker run hello-world
```

Purpose:
Docker is required for building and running container images inside Jenkins pipeline.

---

##  Step 2 – Install Java & Jenkins (jenkins.sh)

```bash
#!/bin/bash

# Install Java (Jenkins prerequisite)
sudo apt install openjdk-17-jre-headless -y

# Add Jenkins GPG key
sudo wget -O /etc/apt/keyrings/jenkins-keyring.asc \
https://pkg.jenkins.io/debian-stable/jenkins.io-2026.key

# Add Jenkins repository
echo "deb [signed-by=/etc/apt/keyrings/jenkins-keyring.asc]" \
https://pkg.jenkins.io/debian-stable binary/ | sudo tee \
/etc/apt/sources.list.d/jenkins.list > /dev/null

# Update packages
sudo apt update

# Install Jenkins
sudo apt install jenkins -y

# Start Jenkins
sudo systemctl start jenkins
sudo systemctl enable jenkins

# Check Jenkins status
sudo systemctl status jenkins
```

Default Port:
```
8080
```

Access:
```
http://<Jenkins-Server-IP>:8080
```

To get initial admin password:
```bash
sudo cat /var/lib/jenkins/secrets/initialAdminPassword
```

---

##  Step 3 – Install kubectl (kubectl.sh)

```bash
# Download kubectl
curl -o kubectl https://amazon-eks.s3.us-west-2.amazonaws.com/1.19.6/2021-01-05/bin/linux/amd64/kubectl

# Make executable
chmod +x ./kubectl

# Move to system path
sudo mv ./kubectl /usr/local/bin

# Verify installation
kubectl version --short --client
```

Purpose:
Allows Jenkins to deploy applications to Kubernetes cluster.

---

##  Step 4 – Install Trivy (trivy.sh)

```bash
# Install dependencies
sudo apt-get install wget apt-transport-https gnupg lsb-release -y

# Add Trivy GPG key
wget -qO - https://aquasecurity.github.io/trivy-repo/deb/public.key | sudo apt-key add -

# Add Trivy repository
echo deb https://aquasecurity.github.io/trivy-repo/deb $(lsb_release -sc) main | sudo tee -a /etc/apt/sources.list.d/trivy.list

# Update and install Trivy
sudo apt update -y
sudo apt install trivy -y

# Verify installation
trivy --version
```

Purpose:
Trivy scans Docker images for vulnerabilities before deployment.

---

# 1️1 GitHub Repository Setup

##  Step 1 – Create Repository on GitHub

Created a new repository in GitHub to store application source code.

---

##  Step 2 – Push Project Code to GitHub

Used following commands to push project:

```bash
git init
git add .
git commit -m "push project"
git remote add origin <repository-url>
git push -u origin master
```

Purpose:
Version control and integration with Jenkins CI/CD pipeline.

---

# 1️2 Jenkins Plugin Installation

Installed required plugins from:

Manage Jenkins → Manage Plugins

## Installed Plugins:

###  JDK
- Eclipse Temurin Installer

###  Maven
- Maven Integration
- Pipeline Maven Integration
- Config File Provider

###  SonarQube
- SonarQube Scanner

###  Docker
- Docker
- Docker Pipeline

###  Kubernetes
- Kubernetes
- Kubernetes CLI
- Kubernetes Client API
- Kubernetes Credentials

###  Monitoring
- Prometheus Metrics Plugin

Purpose:
These plugins integrate Jenkins with build tools, security scanners, container runtime, Kubernetes, and monitoring systems.

---

# 1️3 Jenkins Tool Configuration

Configured tools under:

Manage Jenkins → Global Tool Configuration

---

##  JDK Installation

Name:
```
jdk17
```

Install Automatically:
Enabled
Source:
Adoptium (Temurin) – Version 17

---

##  Maven Installation

Name:
```
maven3
```

Version:
```
3.6.1
```

Install Automatically:
Enabled

---

##  SonarQube Scanner

Name:
```
sonar-scanner
```

Version:
Latest

Install Automatically:
Enabled

---

##  Docker Installation

Name:
```
docker
```

Version:
Latest

Install Automatically:
Enabled

---

After configuring all tools → Click Save.

---

# 1️4 Jenkins Pipeline Creation

Created a new Pipeline Job.

Item Name:
```
BoardGame
```

Pipeline Configuration:

- Selected: Pipeline script from SCM
- SCM: Git
- Repository URL: <your-github-repo-url>
- Branch: master
- Script Path:

```
Jenkins_Pipeline_File
```

Purpose:
This file contains the complete CI/CD pipeline stages including:

• Git Checkout  
• Maven Build  
• SonarQube Analysis  
• Artifact Upload to Nexus  
• Docker Build  
• Trivy Scan  
• Push Image  
• Kubernetes Deployment 


---

# 1️5 Jenkins Credentials Configuration

Configured credentials from:

Manage Jenkins → Credentials → System → Global Credentials → Add Credentials

---

##  1. GitHub Credentials

Kind: Username with Password  
ID:
```
git-cred
```
Username & Password:
```
Purpose:
Used by Jenkins pipeline to clone private GitHub repository.

---

##  2. SonarQube Token

Kind: Secret Text  
ID:
```
sonar-token
```

Purpose:
Used in pipeline for SonarQube authentication during code quality analysis.

---

##  3. DockerHub Credentials

Kind: Username with Password  
ID:
```
docker-cred
```
Username:
```
samarthfunde
```
Purpose:
Used to login and push Docker images to DockerHub from Jenkins pipeline.

---

##  4. Email Credentials

Kind: Username with Password  
ID:
```
mail-cred
```
Username:
```
Purpose:
Used for sending build notifications from Jenkins.

---

##  5. Kubernetes Credentials

Kind: Secret File  
ID:
```
k8s-cred
```
File:
```
kubeconfig.txt
```
Purpose:
Allows Jenkins to authenticate and communicate with Kubernetes cluster using kubectl.

---

# 1️6 Jenkins Pipeline Job

Created Pipeline Job:

Name:
```
BoardGame
```
Pipeline

i added pipeline code in this repo file name is  "Jenkins_Pieline_File"

```

This file contains all CI/CD stages:
• Checkout  
• Build  
• Code Analysis  
• Docker Build  
• Security Scan  
• Push Image  
• Kubernetes Deployment  

---

# 1️7 Kubernetes Configuration Files

Created a folder inside repository:
```
k8s/
```
Inside k8s folder created following files:

---

##  1. namespace.yaml

Creates a dedicated namespace for the application.

```bash
nano namespace.yaml
```
Purpose:
Logical isolation of resources inside Kubernetes cluster.

---

##  2. 1_Service_Account.yaml

```bash
nano 1_Service_Account.yaml
```
Purpose:
Creates Service Account used by Jenkins to interact with cluster.

---

##  3. 2_role.yaml

```bash
nano 2_role.yaml
```
Purpose:
Defines RBAC permissions (what actions are allowed).

---

##  4. 3_bind_role_to_service_acc.yaml

```bash
nano 3_bind_role_to_service_acc.yaml
```
Purpose:
Binds Role to Service Account so Jenkins gets required permissions.

<img width="1712" height="606" alt="image" src="https://github.com/user-attachments/assets/9ba9f6f4-3443-46b8-8b81-4fe491268a8d" />

<img width="1919" height="748" alt="image" src="https://github.com/user-attachments/assets/6793c304-2f24-4e34-acad-f89ede817eab" />

<img width="1917" height="927" alt="image" src="https://github.com/user-attachments/assets/6d6303e9-2eee-447e-8ec3-28a9645aa6c0" />

<img width="1919" height="729" alt="image" src="https://github.com/user-attachments/assets/b376a2e6-ed5f-4c6d-8775-ea123d68a8d2" />


---

# 1️8 kubeconfig File Usage

Generated kubeconfig file from Kubernetes cluster and saved as:

```
kubeconfig.txt
```
Uploaded in Jenkins as:
Credential ID:
```
k8s-cred
```

---

##  Why kubeconfig File is Required?

kubeconfig file contains:

• Cluster API Server endpoint  
• Cluster certificate  
• User authentication token  
• Context information  

Without kubeconfig:
Jenkins cannot connect to Kubernetes cluster.
Pipeline uses this credential like:

```groovy
withCredentials([file(credentialsId: 'k8s-cred', variable: 'KUBECONFIG')]) {
    sh 'kubectl apply -f k8s/'
}
```

This allows secure deployment from Jenkins to Kubernetes.


# 19 Jenkins Email Notification Setup (Gmail SMTP)

---

##  Step 1 – Generate Gmail App Password

Opened:

https://myaccount.google.com/apppasswords

Steps:

1. Login with Gmail account  
2. Go to **Security**
3. Enable **2-Step Verification**
4. Go to **App Passwords**
5. Select:
   - App → Mail
   - Device → Other (Jenkins)
6. Click **Generate**
7. Copy the 16-digit App Password

Note:
We use App Password instead of normal Gmail password for security reasons.

---

##  Step 2 – Add Email Credential in Jenkins

Go to:
Manage Jenkins → Credentials → System → Global → Add Credentials
Kind:
```
Username with password
```
ID:
```
mail-cred
```
Username:
```
samarthfunde45@gmail.com
```
Password:
```
<Generated App Password>
```
Click Save.

---

##  Step 3 – Configure SMTP in Jenkins

Go to:
Manage Jenkins → Configure System

---

### 1. Configure "Extended E-mail Notification"

SMTP Server:
```
smtp.gmail.com
```
SMTP Port:
```
465
```

---

###  2. Configure "E-mail Notification"

SMTP Server:
```
smtp.gmail.com
```

Click Advanced and configure:

Use SMTP Authentication → Enabled  
Username:
```
samarthfunde45@gmail.com
```
Password:
```
<App Password>
```
Use SSL → Enabled  
SMTP Port:
```
465
```

Click **Test Configuration** to verify.

Then click **Save**.

---

# Why Email Notification is Required?

Email notification helps to:

• Notify build success or failure  
• Alert team if deployment fails  
• Send security scan results  
• Improve monitoring and communication  

<img width="1914" height="901" alt="image" src="https://github.com/user-attachments/assets/4591ed21-d193-490b-83fc-812ed4d2ad21" />

---

#  PHASE 3 – Monitoring & Observability Setup


In this phase, I implemented centralized monitoring for the complete DevSecOps pipeline.

Tools Used:

• Prometheus – Metrics Collection  
• Grafana – Visualization  
• Blackbox Exporter – Website Uptime Monitoring  

Monitoring server configured separately with different ports.

---

#  Why Monitoring is Required?

Monitoring helps to:

• Track application uptime  
• Monitor Kubernetes cluster health  
• Monitor Jenkins performance  
• Monitor server CPU & memory  
• Get real-time alerts  
• Visualize metrics using dashboards  

Monitoring snapshots were captured for documentation.

---

# 20 Prometheus Setup

File:
```
nano prometheus_setup.sh
```

```bash
sudo apt update -y

# Download Prometheus from official documentation
wget https://github.com/prometheus/prometheus/releases/download/v3.10.0/prometheus-3.10.0.linux-amd64.tar.gz

# Extract the file
tar -xvf prometheus-3.10.0.linux-amd64.tar.gz

# Navigate into directory
cd prometheus-3.10.0.linux-amd64

# Run Prometheus in background
./prometheus &
```

Default Port:
```
9090
```

Access Prometheus:
```
http://<Monitoring-Server-IP>:9090
```

Purpose:
Prometheus collects metrics from:

• Kubernetes cluster  
• Jenkins server  
• Node Exporter  
• Blackbox Exporter  

---

# 21 Blackbox Exporter Setup

File:
```
nano blackbox_exporter.sh
```

```bash
# Download Blackbox Exporter
wget https://github.com/prometheus/blackbox_exporter/releases/download/v0.28.0/blackbox_exporter-0.28.0.linux-amd64.tar.gz

# Extract
tar -xvf blackbox_exporter-0.28.0.linux-amd64.tar.gz

# Remove tar file
rm -rf blackbox_exporter-0.28.0.linux-amd64.tar.gz

# Navigate to folder
cd blackbox_exporter-0.28.0.linux-amd64

# Run in background
./blackbox_exporter &
```

Default Port:
```
9115
```

Access:
```
http://<Monitoring-Server-IP>:9115
```

Purpose:

Blackbox Exporter monitors:

• Website URL availability  
• HTTP response status  
• Response time  
• SSL certificate validity  

It ensures the deployed application is publicly accessible.

---

# 22 Grafana Setup

File:
```
nano grafana_setup.sh
```

```bash
# Install dependencies
sudo apt-get install -y adduser libfontconfig1 musl

# Download Grafana Enterprise
wget https://dl.grafana.com/grafana-enterprise/release/12.4.0/grafana-enterprise_12.4.0_22325204712_linux_amd64.deb

# Install Grafana
sudo dpkg -i grafana-enterprise_12.4.0_22325204712_linux_amd64.deb

# Start Grafana service
sudo systemctl start grafana-server

# Enable Grafana at boot
sudo systemctl enable grafana-server
```

Default Port:
```
3000
```

Access Grafana:
```
http://<Monitoring-Server-IP>:3000
```

Default Login:
```
Username: admin
Password: admin
```

After login:

1. Add Data Source → Prometheus  
   URL:
   ```
   http://localhost:9090
   ```

2. Create Dashboards  
3. Import Kubernetes / Node Exporter dashboards  

---

#  Ports Used on Monitoring Server

| Tool              | Port              |
|------------------ |------------------ |
| Prometheus        | 9090              |
| Grafana           | 3000              |
| Blackbox Exporter | 9115              |

All tools configured on a single monitoring server.

---

#  Monitoring Flow Architecture

Application  
↓  
Kubernetes  
↓  
Prometheus (collect metrics)  
↓  
Grafana (visualization dashboard)

External Website URL  
↓  
Blackbox Exporter  
↓  
Prometheus  
↓  
Grafana Dashboard  

<img width="1918" height="916" alt="Screenshot 2026-02-27 110705" src="https://github.com/user-attachments/assets/67fb6bbb-11b7-46a7-ba80-2fe10e39a799" />

<img width="1919" height="972" alt="Screenshot 2026-02-27 114140" src="https://github.com/user-attachments/assets/0e4ea363-5e63-41e4-bfe9-64db50b4b5f4" />

<img width="722" height="542" alt="Screenshot 2026-02-27 115155" src="https://github.com/user-attachments/assets/05e9cc37-5c82-417c-aa31-171352748c30" />

<img width="1919" height="951" alt="Screenshot 2026-02-27 124639" src="https://github.com/user-attachments/assets/5dbc9cb2-8b9b-46a8-89a2-846e4a759bba" />

<img width="701" height="931" alt="Screenshot 2026-02-27 130804" src="https://github.com/user-attachments/assets/203f1b74-61f0-4fee-8abc-af8a0bb981cc" />

<img width="1919" height="950" alt="Screenshot 2026-02-27 125210" src="https://github.com/user-attachments/assets/6fd78a6a-08f8-4091-b0e9-45983ffc0c2c" />

<img width="1919" height="908" alt="Screenshot 2026-02-27 125219" src="https://github.com/user-attachments/assets/3320d611-7f17-4684-8ea3-6ad418f7a645" />

<img width="1919" height="925" alt="Screenshot 2026-02-27 125720" src="https://github.com/user-attachments/assets/5cad40e8-1083-4581-9721-e8b8576ccf85" />

<img width="1917" height="946" alt="Screenshot 2026-02-27 134256" src="https://github.com/user-attachments/assets/ae779f7b-f3d3-4750-8c58-a3c8373eb654" />

<img width="1919" height="962" alt="Screenshot 2026-02-27 142155" src="https://github.com/user-attachments/assets/04ab92c8-07d1-45e2-8d89-83ce04c5290a" />

<img width="1919" height="958" alt="Screenshot 2026-02-27 142300" src="https://github.com/user-attachments/assets/08ea6ca4-0df2-4e77-8de2-183f23c4d55c" />

<img width="1919" height="951" alt="Screenshot 2026-02-27 145429" src="https://github.com/user-attachments/assets/08699ecb-8662-4107-883a-ddd448f09749" />




---
