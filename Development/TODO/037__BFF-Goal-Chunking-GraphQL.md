# 🎯 Card 037 - BFF: Goal Chunking GraphQL (Mutations e Queries)

**Agente Responsável:** Gabriela  
**Microserviço:** moversemais-store-graphql (BFF)  
**Prioridade:** Alta  
**Status:** TODO  
**Estimativa:** 4-6 horas

---

## 🔍 **ESTUDO PRÉVIO REALIZADO**

**Arquiobaldo estudou a aplicação antes de criar este card:**
- ✅ Leu `objectives-service.ts` (296 linhas)
- ✅ Leu `backend-client.ts` (padrão de chamadas HTTP)
- ✅ Leu `objectives.ts` (resolvers existentes)
- ✅ Leu `types/interfaces.ts` (interfaces TypeScript)
- ✅ Verificou padrão de error handling (processBackendError)
- ✅ Backend COMPLETO (Cards 035, 036.2) - Endpoints prontos

---

## 📋 CONTEXTO

### **Situação Atual**

**Backend COMPLETO** ✅ (Osvaldo completou!)
- Endpoints prontos:
  - POST /objectives (aceita motive, context, mode)
  - POST /objectives/{id}/generate-plan (IA gera plano)
  - GET /objectives/{id} (retorna plano completo)
  - PUT /objectives/{id} (atualiza motive, context)
- Status DRAFT e ACTIVE implementados
- Goal Chunking funcional

**BFF DESATUALIZADO** ❌
- Mutation createObjective **NÃO envia** motive, context, mode
- **NÃO existe** mutation generateGoalPlan
- **NÃO existe** query getGoalWithPlan
- Types GraphQL desatualizados

**Frontend BLOQUEADO** ❌
- Não consegue criar metas com Goal Chunking
- Não consegue solicitar geração de plano
- Não consegue buscar plano completo

### **Problema Identificado**

Para que **Goal Chunking funcione end-to-end**, precisamos que:

**BFF:**
- Mutation `createObjective` envie motive, context, mode ao backend
- Mutation `generateGoalPlan` chame backend para gerar plano
- Query `getGoalWithPlan` busque meta com plano completo
- Types GraphQL incluam novos campos

**Atualmente:**
- ❌ Frontend não consegue criar metas com Goal Chunking
- ❌ Frontend não consegue solicitar geração de plano
- ❌ Frontend não consegue buscar plano completo

### **Solução Proposta**

Atualizar BFF GraphQL para expor Goal Chunking:
1. Atualizar mutation `createObjective` (motive, context, mode)
2. Criar mutation `generateGoalPlan` (gerar plano com IA)
3. Atualizar query `getObjective` (retornar plano completo)
4. Atualizar Types GraphQL (Stage, Action, KeyResult)

### **Onde se Encaixa na Arquitetura**

```
Sprint 2: Goal Chunking
├─ Card 035: Backend ✅ DONE
├─ Card 036: Segurança ✅ DONE
├─ Card 036.2: Endpoints ✅ DONE
├─ Card 037: BFF (Gabriela) ← ESTE CARD
└─ Card 038: Frontend (Lisa) ⏸️ Aguardando este card
```

### **Impacto se Não For Feito**

- Frontend não consegue usar Goal Chunking
- Backend pronto mas inacessível
- Sprint 2 incompleta

---

## 📊 **DIAGRAMA DE SEQUÊNCIA (REFERÊNCIA)**

**Leia o diagrama completo em:**
`../moversemais-arquitetura/SEQUENCE_DIAGRAMS__Goals-Module.md`

**Resumo do fluxo (Modo AUTO):**

```
Frontend → BFF → Backend → DB → ChatGPT

1. Frontend: createObjective (motive, context, mode=AUTO)
2. BFF: Chama POST /objectives
3. Backend: Cria objective (status=DRAFT)
4. BFF: Retorna objectiveId

5. Frontend: generateGoalPlan (objectiveId)
6. BFF: Chama POST /objectives/{id}/generate-plan
7. Backend: ChatGPT gera plano → Salva no DB → status=ACTIVE
8. BFF: Retorna plano completo

9. Frontend: Exibe plano para revisão
10. Usuário: Aprova ou regenera
```

