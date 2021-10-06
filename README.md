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
Files: 
- Node-App:
```
        - node_deployment.yml
        - node_svc.yml
        - node_hap.yml
```
-   DB-app:
```
        - DB-Deploy.yml
        - DB-services.yml
```
- persistent volume

```
        - pv.yml
        - pvc.yml
```

____
### Adding the Node app :
- This creates the node app using the docker images previouly created:

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
`kubectl create -f node_deployment.yml`\
It will create 3 pods with the Node app instances: To show the running pods: \
`kubectl get  pods`
```
NAME                     READY   STATUS    RESTARTS   AGE
node-869c6747bb-52s8q    1/1     Running   1          23m
node-869c6747bb-g9lcf    1/1     Running   1          23m
node-869c6747bb-mcs9z    1/1     Running   1          23m
```

### Node Service:

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
`kubectl create -f nginx_svc.yml`

### node_hpa.yml
- This is the nodes horazontal scaling add-on

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
`kubectl create -f node_hpa.yml`


### DB-Deploy.yml
- Creates the DB vm using the mongo image from dockerhub. This DB is configured on port 27017

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

        volumeMounts:
          - mountPath: /data/db
            name: storage
      
      volumes:
        - name: storage
          persistentVolumeClaim:
            claimName: mypvc

```
`kubectl create -f DB-Deploy.yml`

### DB-service.yml
- This allows the service to forward data to this container as well as maintain its ip if it does down.

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
`kubectl create -f  DB-service.yml` \
This will seed the databse allowing the data to be accessed \
` kubectl exec [pod-name]  env node seeds/seed.js`
___
## persistent volume kubernetes 
file:
  - pv.yml 
  - pvc.yml

![images](https://www.infinidat.com/sites/default/files/uploads/images/Matching_PV.png)

- pv.yml
This is the presistent volume that get created with a storage volume. When the PVC (claim) is created, it links to the pod that has the correct ammount of memory. 
```
apiVersion: v1
kind: PersistentVolume
metadata:
  name: mypv
spec:
  capacity:
    storage: 256Mi
  volumeMode: Filesystem
  accessModes:
    - ReadWriteOnce
  persistentVolumeReclaimPolicy: Recycle
  storageClassName: slow
  mountOptions:
    - hard
    - nfsvers=4.1
  hostPath:
    path: /data/db
    type: Directory
```


- pvc.yml 

```
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: mypvc
spec:
  resources:
    requests:
      storage: 256Mi
  accessModes:
    - ReadWriteOnce
```
To see if the results: \
`kubectl get  pv`

```
NAME                                       CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS      CLAIM           STORAGECLASS   REASON   AGE
mypv                                       256Mi      RWO            Recycle          Available                   slow                    56m
pvc-498faca5-965e-4886-bc6c-55066eb6cf2f   256Mi      RWO            Delete           Bound       default/mypvc   hostpath                55m

```
To see the PVC for the presistent volume claim \
`kubectl get  pvc`

```
NAME    STATUS   VOLUME                                     CAPACITY   ACCESS MODES   STORAGECLASS   AGE
mypvc   Bound    pvc-498faca5-965e-4886-bc6c-55066eb6cf2f   256Mi      RWO            hostpath       55m
```

___

To see all of the configurations use this. \
` kubectl get all `

At the end it should look like this:
- mongo 
```
NAME                         READY   STATUS    RESTARTS   AGE
pod/mongo-54d9fb44b7-8sdwr   1/1     Running   0          49m
pod/mongo-54d9fb44b7-zskvv   1/1     Running   0          49m
```
- node
```
NAME                         READY   STATUS    RESTARTS   AGE
pod/node-869c6747bb-52s8q    1/1     Running   2          49m
pod/node-869c6747bb-g9lcf    1/1     Running   2          49m
pod/node-869c6747bb-mcs9z    1/1     Running   1          49m
```
- services 
```
NAME                 TYPE           CLUSTER-IP       EXTERNAL-IP   PORT(S)           AGE
service/kubernetes   ClusterIP      10.96.0.1        <none>        443/TCP           44h
service/mongo        LoadBalancer   10.104.231.183   localhost     27017:30770/TCP   18h
service/node         LoadBalancer   10.110.195.225   localhost     3000:30260/TCP    19h
```
- Deployments
```
NAME                    READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/mongo   2/2     2            2           18h
deployment.apps/node    3/3     3            3           18h

NAME                               DESIRED   CURRENT   READY   AGE
replicaset.apps/mongo-54d9fb44b7   2         2         2       49m
replicaset.apps/mongo-697477455c   0         0         0       18h
replicaset.apps/node-869c6747bb    3         3         3       49m
replicaset.apps/node-d5c8478b4     0         0         0       18h

NAME                                                         REFERENCE         TARGETS         MINPODS   MAXPODS   REPLICAS   AGE
horizontalpodautoscaler.autoscaling/sparta-node-app-deploy   Deployment/node   <unknown>/50%   3         9         3          20h
```









