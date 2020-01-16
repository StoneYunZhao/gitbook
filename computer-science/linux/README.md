# Operating System

## 内核

**输入设备**：鼠标、键盘等。  
**输入设备驱动**：  
**中断事件（Interrupt Event）**：  
**输出设备**：显示器等。  
**输出设备驱动**：如显卡驱动  
**文件管理子系统（File Management Subsystem）**：硬盘是个物理设备，要按照规定格式化成为文件系统，文件管理子系统负责管理文件系统。  
**进程管理子系统（Process Management Subsystem）**：在操作系统中，进程需要分配 CPU 进行执行。  
**内存管理子系统（Memory Management Subsystem）**：不同的进程有不同的内存空间，但是整个电脑内存就这么点儿，所以需要统一管理和分配。  
**程序（Program）**：二进制的执行文件，是静态的。  
**进程（Process）**：运行起来的程序。  
**系统调用（System Call）**：很多操作是放在操作系统内核的，进程不能随便调用，比如打印机，进程需要通过系统调用来使用打印机。

![](../../.gitbook/assets/image%20%28200%29.png)

## 常用命令

### 用户管理

```bash
# 创建用户
# 创建之后不会让你输入密码，需要通过 passwd 设置密码
# 若没有指定组，则默认创建一个同名的组
useradd [username]

# 设置密码
passwd

# 用户名:密码:用户ID:组ID:用户描述:用户目录:默认命令行
/etc/passwd
root:x:0:0:root:/root:/bin/zsh
...

# 组名::组ID
/etc/group
root:x:0:
...
```

### 文件管理

```bash
# 切换目录 change directory
cd
cd ..

# 查看目录 list
ls

# 第一个字符是文件类型，d 表示目录
# 剩下的9个是权限位，三个一组，分别为用户、组、其它。
# 第二个字段是硬链接（hard link）数目
# 第三个是所属用户，第四个是所属组，第五个是文件大小，第六个是被修改日期
ls -l
drwxr-xr-x 11 root root 4096 Mar 29 17:25 resource
-rw-r--r--  1 root root   24 Mar 18 11:23 token

# 打印文件内容到命令行
cat

# 合并多个文件
cat file1.sql doc*.sql >> merge.sql

# 编辑文件
vim

# 修改文件所属用户
chown

# 修改文件所属组
chgrp

# 下载文件
wget

# 过滤文件
grep

# 分页查看文件，仅支持向后翻页
more

# 分页查看文件，可以前后翻页
less
```

### 软件管理

Linux 常用的有两种体系，CentOS 和 Ubuntu，前者使用 rpm 安装软件，后者使用 deb 安装软件。

```bash
# 安装 -i: install
rpm -i XXX.rpm # CentOS
dpkg -i XXX.rpm # Ubuntu

# 查找
rpm -qa # q: query, a: all
dpkg -l # l: list

# 删除
rpm -e # e: erase
dpkg -r # r: remove

# 搜索
yum search jdk
apt-cache jdk

# 安装
yum install XXX
apt-get install XXX

# 删除
yum erase XXX
apt-get purge XXX

# 配置文件
/etc/yum.repos.d/
/etc/apt/sources.list

```

### 运行程序

```bash
# 运行当前目录的一个程序
./program
# 退出该程序
ctrl + c

# 后台运行 no hang up
# 1文件描述符表示标准输出，2表示标准错误输出，&表示合并
# 最后一个&表示后台运行
nohup [command] > [filename] 2>&1 &

# 退出一个进程
ps -ef |grep [keyword] |awk '{print $2}'  |xargs kill -9

# 设置开机启动 /usr/lib/systemd/system 会创建一个 XXX.service
systemctl enable XXX
# 启动
systemctl start XXX
# 关闭
systemctl stop XXX

# 跟踪进程执行时系统调用和所接受的信号量
strace
```

### 系统管理

```bash
# 立即关机
shutdown -h now

# 重启
reboot

# 环境变量
.bashrc

export XXX=sss
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

### ssh 免密登录

​SSH 是一个安全性协议，默认 SSH 链接需要密码，可以通过添加配置公钥、私钥来实现免密码登录。

​对信息的加密采用 private key，对信息的解密采用 public key。若客户端A想要登录服务器B，public key 放在 B 上，private key 放在 A 上;

​当 A 连接 B 时，B 生成一个随机数并用 public key 加密发送给 A，A 用 private key 解密发送给 B，B 确认无误后，允许 A 登录。

```bash
ssh-keygen -t rsa

ip="10.4.34.47" && \
ssh root@${ip} 'mkdir -p ~/.ssh' && \
cat ~/.ssh/id_rsa.pub | ssh root@${ip} 'cat >> ~/.ssh/authorized_keys' && \
ssh root@${ip} 'chmod 600 .ssh/authorized_keys'
```

## 系统调用

`strace` 可以跟踪进程执行时系统调用和所接受的信号量。

### 进程管理

`fork`

* 在Linux 里，创建一个进程需要一个老的进程调用 fork 实现，老的进程叫做父进程（Parent Process），新的叫做子进程（Child Process）。
* 操作系统在启动的时候先创建一个所有用户进程的“祖宗进程”。
* 调用 fork 后，就有两个一样的进程，根据返回值来判断进程。若是子进程，则返回 0；若是父进程，则返回子进程的进程号。

Unix fork 使用了[写时复制](../../java/concurrency/concurrency-design-patterns/copy-on-write.md)的思想，子进程会创建父进程的完整副本，比如父进程有 1G 内存，那么子进程会复制完整的 1G 内存。但是 Linux 优化了，fork 的时候让父子进程共享地址空间，仅在父进程或子进程写入的时候才会复制。

`execve`

调用 fork 后，判断 fork 返回0，再调用 execve 来执行另一个程序。

`waitpid`

父进程调用它，将子进程的 ID 作为参数，父进程就可以知道子进程运行是否结束，是否成功。

### 内存管理

每个进程都有自己的**进程内存空间**，又分为代码段（Code Segment）、数据段（Data Segment）。数据段又分为局部变量的部分，函数进入与退出的时候创建与销毁；动态分配部分，保存较长、显式才销毁，这部分称为堆（Heap）。

`brk`：适合申请的内存较小，会和原来的堆数据连在一起。

`mmap`：重新划分一块区域。

### 文件管理

Linux 一切皆文件，标准输出（stdout）、管道、Socket、设备、文件夹等都是文件，Linux 给 每个文件分配一个文件描述符（File Descriptor），是一个整数。

`open`

`close`

`create`

`lseek`：跳到文件的某个位置。

`read`

`write`

### 信号处理

`kill`

将一个用户信号发送给另一个进程。

`sigaction`

注册一个信号处理函数。

### 进程间通信

`msgget`

创建消息队列。

`msgsnd`

发送消息给消息队列。

`msgrcv`

从消息队列中取消息。

`shmget`

创建共享内存。

`shmat`

将共享内存映射到自己的内存空间。

`sem_wait`

占用信号量，若无人占用则可以占用；若已经被占用，则需要等待。

`sem_post`

释放信号量。

### 网络通信

`socket`

创建套接字。

`bind`

绑定端口。

`connect`

发起连接。

`accept`

接受连接。

`listen`

监听。

### Glibc

Linux 下标准的 C 库，GUN 发布的 libc 库。Glibc 为程序员提供丰富的 API，除了例如字符串处理、数学运算等用户态服务之外，最重要的是封装了操作系统提供的系统提供的系统服务，即系统调用的封装。

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

#### Reference

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

