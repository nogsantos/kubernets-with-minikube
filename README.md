# Kubernetes with Minikube

O Minikube inicia um cluster de kubernetes de nó único localmente para fins de desenvolvimento e testes. O Minikube empacota e configura uma VM Linux, Docker e todos os componentes do Kubernetes, otimizados para desenvolvimento local. O Minikube suporta recursos do Kubernetes como:

- DNS
- NodePorts
- ConfigMaps and Secrets
- Dashboards

### Terminologia e objetos Kubernetes

- Objetos:
  - Pod
    - Os pods são os menores objetos que podem ser criados e gerenciados pelo Kubernetes, tais objetos são utilizados para abstrair os containers que formam uma aplicação.
  - Deployment
    - Os pods são os objetos mais básicos existentes no Kubernetes e dessa forma, eles não oferecem alguns recursos, entre eles o de informar o estado desejado da nossa aplicação para o Kubernetes gerenciar. Dessa forma, quando trabalhamos com o Kubernetes, realizamos a abstração desses objetos Pods em objetos que oferecem mais recursos, como por exemplo o objeto Deployment que é capaz de adicionar o estado desejado da aplicação para o Kubernetes gerenciar
  - Service
    - Coleção de pods que funcionam juntos e expostos como um ponto final. Um serviço (coleção de pods) é definido por um label selector. O Kubernetes fornece descoberta de serviço e roteamento de solicitação, atribuindo um endereço IP estável e um nome DNS ao serviço, além de balancear o tráfego de maneira circular para as conexões de rede desse endereço IP entre os conjuntos correspondentes ao seletor.
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

[Docs Minikube](https://kubernetes.io/docs/tasks/tools/install-minikube/)

[Install Minikube](https://kubernetes.io/docs/setup/minikube/)

[Minikube Drivers](https://github.com/kubernetes/minikube/blob/master/docs/drivers.md)

[Kubernetes tutoriais](https://kubernetes.io/docs/tutorials/)

### Linux

- Disable security boot on bios
- Install Virtual Box
- `curl -Lo minikube https://storage.googleapis.com/minikube/releases/latest/minikube-linux-amd64 && sudo chmod +x minikube && sudo mv minikube /usr/local/bin/`

### Kubectl

- `curl -LO https://storage.googleapis.com/kubernetes-release/release/$(curl -s https://storage.googleapis.com/kubernetes-release/release/stable.txt)/bin/linux/amd64/kubectl`
- `chmod +x ./kubectl`
- `sudo mv ./kubectl /usr/local/bin/kubectl`

## Commands

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

#### Verificar pods

```shell
$ kubectl get pods
NAME                          READY     STATUS    RESTARTS   AGE
hello-node-1644695913-l4gkl   1/1       Running   0          49m
```

#### Verificar all pods

```shell
$ kubectl get pods --all-namespaces
NAMESPACE     NAME                          READY     STATUS    RESTARTS   AGE
kube-system   kube-addon-manager-minikube   1/1       Running   0          9h
kube-system   kube-dns-v20-kqlbc            3/3       Running   0          9h
kube-system   kubernetes-dashboard-52wwl    1/1       Running   0          9h
```

#### Deletar um pod

```shell
$ kubectl delete pods [pod-name]
```

#### Informações sobre os pods

```shell
$ kubectl describe pods
```

#### Filtrando apenas o Ip dos pods

```shell
$ kubectl describe pods | grep IP
```

#### Consultar todos os nós

```shell
$ kubectl get nodes
NAME       STATUS    AGE       VERSION
minikube   Ready     24m       v1.6.0
```

#### Consultar o log de um pod

```shell
$ kubectl logs <POD-NAME>
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
