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


Após a criação, basta seguir os passos mostrados para efetura login no Docker hub pela sua máquina.
Depois de logado, vamos fazer um push da nossa imagem no Dockerhub, para isso utilize o comando abaixo:

    docker push nome-usuario-dockerhub:nome-app:versão 

O nome utilizado é o mesmo usado na criação da imagem.
Agora temos nossa imagem no Dockerhub









