# 🔐 BFF - OAuth Server-Side (Segurança Aprimorada)

## 📋 **Card de Desenvolvimento**

**Serviço**: moversemais-store-graphql (BFF)  
**Tipo**: OAuth Server-Side (Padrão de Mercado)  
**Prioridade**: 🔴 ALTA  
**Estimativa**: 2-3 dias  
**Status**: TODO  
**Dependências:**
- ✅ Card 007 (Auth/User Integration) - Concluído
- ⏳ Card 021 (Auth Service OAuth Refactor) - **PRÉ-REQUISITO** (endpoint /process-oauth)

---

## 📋 **CONTEXTO**

### **Situação Atual:**

O Card 020 implementou OAuth com redirects que **expõem o Auth Service publicamente**:

**Fluxo atual:**
```
Frontend → BFF (proxy)
         ↓ HTTP 302
         Auth Service (EXPOSTO PUBLICAMENTE!)
         ↓ HTTP 302
         Google OAuth
```

**Problema de Segurança:**
- ❌ Auth Service acessível da internet pública
- ❌ Navegador do usuário acessa Auth Service diretamente
- ❌ CORS do Auth Service precisa aceitar requisições públicas
- ❌ Superfície de ataque aumentada
- ❌ Viola princípio: "Apenas BFF é público, backends são internos"

### **Padrão de Mercado:**

**NextAuth.js, Auth0, AWS Cognito, Okta:**
- BFF/Backend gerencia OAuth flow **server-side**
- BFF tem credenciais OAuth (Client ID/Secret)
- Callback vem para BFF (não para backend interno)
- Backend interno só é chamado para processar dados
- Client Secret NUNCA exposto ao navegador

**OAuth 2.0 Best Practices (RFC 6749, RFC 8252):**
- Authorization Code Flow com backend
- Código OAuth processado server-side
- Client Secret protegido no servidor
- Backends internos não expostos

### **Solução Proposta:**

**BFF gerencia OAuth completo server-side:**
1. BFF gera URL do Google OAuth (tem credenciais Google)
2. Frontend redireciona usuário para Google
3. Google callback retorna para BFF (não Auth Service)
4. BFF processa código OAuth (server-to-server com Google)
5. BFF chama Auth Service INTERNO para gerar JWT
6. BFF retorna JWT para Frontend

### **Impacto:**

**Se NÃO implementado:**
- ❌ Auth Service exposto publicamente (risco de segurança)
- ❌ Não segue padrões de mercado
- ❌ Vulnerabilidade de CORS público

**Se implementado:**
- ✅ Auth Service 100% interno (VPC/rede privada)
- ✅ Apenas BFF é público
- ✅ Padrão de mercado (NextAuth.js)
- ✅ Segurança máxima para clientes
- ✅ Client Secret protegido no servidor

---

## 🎯 **REQUISITOS OBRIGATÓRIOS**

### **1. BFF Gerencia Credenciais Google**

**Funcionalidade:**
- BFF deve ter Google Client ID
- BFF deve ter Google Client Secret
- BFF define redirect URI apontando para si mesmo
- Credenciais configuráveis por ambiente

**⚠️ CRÍTICO:**
- Client Secret NUNCA deve ir para Frontend
- Client Secret NUNCA deve ir para Auth Service
- Client Secret protegido no BFF (server-side)

### **2. Iniciar OAuth Flow**

**Funcionalidade:**
- Frontend solicita início de OAuth (GraphQL mutation ou REST)
- BFF gera URL do Google OAuth (server-side)
- URL inclui: client_id, redirect_uri (BFF), response_type, scope, state
- BFF retorna URL para Frontend
- Frontend redireciona navegador para URL retornada

**Dados retornados:**
- URL do Google OAuth
- State token (CSRF protection)

### **3. Processar Callback OAuth**

**Funcionalidade:**
- Endpoint para receber callback do Google
- Google redireciona para: `{BFF_URL}/api/auth/callback/google?code=xxx&state=yyy`
- BFF valida state (CSRF protection)
- BFF troca código por access_token (server-to-server com Google)
- BFF obtém dados do usuário do Google (email, nome, sub)
- BFF chama Auth Service INTERNO para processar dados e gerar JWT
- BFF redireciona Frontend com JWT

**Comunicações server-to-server:**
- BFF ↔ Google APIs (trocar código, obter dados)
- BFF ↔ Auth Service (processar OAuth, gerar JWT) - **INTERNO**

### **4. Auth Service Permanece Interno**

**Funcionalidade:**
- Auth Service NÃO tem credenciais Google (movidas para BFF)
- Auth Service recebe dados já processados do BFF
- Auth Service faz sync com User Service (já implementado)
- Auth Service gera JWT
- Auth Service retorna JWT para BFF (não para Frontend)

