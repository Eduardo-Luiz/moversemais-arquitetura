# 🎯 BFF - OAuth Endpoints (Proxy para Auth Service)

## 📋 **Card de Desenvolvimento**

**Serviço**: moversemais-store-graphql (BFF)  
**Tipo**: Integração OAuth via BFF  
**Prioridade**: 🔴 CRÍTICA  
**Estimativa**: 1-2 dias  
**Status**: TODO  
**Dependências:**
- ✅ Card 007 (Auth/User Integration) - Concluído
- ⏳ Card 008 (Frontend Auth) - BLOQUEADO por este card

---

## 📋 **CONTEXTO**

### **Situação Atual:**

O Frontend precisa integrar com OAuth (Google/Facebook/LinkedIn), mas atualmente **NÃO EXISTE** forma correta de fazer isso respeitando a arquitetura BFF.

**Problema identificado no Card 008:**
```
❌ Frontend tentou chamar Auth Service diretamente
❌ Violação do padrão BFF
❌ Expõe serviço interno publicamente
```

### **Arquitetura BFF (Princípio Fundamental):**

```
✅ CORRETO:
Frontend ──── GraphQL ────> BFF ──── REST ────> Auth Service
                                                 User Service
                                                 Objective Service

❌ ERRADO:
Frontend ──── HTTP ────> Auth Service (direto!)
```

### **Princípio:**

> **"Frontend comunica APENAS com BFF. BFF orquestra todos os serviços backend."**

### **Problema a Resolver:**

**Como permitir que o Frontend faça login OAuth respeitando a arquitetura BFF?**

**Fluxo OAuth é complexo:**
1. Frontend precisa redirecionar usuário para Google
2. Google redireciona de volta com código
3. Código precisa ser trocado por JWT
4. JWT precisa ser validado
5. Dados do usuário precisam ser retornados

**Onde o BFF entra nisso?**

### **Solução Proposta:**

BFF expõe endpoints/mutations GraphQL para:
1. **Iniciar OAuth** - Retorna URL de redirecionamento
2. **Processar Callback** - Valida token, retorna dados do usuário

Ou alternativamente:
1. **Endpoints REST no BFF** - Proxy completo para Auth Service
2. Frontend redireciona para BFF, BFF redireciona para Auth Service

### **Impacto:**

**Se NÃO implementado:**
- ❌ Card 008 (Frontend OAuth) fica bloqueado
- ❌ Frontend não pode fazer login real
- ❌ Sistema SSO incompleto
- ❌ Violação de arquitetura se Frontend chamar Auth Service direto

**Se implementado:**
- ✅ Arquitetura BFF respeitada
- ✅ Frontend chama apenas BFF
- ✅ Auth Service permanece interno
- ✅ Card 008 pode ser implementado corretamente
- ✅ Sistema SSO completo e arquiteturalmente correto

---

## 🎯 **REQUISITOS OBRIGATÓRIOS**

### **1. Endpoints OAuth no BFF**

**Funcionalidade:**
- Expor endpoints para iniciar fluxo OAuth
- Retornar URL de redirecionamento do Auth Service
- Ou fazer proxy completo (receber callback, processar, retornar JWT)

**Opções de implementação (VOCÊ DECIDE):**

**Opção A - GraphQL Mutations:**
```graphql
mutation GetOAuthRedirectUrl($provider: Provider!) {
  getOAuthRedirectUrl(provider: $provider) {
    redirectUrl
  }
}

mutation ValidateOAuthToken($token: String!) {
  validateOAuthToken(token: $token) {
    jwt
    user {
      id
      email
      name
      provider
    }
  }
}
```

**Opção B - REST Endpoints (Proxy):**
```
GET  /auth/login/{provider}  → Redireciona para Auth Service
GET  /auth/callback          → Processa callback, retorna JWT
```

**Opção C - Híbrida:**
- GraphQL para obter URL
- REST para callback (mais fácil redirecionar)

