---
title: Separating Unoconv as a service
---

At CloudInn we had unoconv (depends on openoffice) in our backend so we coult convert our reports from ods to PDF or xlsx. That meant that we had to add other 400mb to our base image. Also, It meant longer time for deployments to finish. We were annoyed, we must do something about that.

I took the initiave to rip out unoconv from the base image, Picked one of the unoconv images on docker hub, then created a new namespace, deployment and service on Kubernetes for unocov with 3 replicas. Now our base image decreased from 800mb to 400mb good job (Y).

No the only thing left is to make our backend from test, staging, live to consume the new unoconv service from the new namespace. At first, I didn’t know how to make services from differnet namespaces to talk to each others, After some googling, the solution was easy.

for example let’s take the test namespace, and let’s assume that the new namespace we created for unoconv is dcs document-converter-service.

 - Create a new service inside test namespace. this service won’t have a selector, it will have ExternalName. Think of this service as an alias of the unoconv kubernetes service.

```yaml
kind: Service   
apiVersion: v1   
metadata:     
    name: unoconv    
    namespace: test  
spec:  
    type: ExternalName
    externalName: unoconv.unoconv.svc.cluster.local 
    ports:   
        - port: 80
```

As you can see, the externalName here corresponds to the-unoconv-service.in-this-namespace.svc.cluster.local let’s say you want to alias service called elephant in the namespace zoo the externalName would be:

```
elephant.zoo.svc.cluster.local
```

 - Now create the service using kubectl, assuming the file you created named unoconv-local-service.yml:

```
kubectl -n test create -f unoconv-local-service.yml
```
 - In the backend code you can make api calls to the service using unoconv.unoconv.cluster.local as the host, like this:

POST http://unoconv.unoconv.svc.cluster.local/unoconv/pdf

 - In order for the backend to be able to consume the new service, you have to re-deploy the backend, you can get the deployment name from kubectl -n test get deployments, assuming it’s called ‘backend’

```
kubectl -n test scale deployment backend --replicas=0
kubectl -n test scale deployment backend --replicas=1
```

That’s all, Have Fun !
