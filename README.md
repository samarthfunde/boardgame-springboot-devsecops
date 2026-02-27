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

---



