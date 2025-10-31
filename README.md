# 🧙‍♂️ O Desafio dos Portais (CloudFront + S3)

Este guia é um grimório para forjar um "portal mágico" (Amazon CloudFront) capaz de distribuir artefatos (um site estático) globalmente.

Nossa missão é construir um sistema seguro: o "cofre" (Bucket S3) ficará trancado para o público, e apenas o CloudFront terá a "chave-mestra" para acessar e distribuir o conteúdo.

## 🏛️ Arquitetura

A arquitetura que vamos forjar segue o fluxo do diagrama original:

* `Usuário` ➡️ `CloudFront` (Verifica o Cache) ➡️ `Origem` (S3 Privado)

## 📂 Arquivos do Laboratório

Para este lab, você precisará dos artefatos (`index.html` e `witcher_escola.jpg`) disponíveis neste repositório:

* `https://github.com/weritonpetreca/cloudfront-lab`

> **Nota:** Conforme o guia, a forma mais simples de baixar os arquivos sem usar Git é clicar em `Code` > `Download ZIP` no repositório.

---

## 🛠️ Fases do Guia

Seguiremos o caminho do "Artesão Arcano", configurando cada componente manualmente para o máximo controle.

### Fase 1: Forjando o Cofre (Configuração do S3)

1.  **Criar o Bucket S3:** No console S3, crie um bucket com um nome globalmente único (ex: `lab-bucket-cloudfront-weritonpetreca`).
2.  **Bloquear Acesso Público:** Mantenha a opção **"Block all public access"** (Bloquear todo o acesso público) marcada.

> 🛡️ **Ponto Crítico de Segurança:** Este é o ponto crucial. Ninguém deve acessar nosso cofre diretamente. Daremos uma chave especial apenas para o CloudFront.

### Fase 2: Colocando os Artefatos no Cofre (Upload)

1.  **Upload:** Acesse o bucket criado e faça o upload dos arquivos `index.html` e `witcher_escola.jpg`.

### Fase 3: Criando o Portal Mágico (CloudFront)

1.  **Criar Distribuição:** Navegue até o serviço CloudFront e clique em "Create distribution".
2.  **Configurar Nome e Tipo:** Dê um nome (ex: "Distribution-1") e mantenha "Single website or app".
3.  **Configurar Origem:**
    * **Origin type:** `Amazon S3`
    * **Origin (S3 origin):** Selecione seu bucket S3 na lista (ex: `lab-bucket-cloudfront-....s3.amazonaws.com`).
    * **Settings:** Marque `Allow private S3 bucket…`
4.  **Criar:** Clique em `Create Distribution`.

### 🔑 Fase 4: Verificando a Chave Mestra (S3 Bucket Policy)

Enquanto o CloudFront faz o deploy, vamos verificar a "chave" no cofre.

1.  **Verificar S3:** Volte ao console do S3 -> seu bucket -> aba **"Permissions"** (Permissões).
2.  **Confirmar Política:** Você verá uma política JSON em **"Bucket policy"** que **não estava lá antes**.

> Esta política foi criada automaticamente pelo CloudFront e permite (Allow) que o serviço `cloudfront.amazonaws.com` execute a ação `s3:GetObject` (ler arquivos) no seu bucket, *desde que* a requisição venha da sua distribuição específica.

### 📜 Fase 5: O Pergaminho Padrão (Default Root Object)

Por padrão, acessar o domínio raiz do CloudFront (ex: `https://[d123].cloudfront.net/`) resultará em "Access Denied", pois não especificamos qual arquivo queremos. (Atualmente, seria preciso usar a URL completa: `.../index.html`).

1.  **Editar Configurações:** No console do CloudFront, clique no ID da sua distribuição -> aba **"Settings"** (Configurações) -> **"Edit"**.
2.  **Definir Objeto Raiz:** No campo **"Default root object"** (Objeto raiz padrão), digite: `index.html`
3.  **Salvar Alterações** e aguarde o deploy.

### ⚔️ Fase 6: O Teste Final

1.  **Aguardar Deploy:** Aguarde a propagação da última mudança (o status da distribuição voltará para "Enabled" ou similar).
2.  **Copiar Domínio:** Na aba "Distributions" do CloudFront, copie o **"Domain name (standard)"** (ex: `d123abcdef.cloudfront.net`).
3.  **Testar:** Cole este domínio no seu navegador.

> **Resultado Esperado:** Você deve ver sua página `index.html` temática do Witcher, com a imagem `witcher_escola.jpg`, sendo servida em alta velocidade pelo CloudFront!

### 🌀 Fase 7: O Desmanchar (Desprovisionamento)

Um bom Bruxo sempre limpa seus rastros para não gerar custos.

1.  **Desabilitar o CloudFront:**
    * (Você não pode deletar uma distribuição imediatamente; você deve primeiro desabilitá-la).
    * No console do CloudFront, selecione sua distribuição e clique em **"Disable"** (Desabilitar).

2.  **Esvaziar e Destruir o S3:**
    * (Você não pode deletar um bucket que contém artefatos).
    * No console do S3, acesse seu bucket e clique em **"Empty"** (Esvaziar). Confirme a exclusão permanente (`permanently delete`).
    * Após o esvaziamento, volte à lista de buckets, selecione-o e clique em **"Delete"** (Excluir), confirmando o nome do bucket.

---

**Jornada Concluída!**
