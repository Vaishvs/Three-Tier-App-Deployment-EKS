# Three-Tier-App-Deployment-EKS Overview
Three-tier Web Application using ReactJS, NodeJS, and MongoDB, with deployment on AWS EKS.
In this project, we are going to create a Kubernetes cluster on AWS using Elastic Kubernetes Service. We are also going to create a private container registry using Elastic Container Registry and the Kubernetes cluster is exposed to the internet with the help of the Traefik ingress controller. Images will be built for the front end and the back end. These images will be pushed to the private ECR. EKS will pull these images and create a deployment.

# Architecture

  ![image](https://github.com/user-attachments/assets/78560547-efc6-4a3c-90d6-a12b54230293)


# Step 1: IAM Configuration
- Create a user eks-admin with AdministratorAccess.
- Generate Security Credentials: Access Key and Secret Access Key.

# Step 2: EC2 Setup
- Launch an Ubuntu instance in your favourite region (eg. region us-west-2).
- SSH into the instance from your local machine.

# Step 3: Install AWS CLI v2
    curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
    sudo apt install unzip
    unzip awscliv2.zip
    sudo ./aws/install -i /usr/local/aws-cli -b /usr/local/bin --update
    aws configure

# Step 4: Install Docker
    sudo apt-get update
    sudo apt install docker.io
    docker ps
    sudo chown $USER /var/run/docker.sock

# Step 5: Install kubectl
    curl -o kubectl https://amazon-eks.s3.us-west-2.amazonaws.com/1.19.6/2021-01-05/bin/linux/amd64/kubectl
    chmod +x ./kubectl
    sudo mv ./kubectl /usr/local/bin
    kubectl version --short --client

# Step 6: Install eksctl
    curl --silent --location "https://github.com/weaveworks/eksctl/releases/latest/download/eksctl_$(uname -s)_amd64.tar.gz" | tar xz -C /tmp
    sudo mv /tmp/eksctl /usr/local/bin
    eksctl version

# Step 7: Setup EKS Cluster
    eksctl create cluster --name three-tier-cluster --region us-west-2 --node-type t2.medium --nodes-min 2 --nodes-max 2
    aws eks update-kubeconfig --region us-west-2 --name three-tier-cluster
    kubectl get nodes

# Step 8:
  Change the env. value in k8s_manifests/frontend-deployment.yaml to http://app.<public_ip_kubernets_lb>.nip.io/api/tasks
  and host in traefik-ingress-controller/ingress.yaml to app.<public_ip_kubernets_lb>.nip.io.
  If you own a custom domain, create a subdomain and change app.<public_ip_kubernets_lb>.nip.io to your subdomain name.
  Learn more about https://nip.io/

# Step 9: Run Manifests
    kubectl create namespace workshop
    kubectl apply -f .
    kubectl delete -f .

# Step 10: Install Traefik using helm.

    helm repo add traefik https://helm.traefik.io/traefik
    helm repo update
    kubectl create namespace traefik
    helm install traefik traefik/traefik -n traefik

# Step 11:
  Application is deployed on the AKS cluster. To access it run http://app.<public_ip_kubernets_lb>.nip.io/ or the sub domain on your custom domain provided during step 9

# Cleanup
  -To delete the EKS cluster:
    eksctl delete cluster --name three-tier-cluster --region us-west-2
