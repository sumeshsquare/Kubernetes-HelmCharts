# Kubernetes-HelmCharts

Helm charts directory structure for simple nginx app with Varnish:

```
├── nginx-app
│   ├── Chart.yaml
│   ├── config
│   │   └── default.vcl
│   ├── templates
│   │   ├── nginx-deployment.yaml
│   │   ├── nginx-service.yaml
│   │   ├── varnish-config-map.yaml
│   │   ├── varnish-deployment.yaml
│   │   └── varnish-service.yaml
│   └── values.yaml
└── README.md


minikube version: v1.11.0
helm version: v3.2.4+g0ad800e

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

List the helm apps: `helm list`

```
NAME    	NAMESPACE	REVISION	UPDATED                               	STATUS  	CHART        	APP VERSION
my-nginx	default  	6       	2020-06-30 15:05:33.07778323 +0530 IST	deployed	nginx-app-0.2	1    
```

To rollback (value is REVISION): `helm rollback my-nginx 5`

To delete the app: `helm delete my-nginx`




#### Setup Varnish

varnish-deployment.yaml

```
kind: Deployment
apiVersion: apps/v1
metadata:
  name: varnish-proxy
  labels:
    app: varnish-proxy
spec:
  replicas: 1
  selector:
    matchLabels:
      app: varnish-proxy
  template:
    metadata:
      name: varnish-proxy
      creationTimestamp: null
      labels:
        app: varnish-proxy
    spec:
      volumes:
        - name: varnish-config
          configMap:
            name: varnish-vcl
            items:
              - key: default.vcl
                path: default.vcl
            defaultMode: 420
      containers:
        - name: varnish
          image: million12/varnish
          ports:
            - containerPort: 80
              protocol: TCP
          env:
            - name: VCL_CONFIG
              value: /etc/varnish/default.vcl
          resources: {}
          volumeMounts:
            - name: varnish-config
              mountPath: /etc/varnish/
          terminationMessagePath: /dev/termination-log
          terminationMessagePolicy: File
          imagePullPolicy: Always
      restartPolicy: Always
      terminationGracePeriodSeconds: 30
      dnsPolicy: ClusterFirst
      securityContext: {}
      schedulerName: default-scheduler

```

varnish-service.yaml

```
kind: Service
apiVersion: v1
metadata:
  name: varnish-svc
spec:
  ports:
    - protocol: TCP
      port: 80
      targetPort: 80
      nodePort: 32055
  selector:
    app: varnish-proxy
  clusterIP: 10.105.57.236
  type: NodePort

```

copy the varnish config file `default.vcl` in config directory

create varnish-config-map.yaml with varnish config file location from relative path

```
kind: ConfigMap
apiVersion: v1
metadata:
  name: varnish-vcl

data:
  {{ (.Files.Glob "config/default.vcl").AsConfig | indent 2 }}
```

Run: `helm upgrade my-nginx .`

To get the service url, run `minikube service varnish-svc --url`
