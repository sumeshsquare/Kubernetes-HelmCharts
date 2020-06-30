# Kubernetes-HelmCharts

Helm charts directory structure for simple nginx app:

```
├── nginx-app
│   ├── Chart.yaml
│   └── templates
│       ├── nginx-deployment.yaml
│       └── nginx-service.yaml
└── README.md
```


#### Code below: 

Chart.yaml

```
apiVersion: v1
name: nginx-app
version: 0.2
appVersion: 1.0
description: nginx template with helm charts

```


To create nginx-deployment.yaml, execute cmd

`kubectl create deploy my-nginx --image nginx --dry-run -o yaml > nginx-deployment.yaml`

```
apiVersion: apps/v1
kind: Deployment
metadata:
  creationTimestamp: null
  labels:
    app: my-nginx
  name: my-nginx
spec:
  replicas: 3
  selector:
    matchLabels:
      app: my-nginx
  strategy: {}
  template:
    metadata:
      creationTimestamp: null
      labels:
        app: my-nginx
    spec:
      containers:
      - image: nginx
        name: nginx
        resources: {}
status: {}

```

To create nginx-service.yaml, execute cmd

`kubectl expose deploy my-nginx --port 80 --dry-run -o yaml > nginx-service.yaml`

```
apiVersion: v1
kind: Service
metadata:
  creationTimestamp: null
  labels:
    app: my-nginx
  name: my-nginx
spec:
  type: NodePort
  ports:
  - port: 80
    protocol: TCP
    targetPort: 80
    
  selector:
    app: my-nginx
status:
  loadBalancer: {}

```


#### Start apps with Helm

To deploy first time :`helm install my-nginx`

To update apps: `helm upgrade my-nginx .`
