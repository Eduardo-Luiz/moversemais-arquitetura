# 🎯 Card 025 - BFF: Mutation registerLead

**Agente Responsável:** Gabriela  
**Microserviço:** moversemais-store-graphql  
**Prioridade:** Alta  
**Status:** TODO  
**Estimativa:** 0.5 dia

---

## 📋 CONTEXTO

### **Situação Atual**
O User Service (Card 026) implementou sistema de gestão de leads com endpoint `POST /api/v1/leads`. Frontend precisa de uma mutation GraphQL para registrar leads de forma simples e consistente com a arquitetura BFF.

### **Problema Identificado**
- Frontend não tem como chamar User Service diretamente (viola arquitetura)
- BFF ainda não expõe funcionalidade de registro de leads
- Mutation GraphQL necessária para manter padrão de comunicação

### **Solução Proposta**
Criar mutation GraphQL `registerLead` no BFF que orquestra chamada ao User Service, mantendo a camada de abstração e seguindo padrões existentes do projeto.

### **Onde se Encaixa na Arquitetura**
```
Frontend (Formulário de Lead)
    ↓
BFF GraphQL - VOCÊ (Mutation registerLead) ← ESTE CARD
    ↓
User Service (POST /api/v1/leads)
    ↓
PostgreSQL
```

### **Impacto se Não For Feito**
- Frontend não consegue registrar leads
- Quebra de arquitetura se Frontend chamar User Service direto
- Impossível capturar interesse de usuários

---

## 🎯 REQUISITOS OBRIGATÓRIOS

### **1. GraphQL Schema**

**Nova Mutation:**
- Nome: `registerLead`
- Input: email (String!), optInPlatformNews (Boolean!), optInBlogNews (Boolean!)
- Output: LeadResponse (success, message, leadId)

**Tipo de Resposta:**
- success: Boolean (true/false)
- message: String (mensagem amigável)
- leadId: ID nullable (UUID do lead se criado)

### **2. Resolver**

**Responsabilidades:**
- Validar email no lado do BFF (formato básico)
- Chamar User Service `POST /api/v1/leads`
- Body: { email, optInPlatformNews, optInBlogNews, source: "ASSESSMENT" }
- Headers: Content-Type, X-Request-Id, X-Timestamp (padrão de segurança)
- Tratar resposta do User Service
- Retornar LeadResponse para Frontend

**Tratamento de Erros:**
- User Service indisponível: Retornar success=false, mensagem amigável
- Email inválido: Retornar success=false, mensagem de validação
- Duplicate email: Retornar success=true (idempotente - UX)
- Qualquer outro erro: Retornar success=false, mensagem genérica

### **3. Orquestração**

**Chamar User Service via HTTP:**
- URL: `${USER_SERVICE_URL}/api/v1/leads`
- Method: POST
- Headers obrigatórios:
  - Content-Type: application/json
  - X-Request-Id: UUID gerado
  - X-Timestamp: ISO timestamp
- Body: JSON com email, optInPlatformNews, optInBlogNews, source

**Source Fixo:**
- Sempre enviar `source: "ASSESSMENT"` para User Service
- Frontend não controla source (decisão de negócio)

### **4. Logs**

**Logs Estruturados:**
- Início: "🔍 [BFF-LEAD] Registrando lead: ${emailHash}"
- Sucesso: "✅ [BFF-LEAD] Lead registrado: ${leadId}"
- Erro: "❌ [BFF-LEAD] Erro ao registrar lead: ${error}"

**Não Expor Email em Logs:**
- Usar hash ou mascarar email (ex: "t***@moversemais.com")

---

## ⚠️ RESTRIÇÕES

### **O que NÃO PODE ser alterado:**

1. ❌ **NÃO quebrar schema GraphQL existente**
2. ❌ **NÃO alterar resolvers de objectives ou assessments**
3. ❌ **NÃO alterar tratamento de erros existente** (processBackendError)
4. ❌ **NÃO expor User Service diretamente** ao Frontend
5. ❌ **NÃO exigir autenticação** para esta mutation (lead é anônimo)

### **O que DEVE ser preservado:**

1. ✅ **Padrão de resolvers** existente
2. ✅ **Padrão de chamadas HTTP** para backends
3. ✅ **Padrão de headers de segurança** (X-Request-Id, X-Timestamp)
4. ✅ **Padrão de tratamento de erros**
5. ✅ **Padrão de logs estruturados**

---

## 📚 DOCUMENTAÇÃO DE REFERÊNCIA

### **Arquivos para Estudar (OBRIGATÓRIO):**

1. **Schema GraphQL Existente:**
   - `app/api/graphql/schema/types.ts` - Tipos GraphQL
   - Estudar como outros tipos de Response estão estruturados

2. **Resolvers Existentes:**
   - `app/api/graphql/resolvers/` - Padrão de resolvers
   - Estudar como objectives ou assessments chamam backend

3. **Backend Client:**
   - Verificar como BFF chama outros microserviços via HTTP
   - Estudar padrão de headers e tratamento de erros

4. **Configuração:**
   - `.env` ou configuração de URLs de serviços
   - Verificar como `USER_SERVICE_URL` está configurado

5. **Documentação:**
   - `../moversemais-store-graphql/AGENTS.md` - Políticas do BFF
   - `../moversemais-arquitetura/AGENTS.md` - Visão geral

### **Cards Relacionados:**
- Card 026: User Service Lead Management (pré-requisito - DEVE estar pronto)
- Card 024: Frontend Formulário de Lead (depende deste card)

---

## 🔧 WORKFLOW

### **1. ESTUDAR (OBRIGATÓRIO - Antes de Codificar)**