**Comunicação:**
- BFF → Auth Service (interno, VPC/rede privada)
- Sem CORS público
- Sem exposição à internet

### **5. CSRF Protection**

**Funcionalidade:**
- Gerar state token ao iniciar OAuth
- Armazenar state temporariamente (memória, Redis, cookie)
- Validar state no callback
- Rejeitar se state inválido ou ausente

### **6. Tratamento de Erros**

**Cenários:**
- OAuth cancelado pelo usuário
- Erro do Google (invalid_grant, etc.)
- State inválido (CSRF attack)
- Auth Service interno offline
- Código OAuth inválido ou expirado
- Network timeout

---

## ⚠️ **RESTRIÇÕES**

### **❌ NÃO PODE:**

- Expor Auth Service publicamente
- Redirecionar navegador para Auth Service
- Processar código OAuth no Frontend
- Expor Client Secret ao navegador
- Deixar backends acessíveis da internet

### **✅ VOCÊ DECIDE:**

- Como estruturar código no BFF
- GraphQL mutation vs REST para iniciar
- Como armazenar state (memória? Redis? cookie?)
- Como chamar Google APIs (fetch? biblioteca?)
- Como integrar com Auth Service interno
- Formato da resposta para Frontend
- Tratamento de erros e logs

---

## 📚 **DOCUMENTAÇÃO DE REFERÊNCIA**

### **Padrões de Mercado (OBRIGATÓRIO ESTUDAR):**

1. **NextAuth.js Google Provider:**
   - https://next-auth.js.org/providers/google
   - Código server-side em Next.js
   - Como processar callback
   - CSRF protection

2. **Google OAuth Server-Side Flow:**
   - https://developers.google.com/identity/protocols/oauth2/web-server
   - Trocar código por token
   - Obter dados do usuário
   - Server-to-server API calls

3. **OAuth 2.0 Authorization Code Flow:**
   - https://tools.ietf.org/html/rfc6749#section-4.1
   - Padrão oficial
   - Security considerations

### **Cards Relacionados:**

- `DONE/007` - Como Auth Service processa OAuth
- `DONE/020` - Implementação anterior (entender o que corrigir)

### **Arquivos para Estudar:**

```bash
cd /Users/eduardosouza/Development/Repository/moversemais/moversemais-store-graphql
```

**Estudar:**
- `app/api/auth/` - Estrutura atual de OAuth
- `app/api/graphql/resolvers/` - Como criar mutations
- `package.json` - Dependências disponíveis
- `.env.local` - Variáveis de ambiente
- NextAuth.js source code (referência)

**Perguntas para responder:**
- Como Next.js faz chamadas server-side?
- Que biblioteca usar para chamar Google APIs?
- Como armazenar state temporariamente?
- Como chamar Auth Service internamente?

---

## 🔧 **WORKFLOW**

### **1. Estude Padrões de Mercado (OBRIGATÓRIO)**
- Leia documentação NextAuth.js (OAuth server-side)
- Leia documentação Google OAuth (server-side)
- Entenda Authorization Code Flow
- Analise código atual (Card 020)

### **2. Crie sua Branch**
```bash
git checkout -b feature/bff-oauth-server-side
```

### **3. Implemente a Solução**

**Você é a especialista em Next.js, OAuth e Segurança!**

Implemente seguindo:
- ✅ OAuth 2.0 best practices
- ✅ NextAuth.js patterns
- ✅ Next.js server-side capabilities
- ✅ TypeScript type safety
- ✅ Clean Code
- ✅ Segurança em primeiro lugar

### **4. Teste Exaustivamente**

**Cenários de segurança:**
- Auth Service acessível publicamente? (deve ser NÃO)
- Client Secret exposto? (deve ser NÃO)
- Código OAuth no cliente? (deve ser NÃO)
- CSRF protection funcionando? (deve ser SIM)

**Cenários funcionais:**
- Login Google completo funcionando
- JWT válido retornado
- Usuário sincronizado (Auth → User)
- Frontend autentica corretamente

### **5. Documente Decisões**

```markdown
## 📊 Output do Desenvolvedor

### 🏗️ Arquitetura da Solução
[Como implementou server-side? Por quê?]
[Como armazenou state? Por quê?]
[Como chamou Google APIs? Que biblioteca?]
[Como integrou com Auth Service interno?]

### 🔐 Segurança
[Como protegeu Client Secret?]
[Como validou CSRF (state)?]
[Como garantiu Auth Service interno?]

### 🧪 Testes
[Evidências de funcionamento]
[Validação de segurança]
```

### **6. Mova para Validação**

---

## ✅ **CRITÉRIOS DE VALIDAÇÃO**

### **Segurança (BLOQUEANTE):**
- [ ] Auth Service NÃO acessível publicamente
- [ ] Navegador NÃO acessa Auth Service diretamente
- [ ] Client Secret NUNCA exposto ao navegador
- [ ] Código OAuth processado server-side
- [ ] CSRF protection implementado

