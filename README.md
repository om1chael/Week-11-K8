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