**Seu papel (Gabriela):**
- Passos 2, 3, 4: Mutation createObjective
- Passos 6, 7, 8: Mutation generateGoalPlan
- Query getGoalWithPlan: Buscar plano completo

---

## 🎯 REQUISITOS OBRIGATÓRIOS

### **1. Atualizar Mutation createObjective**

**Função de Negócio:**
Permitir que Frontend crie metas com campos de Goal Chunking (motive, context, mode).

**Schema GraphQL (atualizar):**
```graphql
input CreateObjectiveInput {
  originalInput: String!
  confirmedObjectiveText: String!
  description: String
  startDate: String!
  endDate: String!
  priority: Int
  objectiveType: ObjectiveType
  
  # NOVOS CAMPOS (Card 037)
  motive: String
  context: String
  mode: String  # AUTO ou MANUAL (default: MANUAL)
}

type Mutation {
  createObjective(input: CreateObjectiveInput!): Objective!
}
```

**Lógica (objectives-service.ts):**
- Receber motive, context, mode do Frontend
- Chamar backend POST /objectives com novos campos
- Retornar objective criado (com status DRAFT ou ACTIVE)

**Você decide:**
- Se valida mode (AUTO ou MANUAL) no BFF
- Mensagens de erro
- Logs

**Restrições:**
- NÃO quebrar mutation existente (backward compatible)

---

### **2. Criar Mutation generateGoalPlan**

**Função de Negócio:**
Permitir que Frontend solicite geração de plano para uma meta específica.

**Schema GraphQL (criar):**
```graphql
input GenerateGoalPlanInput {
  objectiveId: ID!
  regenerate: Boolean  # true = deletar plano antigo e gerar novo
}

type Mutation {
  generateGoalPlan(input: GenerateGoalPlanInput!): GoalPlanResponse!
}

type GoalPlanResponse {
  success: Boolean!
  message: String!
  objective: ObjectiveWithPlan
  errors: [String!]
}

type ObjectiveWithPlan {
  id: ID!
  title: String!
  motive: String
  context: String
  mode: String!
  status: String!
  startDate: String!
  endDate: String!
  stages: [Stage!]!
  keyResults: [KeyResult!]!
}

type Stage {
  id: ID!
  title: String!
  description: String
  orderIndex: Int!
  actions: [Action!]!
}

type Action {
  id: ID!
  title: String!
  description: String
  status: String!
  orderIndex: Int!
  linkedKeyResults: [ID!]!  # IDs dos KRs vinculados
}

type KeyResult {
  id: ID!
  title: String!
  description: String
  type: String!  # NUMERIC, PERCENTAGE, BINARY, CURRENCY, TIME
  targetValue: Float!
  currentValue: Float!
  unit: String
  weight: Float!
  progressPercentage: Int!
  orderIndex: Int
}
```

**Lógica (objectives-service.ts):**
- Receber objectiveId e regenerate do Frontend
- Chamar backend POST /objectives/{id}/generate-plan
- Passar headers de segurança (X-User-Id, X-Internal-Service-Key)
- Retornar plano completo estruturado
- Tratar erros (404, 403, 400, 500)

**Você decide:**
- Estrutura do service method
- Mapeamento de dados (backend → GraphQL)
- Error handling

**Restrições:**
- Seguir padrão de objectives-service.ts
- Passar X-Internal-Service-Key (segurança)

---

### **3. Atualizar Query getObjective (ou criar getGoalWithPlan)**

**Função de Negócio:**
Permitir que Frontend busque meta com plano completo (stages, actions, KRs).

**Opção A: Atualizar getObjective existente**
```graphql
type Query {
  getObjective(id: ID!): ObjectiveWithPlan
}
```
- ✅ Reutiliza query existente
- ✅ Retorna plano se existir
- ❌ Pode quebrar Frontend se espera estrutura antiga

**Opção B: Criar getGoalWithPlan nova**
```graphql
type Query {
  getObjective(id: ID!): Objective  # Mantém antiga
  getGoalWithPlan(id: ID!): ObjectiveWithPlan  # Nova
}
```
- ✅ Não quebra query existente
- ✅ Frontend escolhe qual usar
- ❌ Duplicação de código

