# Actualizar sistema
sudo apt update

# Instalar Git
sudo apt install -y git
git clone https://github.com/AaronSanchezBelber/eficiencia.git

# Instalar dependencias base
sudo apt install -y ca-certificates curl

# Instalar Docker (repo oficial)
sudo install -m 0755 -d /etc/apt/keyrings
sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg -o /etc/apt/keyrings/docker.asc
sudo chmod a+r /etc/apt/keyrings/docker.asc

sudo tee /etc/apt/sources.list.d/docker.sources <<EOF
Types: deb
URIs: https://download.docker.com/linux/ubuntu
Suites: noble
Components: stable
Signed-By: /etc/apt/keyrings/docker.asc
EOF

sudo apt update
sudo apt install -y docker-ce docker-ce-cli containerd.io \
  docker-buildx-plugin docker-compose-plugin

# Verificar Docker
sudo docker run hello-world

# Permitir Docker sin sudo
sudo usermod -aG docker $USER
newgrp docker
docker run hello-world

# Habilitar Docker al arranque
sudo systemctl enable docker
sudo systemctl enable containerd
systemctl status docker

# Instalar Minikube
curl -LO https://github.com/kubernetes/minikube/releases/latest/download/minikube-linux-amd64
sudo install minikube-linux-amd64 /usr/local/bin/minikube

# Arrancar Kubernetes local
minikube start

# Instalar kubectl
sudo snap install kubectl --classic

# Verificar Kubernetes
kubectl version --client
kubectl get nodes
kubectl cluster-info
minikube status


VM Ubuntu (GCP e2)
 ├─ Docker
 │   └─ Contenedores
 │       └─ Minikube
 │           └─ Kubernetes Cluster
 │               ├─ API Server
 │               ├─ CoreDNS
 │               └─ Storage
 └─ kubectl

Azure VM (solo para admin)
 └─ kubectl + az
     └─ AKS
         ├─ Node Pools
         ├─ Load Balancer
         ├─ Azure Disk
         └─ ACR (Registry)

EC2 Admin / Bastion
 └─ kubectl + aws
     └─ EKS
         ├─ Managed Node Groups
         ├─ ALB Ingress
         ├─ EBS / EFS
         └─ ECR

VR-Git-Docker-minikube (levantar un cluster real, usar docker como driver, (orquestacion y kubernetes local)), kubectl (controlar el cliente (cluster, manifest, pods,servicios))


PARA ENCHUFA JENKINS EN TU CI-CD DESDE DOCKER

-------------------------------------------------------------------  JENKINS SETUP ------------------------------------------------------------

----------------------------------------------------------

docker run -d --name jenkins \
-p 8080:8080 \
-p 50000:50000 \
-v /var/run/docker.sock:/var/run/docker.sock \
-v $(which docker):/usr/bin/docker \
-u root \
-e DOCKER_GID=$(getent group docker | cut -d: -f3) \
--network minikube \
jenkins/jenkins:lts

------------------------------------------------------------


docker exec -it jenkins bash

apt update -y
apt install -y python3
python3 --version
ln -s /usr/bin/python3 /usr/bin/python
python --version
apt install -y python3-pip
apt install -y python3-venv
exit


---------------------------------------------------------------

----------------------------------------------------------------------- JENKINSFILE CODE ---------------------------------------------------------------------

pipeline {
    agent any
    stages {
        stage('Checkout Github') {
            steps {
                echo 'Checking out code from GitHub...'
		checkout scmGit(branches: [[name: '*/main']], extensions: [], userRemoteConfigs: [[credentialsId: 'github-token', url: 'https://github.com/data-guru0/GITOPS-PROJECT-9.git']])
		    }
        }        
        stage('Build Docker Image') {
            steps {
                echo 'Building Docker image...'
            }
        }
        stage('Push Image to DockerHub') {
            steps {
                echo 'Pushing Docker image to DockerHub...'
            }
        }
        stage('Install Kubectl & ArgoCD CLI') {
            steps {
                echo 'Installing Kubectl and ArgoCD CLI...'
            }
        }
        stage('Apply Kubernetes & Sync App with ArgoCD') {
            steps {
                echo 'Applying Kubernetes and syncing with ArgoCD...'
            }
        }
    }
}



------------------------------------------------------------------------- ARGOCD Configurations ----------------------------------------------------------------------



--------------------------------------------------

kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml

--------------------------------------------------

kubectl port-forward --address 0.0.0.0 service/argocd-server 30751:80 -n argocd 

-------------------------------------------------

kubectl get secret -n argocd argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 -d

---------------------------------------------------

echo 'installing Kubectl & ArgoCD cli...'
curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
chmod +x kubectl
mv kubectl /usr/local/bin/kubectl
curl -sSL -o /usr/local/bin/argocd https://github.com/argoproj/argo-cd/releases/latest/download/argocd-linux-amd64
chmod +x /usr/local/bin/argocd

---------------------------------------------------------------------------


sh '''
argocd login 34.72.5.170:31704 --username admin --password $(kubectl get secret -n argocd argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 -d) --insecure
'''

--------------------------------------------------

