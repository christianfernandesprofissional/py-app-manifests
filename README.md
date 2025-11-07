# Automatização de Deploy com GitHub Actions, Docker Hub e ArgoCD


Este projeto demonstra como automatizar o processo de build, versionamento e deploy contínuo de uma aplicação utilizando GitHub Actions, Docker Hub e ArgoCD.

O objetivo é garantir que, sempre que uma nova alteração for enviada para o repositório da aplicação, uma nova imagem Docker seja criada e publicada automaticamente no Docker Hub.
Em seguida, um segundo repositório que contém os manifests Kubernetes usados pelo ArgoCD (rodando no Rancher Desktop) é atualizado com a nova versão da imagem, fazendo com que o ArgoCD sincronize e aplique automaticamente a nova versão no cluster.

Essa pipeline automatiza o ciclo completo de CI/CD (Continuous Integration / Continuous Deployment).

## ⚙️ Fluxo de funcionamento

O fluxo automatizado é composto por duas partes principais:

### Repositório da Aplicação localizado em: https://github.com/christianfernandesprofissional/py-app

- Contém o código-fonte e o arquivo Dockerfile.

- Possui um workflow GitHub Actions responsável por:

  - Construir a imagem Docker.

  - Enviar a imagem para o Docker Hub.

  - Atualizar automaticamente o repositório de manifests (utilizado pelo ArgoCD).

### Manifests que estão contidos neste repositório

- Os arquivos YAML do Kubernetes estão contidos no diretório py-app-manifests/manifests/

- Os manifests estão sendo monitorado pelo ArgoCD, que detecta alterações e atualiza o ambiente no Rancher Desktop.

## 🔁 Fluxo da Automação

- Um push é feito na branch main do repositório da aplicação.

- O GitHub Actions é acionado automaticamente.

- O workflow executa as seguintes etapas:

  - Faz o checkout do repositório da aplicação.

  - Constrói a imagem Docker da aplicação.

  - Envia a imagem para o Docker Hub com a nova tag (ex: v24).

  - Clona o repositório de manifests.

  - Atualiza o campo image: do arquivo deployment.yaml para apontar para a nova versão da imagem.

  - Cria um Pull Request no repositório de manifests com essa atualização.

- O ArgoCD (rodando no Rancher Desktop) detecta a mudança e atualiza automaticamente o deployment no cluster.

## Pré-requisitos

- Conta no GitHub (repositório público)
- Conta no Docker Hub com token de acesso
- Rancher Desktop com Kubernetes habilitado
- kubectl configurado corretamente (kubectl get nodes)
- ArgoCD instalado no cluster local (Rancher)
- Git instalado
- Python 3 e Docker instalados


