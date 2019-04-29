# Kubernetes

## 外部 service

### 指定 Endpoint

```yaml
kind: Endpoints
apiVersion: v1
metadata:
  name: minio
subsets:
  - addresses:
    - ip: 172.16.1.18
    ports:
    - port: 9000

---
kind: Service
apiVersion: v1
metadata:
  name: minio
spec:
  ports:
  - port: 80
    targetPort: 9000
```

### ExternalName

```yaml
kind: Service
apiVersion: v1
metadata:
  name: m
spec:
  type: ExternalName
  externalName: bf-dev-databag.oss-cn-hangzhou-internal.aliyuncs.com
```

## 安装

```bash
# 1. 设置免密登录
ip="10.4.34.47" && \
ssh root@${ip} 'mkdir -p ~/.ssh' && \
cat ~/.ssh/id_rsa.pub | ssh root@${ip} 'cat >> ~/.ssh/authorized_keys' && \
ssh root@${ip} 'chmod 600 .ssh/authorized_keys'

# 2. 设置网络、hostname
chkconfig NetworkManager on
systemctl start NetworkManager.service
nmtui

# 3. 安装常用的工具
yum update -y && \
yum install -y tree git wget nfs-utils zsh net-tools ntp vim bind-utils traceroute

# 4. 安装 zsh 和插件
sh -c "$(wget https://raw.github.com/robbyrussell/oh-my-zsh/master/tools/install.sh -O -)"
git clone https://github.com/zsh-users/zsh-syntax-highlighting.git ${ZSH_CUSTOM:-~/.oh-my-zsh/custom}/plugins/zsh-syntax-highlighting
git clone https://github.com/zsh-users/zsh-autosuggestions $ZSH_CUSTOM/plugins/zsh-autosuggestions

vi ~/.zshrc 
	change theme to af-magic
	zsh-syntax-highlighting
	zsh-autosuggestions
	alias vi="vim"
source ~/.zshrc	

# 5. 关闭防火墙
systemctl stop firewalld
systemctl disable firewalld

# 6. 关闭内存交换
swapoff -a
vi /etc/fstab 

free -h

# 7. 安装 docker
yum remove docker \
docker-client \
docker-client-latest \
docker-common \
docker-latest \
docker-latest-logrotate \
docker-logrotate \
docker-selinux \
docker-engine-selinux \
docker-engine

rm -rf /var/lib/docker

yum install -y yum-utils \
device-mapper-persistent-data \
lvm2

cat <<EOF > /etc/yum.repos.d/docker-ce.repo
[docker-ce-stable]
name=Docker CE Stable - \$basearch
baseurl=https://mirrors.tuna.tsinghua.edu.cn/docker-ce/linux/centos/7/\$basearch/stable
enabled=1
gpgcheck=1
gpgkey=https://mirrors.tuna.tsinghua.edu.cn/docker-ce/linux/centos/gpg
EOF
yum makecache fast

yum list docker-ce --showduplicates | sort -r
yum install -y --setopt=obsoletes=0 docker-ce-17.03.2.ce docker-ce-selinux-17.03.2.ce-1.el7.centos
iptables -P FORWARD ACCEPT
systemctl enable docker && systemctl start docker

yum-config-manager --disable docker-ce-stable

# 8. 安装 Kuberntes
cat <<EOF > /etc/yum.repos.d/kubernetes.repo
[kubernetes]
name=Kubernetes
baseurl=http://mirrors.aliyun.com/kubernetes/yum/repos/kubernetes-el7-x86_64
enabled=1
gpgcheck=0
repo_gpgcheck=0
gpgkey=http://mirrors.aliyun.com/kubernetes/yum/doc/yum-key.gpg
        http://mirrors.aliyun.com/kubernetes/yum/doc/rpm-package-key.gpg
EOF
yum makecache fast

yum list kubelet --showduplicates | sort -r
yum list kubeadm --showduplicates | sort -r
yum list kubectl --showduplicates | sort -r

setenforce 0
vi /etc/selinux/config
	SELINUX=disabled

yum install -y kubelet-1.11.0 kubectl-1.11.0 kubeadm-1.11.0
systemctl enable kubelet && systemctl start kubelet
yum-config-manager --disable kubernetes

cat <<EOF >  /etc/sysctl.d/k8s.conf
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
EOF

sysctl net.bridge.bridge-nf-call-iptables=1

sysctl --system

# docker info | grep cgroup
# cat /etc/systemd/system/kubelet.service.d/10-kubeadm.conf
# sed -i "s/cgroup-driver=systemd/cgroup-driver=cgroupfs/g" /etc/systemd/system/kubelet.service.d/10-kubeadm.conf

DOCKER_CGROUPS=$(docker info | grep 'Cgroup' | cut -d' ' -f3)
echo $DOCKER_CGROUPS
cat >/etc/sysconfig/kubelet<<EOF
KUBELET_EXTRA_ARGS="--cgroup-driver=$DOCKER_CGROUPS --pod-infra-container-image=registry.cn-hangzhou.aliyuncs.com/bimface_common/pause-amd64:3.1"
EOF

systemctl daemon-reload && systemctl restart kubelet

# master初始化
kubeadm init --config kubeadm/kubeadm.yaml

mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config

# 安装网络插件
kubectl apply -f flannel/kube-flannel.yml

# 重置 master
kubeadm reset -f
ifconfig cni0 down
ip link delete cni0
ifconfig flannel.1 down
ip link delete flannel.1
rm -rf /var/lib/cni/
rm -rf $HOME/.kube
rm -rf /etc/kubernetes

# 查看 kubelet 日志
journalctl -l -u kubelet

# 确认 kubernetes 安装成功
curl http://kubernetes-dashboard.kube-system/api/v1/login/status

# 重新生产 join token
kubeadm token list
kubeadm token create
openssl x509 -pubkey -in /etc/kubernetes/pki/ca.crt | openssl rsa -pubin -outform der 2>/dev/null | openssl dgst -sha256 -hex | sed 's/^.* //'
# or
kubeadm token create --print-join-command

```

