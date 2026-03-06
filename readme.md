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

terraform destroy --auto-approve





