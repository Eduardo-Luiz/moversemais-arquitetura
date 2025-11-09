# 🎯 Card 035 - Sprint 2: Goal Chunking Completo (Backend + BFF + Frontend)

**Agentes Responsáveis:** Osvaldo (Backend) + Gabriela (BFF) + Lisa (Frontend)  
**Microserviços:** moversemais-objective + moversemais-store-graphql + moversemais-store-front  
**Prioridade:** Alta  
**Status:** TODO  
**Estimativa:** 12-16 horas (distribuído entre 3 IAs)

---

## 🔍 **ESTUDO PRÉVIO REALIZADO**

**Arquiobaldo estudou as aplicações antes de criar este card:**
- ✅ Leu PRD 001 (Goal Chunking) - 95 linhas
- ✅ Verificou integração ChatGPT existente (AssessmentDiagnosisService, KeyResultsService)
- ✅ Verificou CreateGoalPage.tsx (430 linhas - frontend)
- ✅ Verificou BFF GraphQL (mutations e resolvers existentes)
- ✅ Sprint 1 COMPLETA (entities, repositories, migrations)

---

## 📋 CONTEXTO

### **Situação Atual**

**Sprint 1 COMPLETA** ✅
- Tabelas criadas: stages, actions, key_results, action_kr_links, checkins
- Entities JPA: Stage, Action, KeyResult, ActionKrLink, Checkin
- Repositories: Todos criados com queries customizadas
- Campos em objectives: motive, context, mode

**O que falta:**
- ❌ Backend não gera planos com IA
- ❌ BFF não tem mutations para Goal Chunking
- ❌ Frontend não permite criação com IA

### **Problema Identificado**

Para implementar **PRD 001 (Goal Chunking)**, precisamos que:

**Backend:**
- IA transforme meta em plano estruturado (Etapas + Ações + KRs)
- Endpoint receba objective_id e gere plano completo
- Plano seja salvo no banco (stages, actions, key_results, action_kr_links)

**BFF:**
- Mutation `generateGoalPlan` chame backend
- Query `getGoalWithPlan` retorne meta + plano completo
- Tratamento de erros adequado

**Frontend:**
- CreateGoalPage evolua com campos motive, context, mode
- Tela de revisão mostre plano gerado pela IA
- Usuário possa aprovar, editar ou regenerar plano

### **Solução Proposta**

Implementar **Goal Chunking completo** em 3 camadas:
1. **Backend**: GoalChunkingService + Endpoint
2. **BFF**: Mutations e Queries GraphQL
3. **Frontend**: Evoluir CreateGoalPage + Tela de Revisão

### **Onde se Encaixa na Arquitetura**

```
Sprint 2: Goal Chunking (IMPLEMENTAR)
├─ Backend (Osvaldo)
│   ├─ GoalChunkingService (IA gera plano)
│   ├─ POST /objectives/{id}/generate-plan
│   └─ DTOs (GoalPlanRequest, GoalPlanResponse)
│
├─ BFF (Gabriela)
│   ├─ Mutation generateGoalPlan
│   ├─ Query getGoalWithPlan
│   └─ Types GraphQL
│
└─ Frontend (Lisa)
    ├─ Evoluir CreateGoalPage.tsx
    ├─ Tela de Revisão de Plano
    └─ Integração com BFF
```

### **Impacto se Não For Feito**

- Usuário não consegue usar IA para criar planos
- Metas ficam sem estrutura (etapas/ações/KRs)
- PRD 001 não implementado
- Sprint 2 incompleta

---

## 🎯 REQUISITOS OBRIGATÓRIOS

### **PARTE 1: BACKEND (Osvaldo)**

#### **1. GoalChunkingService - IA Gera Plano**

**Função de Negócio:**
Transformar uma meta (objective) em plano estruturado usando ChatGPT, gerando:
- **Etapas** (stages): Fases sequenciais do plano
- **Ações** (actions): Tarefas executáveis dentro de cada etapa
- **Key Results** (KRs): Indicadores mensuráveis de progresso
- **Vinculações** (action_kr_links): Quais ações impactam quais KRs

**Requisitos Funcionais:**
- Buscar objective por ID
- Validar que objective existe e pertence ao usuário
- Montar prompt estruturado para ChatGPT incluindo:
  - Título da meta (objectiveText)
  - Motivo (motive) - por que importa
  - Contexto (context) - como pretende alcançar
  - Prazo (startDate, endDate)
  - Tipo (objectiveType)
