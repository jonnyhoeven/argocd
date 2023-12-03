# k3d

## Links
- https://github.com/guilhem/freeipa-issuer
- https://argo-cd.readthedocs.io/en/stable
- https://argo-cd.readthedocs.io/en/stable/operator-manual/disaster_recovery
- https://github.com/argoproj/argo-cd/tree/stable/manifests

- https://benbrougher.tech/posts/microk8s-ingress
- https://microk8s.io/docs/addon-metallb 
- https://k3d.io/v5.6.0/


## Todo
- [ ] setup freeipa issuer
- [ ] setup nexus repository
- [ ] setup storage
- [ ] add argocd backup and restore
- [x] add argocd cli
- [x] add argocd ingress
- [x] add traefik ingress
- [x] add argocd install
 

## Installation
This document outlines a k3s local dev cluster with argocd and traefik ingress.


### Install k3d

```bash
wget -q -O - https://raw.githubusercontent.com/k3d-io/k3d/main/install.sh | TAG=v5.6.0 bash
```

```bash
k3d cluster delete sidcluster
k3d cluster create sidcluster --servers 1 --agents 2 --api-port 6550 --subnet 172.23.0.0/16 #\
#-p "80:8000@loadbalancer" -p "443:8443@loadbalancer" 
kubectl get nodes -o wide
```

## setup cluster
```bash
cat /etc/hosts
kubectl create namespace argocd
kubectl apply -f ./namespaces/argocd/argocd-cmd-params-cm.yaml -n argocd 
kubectl apply -f ./namespaces/argocd/argocd.install.argocd.yaml -n argocd
#kubectl apply -f ./namespaces/argocd/argocd-ha.install.argocd.yaml -n argocd
kubectl apply -f ./namespaces/argocd/argocd.ingress.argocd.yaml -n argocd
```

```bash
#k3d cluster delete sidcluster
k3d cluster create sidcluster --servers 1 --agents 2 --api-port 6550 --subnet 172.23.0.0/16 \
-p "80:8000@loadbalancer" -p "443:8443@loadbalancer" 

k3d kubeconfig merge sidcluster --kubeconfig-switch-context
kubectl cluster-info
kubectl get nodes -o wide
```
Use the internal IP numbers from node output for your hosts file.

### Setup hostname
```/etc/hosts
172.23.0.2        argocd.k8s.test dashboard.k8s.test traefik.k8s.test guestbook.stb.k8s.test guestbook.sid.k8s.test
172.23.0.3        argocd.k8s.test dashboard.k8s.test traefik.k8s.test guestbook.stb.k8s.test guestbook.sid.k8s.test
172.23.0.4        argocd.k8s.test dashboard.k8s.test traefik.k8s.test guestbook.stb.k8s.test guestbook.sid.k8s.test
```

### expose traefik dashboard in k3s
```bash
#kubectl apply -f ./namespaces/kube-system/traefik.hconfig.yaml.dis -n kube-system
#traffic will now redeploy traefik this can take a while
kubectl apply -f ./namespaces/kube-system/traefik.ingress.yaml -n kube-system
#kubectl port-forward -n kube-system "$(kubectl get pods -n kube-system| grep '^traefik-' | awk '{print $1}')" 9000:9000
curl -k -H "Host: traefik.k8s.test" https://172.23.0.3/dashboard/
```

```obsolete
If you want to persist the changes on k3s restart you can create a new file in the /var/rancher/k3s/server/manifests/
named traefik-anything-you-want.yaml and k3s will pick up the configuration on file change and k3s restarts.
https://traefik.k8s.test/dashboard/
```


### install argocd in k3s
```bash
kubectl create namespace argocd
kubectl apply -f ./namespaces/argocd/argocd-cmd-params-cm.yaml -n argocd 
#kubectl apply -f ./namespaces/argocd/argocd.install.argocd.yaml -n argocd
kubectl apply -f ./namespaces/argocd/argocd-ha.install.argocd.yaml -n argocd
kubectl apply -f ./namespaces/argocd/argocd.ingress.argocd.yaml -n argocd

kubectl port-forward svc/argocd-server -n argocd 8080:443
# get argocd initial admin secret with kubectl and delete it
kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 -d
#curl -k -H "Host: argocd.k8s.test" https://172.23.0.3/
```

Log in to argocd using the initial admin secret and change the password.
default username is admin.
localhost:8080

