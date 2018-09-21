# Instalando SDK

Para que nós possamos realizar a interação com o Google Cloud, nós precisamos instalar o SDK.

[Site](https://cloud.google.com/sdk/)

[Documentação](https://cloud.google.com/sdk/docs/)

## MAC

.1 Verifique se você possui o Python instalado, caso não tenha, instalar

```shell
$ python -V
Python 2.7.10
```

.2 SDK

```shell
$ curl https://sdk.cloud.google.com | bash
```

## Linux (base debian)

.1 Crie uma variável de ambiente para a distribuição:

```
$ export CLOUD_SDK_REPO="cloud-sdk-$(lsb_release -c -s)"
```

.2 Adicione a url de distribuição do Cloud SDK:

```
$ echo "deb https://packages.cloud.google.com/apt $CLOUD_SDK_REPO main" | sudo tee -a /etc/apt/sources.list.d/google-cloud-sdk.list
```

.3 Importe a chave pública do Google Cloud:

```
$ curl https://packages.cloud.google.com/apt/doc/apt-key.gpg | sudo apt-key add -
```

.4 Devemos agora atualizar e instalar o SDK:

```
$ sudo apt-get update && sudo apt-get install google-cloud-sdk
```
