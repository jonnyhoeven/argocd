# Rancher desktop

https://docs.rancherdesktop.io/getting-started/installation/#linux


## install rachner desktop on debian

Preparations
```bash
sudo apt install pass
gpg --generate-key
pass init *keyid*

# check kvm
[ -r /dev/kvm ] && [ -w /dev/kvm ] || echo 'insufficient privileges'
# run the following command ONLY if you see the above message
# sudo usermod -a -G kvm "$USER"

# allow non-root users to bind to ports below 1024
sudo sysctl -w net.ipv4.ip_unprivileged_port_start=80
# if you want to preserve setting on reboot:
echo sudo sysctl -w net.ipv4.ip_unprivileged_port_start=80 >> /etc/sysctl.conf
```

Download deb and install
```bash
curl -s https://download.opensuse.org/repositories/isv:/Rancher:/stable/deb/Release.key | gpg --dearmor | sudo dd status=none of=/usr/share/keyrings/isv-rancher-stable-archive-keyring.gpg
echo 'deb [signed-by=/usr/share/keyrings/isv-rancher-stable-archive-keyring.gpg] https://download.opensuse.org/repositories/isv:/Rancher:/stable/deb/ ./' | sudo dd status=none of=/etc/apt/sources.list.d/isv-rancher-stable.list
sudo apt update
sudo apt install rancher-desktop
```

Or for later use uninstall:
```bash
sudo apt remove --autoremove rancher-desktop
sudo rm /etc/apt/sources.list.d/isv-rancher-stable.list
sudo rm /usr/share/keyrings/isv-rancher-stable-archive-keyring.gpg
sudo apt update
```

Start docker desktop
```bash
kubectl get nodes
```

Install argocd:
```bash
kubectl create namespace argocd
kubectl apply -f namespaces/argocd/argocd.install.argocd.yaml -n argocd
kubectl apply -f namespaces/argocd/argocd-cmd-params-cm.yaml -n argocd 
```