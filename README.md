
# Netflix Clone written in Next.js with CI/CD Pipeline Setup with Jenkins, Docker, SonarQube, EKS, Prometheus, Grafana, and ArgoCD

This document provides a step-by-step guide to set up a CI/CD pipeline using various tools such as Jenkins, Docker, SonarQube, Kubernetes (EKS), Prometheus, Grafana, and ArgoCD.

## Steps

### 1. Server Setup and Tool Installation

1. Created a server and installed the following tools/packages:
   - Jenkins
   - Docker
   - Helm
   - Kubectl
   - AWS CLI
   - Trivy
   - SonarQube (via Docker)

### 2. Source Code Retrieval

- Retrieved the source code from GitHub using the `git clone` method.

### 3. Dockerfile Creation and Build

- Created a `Dockerfile` and built the Docker image for testing purposes.

### 4. Jenkins Setup

1. **Jenkins Access**: Opened Jenkins on port `8080`.
2. **Installed Required Plugins**:
   - Docker
   - SonarQube
   - OWASP
   - NodeJS
   - Pipeline Stage View
3. **Added Tools**:
   - SonarScanner
   - OWASP
4. **Credentials Configuration**:
   - Added sensitive information such as GitHub credentials, SonarQube tokens, Docker credentials, etc., into Jenkins credentials.

### 5. Jenkins Pipeline Creation

1. **Pipeline Job**: Created a pipeline job and wrote a Groovy script that:
   - Checks out code from GitHub.
   - Builds and pushes changes to GitHub.
2. **Build Execution**:
   - Ran the job and, after fixing some errors in the script, successfully pushed the Docker image to DockerHub and updated the deployment files.

### 6. EKS Cluster Setup

1. **EKS Cluster Creation**: Created an Amazon EKS cluster with 3 nodes.
2. **AWS Configuration**: Configured AWS using IAM credentials.
3. **Cluster Access**: Used `kubectl` and `kubeconfig` to access the cluster.

### 7. Prometheus and Grafana Installation

1. **Installation**:
   - Installed Prometheus and Grafana using Helm and `kubectl`.
2. **Service Type Change**:
   - Changed the service type for both Prometheus and Grafana from `ClusterIP` to `LoadBalancer`.
3. **Access**:
   - Accessed Prometheus on port `9090` and Grafana on port `3000`.
4. **Dashboards**:
   - Created dashboards in Grafana using Prometheus as the data source.

### 8. ArgoCD Installation and Configuration

1. **ArgoCD Installation**: Installed ArgoCD in the cluster using `kubectl`.
2. **Service Patch**: Patched the ArgoCD service to use `LoadBalancer`.
3. **Access ArgoCD**:
   - Accessed ArgoCD via the load balancer IP/URL.
4. **Login**:
   - Retrieved the ArgoCD admin password from the server and logged in.
5. **Application Setup**:
   - Created an application in ArgoCD by specifying the GitHub repository and path to the deployment files.
6. **Application Sync**:
   - Successfully synced the application, resulting in the creation of 3 pods.
7. **Monitoring**:
   - The application is now live and can be regularly monitored using Prometheus and Grafana.

---

## Installation Commands

## Jenkins Installation

```bash
sudo wget -O /usr/share/keyrings/jenkins-keyring.asc https://pkg.jenkins.io/debian-stable/jenkins.io-2023.key
echo "deb [signed-by=/usr/share/keyrings/jenkins-keyring.asc] https://pkg.jenkins.io/debian-stable binary/" | sudo tee /etc/apt/sources.list.d/jenkins.list > /dev/null
sudo apt-get update
sudo apt-get install jenkins
```

## Check Jenkins Status

```bash
sudo systemctl status jenkins
```

# Trivy Installation

```bash
sudo apt-get install wget apt-transport-https gnupg lsb-release
wget -qO - https://aquasecurity.github.io/trivy-repo/deb/public.key | sudo apt-key add -
echo deb https://aquasecurity.github.io/trivy-repo/deb $(lsb_release -sc) main | sudo tee -a /etc/apt/sources.list.d/trivy.list
sudo apt-get update
sudo apt-get install trivy
```

# Docker Installation

```bash
sudo apt-get update
sudo apt-get install ca-certificates curl
sudo install -m 0755 -d /etc/apt/keyrings
sudo curl -fsSL https://download.docker.com/linux/debian/gpg -o /etc/apt/keyrings/docker.asc
sudo chmod a+r /etc/apt/keyrings/docker.asc
echo "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] https://download.docker.com/linux/debian $(. /etc/os-release && echo "$VERSION_CODENAME") stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
sudo apt-get update
sudo apt-get install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
```

## Test Docker

```bash
sudo docker run hello-world
```

# Docker Configuration for Jenkins

```bash
sudo usermod -aG docker jenkins
sudo systemctl restart docker
sudo chmod 666 /var/run/docker.sock
```

# Install and Run SonarQube

```bash
sudo docker run -itd --name sonar -p 9000:9000 sonarqube
```

# AWS CLI Installation

```bash
curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
unzip awscliv2.zip
sudo ./aws/install
```

## AWS Configuration

```bash
aws configure
```

# Kubectl Installation

```bash
curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl.sha256"
echo "$(cat kubectl.sha256)  kubectl" | sha256sum --check
sudo install -o root -g root -m 0755 kubectl /usr/local/bin/kubectl
kubectl version --client
```

## Connect to EKS Cluster

```bash
aws eks --region <region-code> update-kubeconfig --name <cluster-name>
```

# Helm Installation

```bash
curl -fsSL -o get_helm.sh https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3
chmod 700 get_helm.sh
./get_helm.sh
```

# Prometheus-Grafana Installation

```bash
helm repo add stable https://charts.helm.sh/stable
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
kubectl create namespace prometheus
helm install stable prometheus-community/kube-prometheus-stack -n prometheus
```

## Change Prometheus & Grafana Service Type to LoadBalancer

```bash
kubectl edit svc stable-kube-prometheus-sta-prometheus -n prometheus
kubectl edit svc stable-grafana -n prometheus
```

## Get Grafana Password

```bash
kubectl get secret --namespace <namespace> grafana -o jsonpath="{.data.admin-password}" | base64 --decode ; echo
```

# ArgoCD Installation

```bash
kubectl create namespace argocd
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
```

## Patch ArgoCD Service to LoadBalancer

```bash
kubectl patch svc argocd-server -n argocd -p '{"spec": {"type": "LoadBalancer"}}'
```

## Get ArgoCD IP

```bash
kubectl get svc argocd-server -n argocd
```

## Get ArgoCD Admin Password

```bash
kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 -d ; echo
```
