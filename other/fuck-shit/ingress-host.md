# Ingress Host

有两个 ingress，一个配置了 host: m.com，另一个没有配置 host，所以意义为匹配任何 host。

**坑**：以 m.com/n 访问此集群，不能匹配到 n。

**原因**：在用 nginx ingress 的时候，会为集群中的 ingress 的所有 host 增加一个 server 配置，而 nginx 的匹配规则是有限进入 server 模块匹配，再根据 path 匹配。

```yaml
apiVersion: extensions/v1beta1
kind: Ingress
spec:
  rules:
  - host: m.com
    http:
      paths:
      - path: /m
          
apiVersion: extensions/v1beta1
kind: Ingress
spec:
  rules:
  - http:
      paths:
      - path: /n
```

由 nginx ingress 解析后的 nginx 配置：

```text
# 由 m ingress 生成的 nginx 配置
server {
    server_name  m.com;

    location /m { 
    }
}

# 由 n ingress 生成的 nginx 配置
server {
    server_name  n.com;

    location /n { 
    }
}

# 所以通过 m.com/n 访问的时候，不能匹配到 n 服务。
# 与 n ingress 的定义（匹配来自任何 host /n 路径）不符。
```