- Chamar ChatGPT solicitando JSON estruturado
- Parsear resposta da IA
- Validar estrutura do plano gerado
- Salvar no banco: stages → actions → key_results → action_kr_links
- Retornar plano completo

**Estrutura do Plano (JSON esperado da IA):**
```json
{
  "stages": [
    {
      "title": "Etapa 1: Avaliar condicionamento físico",
      "description": "...",
      "order": 1,
      "actions": [
        {
          "title": "Fazer corrida teste de 2km",
          "description": "...",
          "order": 1,
          "linkedKRs": [0, 1]  // Índices dos KRs que esta ação impacta
        }
      ]
    }
  ],
  "keyResults": [
    {
      "title": "Tempo de corrida",
      "description": "...",
      "type": "TIME",
      "targetValue": 30.00,
      "unit": "min",
      "weight": 0.50
    }
  ]
}
```

**Validações Obrigatórias:**
- Mínimo 1 etapa
- Mínimo 3 ações (total)
- Mínimo 1 KR
- Soma dos weights dos KRs = 1.0
- Tipos de KR válidos (NUMERIC, PERCENTAGE, BINARY, CURRENCY, TIME)
- linkedKRs válidos (índices existentes)

**Referências Existentes:**
- `AssessmentDiagnosisService.kt` - Como chamar ChatGPT e parsear JSON
- `KeyResultsService.kt` - Exemplo de prompt estruturado
- `ChatGPTService.kt` - Serviço base de comunicação com IA

**Restrições:**
- NÃO alterar Objective entity (apenas usar)
- NÃO alterar entities criadas na Sprint 1
- NÃO criar novos endpoints além do especificado
- Reutilizar ChatGPTService existente

---

#### **2. Endpoint POST /objectives/{id}/generate-plan**

**Função de Negócio:**
Permitir que BFF solicite geração de plano para uma meta específica.

**Requisitos Funcionais:**
- Path: `POST /api/v1/objectives/{objectiveId}/generate-plan`
- Headers obrigatórios: `X-User-Id` (UUID do usuário)
- Body: vazio ou opcional (regenerate: boolean)
- Response: Plano completo (stages, actions, key_results, vinculações)
- Status 200: Plano gerado com sucesso
- Status 404: Objective não encontrado
- Status 403: Objective não pertence ao usuário
- Status 400: Objective sem motive/context (modo AUTO exige)
- Status 500: Erro ao chamar IA

**Comportamento:**
- Se objective já tem plano (stages existentes):
  - Se `regenerate=false`: retornar erro "Plano já existe"
  - Se `regenerate=true`: deletar plano antigo e gerar novo
- Se objective não tem plano: gerar novo

**DTOs Necessários:**
- `GoalPlanRequest` (opcional - apenas regenerate flag)
- `GoalPlanResponse` (plano completo estruturado)
- `StageDTO`, `ActionDTO`, `KeyResultDTO`, `ActionKrLinkDTO`

**Restrições:**
- NÃO alterar endpoints existentes
- NÃO alterar ObjectiveController (criar novo ou adicionar método)
- Seguir padrão de UseCases (Clean Architecture)

---

#### **3. DTOs e Validações**

**Criar DTOs:**
- `GoalPlanResponse` - Resposta completa do plano
- `StageDTO` - Etapa com ações
- `ActionDTO` - Ação com vinculações
- `KeyResultDTO` - KR completo
- `ActionKrLinkDTO` - Vinculação action ↔ KR

**Validações Bean Validation:**
- @NotNull, @NotBlank onde apropriado
- @Min, @Max para valores numéricos
- @Size para listas (mínimo 1 etapa, 3 ações, 1 KR)

---

### **PARTE 2: BFF (Gabriela)**

#### **1. Mutation generateGoalPlan**

**Função de Negócio:**
Permitir que Frontend solicite geração de plano para uma meta.

**Schema GraphQL:**
```graphql
type Mutation {
  generateGoalPlan(
    objectiveId: ID!
    regenerate: Boolean
  ): GoalPlanResponse!
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
  linkedKeyResults: [KeyResult!]!
}

type KeyResult {
  id: ID!
  title: String!
  description: String
  type: String!
  targetValue: Float!
  currentValue: Float!
  unit: String
  weight: Float!
  progressPercentage: Int!
}
```

