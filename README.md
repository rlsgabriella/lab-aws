# Wild Rydes - Serverless Web Application (AWS Lab)

Aplicação web serverless desenvolvida com base no laboratório prático da AWS, utilizando uma arquitetura moderna baseada na nuvem para gerir pedidos de viagens de unicórnio interativas. 

### Nota: O projeto foi adaptado para suportar as versões mais recentes dos serviços e frameworks da AWS.

## Registro dos pontos em que este lab precisou ser adaptado

Como a AWS desatualizou o tutorial e/ou o console mudou desde a publicação.

Tutorial base (PT): https://aws.amazon.com/pt/getting-started/hands-on/build-serverless-web-app-lambda-apigateway-s3-dynamodb-cognito/

---

## Módulo 2 - Amazon Cognito (gerenciamento de usuários)

Motivo: o console do Cognito foi redesenhado. O assistente antigo
("Configurar experiência de login" -> "Requisitos de segurança" -> ...)
não aparece mais. Agora o fluxo é guiado por **tipo de aplicação**.

### Antes (lab original) x Agora (console novo)

| Lab original | Console novo |
| --- | --- |
| Assistente em várias páginas com nomes fixos | Fluxo "Define your application" escolhendo o Application type |
| Página "Configurar experiência de login" | "Options for sign-in identifiers" (Username / Email / Phone) |
| Página "Configurar requisitos de segurança" | Não existe; MFA e política de senha ficam nas configurações do pool após a criação |
| Página "Integrar sua aplicação" > "Análise e clientes de aplicação" | Menu lateral **App clients** |
| App client com "ID do cliente" | Idem, mas sem a seção "Análise" |

### Como fizemos

1. Cognito console -> **User pools** -> **Create user pool**.
2. **Application type**: **Single-page application (SPA)**.
3. **Name**: `WildRydes`.
4. **Options for sign-in identifiers**: **Username**.
   **Required attributes for sign-up**: `email`.
5. **Add a return URL**: URL do site Amplify (não é usado pelo SDK JS, mas é obrigatório).
6. **Create your application**.
7. IDs usados no `js/config.js`:
   - **User Pool ID**: em **Overview**.
   - **Client ID**: em **App clients**.

### Pegadinhas

- **Entrega de e-mail**: usar **Send email with Cognito**.
  NÃO usar "Send email with Amazon SES" (exigiria verificar uma identidade
  no SES na mesma região - foi onde o lab travou).
- **Client secret**: o site é um app público -> **não gerar segredo**.
  (O perfil SPA já cria assim.)
- O tutorial em PT traduziu indevidamente nomes de UI/valores. Sempre
  confira o texto original em inglês quando algo não bater.

---

## Módulo 3 - AWS Lambda

Motivo: o runtime **Node.js 16.x não existe mais** (depreciado em jun/2024)
e os runtimes novos (Node.js 22.x / 24.x) **não incluem o AWS SDK v2**.
O código original usa `require('aws-sdk')`, que falha ("Cannot find module
'aws-sdk'") nesses runtimes.

### Mudança de runtime

- Antes: **Node.js 16.x** (indisponível).
- Agora: **Node.js 22.x** (ou 24.x).

### Mudanças no código da função

O código da **aplicação/site não mudou**. Só a Lambda.

| Item | Antes (tutorial) | Agora |
| --- | --- | --- |
| SDK | `aws-sdk` (v2) | `@aws-sdk/client-dynamodb` + `@aws-sdk/lib-dynamodb` (v3) |
| Import | CommonJS `require(...)` | ESM `import ... from ...` |
| Arquivo | `index.js` | `index.mjs` (padrão do console novo) |
| Handler | `exports.handler = (event, context, callback)` + `callback(...)` | `export const handler = async (event, context)` retornando o objeto |
| Gravação DynamoDB | `ddb.put({...}).promise()` | `ddb.send(new PutCommand({...}))` |

O restante (nome da tabela `Rides`, `cognito:username`, formato de resposta
do proxy integration) permanece igual.

O SDK v3 já vem embutido no runtime Node.js 22/24, então **não** precisa de
layer nem `npm install` ao colar o código no console.

### Observações

- Em ESM (`index.mjs`) `require` não funciona; por isso o código foi
  convertido para `import`.
- O nome da tabela DynamoDB no código (`Rides`) deve ser igual, maiúsculas
  e minúsculas incluídas.

---

## Módulo 4 - API Gateway (autorizador do Cognito)

Motivo: tradução ruim do tutorial.

- O texto "Insira **Autorização** para a Origem do token" é a tradução
  automática do nome do header HTTP `Authorization`.
- No campo **Origem do token / Token source** deve-se digitar
  literalmente:

  ```
  Authorization
  ```

- Erro obtido ao deixar em branco / digitar o termo traduzido:

  ```
  Invalid token source expression: method.request.header..
  The source must be a method request header, matching
  'method.request.header.[a-zA-Z0-9._-]+'
  ```

  Os dois pontos seguidos (`..`) indicam que o nome do header veio vazio.

Configuração correta do autorizador:

- Nome: `WildRydes`
- Tipo: Cognito
- Região: a mesma do user pool
- Grupo de usuários: `WildRydes`
- Origem do token: `Authorization`

---

## Observação geral

- Os templates "Executar pilha" do CloudFormation do lab retornaram
  **403 (Forbidden)** ao tentar baixar diretamente, então o caminho manual
  do console foi mantido.
- No `js/config.js`, o `invokeUrl` só é preenchido após criar a API no
  Módulo 4 (stage `prod`); no Módulo 2 ele fica vazio.




## 🚀 Arquitetura e Serviços AWS Utilizados
* **Frontend Estático:** Hospedagem com suporte a assets modernos.
* **Autenticação:** **Amazon Cognito** (Gestão de utilizadores, registo, login e tokens JWT).
* **API & Backend:** **Amazon API Gateway** em conjunto com **AWS Lambda** (Funções em Node.js atualizado para processar os pedidos).
* **Base de Dados:** **Amazon DynamoDB** (Armazenamento de dados das viagens efetuadas).
* **Mapeamento & Geolocalização:** **ArcGIS API for JavaScript** para renderização do mapa e animações de rota.

---

## 📁 Estrutura do Projeto
* `js/config.js` — Configurações globais e credenciais dos endpoints (`UserPoolId`, `ClientId`, `region` e `invokeUrl` do API Gateway).
* `js/esri-map.js` — Inicialização do mapa, gestão de pontos de recolha (*pickup*) e animações vetoriais.
* `js/ride.js` — Lógica de autenticação de sessão, envio de requisições POST para a API e tratamento de respostas.
* `ride.html` — Interface gráfica principal para a seleção de rotas e pedidos.

---

## ⚙️ Configuração e Execução

1. **Configurar as credenciais da AWS:**
   Edite o ficheiro `js/config.js` com os dados gerados na sua consola da AWS:
   ```javascript
   window._config = {
       cognito: {
           userPoolId: 'SEU_USER_POOL_ID',
           userPoolClientId: 'SEU_CLIENT_ID',
           region: 'us-east-1'
       },
       api: {
           invokeUrl: 'SUA_URL_DO_API_GATEWAY'
       }
   };