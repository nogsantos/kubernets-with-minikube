# Kubernets with Minikube

### Objetos

- Pod
  - Os pods são os menores objetos que podem ser criados e gerenciados pelo Kubernetes, tais objetos são utilizados para abstrair os containers que formam uma aplicação.
- Deployment
  - Os pods são os objetos mais básicos existentes no Kubernetes e dessa forma, eles não oferecem alguns recursos, entre eles o de informar o estado desejado da nossa aplicação para o Kubernetes gerenciar. Dessa forma, quando trabalhamos com o Kubernetes, realizamos a abstração desses objetos Pods em objetos que oferecem mais recursos, como por exemplo o objeto Deployment que é capaz de adicionar o estado desejado da aplicação para o Kubernetes gerenciar
- Service
  - Abstrair o acesso em um cluster aos pods. Se trata de um objeto mais estável

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
minikube start
```

Linux [Kvm](https://www.linux-kvm.org/page/Main_Page) driver

```shell
minikube start --vm-driver kvm2
```

#### Criação

```shell
kubectl create -f [nome-arquivo.yml]
```

#### Get service URL

```shell
minikube service [service-name] --url
```

#### Acessar o painel administrativo visual

```shell
minikube dashboard
```

### Kubectl

#### Verificar pods

```shell
kubectl get pods
```

#### Deletar um pod

```shell
kubectl delete pods [pod-name]
```

#### Informações sobre os pods

```shell
kubectl describe pods
```

#### Filtrando apenas o Ip dos pods

```shell
kubectl describe pods | grep IP
```

#### Create algo

```shell
kubectl create -f [file.yml]
```
