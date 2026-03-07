**STEP-1**
**You do this**:
Login AWS
EC2 → Launch Instance
Name: supermario-dev
Select Ubuntu
Select c7i-flex.large
Create key pair
Allow SSH + HTTP
Click Launch

**STEP-2**
What they did:
EC2 → Connect → EC2 Instance Connect
**Then run:**
sudo su
apt update

**STEP-3**

**Install Docker**
apt install docker.io -y
usermod -aG docker ubuntu
newgrp docker
docker --version

**Install Terraform**
wget -O- https://apt.releases.hashicorp.com/gpg | sudo gpg --dearmor -o /usr/share/keyrings/hashicorp-archive-keyring.gpg
echo "deb [signed-by=/usr/share/keyrings/hashicorp-archive-keyring.gpg] https://apt.releases.hashicorp.com $(lsb_release -cs) main" | sudo tee /etc/apt/sources.list.d/hashicorp.list
apt update
apt install terraform -y
terraform --version

**Install AWS CLI**
curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
apt install unzip -y
unzip awscliv2.zip
./aws/install
aws --version

**Install Kubectl**
apt install curl -y
curl -LO https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl
install -o root -g root -m 0755 kubectl /usr/local/bin/kubectl
kubectl version --client

**STEP-4**

**What they did:**
IAM → Roles → Create Role
Select EC2
Attach AdministratorAccess policy
Create role

**STEP-5**

EC2 → Select Instance
Actions → Security → Modify IAM Role
Select created role
Update

**STEP-6**

**Clone GitHub Repo**
mkdir supermario
cd supermario
git clone https://github.com/akshu20791/supermario-game
cd supermario-game
cd EKS-TF
ls

**STEP-7**

**Create S3 Bucket** 
AWS → S3 → Create Bucket
Give unique name

**STEP-8**
vim backend.tf
inside that we have to change the bucket name and region
vim main.tf
change the instance type to c7i-flex.large
vim provider.tf
change the region to ap-south-1

**STEP-9**

**Run Terraform**
terraform init
terraform validate
terraform plan
terraform apply --auto-approve

(cluster and node will be created)

**STEP-10**

**Update Kubeconfig**
aws eks update-kubeconfig --name EKS_CLOUD --region ap-south-1

**Deploy Application**
cd ..
kubectl apply -f deployment.yaml
kubectl apply -f service.yaml
kubectl get all

**STEP-11**
**Get Load Balancer URL**
kubectl describe service mario-service

at last copy the load balencer url and paste in the browser

http://<loard balencer url>


QUESTION 2 – PROMETHEUS + GRAFANA (FROM SCRATCH)

We will do everything inside the same EC2 instance where kubectl is installed.

STEP 1 — Create Monitoring Namespace

Run:

kubectl create namespace monitoring

Check:

kubectl get ns

You should see

monitoring
default
kube-system
STEP 2 — Install HELM

Prometheus and Grafana will be installed using Helm.

Run:

curl -fsSL https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3 | bash

Check version:

helm version
STEP 3 — Add Prometheus Helm Repo

Run:

helm repo add prometheus-community https://prometheus-community.github.io/helm-charts

Update repo

helm repo update
STEP 4 — Install Prometheus + Grafana

Run:

helm install monitoring prometheus-community/kube-prometheus-stack \
--namespace monitoring

This installs:

Prometheus

Grafana

Alertmanager

Node Exporter

Wait 2 minutes.

Check pods:

kubectl get pods -n monitoring

You should see pods like:

prometheus
grafana
alertmanager
node-exporter
kube-state-metrics
STEP 5 — Expose Grafana

Check services:

kubectl get svc -n monitoring

Find:

monitoring-grafana

Edit service:

kubectl edit svc monitoring-grafana -n monitoring

Change:

ClusterIP

to

NodePort

Save and exit.

STEP 6 — Get Grafana Port

Run:

kubectl get svc -n monitoring

Example output:

monitoring-grafana   NodePort   3000:32000

Here

NodePort = 32000
STEP 7 — Open Security Group Port

Go to AWS

EC2 → Security Group → Edit inbound rules

Add:

Custom TCP
Port: 32000
Source: Anywhere

Save.

STEP 8 — Access Grafana

Open browser:

http://<EC2-PUBLIC-IP>:32000

Example:

http://13.234.xx.xx:32000
STEP 9 — Get Grafana Password

Run:

kubectl get secret monitoring-grafana -n monitoring -o jsonpath="{.data.admin-password}" | base64 --decode

Login credentials:

Username: admin
Password: <above command output>
STEP 10 — Connect Prometheus to Grafana

Inside Grafana UI:

Settings
 → Data Sources
 → Add Data Source
 → Prometheus

URL:

http://prometheus-operated.monitoring.svc.cluster.local:9090

Click:

Save & Test
STEP 11 — Import Dashboard

Go to

Dashboards
 → Import

Enter ID:

1860

This is Node Exporter Dashboard.

Select:

Prometheus

Click

Import

Now you will see:

CPU Usage

Memory Usage

Disk Usage

Network

Pod metrics

STEP 12 — Verify Metrics

Run:

kubectl top nodes

and

kubectl top pods

Metrics will also appear in Grafana dashboards.








