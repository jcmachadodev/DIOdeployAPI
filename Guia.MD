🚀 Passo a Passo: Deploy de uma API na Azure
📋 Pré-requisitos

Antes de começar, garanta que você tenha:

Uma conta na Microsoft Azure

O código da sua API (pode ser Java, Node.js, .NET, Python, etc.)

O Azure CLI instalado localmente
👉 Guia oficial

(Opcional) Git instalado e configurado

🧭 1. Faça login na sua conta Azure
az login


Após o comando, será aberta uma janela para login no navegador.
Depois do login, o CLI exibirá as assinaturas disponíveis.

🗂️ 2. Crie um Resource Group

Um Resource Group agrupa todos os recursos do seu projeto.

az group create --name meu-grupo-api --location eastus


💡 Substitua meu-grupo-api pelo nome desejado e eastus pela região mais próxima.

☁️ 3. Crie um App Service Plan

O App Service Plan define a capacidade de computação e o preço.

az appservice plan create \
  --name meu-plano-api \
  --resource-group meu-grupo-api \
  --sku B1


O plano B1 é uma opção básica com custo baixo (ideal para testes).

⚙️ 4. Crie o App Service (sua API)
az webapp create \
  --resource-group meu-grupo-api \
  --plan meu-plano-api \
  --name minha-api-azure \
  --runtime "JAVA:8-java11" 


🔄 Substitua o --runtime conforme sua linguagem:

Node.js → "NODE:18-lts"

.NET → "DOTNETCORE:6.0"

Python → "PYTHON:3.9"

Java → "JAVA:8-java11" (ou outro compatível)

🧱 5. Configure variáveis de ambiente (opcional)

Caso sua API precise de variáveis de ambiente (como banco de dados):

az webapp config appsettings set \
  --resource-group meu-grupo-api \
  --name minha-api-azure \
  --settings DATABASE_URL="jdbc:mysql://..." API_KEY="minhachave"

🚢 6. Faça o deploy da API

Existem três formas principais de deploy:

🔹 Opção 1 — Via ZIP (direto do código)

Crie um arquivo .zip com seu código e rode:

az webapp deployment source config-zip \
  --resource-group meu-grupo-api \
  --name minha-api-azure \
  --src ./minha-api.zip

🔹 Opção 2 — Via GitHub Actions (CI/CD)

Configure integração contínua:

az webapp deployment source config \
  --name minha-api-azure \
  --resource-group meu-grupo-api \
  --repo-url https://github.com/usuario/meu-repo-api \
  --branch main \
  --manual-integration


💡 Assim, cada push no GitHub faz o deploy automático.

🔹 Opção 3 — Via Docker (Container)

Se sua API está containerizada:

az webapp create \
  --resource-group meu-grupo-api \
  --plan meu-plano-api \
  --name minha-api-azure \
  --deployment-container-image-name usuario/minhaapi:latest

🔍 7. Teste o deploy

Acesse no navegador:

https://minha-api-azure.azurewebsites.net


Você também pode checar logs:

az webapp log tail --name minha-api-azure --resource-group meu-grupo-api
