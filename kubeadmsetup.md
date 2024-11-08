# root user

sudo -i 

#######################################

sudo swapoff -a

#######################################

## enable IPv4 packet forwarding

cat <<EOF | sudo tee /etc/sysctl.d/k8s.conf
net.ipv4.ip_forward = 1
EOF

sudo sysctl --system

sysctl net.ipv4.ip_forward

#######################################

# containerd

sudo apt-get update
sudo apt-get install ca-certificates curl
sudo install -m 0755 -d /etc/apt/keyrings
sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg -o /etc/apt/keyrings/docker.asc
sudo chmod a+r /etc/apt/keyrings/docker.asc

echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] https://download.docker.com/linux/ubuntu \
  $(. /etc/os-release && echo "$VERSION_CODENAME") stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
sudo apt-get update

 sudo apt-get install containerd.io

#######################################

containerd config default > /etc/containerd/config.toml

## go to this vi /etc/containerd/config.toml file follow this

SystemdCgroup = false to SystemdCgroup = true

sudo systemctl restart containerd

#######################################

## kubelet kubeadm kubectl

sudo apt-get update

sudo apt-get install -y apt-transport-https ca-certificates curl gpg

curl -fsSL https://pkgs.k8s.io/core:/stable:/v1.31/deb/Release.key | sudo gpg --dearmor -o /etc/apt/keyrings/kubernetes-apt-keyring.gpg

echo 'deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] https://pkgs.k8s.io/core:/stable:/v1.31/deb/ /' | sudo tee /etc/apt/sources.list.d/kubernetes.list

sudo apt-get update
sudo apt-get install -y kubelet kubeadm kubectl
sudo apt-mark hold kubelet kubeadm kubectl

#######################################

## master setup

kubeadm init --apiserver-advertise-address private-ip-of-master-node --pod-network-cidr 10.244.0.0/16 --cri-socket unix:///var/run/containerd/containerd.sock

## exit from the root user to normal user

exit

mkdir -p $HOME/.kube
  sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
  sudo chown $(id -u):$(id -g) $HOME/.kube/config


kubeadm token create --print-join-command

 kubectl apply -f https://reweave.azurewebsites.net/k8s/v1.29/net.yaml