**Requisitos Funcionais:**
- Chamar backend `POST /objectives/{id}/generate-plan`
- Passar `X-User-Id` header (do contexto GraphQL)
- Passar `X-Internal-Service-Key` (segurança)
- Tratar erros do backend (404, 403, 400, 500)
- Mapear response para types GraphQL
- Retornar plano completo estruturado

**Referências Existentes:**
- `objective-service.ts` - Como chamar backend
- `oauth-service.ts` - Como passar headers de segurança
- Resolvers existentes (submitAssessment, createObjective)

**Restrições:**
- NÃO alterar mutations existentes
- NÃO alterar types existentes (criar novos)
- Seguir padrão de error handling (processBackendError)

---

#### **2. Query getGoalWithPlan**

**Função de Negócio:**
Permitir que Frontend busque meta com plano completo (stages, actions, KRs).

**Schema GraphQL:**
```graphql
type Query {
  getGoalWithPlan(objectiveId: ID!): ObjectiveWithPlan
}
```

**Requisitos Funcionais:**
- Chamar backend `GET /objectives/{id}` (endpoint existente)
- Buscar stages relacionados
- Buscar actions relacionados
- Buscar key_results relacionados
- Buscar action_kr_links relacionados
- Montar estrutura hierárquica
- Retornar null se não encontrado

**Restrições:**
- NÃO criar novo endpoint no backend (usar existentes)
- Pode precisar de múltiplas chamadas ao backend
- Ou solicitar ao Osvaldo criar endpoint específico

---

### **PARTE 3: FRONTEND (Lisa)**

#### **1. Evoluir CreateGoalPage.tsx**

**Função de Negócio:**
Adicionar campos para Goal Chunking e permitir escolha entre modo AUTO (IA) e MANUAL.

**Campos Novos:**
- **Motivo** (motive): "Por que essa meta importa para você?"
  - Textarea, opcional mas recomendado para modo AUTO
  - Placeholder: "Ex: Quero melhorar minha saúde e ter mais energia"
- **Contexto** (context): "Como você pretende alcançar?"
  - Textarea, opcional mas recomendado para modo AUTO
  - Placeholder: "Ex: Vou treinar 3x por semana, 30min por dia"
- **Modo** (mode): Toggle ou Radio Button
  - AUTO: "Deixar IA criar plano"
  - MANUAL: "Criar plano manualmente"
  - DEFAULT: MANUAL

**Fluxo Modo AUTO:**
1. Usuário preenche: título, prazo, tipo, motivo, context
2. Clica "Criar Meta com IA"
3. Loading: "Gerando plano estruturado..."
4. Mutation `generateGoalPlan` chamada
5. Redireciona para Tela de Revisão

**Fluxo Modo MANUAL:**
1. Usuário preenche: título, prazo, tipo
2. Clica "Criar Meta"
3. Meta criada sem plano
4. (Futuro: interface para criar etapas/ações manualmente)

**Validações:**
- Se modo AUTO: recomendar preencher motivo e contexto
- Se modo AUTO sem motivo/contexto: mostrar warning (não bloquear)

**Referências Existentes:**
- `CreateGoalPage.tsx` (430 linhas - estudar estrutura)
- `useMutation` do Apollo Client
- Padrão de formulários do projeto

**Restrições:**
- NÃO quebrar funcionalidade existente
- NÃO alterar fluxo de criação manual (apenas adicionar AUTO)
- Seguir padrão de UX do projeto

---

#### **2. Criar GoalPlanReviewPage.tsx (Nova Página)**

**Função de Negócio:**
Mostrar plano gerado pela IA e permitir que usuário revise, edite e aprove.