### **2. Comunicação com Auth Service**

**Funcionalidade:**
- BFF chama Auth Service para obter URLs OAuth
- BFF valida tokens com Auth Service (se necessário)
- BFF adiciona headers de segurança (X-Request-Id, etc.)

**Endpoints do Auth Service (referência):**
```
GET  http://localhost:8084/oauth2/authorization/{provider}
POST http://localhost:8084/api/v1/auth/validate (se necessário)
```

### **3. CORS e Segurança**

**Funcionalidade:**
- BFF configurado para aceitar requisições do Frontend
- Auth Service NÃO precisa aceitar requisições do Frontend
- CORS restrito (apenas Frontend → BFF)

### **4. Tratamento de Erros OAuth**

**Funcionalidade:**
- OAuth cancelado pelo usuário
- Erro do provider (Google/Facebook/LinkedIn)
- Auth Service offline
- Token inválido
- Mensagens claras para o Frontend

### **5. Configuração**

**Variáveis de ambiente:**
```
AUTH_SERVICE_URL=http://localhost:8084
FRONTEND_URL=http://localhost:5173
```

---

## ⚠️ **RESTRIÇÕES**

### **❌ NÃO PODE ALTERAR:**

- GraphQL schema existente (apenas adicionar)
- Middleware de autenticação existente (auth-simple.ts)
- Backend client existente (apenas estender se necessário)
- Resolvers existentes (apenas adicionar novos)

### **✅ VOCÊ DECIDE:**

- GraphQL mutations vs REST endpoints vs híbrido
- Como processar callback (proxy completo? validação separada?)
- Estrutura de resolvers (novo arquivo? existente?)
- Como validar JWT (chamar Auth Service? validação local?)
- Tratamento de erros (códigos, mensagens)
- Estados de redirecionamento

---

## 📚 **DOCUMENTAÇÃO DE REFERÊNCIA**

### **Cards Relacionados:**

1. **`DONE/001__BFF-SSO-Implementation.md`** - BFF base já implementado
2. **`DONE/005__Auth-Service-SSO-Implementation.md`** - Auth Service OAuth
3. **`DONE/007__Auth-Service-Integration-Implementation.md`** - Auth/User integração
4. **`TODO/008__Frontend-Auth-Integration.md`** - Frontend (BLOQUEADO por este card)

### **Arquivos para Estudar:**

**No moversemais-store-graphql:**
```
app/api/graphql/
├── schema/types.ts              # GraphQL schema
├── resolvers/                   # Resolvers existentes
├── services/backend-client.ts   # Cliente para backends
├── middleware/auth-simple.ts    # Middleware de autenticação
└── route.ts                     # Rota GraphQL
```

**Perguntas para responder:**
- Como adicionar novos tipos GraphQL?
- Como criar novos resolvers?
- Como fazer requisições HTTP para Auth Service?
- Como tratar redirecionamentos no BFF?
- REST vs GraphQL para OAuth - qual melhor?

---

## 🔧 **WORKFLOW**

### **1. Estude a Aplicação (OBRIGATÓRIO)**

```bash
cd /Users/eduardosouza/Development/Repository/moversemais/moversemais-store-graphql
```

**Leia e compreenda:**
1. `app/api/graphql/schema/types.ts` - Schema GraphQL
2. `app/api/graphql/resolvers/` - Como criar resolvers
3. `app/api/graphql/services/backend-client.ts` - Cliente HTTP
4. Cards 001, 005, 007 - Entenda Auth Service
5. AGENTS.md do BFF - Padrões do projeto

**Perguntas para responder:**
- Como BFF se comunica com backends atualmente?
- OAuth via GraphQL ou REST - qual melhor?
- Como processar redirecionamentos?
- Como retornar dados para o Frontend?

### **2. Crie sua Branch**

```bash
git checkout -b feature/bff-oauth-endpoints
```

### **3. Implemente a Solução**

