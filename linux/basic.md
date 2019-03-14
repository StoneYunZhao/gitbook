# Basic

## Command

cat

```bash
# 合并多个文件
cat file1.sql doc*.sql >> merge.sql
```

### find

```bash
# 在当前目录查找60天之前的文件
find . -mtime +60

# 删除当前目录下60天之前的文件
find . -mtime +60 -delete
find . -mtime +60 -exec rm -rf {} \;

# 查找当前目录下的空文件夹
find . -type d -empty

# 删除当前目录下的空文件夹
find . -type d -empty -delete
```

### ssh

在本地建立代理到 `10.5.3.32`

```text
ssh -D 0.0.0.0:1082 -f -C -q -N root@10.5.3.32
```

通过代理 Socket5 代理（`10.211.55.4:1081`）连上 `10.5.3.33`

```text
ssh -o ProxyCommand='nc -x 10.211.55.4:1081 %h %p' root@10.5.3.33
```

组合以上两个，在本地建立代理到 `192.166.127.32`，本机是通过`192.166.127.15:1031`的 Socket5 代理连到`192.166.127.32`

```text
ssh -D 0.0.0.0:1082 -f -C -q -N -o ProxyCommand='nc -x 192.166.127.15:1031 %h %p' root@192.166.127.32
```

通过跳板机直连 

```bash
cat ~/.ssh/config

Host 172.16.1.*
        User root
        Port 22
        ProxyCommand ssh root@10.0.197.12 -W %h:%p
Host test
        User root
        Port 22
        HostName 172.16.1.4
        ProxyCommand ssh root@10.0.197.12 -W %h:%p
```

* `ssh 172.16.1.*` 可以直接通过跳板机 `10.0.197.12` 连接`172.16.1.*`，即支持通配符。
* `ssh test` 可以直接通过跳板机 `10.0.197.12` 连接`172.16.1.4`。

## NFS

```bash
yum -y install firewalld
systemctl start firewalld.service
systemctl enable firewalld.service
firewall-cmd --permanent --zone=public --add-service=ssh
firewall-cmd --permanent --zone=public --add-service=nfs
firewall-cmd --zone=public --add-service=rpc-bind --permanent
firewall-cmd --zone=public --add-service=mountd --permanent
firewall-cmd --reload

yum -y install nfs-utils

systemctl enable rpcbind
systemctl start rpcbind
systemctl enable nfs-server
systemctl start nfs-server

mkdir /nfsdata
chown nfsnobody:nfsnobody  /nfsdata
chmod 755 /nfsdata

vi /etc/exports

/nfsdata 172.16.0.0/16(rw,sync,no_subtree_check,no_root_squash)

exportfs -a
```

### Reference

* [http://blog.topspeedsnail.com/archives/4109](http://blog.topspeedsnail.com/archives/4109)
* [https://www.howtoforge.com/tutorial/setting-up-an-nfs-server-and-client-on-centos-7/](https://www.howtoforge.com/tutorial/setting-up-an-nfs-server-and-client-on-centos-7/)

## DNS

### resolve.conf

Linux DNS 解析的配置文件 `/etc/resolve.conf`。

nameserver：dns服务器地址

options：

* ndot 域名中最少包含点的个数
* timeout 超时时间，单位秒，默认5
* attempts 重试次数
* rotate 默认每次都是从头到尾依次请求nameserver，加上此参数会循环请求
* single-request 默认glibc会发送IPv4和IPv6两个并发的请求，此参数会同步发送single-request-reopen CentOS 6中的DNS解析器对于ipv4和ipv6都使用同一个socket接口，在同时发出ipv4 和ipv6解析请求后，只会收到一个ipv4的解析响应，此时socket将一处于“等待”模式，等待ipv6的解析响应，故导致解析缓慢；添加single-request-reopen后就可以重新打开一个新的socket接收ipv6的解析响应，而不影响ipv4的解析响应。

### Docker 配置 DNS

* --dns=IP\_ADDRESS...
* --dns-search=DOMAIN...
* --dns-opt=OPTION...

### Kubernetes 配置 DNS

```text
apiVersion: v1
kind: Pod
metadata:
  namespace: default
  name: dns-example
spec:
  containers:
    - name: test
      image: nginx
  dnsPolicy: "None"
  dnsConfig:
    options:
    - name: timeout
      value: "1"
    - name: attempts
      value: "2"
    - name: rotate
    - name: single-request-reopen
```

### Reference

* [http://man7.org/linux/man-pages/man5/resolv.conf.5.html](http://man7.org/linux/man-pages/man5/resolv.conf.5.html)
* [https://docs.docker.com/v17.09/engine/userguide/networking/default\_network/configure-dns/](https://docs.docker.com/v17.09/engine/userguide/networking/default_network/configure-dns/)
* [https://kubernetes.io/docs/concepts/services-networking/dns-pod-service/](https://kubernetes.io/docs/concepts/services-networking/dns-pod-service/)
* [http://coolnull.com/3820.html](http://coolnull.com/3820.html)

