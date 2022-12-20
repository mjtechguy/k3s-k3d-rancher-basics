# K3s, K3d and Rancher Basics

# General Tasks

## Install Kubectl
- curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
- sudo install -o root -g root -m 0755 kubectl /usr/local/bin/kubectl

## Install Helm
curl https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3 | bash

## Install ingress-nginx
helm upgrade --install ingress-nginx ingress-nginx --repo https://kubernetes.github.io/ingress-nginx --namespace ingress-nginx --create-namespace --set controller.replicaCount=1

## Install Rancher UI in Docker
docker run -d --restart=unless-stopped \
  -p 8080:80 -p 8443:443 \
  -v rancher:/var/lib/rancher \
  --privileged \
  rancher/rancher:v2.6.9

# K3s
https://k3s.io/

## Install latest version
curl -sfL https://get.k3s.io | K3S_KUBECONFIG_MODE="644" sh -

## Install specific version. Versions here: https://github.com/k3s-io/k3s/releases/tag/
curl -sfL https://get.k3s.io | K3S_KUBECONFIG_MODE="644" INSTALL_K3S_VERSION=v1.24.8+k3s1 sh -

## Deploy without Traefik
curl -sfL https://get.k3s.io | K3S_KUBECONFIG_MODE="644" INSTALL_K3S_VERSION=v1.24.8+k3s1 INSTALL_K3S_EXEC="server --no-deploy traefik" sh -

## Export kube-config to ~/.kube/config
cp /etc/rancher/k3s/k3s.yaml ~/.kube/config

## Remove K3s
/usr/local/bin/k3s-uninstall.sh

# K3d - Requires Docker to be installed
https://k3d.io/

## Install K3d
wget -q -O - https://raw.githubusercontent.com/k3d-io/k3d/main/install.sh | bash

## Setup k3d cluster
- export HOSTIP=YOURHOSTIP
- k3d cluster create --k3s-arg "--tls-san=$HOSTIP"@server:* --k3s-arg "--no-deploy=traefik@server:*" --k3s-arg "--disable=traefik" minio-cluster --servers 3 --agents 3 -p "80:80@loadbalancer:*" -p "443:443@loadbalancer:*"
- k3d kubeconfig write minio-cluster
- cp ~/.k3d/kubeconfig-minio-cluster.yaml ~/.kube/config

## Setup k3d cluster with specific version
- export HOSTIP=YOURHOSTIP
- k3d cluster create --k3s-arg "--tls-san=$HOSTIP"@server:* --k3s-arg "--no-deploy=traefik@server:*" --image rancher/k3s:v1.23.3-k3s1 minio-cluster --servers 3 --agents 3 -p "80:80@loadbalancer:*" -p "443:443@loadbalancer:*" -v /data/k3d@agent:* -v /data/k3d:@server:*
- k3d kubeconfig write minio-cluster
- cp ~/.k3d/kubeconfig-minio-cluster.yaml ~/.kube/config


## Deploy k3d with local storage
- export HOSTIP=YOURHOSTIP
- mkdir -p /data/k3d
- chmod 0777 /data/k3d
- k3d cluster create --k3s-arg "--tls-san=$HOSTIP"@server:* --k3s-arg "--no-deploy=traefik@server:*" --image rancher/k3s:v1.23.3-k3s1 minio-cluster --servers 3 --agents 3 -p "80:80@loadbalancer:*" -p "443:443@loadbalancer:*" -v /data/k3d:/data/k3d@agent:* -v /data/k3d:/data/k3d@server:*
- k3d kubeconfig write minio-cluster
- cp ~/.k3d/kubeconfig-minio-cluster.yaml ~/.kube/config

# delete cluster
k3d cluster delete minio-cluster