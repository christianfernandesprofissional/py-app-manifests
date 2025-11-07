# Automatização de Deploy com GitHub Actions, Docker Hub e ArgoCD


Este projeto demonstra como automatizar o processo de build, versionamento e deploy contínuo de uma aplicação utilizando GitHub Actions, Docker Hub e ArgoCD.

O objetivo é garantir que, sempre que uma nova alteração for enviada para o repositório da aplicação, uma nova imagem Docker seja criada e publicada automaticamente no Docker Hub.
Em seguida, um segundo repositório que contém os manifests Kubernetes usados pelo ArgoCD (rodando no Rancher Desktop) é atualizado com a nova versão da imagem, fazendo com que o ArgoCD sincronize e aplique automaticamente a nova versão no cluster.

Essa pipeline automatiza o ciclo completo de CI/CD (Continuous Integration / Continuous Deployment).

## ⚙️ Fluxo de funcionamento

O fluxo automatizado é composto por duas partes principais:

### Repositório da Aplicação 

- **Localizado em:** https://github.com/christianfernandesprofissional/py-app

- Contém o código-fonte e o arquivo Dockerfile.

- Possui um workflow GitHub Actions responsável por:

  - Construir a imagem Docker.

  - Enviar a imagem para o Docker Hub.

  - Atualizar automaticamente o repositório de manifests (utilizado pelo ArgoCD).

### Manifests que estão contidos neste repositório

