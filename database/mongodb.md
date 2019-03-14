# MongoDB

## 安装

```bash
cat > /etc/yum.repos.d/mongodb.repo << EOF
[mongodb-org-3.4]
name=MongoDB Repository
baseurl=https://repo.mongodb.org/yum/redhat/\$releasever/mongodb-org/3.4/x86_64/
gpgcheck=1
enabled=1
gpgkey=https://www.mongodb.org/static/pgp/server-3.4.asc
EOF

yum install mongodb-org-shell -y

mongo remotedb.mlab.com:27017/<dbname> -u <user> -p <pass>
```

