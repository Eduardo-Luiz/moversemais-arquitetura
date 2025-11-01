# 🎯 Card 025 - BFF: Mutation registerLead

**Agente Responsável:** Gabriela  
**Microserviço:** moversemais-store-graphql  
**Prioridade:** Alta  
**Status:** TODO  
**Estimativa:** 0.5 dia

---

## 📋 CONTEXTO

### **Situação Atual**
O User Service implementou sistema de gestão de leads (Cards 026 e 029). Endpoint `POST /api/v1/leads` está pronto e **protegido** por `X-Internal-Service-Key` (apenas BFF pode chamar). Frontend precisa de mutation GraphQL para registrar leads.

### **Problema Identificado**
- Frontend não tem como chamar User Service diretamente (violaria arquitetura BFF)
- Não existe mutation GraphQL para registro de leads
- Padrão de comunicação com User Service já existe (oauth-service.ts usa X-Internal-Service-Key)

### **Padrão Existente no BFF**

**O BFF JÁ USA X-Internal-Service-Key em:**
- `app/api/auth/oauth-service.ts` (linha 252-259)
- Chamadas para Auth Service já enviam header de segurança

**Exemplo do código existente:**
```typescript
const internalServiceKey = process.env.INTERNAL_SERVICE_KEY || ''

const response = await fetch(`${authServiceUrl}/api/v1/auth/process-oauth`, {
  method: 'POST',
  headers: {
    'Content-Type': 'application/json',
    'X-Internal-Service-Key': internalServiceKey,
    'X-Request-Id': crypto.randomUUID(),
    'X-Timestamp': new Date().toISOString(),
  },
  body: JSON.stringify(data)
})
```

**Você deve seguir esse padrão!**

### **Solução Proposta**
Criar mutation GraphQL `registerLead` que chama User Service usando o mesmo padrão de segurança de `oauth-service.ts`.

### **Onde se Encaixa na Arquitetura**
```
Frontend (Formulário de Lead)
    ↓
BFF GraphQL - VOCÊ (Mutation registerLead) ← ESTE CARD
    ↓ (envia X-Internal-Service-Key)
User Service (POST /api/v1/leads - protegido)
    ↓
PostgreSQL
```

### **Impacto se Não For Feito**
- Frontend não consegue registrar leads
- Sistema de captura de leads incompleto
- Impossível lançar assessment com captura de interesse

---

## 🎯 REQUISITOS OBRIGATÓRIOS

### **1. GraphQL Schema**

**Adicionar ao schema (`app/api/graphql/schema/types.ts`):**

Nova mutation `registerLead` com:
- Input: email (String!), optInPlatformNews (Boolean!), optInBlogNews (Boolean!)
- Output: LeadResponse (success, message, leadId)

Tipo LeadResponse com:
- success: Boolean!
- message: String!
- leadId: ID (nullable)

### **2. Resolver**

**Criar/atualizar resolver para leads:**

Responsabilidades:
- Validar formato básico de email no BFF
- Chamar User Service `POST /api/v1/leads`
- **OBRIGATÓRIO:** Enviar header `X-Internal-Service-Key` (seguir padrão de oauth-service.ts)
- Body: { email, optInPlatformNews, optInBlogNews, source: "ASSESSMENT" }
- Tratar resposta do User Service (201, 200, 400, 500)
- Retornar LeadResponse para Frontend

**Source fixo:**
- Sempre enviar `source: "ASSESSMENT"` (decisão de negócio)
- Frontend não controla source

### **3. Chamada HTTP ao User Service**

**Padrão Obrigatório (seguir oauth-service.ts):**

```typescript
const internalServiceKey = process.env.INTERNAL_SERVICE_KEY || ''
const userServiceUrl = process.env.USER_SERVICE_URL || 'http://localhost:8083'

const response = await fetch(`${userServiceUrl}/api/v1/leads`, {
  method: 'POST',
  headers: {
    'Content-Type': 'application/json',
    'X-Internal-Service-Key': internalServiceKey,  // ← OBRIGATÓRIO
    'X-Request-Id': crypto.randomUUID(),
    'X-Timestamp': new Date().toISOString(),
  },
  body: JSON.stringify({
    email: email,
    optInPlatformNews: optInPlatformNews,
    optInBlogNews: optInBlogNews,
    source: 'ASSESSMENT'
  })
})
```

### **4. Variáveis de Ambiente**