- Os arquivos YAML do Kubernetes estão contidos no diretório **py-app-manifests/manifests/**

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

# Passo a passo


### Aplicação

Para inicar o projeto é necessário primeiro ter a aplicação. Aqui estamos rodando uma aplicação simples utilizando Python 3 e uma biblioteca chamada FastAPI, o arquivo da aplicação contém o seguinte código:

    from fastapi import FastAPI

    app = FastAPI()

    @app.get("/")
    async def root():
      return {"message": "Hello World"}


Com a aplicação criada vamos utilizar um comando na raiz do projeto para que as dependências sejam salvas em um arquivo:

    pip freeze > requirements.txt

Agora para poder criar uma imagem Docker, precisamos criar um Dockerfile para nossa aplicação, o Dockerfile utilizado está assim:

    FROM python:3
    
    WORKDIR /app
    
    COPY requirements.txt .
    
    RUN pip install --no-cache-dir -r requirements.txt
    
    COPY main.py .
    
    EXPOSE 8000
    
    CMD ["uvicorn", "main:app", "--host", "0.0.0.0", "--port", "8000", "--reload"]


Agora podemos fazer o push da nossa aplicação no repositório do github responsável por conter a aplicação.

### DockerHub e imagem

Com o Dockerfile em mãos podemos buildar nossa imagem utilizando o comando:

    docker build  -t nome-usuario-dockerhub:nome-app:versão .

Após a criação da imagem é possível verica-la usando:

    docker images

Com a imagem pronta precisamos logar no DockerHub, para isso é necessário entrar no Dockerhub pelo navegador, após logado vá em Settings/Personal-access-token 



<img width="275" height="278" alt="1PAT-dockerhub" src="https://github.com/user-attachments/assets/36ccfae6-655f-45a1-b6fb-5cb903450b4b" /> <br>


E crie um token para seu computador, com permissão de leitura e escrita:


<img width="197" height="98" alt="2create-token-btn-dockerhub" src="https://github.com/user-attachments/assets/9395d569-6454-4732-908f-bce4c93bee8a" /> <br>



<img width="646" height="453" alt="3create-token-dockerhub" src="https://github.com/user-attachments/assets/93a13724-2fda-42fc-adb3-3c56a0158595" />


Após a criação, basta seguir os passos mostrados para efetura login no Docker hub pela sua máquina, mas antes salve o login, e a senha gerados em um arquivo txt para podermos utiliza-los posteriormente.
Depois de logado, vamos fazer um push da nossa imagem no Dockerhub, para isso utilize o comando abaixo:

    docker push nome-usuario-dockerhub:nome-app:versão 

O nome utilizado é o mesmo usado na criação da imagem.
Agora temos nossa imagem no Dockerhub



<img width="1023" height="523" alt="10imagens-dockerhub" src="https://github.com/user-attachments/assets/9ba6198f-5397-41c0-9370-33d466447776" />




### Manifests

Agora vamos criar nosso manifest para que depois ele seja utilizado pelo ArgoCD.
O Manifest utilizado está desta maneira:

- Deployment:


      apiVersion: apps/v1
      kind: Deployment
      metadata: 
        name: py-app-deployment
        labels:
          app: py-app
      spec:
        replicas: 1
        selector:
          matchLabels:
            app: py-app
        template:
          metadata:
            name: py-app
            labels:
              app: py-app
          spec:
            containers:
              - name: py-app
                image: nome-usuario-dockerhub:nome-app:versão
                ports: 
                  - containerPort: 8000
                    name: py-app 
      
- Service:

      apiVersion: v1
      kind: Service
      metadata:
        name: py-app-service
        labels:
          app: py-app
      spec:
        type: NodePort
        selector:
          app: py-app
        ports:
          - port:  5000
            targetPort: 8000
            nodePort: 30500
          
Após a criação dos dois arquivos yaml podemos fazer o push para o repositório que conterá os manifests.


### Configurações Github

Antes de criar nosso workflow precisamos que algumas configurações sejam feitas.
Primeiro, no repositório dos manifests vá em **Settings/Actions**

<img width="332" height="599" alt="8repository-config" src="https://github.com/user-attachments/assets/59dc6d97-db87-46b2-b952-4eefd6e95ddf" />

Desça até o fim da página e altere as seguintes configurações:

<img width="853" height="372" alt="9repository-config" src="https://github.com/user-attachments/assets/ef8338f8-f59e-4d4a-99ba-a07e9129422a" />

Essa configuração é para que o repositório permita que nosso Workflow consiga fazer alterações nos manifests, e depois consiga fazer um Pull Request. <br>

Além disso nas configurações da nossa conta, em **Settings / Developer Settings** crie um Fine grained token para que nosso workflow tenha acesso completo no nosso repositório:

<img width="548" height="270" alt="6fine-grained-tokens" src="https://github.com/user-attachments/assets/354488da-7959-4a62-917d-ca9c39e5ed9a" />

Selecione o repositório que token garantirá acesso e adicione as seguintes permissões:

<img width="926" height="831" alt="7fine-grained-tokens-config" src="https://github.com/user-attachments/assets/78557b07-e35f-45e1-99ff-92342b88c48d" />

Copie o Token gerado e salve.

Agora no repositório onde está a aplicação, vá em **Settings / Secrets and Variables / Actions**:

<img width="339" height="239" alt="4create-secrets" src="https://github.com/user-attachments/assets/be8db78f-7de5-4767-927b-f0f30ed9c4d6" />

E adicione 3 secrets, um contendo o login do Dockerhub, outro contendo a senha gerada pelo Dockehub, e outro contendo o token que geramos na nossa conta do Github:

<img width="976" height="299" alt="5-1secrets" src="https://github.com/user-attachments/assets/374a684d-ccf8-4bb7-8677-89e2f146b5cc" />

Com todas estas configurações feitas podemos agora iniciar a criação do nosso workflow.


### Workflow

Vá até o repositório onde está contida a aplicação, e entre no diretório **/.github/** crie uma pasta chamada **workflows** e crie um arquivo **update-workflow.yaml**.
O workflow da aplicação ficará assim:

    name: Build, Push, and Update Deployment
    
    on: 
      push:
        branches:
         - main
    
    jobs:
      build:
        runs-on: ubuntu-latest
    
        steps:
          # Clona o repositório atual 
          - name: Checkout app repository
            uses: actions/checkout@v4
    
          # Configura Docker Buildx
          - name: Set up Docker Buildx
            uses: docker/setup-buildx-action@v3
    
          # Login no Docker Hub
          - name: Log in to Docker Hub
            uses: docker/login-action@v3
            with:
              username: ${{ secrets.DOCKER_USERNAME }}
              password: ${{ secrets.DOCKER_PASSWORD }}
    
          # Cria variáveis de versão
          - name: Set image version variables
            id: vars
            run: |
              echo "RUN_NUMBER=${GITHUB_RUN_NUMBER}" >> $GITHUB_ENV
              echo "IMAGE_VERSION=v${GITHUB_RUN_NUMBER}" >> $GITHUB_ENV
          # Build e push da imagem
          - name: Build and Push Docker Image
            uses: docker/build-push-action@v6
            with:
              context: .
              file: ./Dockerfile
              push: true
              tags: seu-usuario-github-aqui/nome-do-seu-repositorio-da-aplicação:${{ env.IMAGE_VERSION }}
    
          # Mostra imagens locais (debug opcional)
          - name: Show docker images
            run: docker images
    
          # Checkout do repositório de manifests
          - name: Checkout manifests repository
            uses: actions/checkout@v4
            with:
              repository: seu-usuario-github-aqui/nome-do-seu-repositorio-dos-manifests
              token: ${{ secrets.TOKEN_MANIFESTS }}
              path: nome-do-seu-repositorio-dos-manifests
    
          # Atualiza o arquivo deployment.yaml
          - name: Update deployment image version
            run: |
              cd py-app-manifests/manifests
              sed -i "s|image: seu-usuario-github-aqui/nome-do-seu-repositorio-da-aplicação:.*|image: seu-usuario-github-aqui/nome-do-seu-repositorio-da-aplicação:${IMAGE_VERSION}|" deployment.yaml

          # Cria o Pull Request no repositório dos manifests
          - name: Create Pull Request
            id: create-pull-request 
            uses: peter-evans/create-pull-request@v6
            with:
              token: ${{ secrets.TOKEN_MANIFESTS }}
              base: main
              branch: auto/update-image-${{ env.IMAGE_VERSION }}
              title: "Update image to version ${{ env.IMAGE_VERSION }}"
              body: |
                This PR updates the image tag to `${{ env.IMAGE_VERSION }}`.
              delete-branch: true
              path: ./nome-do-seu-repositorio-dos-manifests

          # Aceita automaticamente o Pull Request feito anteriormente
          - name: Enable Pull Request Automerge
            uses: peter-evans/enable-pull-request-automerge@v3
            with:
              token: ${{ secrets.TOKEN_MANIFESTS }}
              repository: seu-usuario-github-aqui/nome-do-seu-repositorio-dos-manifests
              pull-request-number: ${{ steps.create-pull-request.outputs.pull-request-number }}
              merge-method: squash
    
Este workflow irá gerar a imagem da nossa aplicação já com a versão atualizada de acordo com o número de execução do workflow. Depois irá fazer push da imagem no Docker Hub, em seguida entrará no repositório do manifests, criará uma branch para atualizar a versão da imagem. E por ultimo após as alterações irá fazer um Pull Request para o repositório dos manifests e aceitar automaticamente.

Desta maneira qualquer alteração feita na nossa aplicação será replicada até o nosso ArgoCD.