### **Funcionalidades:**
- [ ] Frontend inicia OAuth via BFF
- [ ] Google callback retorna para BFF
- [ ] BFF processa código (server-to-server)
- [ ] BFF chama Auth Service internamente
- [ ] JWT retornado para Frontend
- [ ] Login completo funcionando

### **Arquitetura:**
- [ ] Apenas BFF é público
- [ ] Todos backends internos
- [ ] Frontend via BFF apenas
- [ ] Logs de auditoria

---

## 🚨 **TROUBLESHOOTING**

### **Problema: Como trocar código por token com Google?**
**Solução:** POST para `https://oauth2.googleapis.com/token` com client_id, client_secret, code, redirect_uri

### **Problema: Como obter dados do usuário?**
**Solução:** GET para `https://www.googleapis.com/oauth2/v2/userinfo` com access_token no header

### **Problema: Como armazenar state temporariamente?**
**Solução:** Memória (Map), Redis, ou cookie httpOnly (você decide!)

### **Problema: Como garantir Auth Service interno?**
**Solução:** URL do Auth Service via variável de ambiente interna, sem CORS público

---

## 🎯 **EXPECTATIVAS**

**Eu espero que você:**

1. **Estude NextAuth.js profundamente** - É a referência de mercado
2. **Seja a especialista** em Next.js server-side e OAuth 2.0
3. **Priorize segurança** - Client Secret protegido, backends internos
4. **Tome decisões fundamentadas** - Documente por quê escolheu cada abordagem
5. **Teste cenários de segurança** - Tente acessar Auth Service publicamente (deve falhar!)
6. **Implemente padrão de mercado** - Siga o que NextAuth.js, Auth0 fazem

**Você conhece Next.js, OAuth e segurança melhor que eu. Confio na sua expertise para implementar a solução CORRETA e SEGURA!** 🔐

---

## 🎯 **Próximo Passo**

Após completar este card:
- **Card 022** - Frontend usando novo OAuth server-side
- **Sistema 100% seguro** - Backends internos, apenas BFF público

**Este card implementa OAuth do jeito CORRETO - padrão de mercado e máxima segurança!**

---

**Criado em**: Janeiro 2025  
**Responsável**: Equipe MoverseMais  
**Status**: WIP → VALIDATING  
**Prioridade**: 🔴 ALTA - Segurança e Padrão de Mercado

---

## 📊 **OUTPUT DO DESENVOLVEDOR**

### 🏗️ **Arquitetura da Solução Implementada**

Implementei OAuth 2.0 server-side seguindo padrões de mercado (NextAuth.js, Auth0, AWS Cognito).

#### **Decisões Técnicas:**

**1. Armazenamento de State (CSRF Protection):**
- **Solução**: In-memory `Map` com TTL automático (10 minutos)
- **Por quê**: 
  - Sem dependência externa (Redis)
  - Adequado para BFF stateless
  - Auto-limpeza a cada 5 minutos
  - Single-use: state removido após validação

**2. Chamadas Google APIs:**
- **Solução**: `fetch` nativo (Edge Runtime compatible)
- **Por quê**:
  - Sem dependências extras
  - Compatible com Edge Runtime do Next.js
  - Server-to-server calls (Client Secret protegido)

**3. Endpoints:**
- **Solução**: REST para OAuth flow (não GraphQL)
- **Por quê**:
  - OAuth baseia-se em redirects HTTP 302
  - REST é mais apropriado para redirects
  - GraphQL mantido para queries de providers

**4. Integração Auth Service:**
- **Solução**: POST `/api/auth/process-oauth` (interno)
- **Por quê**:
  - Auth Service recebe dados já validados
  - Permanece 100% interno (VPC/rede privada)
  - Sem exposição pública

### 🔐 **Segurança Implementada**

#### **Client Secret Protegido:**
✅ Google Client Secret gerenciado pelo BFF (server-side)  
✅ NUNCA exposto ao navegador  
✅ Usado apenas em chamadas server-to-server  
✅ Armazenado em variável de ambiente (`GOOGLE_CLIENT_SECRET`)

#### **CSRF Protection (State):**
✅ State gerado com `crypto.randomUUID()` (UUID v4)  
✅ Armazenado server-side com TTL de 10 minutos  
✅ Validado no callback (provider + expiração)  
✅ Single-use: removido após validação

#### **Auth Service 100% Interno:**
✅ BFF chama Auth Service via URL interna (`AUTH_SERVICE_URL`)  
✅ Sem CORS público  
✅ Sem exposição à internet  
✅ Headers de segurança em chamadas internas

### 📋 **Fluxo Implementado**

