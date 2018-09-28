# Learning Kubernetes with katacoda example

## Creating a deployment

One of the most common Kubernetes object is the deployment object. The deployment object defines the container spec required, along with the name and labels used by other parts of Kubernetes to discover and connect to the application.

This is deployed to the cluster with the command

```shell
$ kubectl create -f deployment.yaml
deployment.extensions/katacoda created
```
As it's a Deployment object, a list of all the deployed objects can be obtained via

```shell
$ kubectl get deployment
NAME                   DESIRED   CURRENT   UP-TO-DATE   AVAILABLE   AGE
katacoda               1         1         1            1           58s
```

Details of individual deployments can be outputted with

```shell
$ kubectl describe deployment katacoda
Name:                   katacoda
Namespace:              default
CreationTimestamp:      Fri, 28 Sep 2018 11:07:15 -0300
Labels:                 app=katacoda
Annotations:            deployment.kubernetes.io/revision=1
Selector:               app=katacoda
Replicas:               1 desired | 1 updated | 1 total | 1 available | 0 unavailable
StrategyType:           RollingUpdate
MinReadySeconds:        0
RollingUpdateStrategy:  1 max unavailable, 1 max surge
Pod Template:
  Labels:  app=katacoda
  Containers:
   katacoda:
    Image:        katacoda/docker-http-server:latest
    Port:         80/TCP
    Host Port:    0/TCP
    Environment:  <none>
    Mounts:       <none>
  Volumes:        <none>
Conditions:
  Type           Status  Reason
  ----           ------  ------
  Available      True    MinimumReplicasAvailable
  Progressing    True    NewReplicaSetAvailable
OldReplicaSets:  <none>
NewReplicaSet:   katacoda-67bd486477 (1/1 replicas created)
Events:
  Type    Reason             Age   From                   Message
  ----    ------             ----  ----                   -------
  Normal  ScalingReplicaSet  2m    deployment-controller  Scaled up replica set katacoda-67bd486477 to 1

```

## Creating a service

Kubernetes has powerful networking capabilities that control how applications communicate. These networking configurations can also be controlled via YAML.

Deploy the Service with

```shell
$ kubectl create -f service.yaml
service/katacoda-svc created
```
As before, details of all the Service objects deployed with

```shell
$ kubectl get svc
NAME           TYPE           CLUSTER-IP       EXTERNAL-IP   PORT(S)        AGE
kubernetes     ClusterIP      10.96.0.1        <none>        443/TCP        8d
katacoda-svc   NodePort       10.101.115.252   <none>        80:30080/TCP   2m
```

By describing the object it's possible to discover more details about the configuration

```shell
$ kubectl describe svc katacoda-svc
Name:                     katacoda-svc
Namespace:                default
Labels:                   app=katacoda
Annotations:              <none>
Selector:                 app=katacoda
Type:                     NodePort
IP:                       10.101.115.252
Port:                     <unset>  80/TCP
TargetPort:               80/TCP
NodePort:                 <unset>  30080/TCP
Endpoints:                172.17.0.10:80
Session Affinity:         None
External Traffic Policy:  Cluster
Events:                   <none>
```

```shell
curl host01:30080
```

## Updating a deployment object

After creating the `deployment` object, to update use.

```shell
$ kubectl apply -f deployment.yaml
```

Ex.:

To increase the number of replicas, update the file set the number of replicas from 1 to 4.

```yaml
replicas: 4
```

Then run the command `apply`, like the example above.

Instantly, the desired state of our cluster has been updated, viewable with

```shell
$ kubectl get deployment
```

Additional Pods will be scheduled to match the request.

```shell
$ kubectl get pods
```

As all the Pods have the same label selector, they'll be load balanced behind the Service NodePort deployed.

Issuing requests to the port will result in different containers processing the request curl `host01:30080`

## Font

[Katacoda kubernetes course](https://www.katacoda.com)