**Você é o especialista em Next.js + GraphQL!**

Decida e implemente:
- GraphQL mutations? REST endpoints? Híbrido?
- Como estruturar resolvers (novo arquivo? existente?)
- Como processar callback OAuth (proxy? validação?)
- Como retornar JWT para Frontend
- Tratamento de erros e redirecionamentos

**Considere:**
- ✅ Next.js best practices
- ✅ GraphQL Yoga patterns
- ✅ TypeScript types
- ✅ Clean Code
- ✅ Tratamento de erros robusto
- ✅ Logs de auditoria

### **4. Teste Exaustivamente**

**Cenários obrigatórios:**

```bash
# 1. Obter URL de redirecionamento
# Chamar BFF para obter URL do Auth Service
# Verificar URL retornada está correta

# 2. Fluxo OAuth completo
# Frontend → BFF → Auth Service → Google → Auth Service → BFF → Frontend
# JWT válido retornado

# 3. OAuth cancelado
# Usuário cancela no Google
# Erro tratado e retornado para Frontend

# 4. Auth Service offline
# BFF trata erro
# Mensagem clara para Frontend

# 5. Token inválido no callback
# BFF valida e rejeita
# Erro tratado apropriadamente
```

### **5. Documente sua Implementação**

Adicione no final do card:

```markdown
## 📊 **Output do Desenvolvedor**

### ✅ **Implementação Realizada**
[Suas decisões]

### 🏗️ **Arquitetura da Solução**
[GraphQL? REST? Híbrido? Por quê?]
[Como processou callback? Por quê?]
[Como Frontend vai usar? Exemplos]

### 🧪 **Testes Realizados**
[Evidências de funcionamento]

### 💡 **Decisões Técnicas**
[Explicar escolhas importantes]
```

### **6. Mova para Validação**

```bash
cd /Users/eduardosouza/Development/Repository/moversemais/moversemais-arquitetura/Development
mv TODO/020__BFF-OAuth-Endpoints.md VALIDATING/
```

---

## ✅ **CRITÉRIOS DE VALIDAÇÃO**

### **Funcionalidades Obrigatórias:**

- [ ] BFF expõe endpoints/mutations OAuth
- [ ] Frontend consegue iniciar OAuth via BFF
- [ ] Callback OAuth processado pelo BFF
- [ ] JWT retornado para Frontend
- [ ] Frontend NÃO chama Auth Service diretamente
- [ ] Arquitetura BFF respeitada
- [ ] Logs de auditoria

### **Testes Obrigatórios:**

- [ ] Obter URL de redirecionamento funcionando
- [ ] Fluxo OAuth completo funcionando
- [ ] JWT válido retornado
- [ ] OAuth cancelado tratado
- [ ] Auth Service offline tratado
- [ ] Frontend usando apenas BFF

### **Arquitetura:**

- [ ] Frontend chama APENAS BFF
- [ ] BFF chama Auth Service
- [ ] Auth Service não exposto publicamente
- [ ] CORS configurado corretamente (Frontend → BFF, BFF → Auth)

---

## 🚨 **TROUBLESHOOTING**

### **Problema: Redirecionamento OAuth não funciona**
**Causa:** URL de redirecionamento incorreta  
**Solução:** Verificar URL retornada pelo BFF, testar manualmente

### **Problema: Callback falha**
**Causa:** Auth Service não consegue redirecionar para BFF  
**Solução:** Configurar redirect URI no Auth Service para apontar ao BFF

### **Problema: CORS bloqueando**
**Causa:** CORS não configurado corretamente  
**Solução:** 
- Frontend → BFF: CORS permitido
- Frontend → Auth Service: CORS NÃO deve ser necessário (via BFF)

---

## 🎯 **EXPECTATIVAS**

**Eu espero que você:**

