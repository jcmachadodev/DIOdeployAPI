🚀 Guia: Deploy de API no Azure App ServiceEste guia cobre a criação dos recursos necessários, a configuração e o deploy do seu código usando o método "Zip Deploy".Pré-requisitosAntes de começar, certifique-se de que você tem:Uma conta do Azure: Você precisa de uma assinatura ativa. (Uma conta gratuita está disponível).Azure CLI: Instalada em sua máquina. (Você pode instalar a partir deste link).Código da API: O código-fonte da sua API pronto e, de preferência, compilado (ex: a pasta dist, bin/Release, ou o build do seu projeto).Um arquivo ZIP: Um arquivo .zip contendo apenas os arquivos de build da sua API. (Não zipe a pasta-pai, entre na pasta de build e zipe o conteúdo).Passo 1: Login no AzureAbra seu terminal ou prompt de comando e faça o login na sua conta do Azure.Bashaz login
Uma janela do navegador será aberta para você se autenticar. Após o login, selecione a assinatura que deseja usar, se tiver mais de uma.Bash# Opcional: Liste suas assinaturas
az account list --output table

# Opcional: Defina a assinatura padrão
az account set --subscription "NOME_OU_ID_DA_SUA_ASSINATURA"
Passo 2: Criar um Grupo de Recursos (Resource Group)Um Grupo de Recursos é um contêiner lógico para agrupar e gerenciar seus recursos do Azure (como seu site, banco de dados, etc.).Bash# Variáveis para facilitar (altere os valores)
LOCATION="brazilsouth"
RESOURCE_GROUP="meu-grupo-recursos-api"

az group create --name $RESOURCE_GROUP --location $LOCATION
Passo 3: Criar um Plano do App ServiceO Plano do App Service define os recursos de computação (hardware, SO, preço) que sua API usará.F1 (Free) é ótimo para testes/dev.B1 (Basic) ou S1 (Standard) são recomendados para produção.Bash# Variável para o plano (altere o valor)
APP_SERVICE_PLAN="meu-plano-api"

az appservice plan create \
    --name $APP_SERVICE_PLAN \
    --resource-group $RESOURCE_GROUP \
    --location $LOCATION \
    --sku F1 \
    --is-linux
Nota: O --is-linux cria um plano baseado em Linux, que é geralmente mais barato e preferido para a maioria das runtimes modernas (Node, Python, .NET Core, Java). Se você precisa de .NET Framework legado, remova este parâmetro.Passo 4: Criar o App Service (Web App)Este é o recurso que de fato hospedará sua API. É aqui que você define o "runtime" (a pilha de tecnologia).Bash# Variável para o nome da API (TEM QUE SER GLOBALMENTE ÚNICO)
API_NAME="minha-api-unica-20251029"

# Veja a seção "Runtimes Comuns" abaixo para escolher o valor correto
RUNTIME="DOTNET|6.0" 

az webapp create \
    --name $API_NAME \
    --resource-group $RESOURCE_GROUP \
    --plan $APP_SERVICE_PLAN \
    --runtime $RUNTIME
Runtimes Comuns (Exemplos para --runtime)TecnologiaValor do Runtime.NET 8`DOTNET.NET 6`DOTNETNode.js 18 LTS`NODENode.js 16 LTS`NODEPython 3.11`PYTHONJava 17 (Tomcat)`JAVAPara ver todos os runtimes disponíveis, use: az webapp list-runtimes --linux ou az webapp list-runtimes --windowsPasso 5: Configurar Variáveis de AmbienteQuase toda API precisa de "secrets", como strings de conexão de banco de dados ou chaves de API. Nunca coloque isso no seu código. Configure-os no Azure:Bash# Exemplo para uma string de conexão de banco de dados
az webapp config appsettings set \
    --name $API_NAME \
    --resource-group $RESOURCE_GROUP \
    --settings "ConnectionStrings__DefaultConnection=SUA_CONNECTION_STRING_AQUI"

# Exemplo para uma chave de API
az webapp config appsettings set \
    --name $API_NAME \
    --resource-group $RESOURCE_GROUP \
    --settings "MinhaChaveSecreta=valor-12345" "ASPNETCORE_ENVIRONMENT=Production"
Passo 6: Fazer o Deploy da API (Zip Deploy)Este é o comando que envia seu código (o arquivo .zip dos pré-requisitos) para o App Service.Bash# Navegue até onde está seu arquivo .zip
# Ex: cd minha-api/build/

# Defina o caminho para o seu arquivo zip
ARQUIVO_ZIP_PATH="./meu-build-da-api.zip"

az webapp deployment source config-zip \
    --resource-group $RESOURCE_GROUP \
    --name $API_NAME \
    --src $ARQUIVO_ZIP_PATH
O Azure irá carregar o zip, descompactá-lo no servidor e reiniciar sua aplicação com o novo código.Passo 7: Testar sua APISeu deploy está concluído! O Azure fornece um URL público para sua API. Você pode encontrá-lo no portal ou com o comando abaixo:Bash# Pega o hostname padrão
HOSTNAME=$(az webapp show --name $API_NAME --resource-group $RESOURCE_GROUP --query "defaultHostName" --output tsv)

echo "Sua API está disponível em: https://$HOSTNAME/"

# Exemplo de teste com curl (se sua API tiver um endpoint /api/health)
curl https://$HOSTNAME/api/health
Você pode agora usar o Postman, Insomnia ou qualquer cliente HTTP para testar seus endpoints.