# Kubernetes with Minikube

O Minikube inicia um cluster de kubernetes de nó único localmente para fins de desenvolvimento e testes. O Minikube empacota e configura uma VM Linux, Docker e todos os componentes do Kubernetes, otimizados para desenvolvimento local. O Minikube suporta recursos do Kubernetes como:

- DNS
- NodePorts
- ConfigMaps and Secrets
- Dashboards

### Terminologia e objetos Kubernetes

- Objetos:
  - [](#pods)Pod
    - Os pods são os menores objetos que podem ser criados e gerenciados pelo Kubernetes, tais objetos são utilizados para abstrair os containers que formam uma aplicação.
  - [](#services)Service
    - Coleção de pods que funcionam juntos e expostos como um ponto final. Um serviço (coleção de pods) é definido por um label selector. O Kubernetes fornece descoberta de serviço e roteamento de solicitação, atribuindo um endereço IP estável e um nome DNS ao serviço, além de balancear o tráfego de maneira circular para as conexões de rede desse endereço IP entre os conjuntos correspondentes ao seletor.
  - [](#deployment)Deployment
    - Um dos objetos mais comuns do Kubernetes é o Deployment. Define a especificação do contêiner necessária, juntamente com o nome e os labels usados ​​por outras partes do Kubernetes para descobrir e se conectar ao aplicativo. Os pods são os objetos mais básicos existentes no Kubernetes e dessa forma, eles não oferecem alguns recursos, entre eles o de informar o estado desejado da nossa aplicação para o Kubernetes gerenciar. Dessa forma, quando trabalhamos com o Kubernetes, realizamos a abstração desses objetos Pods em objetos que oferecem mais recursos, como por exemplo o objeto Deployment que é capaz de adicionar o estado desejado da aplicação para o Kubernetes gerenciar
- Teminologia:
  - Nodes
    - Hosts que executam aplicativos do Kubernetes
  - Containers
    - Unidades de empacotamento
  - Replication Controller
    - Garante disponibilidade e escalabilidade. Ele manipula a replicação e o dimensionamento executando um número especificado de cópias de um pod no cluster. Ele também lida com a criação de pods de substituição se o nó subjacente falhar.
  - Labels
    - Pares de valor-chave de identificação para objeto da API, como pods e nós.
  - Selectors
    - Como os labels, os seletores são o principal mecanismo de agrupamento no Kubernetes e são usados ​​para determinar os componentes aos quais uma operação se aplica. Portanto, "label selectors" são consultas em labels que resolvem os objetos correspondentes.

## Setup

### Linux

- Disable security boot on bios
- Install Virtual Box
- `curl -Lo minikube https://storage.googleapis.com/minikube/releases/latest/minikube-linux-amd64 && sudo chmod +x minikube && sudo mv minikube /usr/local/bin/`

### Kubectl

- `curl -LO https://storage.googleapis.com/kubernetes-release/release/$(curl -s https://storage.googleapis.com/kubernetes-release/release/stable.txt)/bin/linux/amd64/kubectl`
- `chmod +x ./kubectl`
- `sudo mv ./kubectl /usr/local/bin/kubectl`

## Comandos

### Minikube

#### Inicializar minikube cluster

Default Virtual Box

```shell
$ minikube start
Starting local Kubernetes cluster...
Starting VM...
SSH-ing files into VM...
Setting up certs...
Starting cluster components...
Connecting to cluster...
Setting up kubeconfig...
Kubectl is now configured to use the cluster.
```

Linux [Kvm](https://www.linux-kvm.org/page/Main_Page) driver

```shell
minikube start --vm-driver kvm2
Starting local Kubernetes cluster...
Starting VM...
SSH-ing files into VM...
Setting up certs...
Starting cluster components...
Connecting to cluster...
Setting up kubeconfig...
Kubectl is now configured to use the cluster.
```

#### Para os serviços

```shell
$ minikube stop
Stopping local Kubernetes cluster...
Machine stopped.
```

#### Get service URL

```shell
$ minikube service [service-name] --url
```

#### Acessar o painel administrativo visual

```shell
$ minikube dashboard
Opening kubernetes dashboard in default browser...
```

### Kubectl

#### Criação

```shell
$ kubectl create -f [nome-arquivo.yaml]
```

#### Run - Criar um [pod](#pods) baseado em uma imagem do docker

O comando run cria um deployment com base nos parâmetros especificados, como a imagem ou as réplicas. Esse deployment é emitida para o Kubernetes master, que inicia os pods e contêineres necessários. O Kubectl run é semelhante ao docker run, mas em um nível de cluster.

```shell
$ kubectl run [pod name] --image=[docker image] --replicas=[number]

Ex.:

$ kubectl run redis --image=nogsantos/redis:latest --replicas=1
deployment.apps/redis created
```

##### Run e expor a porta do container

```shell
$ kubectl run [pod name] --image=[docker image] --replicas=[number] --port=[number pod port] --hostport=[number exposed port]
```

#### Describe

Para visualizar o que foi criado pelo kubernetes ao criar um [pod](#pods) com `run`, o comando `describe` apresenta maiores detalhes do que foi realizado

```shell
$ kubectl describe deployments [pod name]

Ex.:

$ kubectl describe deployments katacoda
Name:                   katacoda
Namespace:              default
CreationTimestamp:      Fri, 28 Sep 2018 09:51:15 -0300
Labels:                 run=katacoda
Annotations:            deployment.kubernetes.io/revision=1
Selector:               run=katacoda
Replicas:               1 desired | 1 updated | 1 total | 1 available | 0 unavailable
StrategyType:           RollingUpdate
MinReadySeconds:        0
RollingUpdateStrategy:  25% max unavailable, 25% max surge
Pod Template:
  Labels:  run=katacoda
  Containers:
   katacoda:
    Image:        katacoda/docker-http-server:latest
    Port:         <none>
    Host Port:    <none>
    Environment:  <none>
    Mounts:       <none>
  Volumes:        <none>
Conditions:
  Type           Status  Reason
  ----           ------  ------
  Available      True    MinimumReplicasAvailable
  Progressing    True    NewReplicaSetAvailable
OldReplicaSets:  <none>
NewReplicaSet:   katacoda-6dc99c7f7d (1/1 replicas created)
Events:
  Type    Reason             Age   From                   Message
  ----    ------             ----  ----                   -------
  Normal  ScalingReplicaSet  23m   deployment-controller  Scaled up replica set katacoda-6dc99c7f7d to 1
```

#### Verificar [pods](#pods)

```shell
$ kubectl get pods
NAME                          READY     STATUS    RESTARTS   AGE
hello-node-1644695913-l4gkl   1/1       Running   0          49m
```

#### Verificar [serviços](#services)

```shell
$ kubectl get svc
NAME          TYPE           CLUSTER-IP       EXTERNAL-IP   PORT(S)        AGE
hello         ClusterIP      10.97.164.35     <none>        8080/TCP       2d
katacoda      ClusterIP      10.96.72.106     172.17.0.30   8001/TCP       11m
kubernetes    ClusterIP      10.96.0.1        <none>        443/TCP        8d
nogsantos     ClusterIP      10.98.156.146    <none>        5432/TCP       2d
service-app   LoadBalancer   10.106.110.134   <pending>     80:31244/TCP   8d
```

#### Expose [serviços](#services)

Expor a porta do container

```shell
$ kubectl expose deployment [pod name] --external-ip="[endereco externo]" --port=[porta externa] --target-port=[porta no pod]

Ex.:
$ kubectl expose deployment katacoda --external-ip="172.17.0.30" --port=8001 --target-port=80
service/katacoda exposed
```

#### Verificar todos [pods](#pods) ativos

```shell
$ kubectl get pods --all-namespaces
NAMESPACE     NAME                          READY     STATUS    RESTARTS   AGE
kube-system   kube-addon-manager-minikube   1/1       Running   0          9h
kube-system   kube-dns-v20-kqlbc            3/3       Running   0          9h
kube-system   kubernetes-dashboard-52wwl    1/1       Running   0          9h
```

#### Deletar um [pod](#pods)

```shell
$ kubectl delete pods [pod-name]
```

#### Informações sobre os [pods](#pods)

```shell
$ kubectl describe pods
```

#### Filtrando apenas o Ip dos [pods](#pods)

```shell
$ kubectl describe pods | grep IP
```

#### Consultar todos os nós

```shell
$ kubectl get nodes
NAME       STATUS    AGE       VERSION
minikube   Ready     24m       v1.6.0
```

#### Consultar o log de um [pod](#pods)

```shell
$ kubectl logs <POD-NAME>
```

#### Detalhes do cluster

```shell
$ kubectl cluster-info
Kubernetes master is running at https://192.168.39.83:8443
KubeDNS is running at https://192.168.39.83:8443/api/v1/namespaces/kube-system/services/kube-dns:dns/proxy
```

#### Get

Visualizar o status dos [deployments](#deployment) e dos [serviços](#services)

```shell
$ kubectl get deployments,services
```

## Fixes

Ao tentar inicializar o `minikube` pode ocorrer o seguinte erro

```shell
$ minikube start --vm-driver=kvm
Starting local Kubernetes cluster...
Starting VM...
E0409 09:36:09.737093   19497 start.go:116] Error starting host: Temporary Error: Error configuring auth on host: Too many retries waiting for SSH to be available.  Last error: Maximum number of retries (60) exceeded.

 Retrying.
```

Recomenda-se remover o cluster

```shell
$ minikube delete
Deleting local Kubernetes cluster...
Machine deleted.
```

Inicializar novamente

```shell
$ minikube start --vm-driver=kvm
Starting local Kubernetes cluster...
Starting VM...
SSH-ing files into VM...
Setting up certs...
Starting cluster components...
Connecting to cluster...
Setting up kubeconfig...
Kubectl is now configured to use the cluster.
```

### Requested operation is not valid: network 'minikube-net' is not active

Caso a inicialização automática das redes virtuais não esteja definida por padrão, no fedora, quando ocorrer o problema, para inicializar a rede:

[See Virsh](https://libvirt.org/virshcmdref.html)

#### Acessar o `virsh` como root

```shell
$ sudo virsh
Welcome to virsh, the virtualization interactive terminal.

Type:  'help' for help with commands
       'quit' to quit
```

#### Opcional, visualizar as redes disponíveis

```shell
# net-list --all
 Name                 State      Autostart     Persistent
----------------------------------------------------------

```

#### Iniciar a rede

```shell
# net-start minikube-net
Network minikube-net started
```

#### Opcional, confirmar a inicialização

```shell
# net-list --all
 Name                 State      Autostart     Persistent
----------------------------------------------------------
 default              active     yes           yes
 minikube-net         active     no            yes
```

#### Opcional, sair do console do `virsh`

```shell
# exit
```

## Default tasks

### Updating the application

Let's update our app by editing server.js file to return a new message:

Build a new version of our image:

```shell
$ docker build -t hello-node:v2 .
Sending build context to Docker daemon 3.072 kB
Step 1/4 : FROM node:6.9.2
6.9.2: Pulling from library/node
...
Successfully built 70c47e205ce1
```

Then, update the image of our Deployment:

```shell
$ kubectl set image deployment/hello-node hello-node=hello-node:v2
deployment "hello-node" image updated
```

Run the app again to view the new message:

```shell
$ minikube service hello-node
```

### Cleaning up

Now we can clean up the resources we created in our cluster:

```shell
$ kubectl delete service hello-node
service "hello-node" deleted
```

```shell
$ kubectl delete deployment hello-node
deployment "hello-node" deleted
```

Optionally, stop Minikube:

```shell
$ minikube stop
```

## Links e material de apoio

- [Kubernete github](https://github.com/kubernetes)
- [Exemplos kubernete](https://github.com/kubernetes/examples)
- [Docs Minikube](https://kubernetes.io/docs/tasks/tools/install-minikube/)
- [Install Minikube](https://kubernetes.io/docs/setup/minikube/)
- [Minikube Drivers](https://github.com/kubernetes/minikube/blob/master/docs/drivers.md)
- [Kubernetes tutoriais](https://kubernetes.io/docs/tutorials/)
- [kubectl CheatSheet](https://kubernetes.io/docs/reference/kubectl/cheatsheet/)