**Verificar se existe em `.env.local` (ou criar):**
```bash
USER_SERVICE_URL=http://localhost:8083
INTERNAL_SERVICE_KEY=dev-internal-key-change-in-production
```

**Produção (Render.com):**
```bash
USER_SERVICE_URL=http://moversemais-user:8083
INTERNAL_SERVICE_KEY=<secret-forte>
```

### **5. Tratamento de Erros**

**Cenários:**
- User Service indisponível: success=false, mensagem amigável
- Email inválido (400): success=false, mensagem de validação
- Email duplicado (200): success=true (idempotente)
- Erro 500: success=false, mensagem genérica
- API Key inválido (401): logar erro crítico, mensagem genérica

**Não expor detalhes técnicos ao frontend.**

### **6. Logs**

**Logs Estruturados:**
```typescript
console.log('🔍 [BFF-LEAD] Registrando lead: ${emailMasked}')
console.log('✅ [BFF-LEAD] Lead registrado: ${leadId}')
console.error('❌ [BFF-LEAD] Erro ao registrar lead: ${error}')
```

**Mascarar email em logs:**
- `user@example.com` → `u***@example.com`

---

## ⚠️ RESTRIÇÕES

### **O que NÃO PODE ser alterado:**

1. ❌ **NÃO quebrar schema GraphQL existente** (objectives, assessments)
2. ❌ **NÃO alterar resolvers existentes** (objectives, assessment)
3. ❌ **NÃO alterar backend-client.ts** sem necessidade
4. ❌ **NÃO alterar oauth-service.ts** (apenas usar como referência)
5. ❌ **NÃO exigir autenticação** para esta mutation (lead é anônimo)

### **O que DEVE ser preservado:**

1. ✅ **Padrão de segurança** (X-Internal-Service-Key igual oauth-service.ts)
2. ✅ **Padrão de resolvers** existente (estrutura de arquivos)
3. ✅ **Padrão de logs** estruturados
4. ✅ **Padrão de tratamento de erros**
5. ✅ **Variáveis de ambiente** (USER_SERVICE_URL, INTERNAL_SERVICE_KEY)

---

## 📚 DOCUMENTAÇÃO DE REFERÊNCIA

### **Arquivos para Estudar (OBRIGATÓRIO):**

1. **Padrão de Segurança com X-Internal-Service-Key:**
   - `app/api/auth/oauth-service.ts` (linhas 240-265) - **ESTUDAR ESTE PADRÃO**
   - Veja como usa `INTERNAL_SERVICE_KEY`
   - Veja estrutura de headers
   - Veja tratamento de erros

2. **Schema GraphQL Existente:**
   - `app/api/graphql/schema/types.ts` - Como outros tipos estão definidos
   - Procurar por Response types existentes

3. **Resolvers Existentes:**
   - `app/api/graphql/resolvers/objectives.ts` - Padrão de resolvers
   - `app/api/graphql/resolvers/assessment.ts` - Padrão de mutations
   - `app/api/graphql/resolvers/index.ts` - Como registrar resolvers

4. **Services (se existir padrão):**
   - `app/api/graphql/services/objectives-service.ts` - Padrão de service layer

5. **Documentação:**
   - `../moversemais-store-graphql/AGENTS.md` - Políticas do BFF
   - `../moversemais-store-graphql/README.md` - Configuração de variáveis
   - `../moversemais-arquitetura/AGENTS.md` - Visão geral

### **Cards Relacionados:**
- Card 026: User Service Lead Management ✅ (DONE)
- Card 029: User Service Protect Leads ✅ (VALIDATING)
- Card 024: Frontend Formulário de Lead (depende deste card)

### **Referência de Padrão:**
- **oauth-service.ts é sua referência principal** para chamadas ao User Service com X-Internal-Service-Key

---

## 🔧 WORKFLOW

### **1. ESTUDAR (OBRIGATÓRIO - 30 minutos)**

```bash
cd moversemais-store-graphql

# CRÍTICO: Estudar padrão de X-Internal-Service-Key
cat app/api/auth/oauth-service.ts | grep -A 20 "X-Internal-Service-Key"

# Estudar schema GraphQL
cat app/api/graphql/schema/types.ts | grep -A 10 "type.*Response"

# Estudar resolvers
ls -la app/api/graphql/resolvers/
cat app/api/graphql/resolvers/objectives.ts | head -50
cat app/api/graphql/resolvers/assessment.ts | head -50

# Ver como resolvers são registrados
cat app/api/graphql/resolvers/index.ts

# Verificar variáveis de ambiente
cat .env.local | grep -E "USER_SERVICE|INTERNAL_SERVICE"
cat README.md | grep -E "USER_SERVICE|INTERNAL_SERVICE"

# Ler AGENTS.md
cat AGENTS.md
```

