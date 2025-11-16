# GetWay-API
# Kubernetes Gateway API Demo Repository

This repository provides a complete working example of using **NGINX Gateway Fabric** with the **Kubernetes Gateway API**. It includes two demo applications, a Gateway listener, and an HTTPRoute with URL rewriting.

---

## 📁 Repository Structure

```
k8s-gateway-sample-repo/
├─ README.md
├─ LICENSE
├─ .gitignore
├─ install.sh
├─ manifests/
│  ├─ apps/
│  │  ├─ app1.yaml
│  │  └─ app2.yaml
│  ├─ gateway.yaml
│  └─ route.yaml
└─ docs/
   └─ how-to-deploy.md
```

---

## 🚀 Deployment Manifests

### **manifests/apps/app1.yaml**

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: app1
  namespace: default
spec:
  replicas: 3
  selector:
    matchLabels:
      app: prod
  template:
    metadata:
      labels:
        app: prod
    spec:
      containers:
      - name: simple-webserver
        image: menamagdyhalem/simple-webserver
        ports:
        - containerPort: 3000
---
apiVersion: v1
kind: Service
metadata:
  name: app1-svc
  namespace: default
spec:
  type: ClusterIP
  selector:
    app: prod
  ports:
  - port: 2000
    targetPort: 3000
```

### **manifests/apps/app2.yaml**

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: app2
  namespace: default
spec:
  replicas: 3
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: gcr.io/google-samples/hello-app:2.0
        ports:
        - containerPort: 8080
---
apiVersion: v1
kind: Service
metadata:
  name: app2-svc
  namespace: default
spec:
  type: ClusterIP
  selector:
    app: nginx
  ports:
  - port: 2500
    targetPort: 8080
```

---

### **manifests/gateway.yaml**

```yaml
apiVersion: gateway.networking.k8s.io/v1
kind: Gateway
metadata:
  name: my-gateway
  namespace: default
spec:
  gatewayClassName: nginx
  listeners:
  - name: http
    protocol: HTTP
    port: 80
    allowedRoutes:
      namespaces:
        from: All
```

---

### **manifests/route.yaml**

```yaml
apiVersion: gateway.networking.k8s.io/v1
kind: HTTPRoute
metadata:
  name: apps-route
  namespace: default
spec:
  parentRefs:
  - name: my-gateway
  rules:
  - matches:
    - path:
        type: PathPrefix
        value: /app1
    filters:
    - type: URLRewrite
      urlRewrite:
        path:
          type: ReplacePrefixMatch
          replacePrefixMatch: /
    backendRefs:
    - name: app1-svc
      port: 2000

  - matches:
    - path:
        type: PathPrefix
        value: /app2
    filters:
    - type: URLRewrite
      urlRewrite:
        path:
          type: ReplacePrefixMatch
          replacePrefixMatch: /
    backendRefs:
    - name: app2-svc
      port: 2500
```

---

## 📦 install.sh (Optional Installer Script)

```bash
#!/usr/bin/env bash
set -e
VER=v1.6.2

# Install Standard Gateway API CRDs
kubectl kustomize "https://github.com/nginx/nginx-gateway-fabric/config/crd/gateway-api/standard?ref=${VER}" | kubectl apply -f -

# Install NGINX Gateway Fabric CRDs
kubectl apply -f https://raw.githubusercontent.com/nginx/nginx-gateway-fabric/${VER}/deploy/crds.yaml

# Install the controller (NodePort version)
kubectl apply -f https://raw.githubusercontent.com/nginx/nginx-gateway-fabric/${VER}/deploy/nodeport/deploy.yaml
```

---

## 🧪 How to Deploy

1. Install CRDs and Controller:

```bash
bash install.sh
```

2. Deploy applications:

```bash
kubectl apply -f manifests/apps/app1.yaml
kubectl apply -f manifests/apps/app2.yaml
```

3. Deploy Gateway and Route:

```bash
kubectl apply -f manifests/gateway.yaml
kubectl apply -f manifests/route.yaml
```

4. Get the NodePort:

```bash
kubectl get svc -n nginx-gateway
```

5. Access apps:

```
http://<PUBLIC-IP>:<NODEPORT>/app1
http://<PUBLIC-IP>:<NODEPORT>/app2
```

---

## ✔ Ready for GitHub

Use the following commands:

```bash
git init
git add .
git commit -m "Initial commit: Gateway API demo"
git branch -M main
git remote add origin git@github.com:<user>/<repo>.git
git push -u origin main
```

---

If you want, I can also rewrite the README.md in full professional style or add diagrams/documentation.