**Estrutura da Página:**
```
┌─────────────────────────────────────────┐
│ 🎯 Revise seu Plano                     │
│                                         │
│ Meta: Correr 5km em 30 minutos         │
│ Prazo: 3 meses                          │
│                                         │
│ ┌─────────────────────────────────────┐ │
│ │ 📋 Etapa 1: Avaliar condicionamento │ │
│ │   ✓ Fazer corrida teste de 2km     │ │
│ │   ✓ Registrar tempo e frequência   │ │
│ └─────────────────────────────────────┘ │
│                                         │
│ ┌─────────────────────────────────────┐ │
│ │ 📋 Etapa 2: Criar rotina de treinos│ │
│ │   ✓ Planejar 3 treinos semanais    │ │
│ └─────────────────────────────────────┘ │
│                                         │
│ ┌─────────────────────────────────────┐ │
│ │ 📊 Indicadores de Progresso (KRs)  │ │
│ │   • Tempo de corrida: 30 min (50%) │ │
│ │   • Treinos completados: 12 (30%)  │ │
│ │   • Frequência cardíaca: 150 (20%) │ │
│ └─────────────────────────────────────┘ │
│                                         │
│ [🔄 Regenerar Plano] [✅ Aprovar]      │
└─────────────────────────────────────────┘
```

**Funcionalidades:**
- Mostrar etapas com ações (hierárquico)
- Mostrar KRs com pesos (%)
- Botão "Regenerar Plano": chama mutation novamente
- Botão "Aprovar e Ativar": confirma plano e ativa meta
- (Futuro: Editar etapas/ações inline)

**Requisitos Funcionais:**
- Receber objectiveId via URL params
- Query `getGoalWithPlan` para buscar plano
- Loading state durante busca
- Error state se plano não encontrado
- Success state mostrando plano
- Navegação: voltar para lista de metas após aprovar

**Restrições:**
- NÃO implementar edição inline (futuro)
- Focar em visualização e aprovação

---

## ⚠️ RESTRIÇÕES GERAIS

### **O que NÃO PODE ser alterado:**

**Backend:**
1. ❌ NÃO alterar entities da Sprint 1
2. ❌ NÃO alterar repositories da Sprint 1
3. ❌ NÃO alterar migrations V025-V030
4. ❌ NÃO alterar ChatGPTService base
5. ❌ NÃO alterar endpoints existentes de Objective

**BFF:**
1. ❌ NÃO alterar mutations existentes
2. ❌ NÃO alterar queries existentes
3. ❌ NÃO alterar error handling base

**Frontend:**
1. ❌ NÃO quebrar CreateGoalPage existente
2. ❌ NÃO alterar fluxo manual de criação
3. ❌ NÃO alterar componentes existentes

---

## 📚 DOCUMENTAÇÃO DE REFERÊNCIA

### **Backend (Osvaldo):**

**Estudar (OBRIGATÓRIO):**
1. `AssessmentDiagnosisService.kt` - Como chamar ChatGPT e parsear JSON
2. `KeyResultsService.kt` - Exemplo de prompt estruturado
3. `ChatGPTService.kt` - Serviço base
4. Entities da Sprint 1 (Stage, Action, KeyResult, ActionKrLink)
5. Repositories da Sprint 1

**PRD:**
- `../moversemais-arquitetura/PRD/prd_001_goal_chunking.md`

---

### **BFF (Gabriela):**

**Estudar (OBRIGATÓRIO):**
1. `objective-service.ts` - Como chamar backend
2. `oauth-service.ts` - Headers de segurança
3. Resolvers existentes (submitAssessment, createObjective)
4. Types GraphQL existentes

---

### **Frontend (Lisa):**

**Estudar (OBRIGATÓRIO):**
1. `CreateGoalPage.tsx` (430 linhas)
2. Padrão de formulários do projeto
3. `useMutation` e `useQuery` do Apollo Client
4. Componentes de UI existentes

---

## 🔧 WORKFLOW

### **OSVALDO (Backend):**

1. **Estudar** (1 hora):
   - AssessmentDiagnosisService, KeyResultsService, ChatGPTService
   - Entities e Repositories da Sprint 1
   - PRD 001

2. **Criar Branch**:
   ```bash
   git checkout -b feature/goals-chunking-backend
   ```

3. **Implementar** (6-8 horas):
   - GoalChunkingService
   - Endpoint POST /objectives/{id}/generate-plan
   - DTOs (GoalPlanResponse, etc.)
   - UseCase (se seguir Clean Architecture)
   - Testes

4. **Testar**:
   - Chamar endpoint via Postman/curl
   - Verificar plano gerado
   - Verificar dados salvos no banco
   - Testar validações

5. **Commit e Push**

---

### **GABRIELA (BFF):**