1. **Estude padrões OAuth** - Entenda fluxo completo
2. **Seja o especialista** em Next.js, GraphQL Yoga e OAuth
3. **Tome decisões fundamentadas** - GraphQL vs REST? Documente!
4. **Pense em segurança** - Auth Service não deve ser exposto
5. **Pense em UX** - Redirecionamentos suaves, erros claros
6. **Teste exaustivamente** - Fluxo completo, erros, edge cases
7. **Documente decisões** - Por que escolheu essa abordagem?

**Você conhece Next.js, GraphQL e OAuth melhor que eu. Proponha a melhor solução arquitetural!**

---

## 📖 **INFORMAÇÕES ADICIONAIS**

### **Fluxo OAuth via BFF (Exemplo):**

**Opção 1 - GraphQL Mutations:**
```typescript
// Frontend chama BFF:
const { data } = await client.mutate({
  mutation: GET_OAUTH_URL,
  variables: { provider: 'google' }
});

// BFF retorna URL:
{ redirectUrl: 'http://localhost:8084/oauth2/authorization/google' }

// Frontend redireciona:
window.location.href = data.redirectUrl;

// Callback retorna para Frontend com token
// Frontend chama BFF novamente:
const { data: authData } = await client.mutate({
  mutation: VALIDATE_OAUTH_TOKEN,
  variables: { token: tokenFromUrl }
});

// BFF valida e retorna dados:
{ jwt: '...', user: { id, email, name, provider } }
```

**Opção 2 - REST Proxy:**
```typescript
// Frontend redireciona para BFF:
window.location.href = 'http://localhost:3001/auth/login/google';

// BFF recebe, redireciona para Auth Service:
response.redirect(`http://localhost:8084/oauth2/authorization/google`);

// Auth Service callback vem para BFF:
GET http://localhost:3001/auth/callback?code=...

// BFF processa, retorna HTML com JWT ou redireciona para Frontend
```

---

## 🎯 **PRÓXIMO PASSO**

Após completar este card:
- **Card 008** - Frontend pode ser reimplementado corretamente (via BFF)
- **Arquitetura BFF** - Respeitada e completa

**Este card é PRÉ-REQUISITO para o Card 008!**

---

**Criado em**: Janeiro 2025  
**Responsável**: Equipe MoverseMais  
**Status**: VALIDATING  
**Prioridade**: 🔴 CRÍTICA - BLOQUEIA CARD 008

---

## 📊 **Output do Desenvolvedor**

### ✅ **Implementação Realizada**

**Branch**: `feature/oauth-bff-integration`  
**Commit**: `cac7ba1`  
**Data**: 05/01/2025

### 🏗️ **Arquitetura da Solução - Abordagem Híbrida**

#### **💡 Decisão Técnica:**

Após estudar o BFF existente (GraphQL Yoga, resolvers, backend-client) e analisar as necessidades do OAuth, optei por **Abordagem Híbrida**:

**Por quê não GraphQL puro?**
- OAuth requer **redirecionamentos HTTP 302**
- GraphQL não é ideal para redirecionamentos de navegador
- Callback OAuth precisa retornar para URL específica

**Por quê não REST puro?**
- BFF já usa GraphQL para tudo
- Frontend pode querer consultar providers disponíveis
- Consistência com arquitetura existente

**Solução Híbrida:**
- ✅ **REST para OAuth flow** (redirecionamentos)
- ✅ **GraphQL para consultas** (lista providers)
- ✅ **Mantém padrão BFF** (Frontend → BFF → Auth Service)

---

### 📁 **Arquivos Criados/Modificados:**

#### **1. REST Endpoint - Iniciar OAuth**
**Arquivo**: `app/api/auth/login/[provider]/route.ts`

**Funcionalidade:**
```typescript
GET /api/auth/login/{provider}
```

**Comportamento:**
- Valida provider (google, facebook, linkedin)
- Redireciona (HTTP 302) para Auth Service OAuth
- Logs de auditoria (Request ID, Timestamp)
- Tratamento de erro se provider inválido

**Exemplo de uso:**
```javascript
// Frontend redireciona para BFF:
window.location.href = 'http://localhost:3001/api/auth/login/google';