Don't forget to delete the secret after you have changed the password.
```bash
kubectl -n argocd delete secret argocd-initial-admin-secret
```

### install argocli for local use
Cli tool needed for easy backups and restores, but can be handy for setting up apps as well.
```bash
curl -sSL -o argocd-linux-amd64 https://github.com/argoproj/argo-cd/releases/latest/download/argocd-linux-amd64
sudo install -m 555 argocd-linux-amd64 /usr/local/bin/argocd
rm argocd-linux-amd64
argocd admin initial-password -n argocd
# or
argocd login argocd.k8s.test
```

### Setup apps and repositories
```bash
#kubectl config get-contexts -o name
#argocd cluster add k3d-sidcluster # can be used to connect/manage external clusters
#kubectl config set-context --current --namespace=sid
```

#### Add git repo to argocd
```bash
argocd repo add https://github.com/jonnyhoeven/argocd.git \
  --username anything \
  --password *obfuscated*
```


#### argocd
```bash
kubectl create namespace argocd
argocd app create argocd \
  --repo https://github.com/jonnyhoeven/argocd.git \
  --path namespaces/argocd \
  --dest-server https://kubernetes.default.svc \
  --dest-namespace argocd \
  --sync-policy automated \
  --self-heal \
  --directory-recurse
```



#### Sid namespace
```bash
kubectl create namespace sid
argocd app create guestbook-sid \
  --repo https://github.com/jonnyhoeven/argocd.git \
  --path namespaces/sid/guestbook \
  --dest-server https://kubernetes.default.svc \
  --dest-namespace sid \
  --sync-policy automated \
  --self-heal \
  --auto-prune \
  --allow-empty \
  --directory-recurse   
curl -k -H "Host: guestbook.sid.k8s.test" https://guestbook.sid.k8s.test
```

#### Stable namespace
```bash
kubectl create namespace stb
argocd app create guestbook-stb \
  --repo https://github.com/jonnyhoeven/argocd.git \
  --path namespaces/stb/guestbook \
  --dest-server https://kubernetes.default.svc \
  --dest-namespace stb \
  --sync-policy automated \
  --self-heal \
  --auto-prune \
  --allow-empty \
  --directory-recurse   
curl -k -H "Host: guestbook.stb.k8s.test" https://guestbook.stb.k8s.test
```

#### traefik settings and dashboard ingress
```bash
kubectl create namespace kube-system
argocd app create kube-system \
  --repo https://github.com/jonnyhoeven/argocd.git \
  --path namespaces/kube-system \
  --dest-server https://kubernetes.default.svc \
  --dest-namespace kube-system \
  --sync-policy automated \
  --self-heal \
  --auto-prune \
  --allow-empty \
  --directory-recurse   
```

#### kubernetes dashboard with ingress
```bash
kubectl create namespace kubernetes-dashboard
argocd app create kubernetes-dashboard \
  --repo https://github.com/jonnyhoeven/argocd.git \
  --path namespaces/kubernetes-dashboard \
  --dest-server https://kubernetes.default.svc \
  --dest-namespace kubernetes-dashboard \
  --sync-policy automated \
  --self-heal \
  --auto-prune \
  --allow-empty \
  --directory-recurse   

kubectl proxy
#http://localhost:8001/api/v1/namespaces/kubernetes-dashboard/services/https:kubernetes-dashboard:/proxy/#/login

curl -k -H "Host: dashboard.k8s.test" https://dashboard.k8s.test
``` 

#### cloudnative-pg
```bash
kubectl create namespace cloudnative-pg
argocd app create cloudnative-pg \
  --repo https://github.com/jonnyhoeven/argocd.git \
  --path namespaces/cloudnative-pg \
  --dest-server https://kubernetes.default.svc \
  --dest-namespace cloudnative-pg \
  --sync-policy automated \
  --self-heal \
  --auto-prune \
  --allow-empty \
  --directory-recurse
```

#### prometheus
```bash
kubectl create namespace prometheus
argocd app create prometheus \
  --repo https://github.com/jonnyhoeven/argocd.git \
  --path namespaces/prometheus \
  --dest-server https://kubernetes.default.svc \
  --dest-namespace prometheus \
  --sync-policy automated \
  --self-heal \
  --auto-prune \
  --allow-empty \
  --directory-recurse
```


  