**Você decide qual opção usar!**

**Lógica (objectives-service.ts):**
- Chamar backend GET /objectives/{id}
- Backend já retorna plano completo (Card 036.2)
- Mapear para types GraphQL
- Retornar estrutura hierárquica

**Restrições:**
- Não quebrar query existente (se escolher Opção A, garantir backward compatibility)

---

### **4. Atualizar Types GraphQL**

**Criar/Atualizar tipos:**
- `Stage` (novo)
- `Action` (novo)
- `KeyResult` (novo)
- `ObjectiveWithPlan` (novo)
- `GoalPlanResponse` (novo)
- Atualizar `Objective` (adicionar motive, context, mode)
- Atualizar `CreateObjectiveInput` (adicionar motive, context, mode)

**Você decide:**
- Estrutura exata dos types
- Se cria arquivo separado (goals-types.ts) ou adiciona em types.ts
- Nomenclatura

---

## ⚠️ RESTRIÇÕES

### **O que NÃO PODE ser alterado:**

1. ❌ **NÃO alterar mutations existentes** (se quebrar)
2. ❌ **NÃO alterar queries existentes** (se quebrar)
3. ❌ **NÃO alterar backend** (já está pronto)
4. ❌ **NÃO alterar error handling base** (processBackendError)

### **O que DEVE ser preservado:**

1. ✅ **Padrão de services** (objectives-service.ts)
2. ✅ **Padrão de backend-client** (chamadas HTTP)
3. ✅ **Padrão de resolvers** (objectives.ts)
4. ✅ **Padrão de types** (interfaces.ts)
5. ✅ **Headers de segurança** (X-User-Id, X-Internal-Service-Key)
6. ✅ **Error handling** (processBackendError)

---

## 📚 DOCUMENTAÇÃO DE REFERÊNCIA

### **Arquivos para Estudar (OBRIGATÓRIO):**

1. **Services Existentes:**
   - `app/api/graphql/services/objectives-service.ts` (296 linhas)
   - `app/api/graphql/services/backend-client.ts` (padrão de chamadas)

2. **Resolvers Existentes:**
   - `app/api/graphql/resolvers/objectives.ts`
   - Padrão de mutation e query

3. **Types Existentes:**
   - `app/api/graphql/types/interfaces.ts`
   - `app/api/graphql/schema/types.ts`

4. **Error Handling:**
   - `app/api/graphql/utils/graphql-error-handler.ts`
   - `app/api/graphql/utils/error-mapping.ts`

5. **Diagrama de Sequência (IMPORTANTE!):**
   - `../moversemais-arquitetura/SEQUENCE_DIAGRAMS__Goals-Module.md`
   - **Leia o Diagrama 1 completo!**

6. **Backend Endpoints (referência):**
   - POST /objectives (aceita motive, context, mode)
   - POST /objectives/{id}/generate-plan (gera plano)
   - GET /objectives/{id} (retorna plano completo)

---

## 🔧 WORKFLOW

### **1. ESTUDAR (OBRIGATÓRIO - 1 hora)**

```bash
cd moversemais-store-graphql

# Estudar services
cat app/api/graphql/services/objectives-service.ts
cat app/api/graphql/services/backend-client.ts

# Estudar resolvers
cat app/api/graphql/resolvers/objectives.ts

# Estudar types
cat app/api/graphql/types/interfaces.ts

# Ler diagrama de sequência (IMPORTANTE!)
cat ../moversemais-arquitetura/SEQUENCE_DIAGRAMS__Goals-Module.md

# Ler AGENTS.md
cat AGENTS.md
```

**Perguntas para Responder:**
- Como objectives-service chama backend?
- Como passar X-Internal-Service-Key?
- Como mapear response backend → GraphQL?
- Atualizar Objective type ou criar ObjectiveWithPlan?
- Criar arquivo separado para Goal types?

### **2. CRIAR BRANCH**

```bash
git checkout -b feature/goals-chunking-bff
```

### **3. IMPLEMENTAR (VOCÊ DECIDE COMO)**

**Você é a especialista em GraphQL + Next.js!**