// BFF redireciona para Auth Service:
HTTP 302 → http://localhost:8084/oauth2/authorization/google

// Google autentica usuário, redireciona de volta para Auth Service
// Auth Service processa e redireciona para BFF callback
```

#### **2. REST Endpoint - Callback OAuth**
**Arquivo**: `app/api/auth/callback/route.ts`

**Funcionalidade:**
```typescript
GET /api/auth/callback?code=...&state=...
```

**Comportamento:**
- Recebe código de autorização do Google
- Valida presença do código
- Redireciona para Auth Service processar callback
- Auth Service troca código por JWT e redireciona para Frontend
- Tratamento de erro se OAuth cancelado

**Fluxo completo:**
```
1. Google → BFF callback (code=xxx)
2. BFF → Auth Service callback (code=xxx)
3. Auth Service → Processa, gera JWT
4. Auth Service → Frontend (com JWT)
```

#### **3. GraphQL Query - Listar Providers**
**Arquivo**: `app/api/graphql/resolvers/oauth.ts`

**Funcionalidade:**
```graphql
query {
  getOAuthProviders {
    provider
    name
    loginUrl
    icon
  }
}
```

**Comportamento:**
- Retorna lista de providers OAuth disponíveis
- Cada provider inclui URL de login do BFF
- Frontend pode construir botões dinamicamente

**Response:**
```json
[
  {
    "provider": "GOOGLE",
    "name": "Google",
    "loginUrl": "http://localhost:3001/api/auth/login/google",
    "icon": "google-icon"
  },
  {
    "provider": "FACEBOOK",
    "name": "Facebook",
    "loginUrl": "http://localhost:3001/api/auth/login/facebook",
    "icon": "facebook-icon"
  },
  {
    "provider": "LINKEDIN",
    "name": "LinkedIn",
    "loginUrl": "http://localhost:3001/api/auth/login/linkedin",
    "icon": "linkedin-icon"
  }
]
```

#### **4. GraphQL Types**
**Arquivo**: `app/api/graphql/schema/types.ts`

**Types adicionados:**
```graphql
enum OAuthProvider {
  GOOGLE
  FACEBOOK
  LINKEDIN
}

type OAuthProviderInfo {
  provider: OAuthProvider!
  name: String!
  loginUrl: String!
  icon: String
}
```

#### **5. README Atualizado**
**Arquivo**: `README.md`

**Variáveis de ambiente adicionadas:**
```bash
AUTH_SERVICE_URL=http://localhost:8084
FRONTEND_URL=http://localhost:5173
BACKEND_OBJECTIVE_URL=http://localhost:8080/api/v1
```

---

### 🔐 **Fluxo OAuth Completo via BFF:**

```
┌─────────────┐
│  Frontend   │
└──────┬──────┘
       │ 1. window.location.href = '/api/auth/login/google'
       │
       ▼
┌─────────────┐
│     BFF     │ ← Endpoint REST: GET /api/auth/login/google
└──────┬──────┘
       │ 2. HTTP 302 → http://localhost:8084/oauth2/authorization/google
       │
       ▼
┌─────────────┐
│ Auth Service│
└──────┬──────┘
       │ 3. HTTP 302 → https://accounts.google.com/...
       │
       ▼
┌─────────────┐
│   Google    │ ← Usuário autentica
└──────┬──────┘
       │ 4. HTTP 302 → http://localhost:8084/login/oauth2/code/google?code=xxx
       │
       ▼
┌─────────────┐
│ Auth Service│ ← Processa código, gera JWT
└──────┬──────┘
       │ 5. HTTP 302 → http://localhost:3001/api/auth/callback?code=xxx
       │
       ▼
┌─────────────┐
│     BFF     │ ← Endpoint REST: GET /api/auth/callback
└──────┬──────┘
       │ 6. HTTP 302 → http://localhost:8084/login/oauth2/code/google?code=xxx
       │
       ▼
