# Launch a simple, multi-tier web application using Kubernetes and Docker

> Replicated with `minikube` node local instalation.

The Guestbook example application stores notes from guests in Redis via JavaScript API calls. Redis contains a master (for storage), and a replicated set of redis 'slaves'.

## Core concepts

- Pods
- Replication Controllers
- Services
- NodePorts


### Redis Master Controller

The first stage of launching the application is to start the Redis Master. A Kubernetes service deployment has, at least, two parts. A replication controller and a service.

The replication controller defines how many instances should be running, the Docker Image to use, and a name to identify the service. Additional options can be utilized for configuration and discovery. Use the editor above to view the YAML definition.

If Redis were to go down, the replication controller would restart it on an active node.

```shell
$ kubectl create -f redis-master-controller.yaml
replicationcontroller/redis-master created
```

#### What's running?

The above command created a Replication Controller. The Replication

```shell
$ kubectl get rc
NAME           DESIRED   CURRENT   READY     AGE
redis-master   1         1         1         32s
```

All containers described as Pods. A pod is a collection of containers that makes up a particular application, for example Redis. You can view this using `kubectl`

```shell
$ kubectl get pods
NAME                     READY     STATUS    RESTARTS   AGE
redis-master-9s4xg       1/1       Running   0          1m
```

### Redis Master Service

The second part is a service. A Kubernetes service is a named load balancer that proxies traffic to one or more containers. The proxy works even if the containers are on different nodes.

Services proxy communicate within the cluster and rarely expose ports to an outside interface.

When you launch a service it looks like you cannot connect using curl or netcat unless you start it as part of Kubernetes. The recommended approach is to have a LoadBalancer service to handle external communications.

#### Create Service

The YAML defines the name of the replication controller, redis-master, and the ports which should be proxied.

```shell
$ kubectl create -f redis-master-service.yaml
service/redis-master created
```

#### List Services

```shell
$ kubectl get services
NAME           TYPE        CLUSTER-IP     EXTERNAL-IP   PORT(S)    AGE
kubernetes     ClusterIP   10.96.0.1      <none>        443/TCP    8d
redis-master   ClusterIP   10.106.16.23   <none>        6379/TCP   1m
```

#### Describe Services

```shell
$ kubectl describe services redis-master
Name:              redis-master
Namespace:         default
Labels:            name=redis-master
Annotations:       <none>
Selector:          name=redis-master
Type:              ClusterIP
IP:                10.106.16.23
Port:              <unset>  6379/TCP
TargetPort:        6379/TCP
Endpoints:         172.17.0.3:6379
Session Affinity:  None
Events:            <none>
```

### Replication Slave Pods

In this example we'll be running Redis Slaves which will have replicated data from the master. More details of Redis replication can be found at http://redis.io/topics/replication

As previously described, the controller defines how the service runs. In this example we need to determine how the service discovers the other pods. The YAML represents the GET_HOSTS_FROM property as DNS. You can change it to use Environment variables in the yaml but this introduces creation-order dependencies as the service needs to be running for the environment variable to be defined.

#### Start Redis Slave Controller

In this case, we'll be launching two instances of the pod using the image `kubernetes/redis-slave:v2`. It will link to redis-master via DNS.

```shell
$ kubectl create -f redis-slave-controller.yaml
replicationcontroller/redis-slave created
```

#### List Replication Controllers

```shell
$ kubectl get rc
NAME           DESIRED   CURRENT   READY     AGE
redis-master   1         1         1         9m
redis-slave    2         2         0         13s
```

### Redis Slave Service

As before we need to make our slaves accessible to incoming requests. This is done by starting a service which knows how to communicate with redis-slave.

Because we have two replicated pods the service will also provide load balancing between the two nodes.

#### Start Redis Slave Service

```shell
$ kubectl create -f redis-slave-service.yaml
service/redis-slave created
```

```shell
$ kubectl get services
NAME           TYPE        CLUSTER-IP     EXTERNAL-IP   PORT(S)    AGE
kubernetes     ClusterIP   10.96.0.1      <none>        443/TCP    8d
redis-master   ClusterIP   10.106.16.23   <none>        6379/TCP   6m
redis-slave    ClusterIP   10.98.0.26     <none>        6379/TCP   13s
```

### Frontend Replicated Pods

With the data services started we can now deploy the web application. The pattern of deploying a web application is the same as the pods we've deployed before.

#### Launch Frontend

The YAML defines a service called frontend that uses the image _gcr.io/googlesamples/gb-frontend:v3. The replication controller will ensure that three pods will always exist.

```shell
$ kubectl create -f frontend-controller.yaml
replicationcontroller/frontend created
```

#### List controllers and pods

```shell
$ kubectl get rc
NAME           DESIRED   CURRENT   READY     AGE
frontend       3         3         0         16s
redis-master   1         1         1         13m
redis-slave    2         2         2         4m
```

```shell
$ kubectl get pods
NAME                     READY     STATUS              RESTARTS   AGE
frontend-kqj22           0/1       ContainerCreating   0          51s
frontend-pgq85           0/1       ContainerCreating   0          51s
frontend-x47w2           0/1       ContainerCreating   0          51s
redis-master-9s4xg       1/1       Running             0          13m
redis-slave-2qp5r        1/1       Running             0          4m
redis-slave-bxw6q        1/1       Running             0          4m
```

##### PHP Code

The PHP code uses HTTP and JSON to communicate with Redis. When setting a value requests go to redis-master while read data comes from the redis-slave nodes.

### Guestbook Frontend Service

To make the frontend accessible we need to start a service to configure the proxy.

#### Start Proxy

The YAML defines the service as a NodePort. NodePort allows you to set well-known ports that are shared across your entire cluster. This is like `-p 80:80` in Docker.

In this case, we define our web app is running on port 80 but we'll expose the service on 30080.

```shell
$ kubectl create -f frontend-service.yaml
service/frontend created
```

```shell
$ kubectl get services
NAME           TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)        AGE
frontend       NodePort    10.103.232.162   <none>        80:30080/TCP   13s
kubernetes     ClusterIP   10.96.0.1        <none>        443/TCP        8d
redis-master   ClusterIP   10.106.16.23     <none>        6379/TCP       11m
redis-slave    ClusterIP   10.98.0.26       <none>        6379/TCP       5m
```

### Access Guestbook Frontend

With all controllers and services defined Kubernetes will start launching them as Pods. A pod can have different states depending on what's happening. For example, if the Docker Image is still being downloaded then the Pod will have a pending state as it's not able to launch. Once ready the status will change to running.

#### View Pods Status

You can view the status using the following command:

```shell
$ kubectl get pods
frontend-kqj22           1/1       Running   0          21m
frontend-pgq85           1/1       Running   0          21m
frontend-x47w2           1/1       Running   0          21m
redis-master-9s4xg       1/1       Running   0          33m
redis-slave-2qp5r        1/1       Running   0          25m
redis-slave-bxw6q        1/1       Running   0          25m
```

#### Find NodePort

If you didn't assign a well-known NodePort then Kubernetes will assign an available port randomly. You can find the assigned NodePort using kubectl.

```shell
$ kubectl describe service frontend | grep NodePort
Type:                     NodePort
NodePort:                 <unset>  30080/TCP
```

### View UI

Once the Pod is in running state you will be able to view the UI via port 30080. Use the URL to view the page

```shell
$ minikube service frontend --url
http://192.168.39.83:30080
```

Under the covers the PHP service is discovering the Redis instances via DNS. You now have a working multi-tier application deployed on Kubernetes.