**Perguntas para Responder Antes de Implementar:**
- Como oauth-service.ts usa X-Internal-Service-Key?
- Onde USER_SERVICE_URL está configurado?
- Como tipos Response estão estruturados no schema?
- Onde resolvers são registrados?
- Há padrão de service layer ou resolver chama backend direto?

### **2. CRIAR BRANCH**

```bash
git checkout -b feature/bff-lead-mutation
```

### **3. IMPLEMENTAR (VOCÊ DECIDE COMO)**

**Ordem Sugerida (mas você decide):**
1. Adicionar tipos GraphQL (LeadResponse, RegisterLeadInput) em `schema/types.ts`
2. Criar função de chamada ao User Service (pode ser inline no resolver ou service separado)
3. Criar resolver `registerLead` em `resolvers/` (arquivo novo ou existente)
4. Registrar resolver em `resolvers/index.ts`
5. Testar via GraphQL Playground

**Você decide:**
- Estrutura exata dos tipos GraphQL
- Nome do arquivo do resolver (leads.ts novo ou em objectives.ts existente)
- Se cria service layer ou faz inline no resolver
- Validações adicionais
- Mensagens de erro amigáveis
- Estrutura de logs

**Mas DEVE seguir:**
- ✅ Padrão de X-Internal-Service-Key de oauth-service.ts
- ✅ Estrutura de headers igual (X-Request-Id, X-Timestamp)

### **4. TESTAR**

**Testes via GraphQL Playground:**

```bash
# Acessar playground
open http://localhost:3001/api/graphql

# Teste 1: Registrar lead com sucesso
mutation {
  registerLead(
    email: "teste@moversemais.com"
    optInPlatformNews: true
    optInBlogNews: true
  ) {
    success
    message
    leadId
  }
}
# Esperado: success=true, leadId=UUID

# Teste 2: Email inválido
mutation {
  registerLead(
    email: "email-invalido"
    optInPlatformNews: true
    optInBlogNews: false
  ) {
    success
    message
    leadId
  }
}
# Esperado: success=false, mensagem de erro

# Teste 3: Email duplicado (idempotência)
mutation {
  registerLead(
    email: "teste@moversemais.com"
    optInPlatformNews: false
    optInBlogNews: true
  ) {
    success
    message
    leadId
  }
}
# Esperado: success=true (idempotente)
```

**Verificações Obrigatórias:**
- [ ] Mutation funciona no GraphQL Playground
- [ ] Header X-Internal-Service-Key sendo enviado ao User Service
- [ ] User Service aceita requisição (não retorna 401)
- [ ] Source "ASSESSMENT" enviado automaticamente
- [ ] Tratamento de erros funciona
- [ ] Logs estruturados aparecem

**Teste de Segurança:**
- [ ] Verificar logs do User Service
- [ ] Deve mostrar: `INFO [SECURITY-FILTER] API Key válida - acesso autorizado: /api/v1/leads`
- [ ] Se mostrar 401 ou WARN, X-Internal-Service-Key está incorreto

### **5. DOCUMENTAR DECISÕES**

Ao final do card, documente:
- Estrutura de tipos GraphQL escolhida
- Onde colocou o resolver (arquivo novo ou existente)
- Se criou service layer ou fez inline
- Como implementou chamada ao User Service
- Padrão de oauth-service.ts seguido
- Validações adicionadas
- Tratamento de erros implementado
- Dificuldades encontradas

### **6. COMMIT E PUSH**

```bash
git add .
git commit -m "feat(bff): adiciona mutation registerLead

- Nova mutation GraphQL para registro de leads
- Chama User Service POST /api/v1/leads
- Segue padrão oauth-service.ts (X-Internal-Service-Key)
- Headers: X-Internal-Service-Key, X-Request-Id, X-Timestamp
- Source fixo 'ASSESSMENT'
- Tratamento de erros robusto
- Logs estruturados"

git push origin feature/bff-lead-mutation
```

### **7. MOVER PARA VALIDAÇÃO**