**Ordem sugerida:**
1. Atualizar types/interfaces.ts (novos campos e types)
2. Atualizar schema/types.ts (GraphQL schema)
3. Atualizar backend-client.ts (método generateGoalPlan)
4. Atualizar objectives-service.ts (métodos novos)
5. Atualizar resolvers/objectives.ts (mutations e queries)
6. Testar

**Decisões técnicas que você toma:**
- Estrutura dos types GraphQL
- Mapeamento de dados (backend → GraphQL)
- Error handling específico
- Logs
- Nomenclatura

**Mas DEVE seguir:**
- ✅ Padrão de objectives-service.ts
- ✅ Padrão de backend-client.ts
- ✅ Passar X-Internal-Service-Key (segurança)
- ✅ Usar processBackendError

### **4. TESTAR**

**Testes Obrigatórios:**

```bash
# 1. Rodar BFF
npm run dev

# 2. Acessar GraphQL Playground
# http://localhost:3001/api/graphql

# 3. Testar Mutation createObjective (mode=AUTO)
mutation {
  createObjective(input: {
    originalInput: "Quero correr 5km em 30 minutos"
    confirmedObjectiveText: "Correr 5km em 30 minutos"
    startDate: "2025-11-09T00:00:00"
    endDate: "2026-02-09T00:00:00"
    objectiveType: HEALTH_GOAL
    motive: "Melhorar saúde e disposição"
    context: "Treinar 3x por semana, 30min"
    mode: "AUTO"
  }) {
    id
    status  # Esperado: DRAFT
    mode    # Esperado: AUTO
    motive
    context
  }
}

# 4. Testar Mutation generateGoalPlan
mutation {
  generateGoalPlan(input: {
    objectiveId: "{id-do-passo-3}"
    regenerate: false
  }) {
    success
    message
    objective {
      id
      status  # Esperado: ACTIVE (após gerar plano)
      stages {
        id
        title
        orderIndex
        actions {
          id
          title
          status
          linkedKeyResults
        }
      }
      keyResults {
        id
        title
        type
        targetValue
        weight
      }
    }
  }
}

# 5. Testar Query getGoalWithPlan (ou getObjective)
query {
  getGoalWithPlan(id: "{id}") {
    id
    title
    motive
    context
    mode
    status
    stages {
      id
      title
      actions {
        id
        title
      }
    }
    keyResults {
      id
      title
      type
    }
  }
}

# 6. Testar Mutation createObjective (mode=MANUAL)
mutation {
  createObjective(input: {
    originalInput: "Aprender Kotlin"
    confirmedObjectiveText: "Aprender Kotlin em 3 meses"
    startDate: "2025-11-09T00:00:00"
    endDate: "2026-02-09T00:00:00"
    mode: "MANUAL"
  }) {
    id
    status  # Esperado: ACTIVE
    mode    # Esperado: MANUAL
  }
}

# 7. Testar regenerate
mutation {
  generateGoalPlan(input: {
    objectiveId: "{id}"
    regenerate: true  # Deleta plano antigo e gera novo
  }) {
    success
    objective {
      stages {
        title  # Deve ser diferente do anterior
      }
    }
  }
}
```

**Verificações:**
- [ ] Mutation createObjective envia motive, context, mode
- [ ] Mode AUTO cria objective com status=DRAFT
- [ ] Mode MANUAL cria objective com status=ACTIVE
- [ ] Mutation generateGoalPlan chama backend
- [ ] Plano retornado está estruturado (stages, actions, KRs)
- [ ] Status muda de DRAFT para ACTIVE após gerar plano
- [ ] Query getGoalWithPlan retorna plano completo
- [ ] Regenerate funciona (deleta e gera novo)
- [ ] Headers de segurança passados (X-Internal-Service-Key)
- [ ] Erros tratados (404, 403, 400, 500)

### **5. DOCUMENTAR DECISÕES**

Ao finalizar, documente:
- Types GraphQL criados
- Services atualizados
- Resolvers atualizados
- Mapeamento de dados
- Testes realizados
- Dificuldades encontradas

### **6. COMMIT E PUSH**