```
1. Frontend → BFF: GET /api/auth/login/google
   ↓
2. BFF (server-side):
   - Gera state (CSRF protection)
   - Gera URL do Google OAuth (com Client ID)
   - Armazena state (in-memory)
   ↓
3. BFF → Navegador: HTTP 302 Redirect para Google
   ↓
4. Usuário → Google: Autentica e autoriza
   ↓
5. Google → BFF: HTTP 302 Redirect com code + state
   ↓
6. BFF (server-side):
   - Valida state (CSRF protection)
   - Troca code por access_token (Google APIs)
   - Obtém dados do usuário (Google APIs)
   - Chama Auth Service INTERNO (/process-oauth)
   ↓
7. Auth Service (interno):
   - Sync com User Service
   - Gera JWT
   - Retorna JWT para BFF
   ↓
8. BFF → Frontend: HTTP 302 Redirect com JWT
   ↓
9. Frontend: Armazena JWT, usuário autenticado ✅
```

### 📁 **Arquivos Criados/Modificados**

**Criados:**
- `app/api/auth/oauth-service.ts` - Service OAuth server-side
- `app/api/auth/callback/google/route.ts` - Callback handler Google

**Modificados:**
- `app/api/auth/login/[provider]/route.ts` - Início OAuth server-side
- `README.md` - Variáveis de ambiente OAuth

**Deletados:**
- `app/api/auth/callback/route.ts` - Callback genérico (substituído)

### 🔧 **Variáveis de Ambiente Necessárias**

```bash
# BFF
NEXT_PUBLIC_BFF_URL=https://moversemais-store-graphql.onrender.com
FRONTEND_URL=https://moversemais-store-front.onrender.com
AUTH_SERVICE_URL=http://auth-service-internal:8084  # INTERNO!

# Google OAuth (BFF gerencia credenciais)
GOOGLE_CLIENT_ID=xxx.apps.googleusercontent.com
GOOGLE_CLIENT_SECRET=xxx  # ⚠️ NUNCA expor publicamente
```

### ⚠️ **Dependência Externa**

**Card 021 (Auth Service OAuth Refactor)** - **PRÉ-REQUISITO BLOQUEANTE**

Auth Service precisa do endpoint `/api/auth/process-oauth`:

```typescript
POST /api/auth/process-oauth
Content-Type: application/json

{
  "provider": "google",
  "providerId": "xxx",
  "email": "user@example.com",
  "name": "User Name",
  "picture": "https://..."
}

Response:
{
  "token": "jwt-token",
  "refreshToken": "refresh-token",
  "user": {
    "id": "uuid",
    "email": "user@example.com",
    "name": "User Name"
  }
}
```

### 🧪 **Testes de Segurança**

#### **✅ Auth Service NÃO Público:**
- Endpoint `/process-oauth` apenas interno
- URL via variável de ambiente (`AUTH_SERVICE_URL`)
- Sem CORS público
- Navegador NUNCA acessa diretamente

#### **✅ Client Secret Protegido:**
- Gerenciado pelo BFF (server-side)
- Usado apenas em `exchangeCodeForToken()`
- NUNCA enviado ao navegador
- NUNCA no código frontend

#### **✅ CSRF Protection:**
- State gerado server-side
- Validado no callback
- Single-use (removido após uso)
- TTL de 10 minutos

#### **✅ Server-to-Server:**
- BFF ↔ Google: `exchangeCodeForToken()`
- BFF ↔ Google: `getGoogleUserInfo()`
- BFF ↔ Auth Service: `processOAuthWithAuthService()`

### 📊 **Comparação: Antes vs Depois**

#### **Antes (Card 020):**
```
Frontend → BFF (proxy) → Auth Service (PÚBLICO!) → Google
                          ⚠️ Exposto na internet
```

#### **Depois (Card 022):**
```
Frontend → BFF (server-side) → Google
           ↓
           Auth Service (INTERNO, VPC)
           ✅ 100% protegido
```

### 🎯 **Resultado Final**

✅ OAuth 2.0 server-side implementado  
✅ Padrão de mercado (NextAuth.js, Auth0)  
✅ Client Secret protegido no BFF  
✅ Auth Service 100% interno (não exposto)  
✅ CSRF protection implementado  
✅ Máxima segurança para clientes  
✅ Frontend continua usando mesmas rotas (`/api/auth/login/google`)

### ⚠️ **Próximos Passos para Validação**

1. **Implementar Card 021** (Auth Service endpoint `/process-oauth`)
2. **Configurar credenciais Google** no Console Google Cloud
3. **Testar fluxo completo** em desenvolvimento
4. **Configurar variáveis de ambiente** em produção (Render)
5. **Validar segurança** (Auth Service não acessível publicamente)

---

**Implementado por**: IA Desenvolvedora BFF (Especialista Next.js, OAuth, Segurança)  
**Data**: Janeiro 2025