1. **Estudar** (30 min):
   - objective-service.ts
   - Resolvers existentes

2. **Criar Branch**:
   ```bash
   git checkout -b feature/goals-chunking-bff
   ```

3. **Implementar** (3-4 horas):
   - Mutation generateGoalPlan
   - Query getGoalWithPlan
   - Types GraphQL
   - Resolvers
   - Testes

4. **Testar**:
   - GraphQL Playground
   - Verificar integração com backend

5. **Commit e Push**

---

### **LISA (Frontend):**

1. **Estudar** (30 min):
   - CreateGoalPage.tsx
   - Padrão de formulários

2. **Criar Branch**:
   ```bash
   git checkout -b feature/goals-chunking-frontend
   ```

3. **Implementar** (4-5 horas):
   - Evoluir CreateGoalPage.tsx
   - Criar GoalPlanReviewPage.tsx
   - Integração com BFF
   - Testes

4. **Testar**:
   - Fluxo completo: criar meta → gerar plano → revisar → aprovar

5. **Commit e Push**

---

## ✅ CRITÉRIOS DE VALIDAÇÃO

### **Backend (Osvaldo):**
- [ ] GoalChunkingService criado
- [ ] Endpoint POST /objectives/{id}/generate-plan funciona
- [ ] Plano gerado pela IA é válido (JSON estruturado)
- [ ] Dados salvos no banco (stages, actions, key_results, action_kr_links)
- [ ] Validações funcionam (mínimo 1 etapa, 3 ações, 1 KR)
- [ ] Soma dos weights = 1.0
- [ ] Erros tratados (404, 403, 400, 500)
- [ ] Build compilado
- [ ] Aplicação iniciou

### **BFF (Gabriela):**
- [ ] Mutation generateGoalPlan criada
- [ ] Query getGoalWithPlan criada
- [ ] Types GraphQL criados
- [ ] Integração com backend funciona
- [ ] Headers de segurança passados
- [ ] Erros tratados
- [ ] GraphQL Playground testado

### **Frontend (Lisa):**
- [ ] CreateGoalPage.tsx evoluído (campos motive, context, mode)
- [ ] GoalPlanReviewPage.tsx criado
- [ ] Fluxo AUTO funciona (criar → gerar → revisar → aprovar)
- [ ] Fluxo MANUAL preservado
- [ ] Loading states implementados
- [ ] Error states implementados
- [ ] Navegação funciona
- [ ] Responsivo (mobile/desktop)

---

## 🎯 EXPECTATIVAS

### **Para Osvaldo (Backend):**

**Você completou Sprint 1 com Score 10/5!** 🏆

Agora, você é o especialista que vai:
- Integrar ChatGPT para Goal Chunking
- Criar service robusto com validações
- Salvar plano estruturado no banco
- Você decide estrutura de classes, métodos, prompts

**Referências:**
- AssessmentDiagnosisService (você tem acesso)
- Entities da Sprint 1 (você criou!)

---

### **Para Gabriela (BFF):**

**Você é a especialista em GraphQL + Next.js!**

Você decide:
- Estrutura dos resolvers
- Mapeamento de dados
- Error handling
- Types GraphQL

**Referências:**
- Resolvers existentes (submitAssessment, etc.)

---

### **Para Lisa (Frontend):**

**Você é a especialista em React + TypeScript!**

Você decide:
- Estrutura dos componentes
- UX/UI do formulário
- Layout da tela de revisão
- Estados e navegação

**Referências:**
- CreateGoalPage.tsx (430 linhas)

---

## 📊 OUTPUT ESPERADO

Ao finalizar, cada desenvolvedor documenta:

### **Osvaldo:**
- Decisões técnicas (prompt, validações, estrutura)
- Testes realizados
- Dificuldades encontradas

### **Gabriela:**
- Decisões técnicas (resolvers, types)
- Testes realizados
- Dificuldades encontradas

### **Lisa:**
- Decisões técnicas (componentes, UX)
- Testes realizados
- Dificuldades encontradas

---

**Data de Criação:** 08/11/2025  
**Criado por:** Arquiobaldo (Arquiteto MoverseMais)  
**Contexto:** Módulo Goals - Sprint 2 (Goal Chunking)  
**Dependência:** Card 034 ✅ (DONE - Sprint 1 Completa)  
**Versão:** 1.0