```bash
git add .
git commit -m "feat(bff): implementa Goal Chunking GraphQL

- CreateObjectiveInput: adiciona motive, context, mode
- Mutation generateGoalPlan (gera plano com IA)
- Query getGoalWithPlan (busca plano completo)
- Types GraphQL: Stage, Action, KeyResult, ObjectiveWithPlan
- objectives-service: métodos novos
- backend-client: generateGoalPlan method
- Resolvers atualizados
- Headers de segurança (X-Internal-Service-Key)
- Error handling
- Testes no GraphQL Playground
- Ref: Card 037"

git push origin feature/goals-chunking-bff
```

### **7. MOVER PARA VALIDAÇÃO**

```bash
mv Development/TODO/037__BFF-Goal-Chunking-GraphQL.md \
   Development/VALIDATING/037__BFF-Goal-Chunking-GraphQL.md
```

---

## ✅ CRITÉRIOS DE VALIDAÇÃO

### **Funcionalidades:**
- [ ] Mutation createObjective atualizada (motive, context, mode)
- [ ] Mutation generateGoalPlan criada
- [ ] Query getGoalWithPlan criada (ou getObjective atualizada)
- [ ] Types GraphQL criados (Stage, Action, KeyResult, ObjectiveWithPlan)
- [ ] Backend chamado corretamente (POST /objectives, POST /generate-plan, GET /objectives/{id})
- [ ] Headers de segurança passados (X-Internal-Service-Key)
- [ ] Erros tratados (processBackendError)

### **Padrão:**
- [ ] Seguiu padrão de objectives-service.ts
- [ ] Seguiu padrão de backend-client.ts
- [ ] Seguiu padrão de resolvers
- [ ] Seguiu padrão de types
- [ ] Código limpo e documentado

### **Testes:**
- [ ] GraphQL Playground testado
- [ ] Mutation createObjective (AUTO e MANUAL)
- [ ] Mutation generateGoalPlan
- [ ] Query getGoalWithPlan
- [ ] Regenerate testado
- [ ] Erros tratados

---

## 🚨 TROUBLESHOOTING

### **Problema: Backend retorna 401**
**Solução:**
- Verificar se X-Internal-Service-Key está sendo enviado
- Verificar valor da variável INTERNAL_SERVICE_KEY
- Verificar backend-client.ts

### **Problema: Plano não retorna estruturado**
**Solução:**
- Verificar mapeamento backend → GraphQL
- Backend já retorna estruturado (Card 036.2)
- Verificar types GraphQL

### **Problema: Status não muda para ACTIVE**
**Solução:**
- Backend já faz isso (Card 035)
- Verificar se response está sendo mapeado corretamente

---

## 🎯 EXPECTATIVAS

### **Você é a Especialista em BFF GraphQL**

**Gabriela, você já implementou várias integrações com sucesso!**

**Agora, você vai expor Goal Chunking via GraphQL!**

**Eu confio que você:**
- Conhece o BFF (você mantém essa aplicação!)
- Sabe criar mutations e queries GraphQL
- Sabe chamar backend com segurança
- Sabe mapear dados backend → GraphQL

**Referências:**
- objectives-service.ts (você conhece!)
- Diagrama de sequência (leia!)
- Backend pronto (Osvaldo completou!)

**Eu defini O QUE precisa ser feito. Você decide COMO fazer.**

**Este card expõe Goal Chunking para o Frontend!** 🚀

---

## 📊 OUTPUT ESPERADO

Ao finalizar, documente aqui:

### **Decisões Técnicas Tomadas:**
(Você preenche)

### **Types GraphQL Criados:**
(Liste)

### **Services Atualizados:**
(Liste métodos)

### **Resolvers Atualizados:**
(Liste mutations e queries)

### **Testes Realizados:**
(Liste cenários testados no GraphQL Playground)

### **Dificuldades Encontradas:**
(Se houver)

### **Melhorias Implementadas:**
(Além do requisitado)

---

**Data de Criação:** 09/11/2025  
**Criado por:** Arquiobaldo (Arquiteto MoverseMais)  
**Contexto:** Módulo Goals - Sprint 2 (Goal Chunking) - BFF  
**Dependência:** Cards 035, 036, 036.2 ✅ (DONE - Backend Completo)  
**Diagrama:** SEQUENCE_DIAGRAMS__Goals-Module.md (Diagrama 1)  
**Próximo:** Card 038 (Frontend - Lisa)  
**Versão:** 1.0

