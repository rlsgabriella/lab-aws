# 🦄 Wild Rydes - Serverless Web Application (AWS Lab)

Aplicação web serverless desenvolvida com base no laboratório prático da AWS, utilizando uma arquitetura moderna baseada em nuvem para gerir pedidos de viagens de unicórnio interativas.

> **Nota:** O projeto foi adaptado para suportar as versões mais recentes dos serviços e frameworks da AWS.

---

## 📚 Tutorial Base

Tutorial oficial da AWS:

[Build a Serverless Web Application with AWS Lambda, Amazon API Gateway, AWS Amplify, Amazon DynamoDB and Amazon Cognito](https://aws.amazon.com/pt/getting-started/hands-on/build-serverless-web-app-lambda-apigateway-s3-dynamodb-cognito/)

---

# 🔄 Registro das Adaptações do Lab

Como alguns pontos do tutorial ficaram desatualizados e o console da AWS mudou desde sua publicação original, este projeto precisou de algumas adaptações.

As alterações realizadas estão documentadas abaixo para facilitar a reprodução do laboratório utilizando as versões atuais dos serviços AWS.

---

## 🧠 Decisão Arquitetural: Por que usar o API Gateway em vez de chamar o Lambda diretamente?

Embora tecnicamente seja possível invocar uma função **AWS Lambda** diretamente a partir de uma aplicação frontend utilizando o SDK da AWS, a utilização do **Amazon API Gateway** como intermediário é uma prática importante em arquiteturas Serverless.

### Segurança e Autenticação — Cognito Authorizer

O API Gateway atua como um **portão de entrada seguro** para a aplicação.

Ele valida o token JWT emitido pelo **Amazon Cognito** antes de permitir que a requisição seja encaminhada para a função Lambda.

Fluxo simplificado:

```text
Frontend
   │
   │ JWT
   ▼
API Gateway
   │
   │ Cognito Authorizer
   ▼
AWS Lambda
   │
   ▼
DynamoDB
```

### Desacoplamento e Gestão de Rotas

O API Gateway transforma funções Lambda em endpoints HTTP públicos e padronizados.

Por exemplo:

```text
POST /ride
```

Isso permite gerir:

* métodos HTTP;
* endpoints REST;
* ambientes/stages;
* autenticação;
* autorização;
* versionamento da API;

sem precisar alterar diretamente a lógica do frontend ou da função Lambda.

### Controle de Tráfego e CORS

O API Gateway também permite configurar políticas de **CORS (Cross-Origin Resource Sharing)**.

Isso é essencial para permitir que aplicações web hospedadas em um domínio diferente possam comunicar-se com a API de forma controlada.

---

# 🛠️ Adaptações por Módulo

## Módulo 1 — AWS Amplify (Hospedagem Web Estática)

### Motivo da adaptação

No laboratório original, utilizava-se o **Amazon S3** diretamente para hospedar o site estático.

No fluxo utilizado neste projeto, a hospedagem web foi realizada através do **AWS Amplify**, facilitando:

* deploy da aplicação;
* integração com repositórios;
* gestão de domínio;
* publicação de arquivos estáticos;
* entrega dos assets do frontend.

### Como fizemos

1. Acessar o console do **AWS Amplify**.
2. Criar uma nova aplicação.
3. Selecionar a opção para publicar/hospedar os arquivos estáticos:

   * HTML;
   * CSS;
   * JavaScript.
4. Vincular o repositório ou realizar o upload direto dos arquivos.
5. Publicar a aplicação.

A URL gerada pelo Amplify passa a ser utilizada como domínio da aplicação e como referência para algumas configurações posteriores.

---

## Módulo 2 — Amazon Cognito (Gerenciamento de Usuários)

### Motivo da adaptação

O console do **Amazon Cognito** foi redesenhado.

O assistente antigo utilizado no tutorial, com etapas como:

```text
Configurar experiência de login
→ Requisitos de segurança
→ Integrar sua aplicação
```

não aparece mais da mesma maneira.

Atualmente, o fluxo é orientado principalmente pelo **tipo de aplicação**.

### Antes x Agora

| Lab original                                               | Console atual                                                                           |
| ---------------------------------------------------------- | --------------------------------------------------------------------------------------- |
| Assistente em várias páginas com nomes fixos               | Fluxo **Define your application**, escolhendo o `Application type`                      |
| `Configurar experiência de login`                          | `Options for sign-in identifiers` — Username / Email / Phone                            |
| `Configurar requisitos de segurança`                       | Não existe como etapa separada; MFA e política de senha ficam nas configurações do pool |
| `Integrar sua aplicação > Análise e clientes de aplicação` | Menu lateral **App clients**                                                            |
| App client com `ID do cliente`                             | Continua existindo, mas sem a antiga seção `Análise`                                    |

### Como fizemos

No console do Cognito:

```text
Cognito
└── User pools
    └── Create user pool
```

Configurações utilizadas:

| Configuração               | Valor                            |
| -------------------------- | -------------------------------- |
| Application type           | Single-page application (SPA)    |
| Name                       | WildRydes                        |
| Sign-in identifier         | Username                         |
| Required sign-up attribute | Email                            |
| Return URL                 | URL do site hospedado no Amplify |

Depois:

```text
Create your application
```

### IDs utilizados no `js/config.js`

**User Pool ID**

Disponível em:

```text
User Pool
└── Overview
```

**Client ID**

Disponível em:

```text
User Pool
└── App clients
```

### ⚠️ Pegadinhas

#### Entrega de e-mail

Utilizar:

```text
Send email with Cognito
```

Evitar:

```text
Send email with Amazon SES
```

O uso do SES exigiria verificar uma identidade no **Amazon SES** na mesma região, o que pode impedir o avanço do laboratório caso não esteja configurado.

#### Client Secret

Como o site é uma aplicação pública do tipo **SPA**, não deve ser utilizado um Client Secret.

O perfil SPA já é adequado para esse cenário.

#### Traduções do tutorial

Alguns nomes de campos e opções do tutorial em português não correspondem exatamente aos nomes exibidos atualmente no console.

Quando alguma configuração não for encontrada, é recomendável comparar com os nomes originais em inglês.

---

## Módulo 3 — AWS Lambda

### Motivo da adaptação

O runtime **Node.js 16.x** utilizado originalmente no laboratório foi depreciado.

Os runtimes atuais utilizam versões mais recentes do Node.js e o código original baseado em:

```javascript
require('aws-sdk')
```

não funciona da mesma forma.

O erro encontrado foi:

```text
Cannot find module 'aws-sdk'
```

Por isso, a função foi adaptada para utilizar o **AWS SDK for JavaScript v3**.

---

### Mudança de Runtime

**Antes:**

```text
Node.js 16.x
```

**Agora:**

```text
Node.js 22.x
```

ou versões mais recentes suportadas pelo Lambda.

---

### Mudanças no código da função

O código da aplicação frontend não precisou ser alterado nessa etapa.

A principal mudança ocorreu na função Lambda.

| Item       | Tutorial original          | Implementação atual                                       |
| ---------- | -------------------------- | --------------------------------------------------------- |
| SDK        | `aws-sdk` (v2)             | `@aws-sdk/client-dynamodb` + `@aws-sdk/lib-dynamodb` (v3) |
| Importação | CommonJS `require(...)`    | ESM `import ... from ...`                                 |
| Arquivo    | `index.js`                 | `index.mjs`                                               |
| Handler    | `exports.handler = (...)`  | `export const handler = async (...)`                      |
| DynamoDB   | `ddb.put({...}).promise()` | `ddb.send(new PutCommand({...}))`                         |

O restante da implementação permanece seguindo a lógica original, incluindo:

* tabela `Rides`;
* `cognito:username`;
* formato de resposta utilizado pela integração proxy do API Gateway.

### Observações

Em arquivos ESM, como:

```text
index.mjs
```

o comando:

```javascript
require()
```

não funciona da mesma maneira.

Por isso, o código foi convertido para:

```javascript
import ... from ...
```

Também é importante garantir que o nome da tabela DynamoDB utilizado no código seja exatamente:

```text
Rides
```

incluindo letras maiúsculas e minúsculas.

---

## Módulo 4 — API Gateway e Cognito Authorizer

### Motivo da adaptação

Nesta etapa, o principal problema encontrado foi relacionado à tradução do tutorial.

O texto:

```text
Insira Autorização para a Origem do token
```

pode gerar confusão.

O valor esperado pelo API Gateway é o nome real do header HTTP:

```text
Authorization
```

### Erro encontrado

Ao deixar o campo vazio ou utilizar um valor incorreto:

```text
Invalid token source expression: method.request.header..

The source must be a method request header, matching

'method.request.header.[a-zA-Z0-9._-]+'
```

Os dois pontos consecutivos:

```text
header..
```

indicam que o nome do header não foi configurado corretamente.

### Configuração correta do Authorizer

| Configuração      | Valor                     |
| ----------------- | ------------------------- |
| Nome              | WildRydes                 |
| Tipo              | Cognito                   |
| Região            | Mesma região do User Pool |
| Grupo de usuários | WildRydes                 |
| Origem do token   | `Authorization`           |

---

## ⚠️ Observações Gerais do Lab

O template **Executar pilha** do CloudFormation disponibilizado pelo laboratório retornou:

```text
403 Forbidden
```

ao tentar realizar o download diretamente.

Por esse motivo, a configuração manual através do console da AWS foi mantida.

Além disso, no arquivo:

```text
js/config.js
```

o campo:

```javascript
invokeUrl
```

só deve ser preenchido depois da criação da API no **Módulo 4** e da publicação do stage, por exemplo:

```text
prod
```

Durante o Módulo 2, esse valor pode permanecer vazio.

---

# 🚀 Arquitetura e Serviços AWS Utilizados

* **Frontend Estático:** hospedagem dos arquivos HTML, CSS e JavaScript utilizando AWS Amplify.
* **Autenticação:** **Amazon Cognito** para gestão de usuários, registro, login e tokens JWT.
* **API:** **Amazon API Gateway** para disponibilização dos endpoints HTTP e autorização das requisições.
* **Backend:** **AWS Lambda** utilizando Node.js para processamento dos pedidos.
* **Banco de Dados:** **Amazon DynamoDB** para armazenamento dos dados das viagens realizadas.
* **Mapeamento e Geolocalização:** **ArcGIS API for JavaScript** para renderização do mapa e animações das rotas.

---

# 🏗️ Arquitetura da Aplicação

```text
                    ┌──────────────────────┐
                    │     AWS Amplify      │
                    │   Frontend Estático  │
                    └──────────┬───────────┘
                               │
                               ▼
                    ┌──────────────────────┐
                    │   Amazon Cognito     │
                    │  Login / Registro    │
                    │        JWT           │
                    └──────────┬───────────┘
                               │
                               ▼
                    ┌──────────────────────┐
                    │ Amazon API Gateway   │
                    │    POST /ride        │
                    │ Cognito Authorizer   │
                    └──────────┬───────────┘
                               │
                               ▼
                    ┌──────────────────────┐
                    │      AWS Lambda      │
                    │       Node.js        │
                    └──────────┬───────────┘
                               │
                               ▼
                    ┌──────────────────────┐
                    │   Amazon DynamoDB    │
                    │    Tabela Rides      │
                    └──────────────────────┘
```

---

## 🗺️ Interface da Aplicação

![Wild Rydes](image.png)

---

# 📁 Estrutura do Projeto

```text
.
├── js/
│   ├── config.js
│   ├── esri-map.js
│   └── ride.js
│
├── ride.html
├── image.png
└── README.md
```

### `js/config.js`

Responsável pelas configurações globais da aplicação:

* `UserPoolId`;
* `ClientId`;
* `region`;
* `invokeUrl` do API Gateway.

### `js/esri-map.js`

Responsável por:

* inicialização do mapa;
* gestão dos pontos de recolha (*pickup*);
* animações vetoriais;
* interação com o ArcGIS.

### `js/ride.js`

Responsável por:

* validação da sessão;
* autenticação;
* recuperação do token;
* envio de requisições `POST` para a API;
* tratamento das respostas.

### `ride.html`

Interface principal da aplicação para seleção de rotas e solicitação das viagens.

---

# ⚙️ Configuração e Execução

## 1. Configurar as credenciais da AWS

Edite o arquivo:

```text
js/config.js
```

com os dados gerados no console da AWS:

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
```

Substitua:

```text
SEU_USER_POOL_ID
SEU_CLIENT_ID
SUA_URL_DO_API_GATEWAY
```

pelos valores correspondentes à sua infraestrutura AWS.

---

## 2. Configurar o Amazon Cognito

Certifique-se de que o User Pool utilizado pela aplicação possui:

```text
Application type: Single-page application (SPA)
Sign-in identifier: Username
Required attribute: Email
```

Depois, copie o **User Pool ID** e o **Client ID** para o `config.js`.

---

## 3. Configurar o DynamoDB

A função Lambda espera encontrar uma tabela chamada:

```text
Rides
```

Certifique-se de que o nome configurado no DynamoDB seja exatamente igual ao utilizado pela função.

---

## 4. Configurar a função Lambda

Utilize um runtime Node.js atualmente suportado pela AWS.

A função utiliza o **AWS SDK for JavaScript v3**, incluindo:

```javascript
import { DynamoDBClient } from '@aws-sdk/client-dynamodb';
import { DynamoDBDocumentClient, PutCommand } from '@aws-sdk/lib-dynamodb';
```

---

## 5. Configurar o API Gateway

Configure o endpoint utilizado para solicitar uma viagem:

```http
POST /ride
```

Adicione o **Cognito Authorizer** e configure o Token Source como:

```text
Authorization
```

Depois, faça o deploy da API para o stage:

```text
prod
```

Copie a **Invoke URL** gerada pelo API Gateway.

---

## 6. Atualizar o `config.js`

Depois que o API Gateway estiver publicado:

```javascript
api: {
    invokeUrl: 'https://SEU_ID.execute-api.us-east-1.amazonaws.com/prod'
}
```

---

# 🔐 Fluxo de Autenticação

O fluxo utilizado pela aplicação é:

```text
Usuário
   │
   ▼
Cadastro / Login
   │
   ▼
Amazon Cognito
   │
   │ JWT
   ▼
Frontend
   │
   │ Authorization: JWT
   ▼
API Gateway
   │
   │ Cognito Authorizer
   ▼
AWS Lambda
   │
   ▼
Amazon DynamoDB
```

O API Gateway valida o token recebido antes de encaminhar a requisição para a função Lambda.

---

# 🧰 Tecnologias Utilizadas

* HTML
* CSS
* JavaScript
* AWS Amplify
* Amazon Cognito
* Amazon API Gateway
* AWS Lambda
* Amazon DynamoDB
* AWS SDK for JavaScript v3
* ArcGIS API for JavaScript

---

# 📌 Objetivo do Projeto

Este projeto foi desenvolvido para fins de **estudo e prática de arquitetura Serverless na AWS**.

O objetivo principal é compreender a integração entre serviços gerenciados da AWS, passando pelo fluxo completo:

```text
Frontend
    ↓
Autenticação
    ↓
API
    ↓
Função Serverless
    ↓
Banco de Dados
```

Além da implementação do laboratório original, este repositório documenta as adaptações necessárias para executar o projeto utilizando as versões atuais dos serviços e do console da AWS.