```bash
mv Development/TODO/025__BFF-Mutation-Register-Lead.md \
   Development/VALIDATING/025__BFF-Mutation-Register-Lead.md
```

---

## ✅ CRITÉRIOS DE VALIDAÇÃO

### **Funcionalidades:**
- [ ] Mutation `registerLead` disponível no schema GraphQL
- [ ] GraphQL Playground exibe mutation corretamente
- [ ] Email válido: success=true
- [ ] Email inválido: success=false, mensagem clara
- [ ] Email duplicado: success=true (idempotente)
- [ ] leadId retornado quando sucesso

### **Integração:**
- [ ] Chama User Service corretamente
- [ ] URL: `${USER_SERVICE_URL}/api/v1/leads`
- [ ] Method: POST
- [ ] Headers enviados:
  - Content-Type: application/json
  - **X-Internal-Service-Key** (do env var)
  - X-Request-Id (UUID gerado)
  - X-Timestamp (ISO timestamp)
- [ ] Body enviado: { email, optInPlatformNews, optInBlogNews, source: "ASSESSMENT" }

### **Segurança:**
- [ ] X-Internal-Service-Key sendo enviado
- [ ] User Service aceita (não retorna 401)
- [ ] API Key vem de variável de ambiente
- [ ] API Key nunca aparece em logs

### **Qualidade:**
- [ ] Logs estruturados presentes
- [ ] Email mascarado em logs (u***@example.com)
- [ ] Tratamento de erros robusto
- [ ] Código segue padrões do BFF
- [ ] Sem quebra de schema existente

---

## 🚨 TROUBLESHOOTING

### **Problema: User Service retorna 401 Unauthorized**
**Solução:**
- Verificar se `X-Internal-Service-Key` está sendo enviado
- Verificar valor de `INTERNAL_SERVICE_KEY` em .env.local
- Deve ser: `dev-internal-key-change-in-production`
- Verificar logs do User Service: deve mostrar `API Key válida`

### **Problema: INTERNAL_SERVICE_KEY undefined**
**Solução:**
- Verificar `.env.local` tem a variável
- Reiniciar servidor BFF após adicionar variável
- Verificar README.md para ver configuração esperada

### **Problema: Mutation não aparece no Playground**
**Solução:**
- Verificar se tipo foi adicionado ao schema principal
- Verificar se resolver foi registrado em `resolvers/index.ts`
- Reiniciar servidor BFF
- Limpar cache do navegador

### **Problema: User Service não responde**
**Solução:**
- Verificar se USER_SERVICE_URL está correto
- Verificar se User Service está rodando (porta 8083)
- Verificar logs do User Service

### **Problema: CORS error**
**Solução:**
- Não deve acontecer (mutation é server-side no BFF)
- Se acontecer, verificar configuração CORS

---

## 🎯 EXPECTATIVAS

### **Você é a Guardiã da Arquitetura BFF**

**Gabriela, você domina Next.js, GraphQL e TypeScript.** Eu confio que você:

1. ✅ **Conhece GraphQL** melhor que eu
2. ✅ **Conhece Next.js** melhor que eu
3. ✅ **Conhece TypeScript** melhor que eu
4. ✅ **Sabe orquestrar microserviços** melhor que eu

**Por isso:**
- Estude `oauth-service.ts` profundamente (é sua referência principal)
- Siga o padrão de X-Internal-Service-Key estabelecido
- Estude schema e resolvers existentes
- Tome decisões técnicas fundamentadas
- Adicione validações que julgar necessárias

**Eu defini O QUE precisa ser feito. Você decide COMO fazer.**

**Mas OBRIGATÓRIO:**
- Seguir padrão de oauth-service.ts para headers
- Enviar X-Internal-Service-Key (sem isso, User Service bloqueia com 401)

---

## 📊 OUTPUT ESPERADO

Ao finalizar, documente aqui:

### **Decisões Técnicas Tomadas:**
(Você preenche)

### **Estrutura Criada:**
(Liste arquivos criados/modificados)

### **Padrão Seguido:**
(Como seguiu oauth-service.ts)

### **Testes Realizados:**
(Liste cenários testados)

### **Dificuldades Encontradas:**
(Se houver)

### **Melhorias Implementadas:**
(Além do requisitado)

---

**Data de Criação:** 01/11/2025  
**Recriado por:** Arquiobaldo (após estudar código existente)  
**Versão:** 2.0 (reescrito após falha arquitetural)

