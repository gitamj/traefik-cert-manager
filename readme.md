# Traefik v2 + cert-manager

# Kubernetes

#### 1. Enable root user
```
sudo nano /root/.ssh/authorized_keys
```
remove till "ssh..."

#### 2. oracle only -- disable firewall 
```
sudo apt-get update; sudo apt install firewalld -y;sudo firewall-cmd --zone=public --permanent --add-port=8080/tcp; sudo firewall-cmd --zone=public --permanent --add-port=80/tcp;sudo firewall-cmd --zone=public --permanent --add-port=443/tcp;sudo firewall-cmd --zone=public --permanent --add-port=6443/tcp;sudo firewall-cmd --zone=public --permanent --add-port=10250-10252/tcp;sudo firewall-cmd --zone=public --permanent --add-port=30000-33000/tcp;sudo firewall-cmd --reload;sudo systemctl disable firewalld;sudo systemctl stop firewalld;
```

#### 3. oracle only -- disable ufw
```
sudo systemctl stop ufw
sudo systemctl disable ufw
sudo systemctl status ufw
```

#### 4. Setup Kubernetes - Master, disable swap and update hosts
```
sudo hostnamectl set-hostname master

sudo swapoff -a

sudo nano /etc/hosts
10.0.0.4        master
10.0.0.23        node1
```

#### 5. Setup Kubernetes - Node1, disable swap and update hosts
```
sudo hostnamectl set-hostname node1

sudo swapoff -a

sudo nano /etc/hosts
10.0.0.4        master
10.0.0.23        node1
```

#### 6. Installing Kubernetes, kubectl, kubeadmin and docker
```
sudo curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | sudo apt-key add -

sudo apt-add-repository "deb http://apt.kubernetes.io/ kubernetes-xenial main"

sudo apt-get update

sudo apt-get install -y docker.io kubelet kubeadm kubectl;sudo apt-mark hold docker.io kubelet kubeadm kubectl

sudo systemctl enable kubelet.service;sudo systemctl enable docker.service
```

#### 7. Configuring Kubernetes on MASTER ONLY

```
sudo kubeadm init --pod-network-cidr=10.244.0.0/16

mkdir -p $HOME/.kube;sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config;sudo chown $(id -u):$(id -g) $HOME/.kube/config

sudo kubectl apply -f https://raw.githubusercontent.com/coreos/flannel/master/Documentation/kube-flannel.yml

kubectl get pods --all-namespaces

kubectl get nodes 
```

#### 8. If there is not node 1 - master and node are same machine
```
kubectl taint nodes --all node-role.kubernetes.io/master-
```

#### 9. Node Only
```
kubeadm join 10.0.0.4:6443 --token n2besj.c1sy8un8gij7sdyw \
    --discovery-token-ca-cert-hash sha256:XXXXXXXXXXXXXXXXXXX 
```

# CERT-MANAGER

#### 10. Clone the cert manager project
```
sudo mkdir /setup; cd /setup; sudo git clone https://github.com/gitamj/traefik-cert-manager-pvt 
cd /
cd setup/traefik-cert-manager-pvt;
```

#### 10. Install traefik v2 and whoami
```
kubectl apply -f traefik/

kubectl apply -f whoami/
```

#### 11. Install cert-manager
```
kubectl apply --validate=false -f https://raw.githubusercontent.com/jetstack/cert-manager/release-0.12/deploy/manifests/00-crds.yaml

kubectl create namespace cert-manager

echo 'apiVersion: v1
kind: ServiceAccount
metadata:
  name: tiller
  namespace: kube-system
---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: tiller
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: cluster-admin
subjects:
  - kind: ServiceAccount
    name: tiller
    namespace: kube-system'| kubectl apply -f -


sudo snap install helm --classic

sudo helm repo add jetstack https://charts.jetstack.io

sudo helm repo update

sudo helm install cert-manager   --namespace cert-manager   --version v0.12.0   jetstack/cert-manager

kubectl get pods --namespace cert-manager

kubectl apply -f cert-manager/
```
#### 12. Basic Auth
```
apt install apache2-utils

htpasswd -c users admin

kubectl create secret generic basic-auth --from-file=users --namespace=whoami
```
