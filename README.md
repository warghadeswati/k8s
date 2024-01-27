# k8s

Kubernetes installation: https://github.com/shazforiot/kubernetes-cluster-setup-using-kubeadm

hostnamectl set-hostname master
hostnamectl set-hostname worker1
hostnamectl set-hostname worker2

Step1 (Fire on all three nodes )
sudo setenforce 0
sudo sed -i 's/^SELINUX=enforcing$/SELINUX=permissive/' /etc/selinux/config


Step2 : (Fire on all nodes)

# disable swap (Fire on all nodes)
sudo sed -i '/ swap / s/^\(.*\)$/#\1/g' /etc/fstab
swapoff -a

Step3 : (Fire on all nodes)

cat <<EOF | sudo tee /etc/modules-load.d/k8s.conf
overlay
br_netfilter
EOF
===================================================
Step4 : (Fire on all nodes)
sudo modprobe overlay
sudo modprobe br_netfilter
==================================================
Step5 : (Fire on all nodes)
cat <<EOF | sudo tee /etc/sysctl.d/k8s.conf
net.bridge.bridge-nf-call-iptables  = 1
net.bridge.bridge-nf-call-ip6tables = 1
net.ipv4.ip_forward                 = 1
EOF

Step6 : (Fire on all nodes)

sudo sysctl --system
lsmod | grep br_netfilter
lsmod | grep overlay
sysctl net.bridge.bridge-nf-call-iptables net.bridge.bridge-nf-call-ip6tables net.ipv4.ip_forward
==============
Step7 -- fire on all nodes
sudo yum-config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo
sudo yum install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
systemctl enable --now docker
docker ps
 =======================
step 8 ( fire on all nodes)

containerd config default | sudo tee /etc/containerd/config.toml >/dev/null 2>&1

sudo sed -i 's/SystemdCgroup \= false/SystemdCgroup \= true/g' /etc/containerd/config.toml

systemctl restart containerd
==============================================
step 9 ( fire on all nodes)

cat <<EOF | sudo tee /etc/yum.repos.d/kubernetes.repo
[kubernetes]
name=Kubernetes
baseurl=https://pkgs.k8s.io/core:/stable:/v1.28/rpm/
enabled=1
gpgcheck=1
gpgkey=https://pkgs.k8s.io/core:/stable:/v1.28/rpm/repodata/repomd.xml.key
exclude=kubelet kubeadm kubectl cri-tools kubernetes-cni
EOF

sudo yum install -y kubelet kubeadm kubectl --disableexcludes=kubernetes

systemctl enable --now kubelet

systemctl status docker
systemctl status containerd

systemctl status kubelet

kubectl version
kubeadm version
kubelet --version

=================================
Step 10 ( Only fire on master node)

kubeadm init --pod-network-cidr=10.244.0.0/16  --control-plane-endpoint=3.110.134.95 --ignore-preflight-errors=Mem --cri-socket /run/containerd/containerd.sock 

mkdir -p $HOME/.kube  (execute on master node only)
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config  (execute on master node only)
sudo chown $(id -u):$(id -g) $HOME/.kube/config  (execute on master node only)

==============================================
Copy kubeadm join command and fire it on worker1 and worker2
===============================================
Step11 (Only fire on master nodes)
kubectl create -f https://github.com/flannel-io/flannel/releases/latest/download/kube-flannel.yml
curl https://raw.githubusercontent.com/projectcalico/calico/v3.26.1/manifests/canal.yaml -O
kubectl apply -f canal.yaml
(To read more about cni comparision , please https://kubevious.io/blog/post/comparing-kubernetes-container-network-interface-cni-providers)
