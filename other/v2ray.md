# v2ray

## v2ray

### 安装

```text
bash <(curl -L -s https://install.direct/go.sh)
​
statemctl enable v2ray
systemctl start v2ray
systemctl status v2ray
```

###  配置

`cat /etc/v2ray/config.json`

```text
{
  "inbounds": [{
    "port": XXX,
    "listen": "127.0.0.1",
    "protocol": "vmess",
    "settings": {
      "clients": [
        {
          "id": "XXX",
          "alterId": 64
        }
      ]
    },
    "streamSettings": {
      "network": "ws",
      "wsSettings": {
        "path": "/XXX"
      }
    }
  }],
  "outbounds": [{
    "protocol": "freedom",
    "settings": {}
  }]
}
```

## 证书

```text
yum -y install epel-release
yum -y install certbot
certbot certonly --standalone -d XXX
```

## nginx

### 安装

```text
yum install -y nginx
systemctl enable nginx
systemctl start nginx
systemctl status nginx
```

### 配置

`cat /etc/nginx/conf.d/v2ray.conf`

```text
server {
    listen       443 ssl;
    server_name  XXX;
​
    ssl_certificate    /etc/letsencrypt/live/XXX/fullchain.pem;
    ssl_certificate_key    /etc/letsencrypt/live/XXX/privkey.pem;
    ssl_protocols TLSv1 TLSv1.1 TLSv1.2;
    ssl_ciphers ECDHE-RSA-AES128-GCM-SHA256:HIGH:!aNULL:!MD5:!RC4:!DHE;
    ssl_prefer_server_ciphers on;
    ssl_session_cache shared:SSL:10m;
    ssl_session_timeout 10m;
    error_page 497  https://$host$request_uri;
​
location /tian {
    proxy_pass       http://127.0.0.1:XXX;
    proxy_redirect             off;
    proxy_http_version         1.1;
    proxy_set_header Upgrade   $http_upgrade;
    proxy_set_header Connection "upgrade";
    proxy_set_header Host      $http_host;
    }
}
```

