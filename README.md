# Repositório de Manifestos (hello-manifests)

Este repositório armazena os manifestos Kubernetes para a aplicação `hello-app` e serve como a **única fonte da verdade** (Single Source of Truth) para o nosso fluxo de GitOps.

Ele não contém código-fonte da aplicação, apenas os arquivos de configuração `.yaml` que descrevem o estado desejado da nossa aplicação no cluster Kubernetes. Ele é monitorado continuamente pelo ArgoCD, e todo o processo de deploy é gerenciado através de Solicitações de Pull (Pull Requests).

## 🚀 Passo a Passo do que Fizemos (Configuração deste Repositório)

Seguimos os seguintes passos para configurar este repositório e integrá-lo ao nosso pipeline de CI/CD.

### Passo 1: Criação e Nomeação do Repositório

Primeiramente, criamos este repositório público no GitHub para armazenar nossos arquivos de manifesto `.yaml`.

* **Problema de Nomeação:** Inicialmente, o repositório foi nomeado `olá-manifestos`. Isso causou um erro de `is not a valid repository name` (nome de repositório inválido) no pipeline de CI/CD, pois a ferramenta `git` não lidou bem com o caractere especial "á".
* **Solução:** Para corrigir, renomeamos o repositório para `hello-manifests`, que usa apenas caracteres padrão.

### Passo 2: Configuração da Chave de Deploy (Permissão de Escrita)

Para permitir que o pipeline de CI (que roda no repositório `hello-app`) pudesse criar Pull Requests automaticamente aqui, configuramos uma "Deploy Key".

1.  Navegamos até **"Configurações" (Settings)** > **"Implantar chaves" (Deploy keys)** neste repositório.
2.  Clicamos em **"Adicionar chave de implantação"**.
3.  No campo **"Title"**, demos um nome (ex: `olá-app-ci`).
4.  No campo **"Key"**, colamos o conteúdo da nossa chave **pública** (o arquivo `.pub`) que geramos no terminal.
5.  Marcamos a caixa **"Allow write access" (Permitir acesso de escrita)**. Este passo foi crucial para permitir que a automação do `hello-app` pudesse fazer `push` de novos branches para este repositório.

### Passo 3: Adição dos Manifestos Iniciais

Com o repositório configurado, adicionamos os arquivos `.yaml` que descrevem o estado inicial da nossa aplicação.

1.  **Manifesto de Implantação (`deployment.yaml` ou `implantacao.yaml`)**
    * Criamos este arquivo para dizer ao Kubernetes qual imagem rodar.
    * O conteúdo inicial usava a tag `:latest` como um *placeholder*.
    * *(Nota: Tivemos que ajustar o nome deste arquivo no workflow de CI para corresponder exatamente ao nome do arquivo aqui, corrigindo um erro de "No such file or directory").*

2.  **Manifesto de Serviço (`service.yaml` ou `serviço.yaml`)**
    * Criamos este arquivo para expor nossa aplicação dentro do cluster Kubernetes.

3.  Fizemos `git add .`, `git commit -m "Adiciona manifestos iniciais..."` e `git push` para enviar esses arquivos pela primeira vez.

### Passo 4: Conexão com o ArgoCD

Configuramos o ArgoCD (Etapa 4 do projeto) para monitorar este repositório.

1.  Na interface do ArgoCD, criamos um "New App" (`hello-app`).
2.  No campo **Repository URL**, inserimos o link deste repositório: `https://github.com/AndreViniciusDevOps/hello-manifests.git`
3.  Definimos a **Revision** como `HEAD` e o **Path** como `.`.
4.  Isso instruiu o ArgoCD a monitorar a branch `main` deste repositório e aplicar o que estivesse nela.

### Passo 5: Teste do Fluxo GitOps (Recebendo Pull Requests)

Este foi o teste final do nosso fluxo de trabalho.

1.  **Recebimento de PRs:** O pipeline de CI (do `hello-app`) começou a criar **Solicitações de Pull (Pull Requests)** automaticamente neste repositório.
2.  **Conteúdo do PR:** Cada PR sugeria a mudança da `image:` no `deployment.yaml` para uma nova tag SHA (ex: `sha-df78332`).
3.  **Ação de Deploy (Merge):** Nós **mesclamos (merged)** manualmente a solicitação de pull mais recente para aprovar o deploy.
4.  **Sincronização:** O ArgoCD detectou o merge na branch `main`. Ele sincronizou a aplicação, que inicialmente estava com erro `ImagePullBackOff` (porque a tag `:latest` não existia).
5.  **Resultado:** O pod foi atualizado para a imagem correta e mudou seu status para `Running`, concluindo o deploy.
