# rabbitmq-example
实现一些工作模式。

# ubuntu设置全局代理
sudo vi /etc/environment
http_proxy="http://192.168.0.22:1087/"
https_proxy="http://192.168.0.22:1087/"

## 永久关闭swap
sudo swapoff -a  
vim /etc/fstab


# docker设置http代理

sudo mkdir -p /etc/systemd/system/docker.service.d
sudo vim /etc/systemd/system/docker.service.d/http-proxy.conf

[Service]
Environment="HTTP_PROXY=http://192.168.0.22:1087"
Environment="HTTPS_PROXY=http://192.168.0.22:1087"
Environment="NO_PROXY=localhost,127.0.0.1,docker-registry.example.com,.corp"

sudo systemctl daemon-reload
sudo systemctl restart docker
sudo systemctl show --property=Environment docker

wget https://raw.githubusercontent.com/coreos/flannel/v0.12.0/Documentation/kube-flannel.yml
kubectl apply -f kube-flannel.yml 

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


# 初始化集群

kubeadm init --pod-network-cidr=10.244.0.0/16 --apiserver-advertise-address=192.168.0.103