# Latest installation steps
## Docker install --> https://docs.docker.com/engine/install/ubuntu/
 sudo apt-get update
 sudo apt-get install \
    ca-certificates \
    curl \
    gnupg \
    lsb-release
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg
echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] https://download.docker.com/linux/ubuntu \
  $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
sudo apt-get update
sudo apt-get install docker-ce docker-ce-cli containerd.io

## Kubernetes insallation --> https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/install-kubeadm/
sudo apt-get install iptables
iptables -L
sudo iptables -A INPUT -p tcp --match multiport --dport 6443,2379,2380,10250,10259,10257 -j ACCEPT
iptables -A INPUT -i lo -p all -j ACCEPT
sudo swapoff -a
sudo sed -i '/ swap / s/^/#/' /etc/fstab
sudo mkdir /etc/docker
cat <<EOF | sudo tee /etc/docker/daemon.json
{
  "exec-opts": ["native.cgroupdriver=systemd"],
  "log-driver": "json-file",
  "log-opts": {
    "max-size": "100m"
  },
  "storage-driver": "overlay2"
}
EOF
sudo systemctl enable docker
sudo systemctl daemon-reload
sudo systemctl restart docker


lsmod | grep br_netfilter
sudo modprobe br_netfilter
cat <<EOF | sudo tee /etc/modules-load.d/k8s.conf
br_netfilter
EOF

cat <<EOF | sudo tee /etc/sysctl.d/k8s.conf
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
EOF
sudo sysctl --system

reboot

sudo apt-get update
sudo apt-get install -y apt-transport-https ca-certificates curl

sudo curl -fsSLo /usr/share/keyrings/kubernetes-archive-keyring.gpg https://packages.cloud.google.com/apt/doc/apt-key.gpg
echo "deb [signed-by=/usr/share/keyrings/kubernetes-archive-keyring.gpg] https://apt.kubernetes.io/ kubernetes-xenial main" | sudo tee /etc/apt/sources.list.d/kubernetes.list
sudo apt-get update
sudo apt-get install -y kubelet kubeadm kubectl

systemctl status kubelet
sudo apt-mark hold kubelet kubeadm kubectl

kubeadm init --pod-network-cidr=10.244.0.0/16 --ignore-preflight-errors=all (warnings will be ignored)

mkdir $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
export KUBECONFIG=/etc/kubernetes/admin.conf
kubectl get nodes
## Cluster network --> https://v1-22.docs.kubernetes.io/docs/concepts/cluster-administration/networking/
kubectl apply -f https://raw.githubusercontent.com/flannel-io/flannel/master/Documentation/kube-flannel.yml
## troubleshoot
kubedm reset



# Install Kubernetes Using Script

### `Step1: On Master Node Only`
```
## Install Docker,kubeadm,kubelet,kubectl

sudo wget https://raw.githubusercontent.com/lerndevops/labs/master/scripts/installK8S.sh -P /tmp
sudo chmod 755 /tmp/installK8S.sh
sudo bash /tmp/installK8S.sh

## Initialize kubernetes Master Node
 
   sudo kubeadm init --ignore-preflight-errors=all

   sudo mkdir -p $HOME/.kube
   sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
   sudo chown $(id -u):$(id -g) $HOME/.kube/config

   ## install networking driver -- Weave/flannel/canal/calico etc... 

   ## below installs weave networking driver 
    
   sudo kubectl apply -f "https://cloud.weave.works/k8s/net?k8s-version=$(kubectl version | base64 | tr -d '\n')" 

   # Validate:  kubectl get nodes
```
### `Step2: On All Worker Nodes`
```
## Install Docker,kubeadm,kubelet

sudo wget https://raw.githubusercontent.com/lerndevops/labs/master/scripts/installK8S.sh -P /tmp
sudo chmod 755 /tmp/installK8S.sh
sudo bash /tmp/installK8S.sh

## Run Below on Master Node to get join token 

kubeadm token create --print-join-command 

    copy the kubeadm join token from master & run it on all nodes

    Ex: kubeadm join 10.128.15.231:6443 --token mks3y2.v03tyyru0gy12mbt \
           --discovery-token-ca-cert-hash sha256:3de23d42c7002be0893339fbe558ee75e14399e11f22e3f0b34351077b7c4b56
```

# Manual Installation Steps
### `Step1:  On Master Node Only`
```
    ### INSTALL DOCKER 
    
    sudo apt-get update
    sudo apt-get install -y apt-transport-https ca-certificates curl software-properties-common

    sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
    sudo add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable"
    sudo apt-get update ; clear
    sudo apt-get install -y docker-ce
    
    sudo wget https://raw.githubusercontent.com/lerndevops/labs/master/kube/install/daemon.json -P /etc/docker
    sudo service docker restart
    sudo service docker status
   
    ### INSTALL KUBEADM,KUBELET,KUBECTL
    
    sudo echo "deb http://apt.kubernetes.io/ kubernetes-xenial main" | sudo tee /etc/apt/sources.list.d/kubernetes.list
    sudo curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | sudo apt-key add -
    sudo apt-get update ; clear
    sudo apt-get install -y kubelet kubeadm kubectl

    ### Initialize Master Node 
    
    sudo kubeadm init --ignore-preflight-errors=all
	
    sudo mkdir -p $HOME/.kube
    sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
    sudo chown $(id -u):$(id -g) $HOME/.kube/config

    ## install networking driver -- Weave/flannel/canal/calico etc... 

    ## below installs weave networking driver 
    
    sudo kubectl apply -f "https://cloud.weave.works/k8s/net?k8s-version=$(kubectl version | base64 | tr -d '\n')" 
	
    # Validate: kubectl get nodes
```
### Step2: `On All Worker Nodes:`
```
    ### INSTALL DOCKER 
    
    sudo apt-get update
    sudo apt-get install -y apt-transport-https ca-certificates curl software-properties-common

    sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
    sudo add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable"
    sudo apt-get update ; clear
    sudo apt-get install -y docker-ce
    
    sudo wget https://raw.githubusercontent.com/lerndevops/labs/master/kube/install/daemon.json -P /etc/docker
    sudo service docker restart
    sudo service docker status
   
    ### INSTALL KUBEADM,KUBELET,KUBECTL
    
    sudo echo "deb http://apt.kubernetes.io/ kubernetes-xenial main" | sudo tee /etc/apt/sources.list.d/kubernetes.list
    sudo curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | sudo apt-key add -
    sudo apt-get update ; clear
    sudo apt-get install -y kubelet kubeadm kubectl

    ## RUN Below on Master Node to get join token 
    
    kubeadm token create --print-join-command
       
    copy the kubeadm join token from master & run it on all nodes
          
    Ex: kubeadm join 10.128.15.231:6443 --token mks3y2.v03tyyru0gy12mbt \
           --discovery-token-ca-cert-hash sha256:3de23d42c7002be0893339fbe558ee75e14399e11f22e3f0b34351077b7c4b56
```
