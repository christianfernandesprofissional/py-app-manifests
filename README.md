# 📦 Repositório de Manifests – py-app  

Este repositório contém os **manifests Kubernetes** utilizados para realizar o deploy da aplicação **py-app**, cujo processo de automação foi descrito na documentação principal.  

Os arquivos YAML contidos neste repositório são monitorados pelo **ArgoCD**, que realiza a sincronização automática com o cluster Kubernetes no **Rancher Desktop** sempre que há uma atualização.  

Os arquivos são:

- **deployment.yaml** – Define o *Deployment* da aplicação, especificando a imagem Docker e o número de réplicas.  
- **service.yaml** – Cria o *Service* responsável por expor a aplicação na porta configurada.  

## 🔄 Atualizações Automáticas  

As atualizações neste repositório são feitas automaticamente por um **workflow do GitHub Actions** localizado no repositório da aplicação principal:  
🔗 [py-app (repositório da aplicação)](https://github.com/christianfernandesprofissional/py-app)  

Sempre que um novo *push* é feito na branch `main` do repositório da aplicação:  
1. Uma nova imagem Docker é gerada e enviada para o Docker Hub.  
2. Este repositório tem seu arquivo `deployment.yaml` atualizado com a nova versão da imagem.  
3. O **ArgoCD** detecta a alteração e atualiza o ambiente automaticamente.  

## 🚀 Objetivo  

Garantir um fluxo **CI/CD completo com Github Actions**, onde cada mudança no código resulta em uma nova versão implantada automaticamente no cluster Kubernetes.

