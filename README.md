# Wild Rydes - Serverless Web Application (AWS Lab)

Aplicação web serverless desenvolvida com base no laboratório prático da AWS, utilizando uma arquitetura moderna baseada na nuvem para gerir pedidos de viagens de unicórnio interativas. O projeto foi atualizado para suportar as versões mais recentes dos serviços e frameworks da AWS.

##  Arquitetura e Serviços AWS Utilizados
* **Frontend Estático:** Hospedado com suporte a assets modernos.
* **Autenticação:** **Amazon Cognito** (Gestão de utilizadores, registo, login e tokens JWT).
* **API & Backend:** **Amazon API Gateway** em conjunto com **AWS Lambda** (Funções em Node.js/Python para processar os pedidos).
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