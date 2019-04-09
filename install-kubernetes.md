# Installing kubernetes on ubuntu

## Steps for installation

### Install Docker
````sh
# download and add the gpg key for docker repos
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -

# add the debian 64 bit docker repo
sudo add-apt-repository \
   "deb [arch=amd64] https://download.docker.com/linux/ubuntu \
   $(lsb_release -cs) \
   stable"

# update package manager
sudo apt-get update

# install docker (specific version)
sudo apt-get install -y docker-ce=18.06.1~ce~3-0~ubuntu

# mark docker to not automatically update
sudo apt-mark hold docker-ce

# check
sudo docker version
````  

### Install kubeadm, kubelet and kubectl
````sh
# download and add the gpg key for kubernetes repos
curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | sudo apt-key add -

# output a file which contains the url for the kubernetes repo
cat << EOF | sudo tee /etc/apt/sources.list.d/kubernetes.list
deb https://apt.kubernetes.io/ kubernetes-xenial main
EOF

# update package manager
sudo apt-get update

# install kubelet, kubeadm and kubectl (specific versions)
sudo apt-get install -y kubelet=1.12.7-00 kubeadm=1.12.7-00 kubectl=1.12.7-00

# mark kubelet, kubeadm and kubectl to not automatically update
sudo apt-mark hold kubelet kubeadm kubectl

# check
kubeadm version
````

### Bootstrapping the Cluster
**MASTER**
````sh
# initialize the cluster on the MASTER NODE
sudo kubeadm init --pod-network-cidr=10.244.0.0/16

# local kubeconfig (will be output of previous command)
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config

# check (look for server version)
kubectl version
````  

**WORKER**
````sh
# add worker node to cluster
# the ip, token and hash will be output of kubeadm init on master
sudo kubeadm join $some_ip:6443 --token $some_token --discovery-token-ca-cert-hash $some_hash

````

> **Common error**  
> Check the security groups of the cluster, so that all nodes can communicate with each other

**MASTER**
````sh
# check nodes from the master
# STATUS "NotReady" is okay due to missing networking
kubectl get nodes
````

### Configure networking with flannel
**Note:** Flannel is just an example. Different products can be used from https://kubernetes.io/docs/concepts/cluster-administration/networking/.  

````sh
# edit sysctl configuration
echo "net.bridge.bridge-nf-call-iptables=1" | sudo tee -a /etc/sysctl.conf
sudo sysctl -p
````  

**MASTER**
````sh
# install flannel
kubectl apply -f https://raw.githubusercontent.com/coreos/flannel/bc79dd1505b0c8681ece4de4c0d86c5cd2643275/Documentation/kube-flannel.yml

# check
# STATUS schould be Ready
kubectl get nodes

# verify running system pods
kubectl get pods -n kube-system
````

