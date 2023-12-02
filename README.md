
Cool issuer for k8s:
https://github.com/guilhem/freeipa-issuer

https://argo-cd.readthedocs.io/en/stable/
https://cloud.google.com/blog/products/containers-kubernetes/kubernetes-best-practices-organizing-with-namespaces
https://benbrougher.tech/posts/microk8s-ingress/

```bash
sudo apt-get update
# apt-transport-https may be a dummy package; if so, you can skip that package
#sudo apt-get install -y apt-transport-https ca-certificates curl
#curl -fsSL https://pkgs.k8s.io/core:/stable:/v1.28/deb/Release.key | sudo gpg --dearmor -o /etc/apt/keyrings/kubernetes-apt-keyring.gpg
# This overwrites any existing configuration in /etc/apt/sources.list.d/kubernetes.list
#echo 'deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] https://pkgs.k8s.io/core:/stable:/v1.28/deb/ /' | sudo tee /etc/apt/sources.list.d/kubernetes.list
#sudo apt-get update
#sudo apt-get install -y kubectl

sudo snap install microk8s --classic --channel=1.28
microk8s --classic --channel=1.22/stable)
sudo usermod -a -G microk8s john
sudo chown -R john ~/.kube
newgrp microk8s


microk8s inspect
nano /etc/docker/daemon.json
{
    "insecure-registries" : ["localhost:32000"] 
}
sudo iptables -P FORWARD ACCEPT 
sudo apt-get install iptables-persistent

echo fs.inotify.max_user_watches=1048576 | sudo tee -a /etc/sysctl.conf
sudo sysctl --system

microk8s status --wait-ready
#microk8s start
microk8s kubectl get all --all-namespaces
microk8s enable dns && microk8s stop && microk8s start

microk8s dashboard-proxy
#curl -LO https://storage.googleapis.com/minikube/releases/latest/minikube_latest_amd64.deb
#sudo dpkg -i minikube_latest_amd64.deb
#minikube start

microk8s kubectl create namespace argocd
microk8s kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml


brew install argocd
microk8s kubectl port-forward svc/argocd-server -n argocd 8080:443
argocd admin initial-password -n argocd

curl -sSL -o argocd-linux-amd64 https://github.com/argoproj/argo-cd/releases/latest/download/argocd-linux-amd64
sudo install -m 555 argocd-linux-amd64 /usr/local/bin/argocd
rm argocd-linux-amd64




microk8s enable metallb:192.168.1.20-192.168.1.40
microk8s kubectl patch svc argocd-server -n argocd -p '{"spec": {"type": "LoadBalancer"}}'

```
Enabled
    dashboard            # (core) The Kubernetes dashboard
    dns                  # (core) CoreDNS
    ha-cluster           # (core) Configure high availability on the current node
    helm                 # (core) Helm - the package manager for Kubernetes
    helm3                # (core) Helm 3 - the package manager for Kubernetes
    ingress              # (core) Ingress controller for external access
    metrics-server       # (core) K8s Metrics Server for API access to service metrics
