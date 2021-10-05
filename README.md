# Kubernets (K8)
![image](https://www.net4all.ch/wp-content/uploads/sites/3/2020/12/09/kubernetes-net4all.png)
- developed by google
- container-orchestration tool
- Manages containers (Docker containers)

- Deployment as loadbalancer 
## What does K8 do for you:
- self healing 
- load balancing 
- high avaliablety 
- high scalablity 
- Disaster recovery 


## Installing K8
(This installation requires that you have Docker installed)

- Once you have docker installed 

- Access Docker and install k8 by clicking on `enable kubernetes` 
- press `apply and restart`
- after this process has fininshed The k8 symbol on the bottom should go from amber to green.  

![image](https://birthday.play-with-docker.com/images/kubernetes-docker-desktop/settings-kubernetes.png)


- To test the instalations:
access your command promot and type: 
```
kubectl version
```
you should see this as a result:

```
Client Version: version.Info{Major:"1", Minor:"20", GitVersion:"v1.20.6", GitCommit:"8a62859e515889f07e3e3be6a1080413f17cf2c3", GitTreeState:"clean", BuildDate:"2021-04-15T03:28:42Z", GoVersion:"go1.15.10", Compiler:"gc", Platform:"windows/amd64"}
Error from server (InternalError): an error on the server ("") has prevented the request from succeeding
```

# Tuesday 
## Adding mongoDB
- Create PV.yml
(presistant volume) and PVC.yml (presisnat volume claim)
- Create HPA.yml
- Create an env var inside the node-deploy to connect to mongodb on port 27017 


![Capture](https://user-images.githubusercontent.com/17476059/136104964-89a1b24a-d128-426f-ad32-70820067ec71.PNG)

____
### Adding the Database:
- Files:
-   Node-App:
        - node_deployment.yml
        - node_svc.yml
        - node_hap.yml
-   DB-app:
        - DB-Deploy.yml
        - DB-services.yml
____
### Adding the Node app
This creates the node app using the docker images previouly created:

```
# Create a diagram for node deployment and services 
apiVersion: apps/v1
kind: Deployment
metadata:
  name: node
spec:
  selector:
    matchLabels:
      app: node
  replicas: 3
  template:
    metadata:
      labels:
        app: node
    spec:
      containers:
      - name: node
        image: sremichael/sre_node_app:v3
        ports:
          - containerPort: 3000
        env:
          - name: DB_HOST
            value: mongodb://mongo:27017/posts

```

### Node Service

```
---
apiVersion: v1
kind: Service
metadata:
  name: node
spec:
  selector:
    app: node
  ports:
   - port: 3000
     targetPort: 3000
  type: LoadBalancer
```


### node_hap.yml
This is the nodes horazontal scaling add-on

```
apiVersion: autoscaling/v1
kind: HorizontalPodAutoscaler

metadata:
    name: sparta-node-app-deploy
    namespace: default

spec:
  maxReplicas: 9
  minReplicas: 3
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment 
    name: node 
  targetCPUUtilizationPercentage: 50
```


### DB-Deploy.yml
Creates the DB vm using the mongo image from dockerhub. This DB is configured on port 27017

```
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: mongo  

spec:
  selector:
    matchLabels:
      app: mongo
  replicas: 2
  template:
    metadata:
      labels:
        app: mongo

    spec:
      containers:
      - name: mongo
        image: mongo:latest
        ports: 
         - containerPort: 27017

```
### DB-service.yml
This allows the service to forward data to this container as well as maintain its ip if it does down.

```
apiVersion: v1
kind: Service
metadata:
  name: mongo
spec:
  selector:
    app: mongo
  ports:
  - port: 27017
    targetPort: 27017
  type: LoadBalancer 
 ```