```bash
# Estudar estrutura do BFF
cd moversemais-store-graphql
tree app/api/graphql/

# Analisar schema existente
cat app/api/graphql/schema/types.ts

# Analisar resolvers existentes
ls -la app/api/graphql/resolvers/
# Escolher um resolver similar para estudar padrão

# Verificar configuração de serviços
cat .env.local
# ou verificar onde USER_SERVICE_URL está definido

# Ler AGENTS.md
cat AGENTS.md
```

**Perguntas para Responder Antes de Implementar:**
- Onde ficam os tipos GraphQL?
- Qual padrão de nomenclatura de resolvers?
- Como outros resolvers chamam backends via HTTP?
- Qual padrão de tratamento de erros?
- Como gerar X-Request-Id e X-Timestamp?

### **2. CRIAR BRANCH**

```bash
git checkout -b feature/bff-lead-mutation
```

### **3. IMPLEMENTAR (VOCÊ DECIDE COMO)**

**Ordem Sugerida (mas você decide):**
1. Adicionar tipos GraphQL (LeadResponse, RegisterLeadInput)
2. Criar resolver registerLead
3. Implementar chamada HTTP ao User Service
4. Adicionar tratamento de erros
5. Adicionar logs estruturados
6. Testar via GraphQL Playground

**Você decide:**
- Estrutura exata dos tipos GraphQL
- Nome exato dos arquivos
- Organização de código
- Validações adicionais
- Mensagens de erro amigáveis
- Estrutura de logs

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

# Teste 3: User Service indisponível
# Parar User Service temporariamente
mutation {
  registerLead(
    email: "teste2@moversemais.com"
    optInPlatformNews: false
    optInBlogNews: true
  ) {
    success
    message
    leadId
  }
}
# Esperado: success=false, mensagem amigável
```

**Verificações Obrigatórias:**
- [ ] Mutation funciona no GraphQL Playground
- [ ] Headers X-Request-Id e X-Timestamp sendo enviados
- [ ] User Service recebe dados corretamente
- [ ] Source "ASSESSMENT" enviado automaticamente
- [ ] Tratamento de erros funciona
- [ ] Logs estruturados aparecem

### **5. DOCUMENTAR DECISÕES**

Ao final do card, documente:
- Estrutura de tipos GraphQL escolhida
- Como implementou chamada HTTP
- Validações adicionadas
- Tratamento de erros implementado
- Dificuldades encontradas

### **6. COMMIT E PUSH**

```bash
git add .
git commit -m "feat(bff): adiciona mutation registerLead

- Nova mutation GraphQL para registro de leads
- Orquestra chamada ao User Service
- Headers de segurança (X-Request-Id, X-Timestamp)
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
- [ ] Email válido: sucesso
- [ ] Email inválido: erro amigável
- [ ] User Service indisponível: erro amigável (não quebra)
- [ ] Duplicate email: retorna sucesso (idempotente)
- [ ] leadId retornado quando sucesso

### **Integração:**
- [ ] Chama User Service corretamente
- [ ] Body enviado com todos campos (email, optInPlatformNews, optInBlogNews, source)
- [ ] Source fixo "ASSESSMENT" enviado
- [ ] Headers X-Request-Id e X-Timestamp enviados

### **Qualidade:**
- [ ] Logs estruturados presentes
- [ ] Email não exposto em logs (mascarado ou hash)
- [ ] Tratamento de erros robusto
- [ ] Código segue padrões do BFF
- [ ] Sem quebra de schema existente

---

## 🚨 TROUBLESHOOTING

### **Problema: Mutation não aparece no Playground**
**Solução:**
- Verificar se tipo foi adicionado ao schema principal
- Reiniciar servidor BFF
- Limpar cache do navegador

### **Problema: User Service não responde**
**Solução:**
- Verificar se USER_SERVICE_URL está correto
- Verificar se User Service está rodando (porta 8083)
- Verificar logs do User Service

### **Problema: CORS error no Frontend**
**Solução:**
- Verificar configuração CORS do BFF
- Deve permitir origin do Frontend (localhost:5173)

### **Problema: X-Request-Id não sendo enviado**
**Solução:**
- Gerar UUID: `crypto.randomUUID()` ou similar
- Adicionar ao headers da requisição HTTP

---

## 🎯 EXPECTATIVAS

### **Você é a Guardiã da Arquitetura BFF**

**Gabriela, você é a orquestradora de microserviços.** Eu confio que você:

1. ✅ **Conhece GraphQL + Next.js** melhor que eu
2. ✅ **Conhece TypeScript** melhor que eu
3. ✅ **Sabe como orquestrar chamadas** melhor que eu
4. ✅ **Entende tratamento de erros** melhor que eu

**Por isso:**
- Estude o código existente do BFF profundamente
- Siga os padrões já estabelecidos
- Tome decisões técnicas fundamentadas
- Adicione validações que julgar necessárias
- Proponha a melhor estrutura de código

**Eu defini O QUE precisa ser feito. Você decide COMO fazer.**

**Seu papel é proteger a arquitetura: Frontend → BFF → Backends**

---

## 📊 OUTPUT ESPERADO

Ao finalizar, documente aqui:

### **Decisões Técnicas Tomadas:**
(Você preenche)

### **Estrutura Criada:**
(Liste arquivos criados/modificados)

### **Testes Realizados:**
(Liste cenários testados)

### **Dificuldades Encontradas:**
(Se houver)

### **Melhorias Implementadas:**
(Além do requisitado)

---

**Data de Criação:** 01/11/2025  
**Criado por:** Arquiobaldo (Arquiteto MoverseMais)  
**Versão:** 1.0

