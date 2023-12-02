# microk8s

##

https://argo-cd.readthedocs.io/en/stable/
https://github.com/argoproj/argo-cd/tree/stable/manifests

https://github.com/guilhem/freeipa-issuer
https://benbrougher.tech/posts/microk8s-ingress/
https://microk8s.io/docs/addon-metallb




### install microk8s
```bash
sudo snap install microk8s --classic --channel=1.28
sudo usermod -a -G microk8s john
sudo chown -R john ~/.kube
newgrp microk8s
```

### inspect and fix issues
```bash
microk8s inspect
nano /etc/docker/daemon.json
```json
{
    "insecure-registries" : ["localhost:32000"] 
}
```

```
sudo iptables -P FORWARD ACCEPT 
sudo apt-get install iptables-persistent
echo fs.inotify.max_user_watches=1048576 | sudo tee -a /etc/sysctl.conf
sudo sysctl --system
```

### Start microk8s
```bash
microk8s status --wait-ready
microk8s kubectl get all --all-namespaces

microk8s enable dns
microk8s enable ingress
microk8s enable metallb:192.168.1.20-192.168.1.40
microk8s stop && microk8s start

microk8s dashboard-proxy
```

### Install argocd
```bash
microk8s kubectl create namespace argocd
microk8s kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
microk8s kubectl apply -n argocd -f ./argocd/install.yaml
microk8s kubectl port-forward svc/argocd-server -n argocd 8080:443
```


### install argocli
```bash
curl -sSL -o argocd-linux-amd64 https://github.com/argoproj/argo-cd/releases/latest/download/argocd-linux-amd64
sudo install -m 555 argocd-linux-amd64 /usr/local/bin/argocd
rm argocd-linux-amd64
```


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
