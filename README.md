# ubuntu设置全局代理
sudo vim /etc/environment
http_proxy="http://192.168.0.50:1087/"
https_proxy="http://192.168.0.50:1087/"

## 设置root密码
sudo passwd root


## 永久关闭swap
sudo swapoff -a  
vim /etc/fstab


# docker

## install docker ce

apt-get update && apt-get install 
apt-transport-https ca-certificates curl software-properties-common
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | apt-key add -

add-apt-repository \
  "deb [arch=amd64] https://download.docker.com/linux/ubuntu \
  $(lsb_release -cs) \
  stable"

apt-get update && apt-get install -y\
  containerd.io=1.2.13-2 \
  docker-ce=5:19.03.11~3-0~ubuntu-$(lsb_release -cs) \
  docker-ce-cli=5:19.03.11~3-0~ubuntu-$(lsb_release -cs)

cat > /etc/docker/daemon.json <<EOF
{
  "exec-opts": ["native.cgroupdriver=systemd"],
  "log-driver": "json-file",
  "log-opts": {
    "max-size": "100m"
  },
  "storage-driver": "overlay2"
}
EOF

mkdir -p /etc/systemd/system/docker.service.d
systemctl daemon-reload
systemctl restart docker
sudo systemctl enable docker

## docker设置http代理
sudo mkdir -p /etc/systemd/system/docker.service.d
sudo vim /etc/systemd/system/docker.service.d/http-proxy.conf

[Service]
Environment="HTTP_PROXY=http://192.168.0.50:1087"
Environment="HTTPS_PROXY=http://192.168.0.50:1087"
Environment="NO_PROXY=localhost,127.0.0.1,docker-registry.example.com,.corp"

sudo systemctl daemon-reload
sudo systemctl restart docker
sudo systemctl show --property=Environment docker


## 阿里云镜像地址

https://czz4db29.mirror.aliyuncs.com

## cgroups driver set 

cat > /etc/docker/daemon.json <<EOF
{
  "exec-opts": ["native.cgroupdriver=systemd"],
  "log-driver": "json-file",
  "log-opts": {
    "max-size": "100m"
  },
  "storage-driver": "overlay2"
}
EOF


# 搭建集群

## install  kubelet kubeadm kubectl

sudo apt-get update && sudo apt-get install -y apt-transport-https curl
curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | sudo apt-key add -
cat <<EOF | sudo tee /etc/apt/sources.list.d/kubernetes.list
deb https://apt.kubernetes.io/ kubernetes-xenial main
EOF
sudo apt-get update
sudo apt-get install -y kubelet kubeadm kubectl
sudo apt-mark hold kubelet kubeadm kubectl

## 主节点
kubeadm init --pod-network-cidr=10.244.0.0/16 

## 配置config

export KUBECONFIG=/etc/kubernetes/admin.conf

## apply fannel 

docker pull rancher/coreos-flannel:v0.12.0-amd64
wget https://raw.githubusercontent.com/coreos/flannel/v0.12.0/Documentation/kube-flannel.yml
kubectl apply -f kube-flannel.yml 

## join集群

kubeadm join 192.168.0.103:6443 --token h97gmy.tkwor0qfsbyh6fpe \
    --discovery-token-ca-cert-hash sha256:fcf536bfec0578a77815f0fad5b7d4a61a7ed74f1756c89a5030db9a7c682215 

scp /etc/kubernetes/admin.conf hzt@192.168.0.102:/etc/kubernetes/admin.conf

## 管理

journalctl -f -u kubelet
kubectl get pods -n kube-system -o wide
kubectl describe pod yourPodName -n kube-system