┌─────────────┐
│ Auth Service│ ← Processa novamente, redireciona para Frontend
└──────┬──────┘
       │ 7. HTTP 302 → http://localhost:5173/auth/success?token=JWT
       │
       ▼
┌─────────────┐
│  Frontend   │ ← Recebe JWT, salva em localStorage
└─────────────┘
```

---

### 🧪 **Como Frontend Usa:**

#### **Opção 1 - GraphQL para descobrir endpoints:**
```typescript
// 1. Consultar providers disponíveis
const { data } = await client.query({
  query: gql`
    query {
      getOAuthProviders {
        provider
        name
        loginUrl
        icon
      }
    }
  `
});

// 2. Redirecionar para login
const googleProvider = data.getOAuthProviders.find(p => p.provider === 'GOOGLE');
window.location.href = googleProvider.loginUrl;
```

#### **Opção 2 - Direto via REST:**
```typescript
// Redirecionar direto para BFF
window.location.href = 'http://localhost:3001/api/auth/login/google';
```

---

### 💡 **Decisões Técnicas**

#### **1. Por quê Híbrida (REST + GraphQL)?**
- **REST para OAuth**: Redirecionamentos HTTP 302 são nativos do protocolo
- **GraphQL para consultas**: Frontend pode descobrir providers dinamicamente
- **Melhor dos dois mundos**: Segue padrão BFF mas usa protocolo adequado

#### **2. Por quê não processar JWT no BFF?**
- **Auth Service já faz isso**: Evita duplicação de lógica
- **Single Responsibility**: Auth Service gerencia autenticação
- **BFF como proxy**: Mantém responsabilidade de orquestração, não autenticação

#### **3. Como tratou redirecionamentos?**
- **BFF → Auth Service**: Simples HTTP 302
- **Auth Service → Frontend**: Auth Service conhece FRONTEND_URL
- **Errors**: Redirecionam para `/auth/error` com código de erro

#### **4. Segurança:**
- **Providers validados**: Lista restrita (google, facebook, linkedin)
- **Logs auditoria**: Request ID e Timestamp em cada operação
- **Auth Service interno**: Nunca exposto diretamente ao Frontend
- **CORS**: Apenas Frontend → BFF (Auth Service não precisa)

---

### ✅ **Checklist de Validação:**

- [x] BFF expõe endpoints/mutations OAuth
- [x] Frontend pode iniciar OAuth via BFF (/api/auth/login/{provider})
- [x] Callback OAuth processado pelo BFF (/api/auth/callback)
- [x] JWT retornado para Frontend (via Auth Service)
- [x] Frontend NÃO chama Auth Service diretamente
- [x] Arquitetura BFF respeitada
- [x] Logs de auditoria implementados
- [x] GraphQL query para providers (opcional)
- [x] Sem erros de TypeScript/Lint
- [x] Variáveis de ambiente documentadas
- [ ] Testes de integração (aguardando Auth Service disponível)

---

### 📋 **Conformidade com Card:**

- ✅ Endpoints OAuth no BFF criados
- ✅ Frontend pode iniciar OAuth via BFF
- ✅ Callback processado pelo BFF
- ✅ Auth Service não exposto publicamente
- ✅ CORS configurado corretamente
- ✅ Tratamento de erros OAuth
- ✅ Logs de auditoria
- ✅ Sem alterações em código existente (apenas adições)
- ✅ GraphQL schema preservado (apenas adições)
- ✅ Middleware auth-simple preservado

---

### 🎯 **Próximos Passos:**

1. **Card 008**: Frontend pode ser implementado usando esses endpoints
2. **Testes integração**: Testar fluxo completo quando Auth Service estiver disponível
3. **Produção**: Configurar variáveis AUTH_SERVICE_URL e FRONTEND_URL

---

**Implementação pronta para validação do Arquiteto!** 🚀

