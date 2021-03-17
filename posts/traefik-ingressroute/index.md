# 使用 Traefik 暴露内部服务


## 部署测试 web 应用

使用 Deployment 部署 demoapp， 启动两个 pod 实例， 资源配置清单 `demoapp.yaml`  如下：

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    app: demoapp
  name: demoapp
spec:
  replicas: 2
  selector:
    matchLabels:
      app: demoapp
  template:
    metadata:
      labels:
        app: demoapp
    spec:
      containers:
      - image: ikubernetes/demoapp:v1.1
        name: demoapp
---
apiVersion: v1
kind: Service
metadata:
  labels:
    app: demoapp
  name: demoapp
spec:
  ports:
  - name: www
    port: 80
    protocol: TCP
    targetPort: 80
  selector:
    app: demoapp
  type: ClusterIP
```

在集群中应用 `demoapp.yaml`

```bash
kubectl apply -f demoapp.yaml
```

## 配置 traefik IngressRoute

*ingress-route.yaml*

```yaml
apiVersion: traefik.containo.us/v1alpha1
# 使用 ingress route
kind: IngressRoute
metadata:
  name: demo-ingress
  namespace: default
spec:
  # 指定使用的入口，由于是 http 流量，我们使用 web 入口（入口在 traefik 命令行配置参数自行配置）
  entryPoints:
    - web
  routes:
  # 指定匹配规则
  - match: Host(`d.wglee.cn`)
    kind: Rule
    services:
    - name: demoapp
      port: 80
```

在集群中应用 `ingress-route.yaml`

```bash
kubectl apply -f ingress-route.yaml
```

## 配置负载均衡反向代理

我们把只要是 `wglee.cn` 这个域名的流量（配置中使用通配符）统一反代到 traefik 的 web 入口

```bash
upstream k8s_services {
    server 192.168.31.21:80;
    server 192.168.31.22:80;
}

server {
        listen 80;
        server_name *.wglee.cn;

        location / {
        proxy_pass http://k8s_services;
        include proxy.conf;
        }
}
```

## 查看及测试

打开 `traefik-dashboard` 页面，查看配置的规则是否生效，使用域名访问此 web 服务。
