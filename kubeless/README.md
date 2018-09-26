# Kubeless

[The Kubernetes Native Serverless Framework](https://kubeless.io/)

##### Criando um namespace

```shell
$ kubectl create ns [name fo namespace]
```

##### Instalando a última versão estável

```shell
$ kubectl create -f https://github.com/kubeless/kubeless/releases/download/v0.6.0/kubeless-v0.6.0.yaml
```

##### Verificando se o pod foi criado

```shell
$ kubectl get pods -n kubeless
```

##### Function deploy

Exemplo deploy function em python

```shell
$ kubeless function deploy toy --runtime python2.7 \
                               --handler toy.handler \
                               --from-file toy.py
INFO[0000] Deploying function...
INFO[0000] Function toy submitted for deployment
INFO[0000] Check the deployment status executing 'kubeless function ls toy'
```

##### Listando as funções

```shell
$ kubeless function ls
```

##### Chamada a uma função

```shell
$ kubeless function call [nome funcao]

Ex.:

$ kubeless function call toy --data '{"hello":"world"}'
```

##### Log de uma funções

```shell
$ kubeless function logs [nome funcao]

Ex.:

$ kubeless function logs toy
```

##### Descrição de uma funções

```shell
$ kubeless function describe [nome funcao]

Ex.:

$ kubeless function describe toy
```

##### Atualizando uma funções

```shell
$ kubeless function update [nome funcao] --from-file [arquivo atualizacao]

Ex.:

$ kubeless function update toy --from-file toy-update.py
```

##### Deletando uma funções

```shell
$ kubeless function delete [nome funcao]
```

## Exemplo deploy de uma função javascript

### Deploy

Exemplo arquivo `hello.js`

```shell
$ kubeless function deploy hello --runtime nodejs6 \
                                 --handler hello.handler \
                                 --from-file hello.js
```

### Listando

```shell
$ kubeless function ls

Retorno após inicialização:

NAME 	NAMESPACE	HANDLER      	RUNTIME  	DEPENDENCIES	STATUS
hello	default  	hello.handler	nodejs6  	            	1/1 READY

```

### Chamada à função

```shell
$ kubeless function call hello --data '{"kubeless":"rocks"}'
{"kubeless":"rocks"}
```
