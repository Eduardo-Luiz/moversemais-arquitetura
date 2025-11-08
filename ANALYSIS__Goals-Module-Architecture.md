# 🏗️ ANÁLISE ARQUITETURAL - Módulo de Metas com IA

**Arquiteto:** Arquiobaldo  
**Data:** 01/11/2025  
**Status:** Análise Completa - Aguardando Decisão

---

## 📋 **CONTEXTO DA ANÁLISE**

### **Objetivo:**
Decidir entre **REAPROVEITAR** ou **RECRIAR** funcionalidade de criação de objetivos para implementar PRD 001 (Goal Chunking) e PRD 002 (Check-in de Progresso).

### **Metodologia:**
Estudei profundamente 3 camadas da aplicação:
1. ✅ Frontend (`moversemais-store-front`)
2. ✅ BFF (`moversemais-store-graphql`)
3. ✅ Backend (`moversemais-objective`)

---

## 🔍 **O QUE JÁ EXISTE (SISTEMA ATUAL)**

### **Frontend - 2 Componentes de Criação:**

#### **1. CreateGoalPage.tsx (430 linhas)**
**Localização:** `src/components/CreateGoalPage.tsx`

**Funcionalidades:**
- ✅ Formulário estruturado (7 campos)
- ✅ Mutation GraphQL `CREATE_OBJECTIVE`
- ✅ Campos:
  1. Tipo de objetivo (PROFESSIONAL_GOAL, PERSONAL_ACHIEVEMENT, HEALTH, FINANCIAL_GAIN)
  2. Descrição da meta (originalInput)
  3. Meta confirmada (confirmedObjectiveText)
  4. Data de início
  5. Data de conclusão (prazos: 1/3/6 meses ou personalizado)
  6. Prioridade (1-5)
  7. Descrição adicional (opcional)
- ✅ Barra de progresso (% campos preenchidos)
- ✅ Botão "Obter Sugestão de IA" (básico - copia originalInput)
- ✅ Sidebar com preview da meta
- ✅ Tratamento de erros (ErrorModal + NotificationToast)
- ✅ Navegação: `/goals` (lista) ← → `/create-goal` (criar)

**Integração:**
- ✅ Apollo Client (`useMutation`)
- ✅ Mutation `CREATE_OBJECTIVE`
- ✅ RefetchQueries após criar

**Limitações:**
- ❌ **NÃO tem Goal Chunking** (IA não estrutura plano)
- ❌ **NÃO tem Etapas/Ações** (apenas meta única)
- ❌ **NÃO tem KRs vinculados** (apenas meta)
- ❌ **IA básica** (apenas copia texto)

---

#### **2. SmartGoalCreator.tsx (415 linhas)**
**Localização:** `src/components/SmartGoalCreator.tsx`

**Funcionalidades:**
- ✅ Modal/Overlay (não página)
- ✅ Formulário extenso (10 perguntas)
- ✅ Campos adicionais:
  - Categoria (financeira, saúde, aprendizado, carreira, pessoal)
  - Conexão/Motivação (por que importa)
  - Critérios de sucesso (número, data, reconhecimento)
  - Resultado esperado
  - Recursos necessários
  - Viabilidade (slider 1-10)
  - Primeiro passo
  - Acompanhamento (diário/semanal/mensal)
  - Compartilhar com grupo (sim/não)
- ✅ Barra de progresso
- ✅ Sidebar com resumo
- ✅ Botão "Obter Sugestão de IA" (básico)

**Integração:**
- ❌ **NÃO integra com BFF** (mock local)
- ❌ **NÃO salva no backend** (apenas callback local)
- ❌ **NÃO usa mutation GraphQL**

**Status:**
- ⚠️ **Protótipo/Mock** (não funcional end-to-end)

---

### **Backend - moversemais-objective**

#### **Entidade Objective (Atual):**

**Campos:**
```kotlin
- id: UUID
- userId: UUID
- originalInput: String (descrição original)
- objectiveText: String (texto confirmado)
- description: String? (opcional)
- source: Source (enum)
- objectiveType: ObjectiveType? (PROFESSIONAL_GOAL, etc.)
- startDate: LocalDateTime
- endDate: LocalDateTime
- status: Status (ACTIVE, CONCLUDED, ARCHIVED)
- progressStatus: ProgressStatus (NOT_STARTED, IN_PROGRESS, etc.)
- priority: Int? (1-5)
- completionPercentage: Int (0-100)
- completedAt: LocalDateTime?
- archivedAt: LocalDateTime?
- createdAt, updatedAt
```

**Funcionalidades:**
- ✅ CRUD completo
- ✅ Métodos auxiliares (isActive, isOverdue, daysRemaining)
- ✅ Tipos de objetivo (PROFESSIONAL, PERSONAL, HEALTH, FINANCIAL)

**Limitações:**
- ❌ **NÃO tem tabela Stages** (etapas)
- ❌ **NÃO tem tabela Actions** (ações)
- ❌ **NÃO tem tabela KeyResults** (KRs)
- ❌ **NÃO tem Goal Chunking** (IA estruturação)

---

#### **Integração IA Existente:**

**KeyResultsService.kt:**
- ✅ Integra ChatGPT para sugestão de Key Results
- ✅ Prompt estruturado
- ✅ Parsing de JSON
- ✅ Tratamento de erros

**AssessmentDiagnosisService.kt:**
- ✅ Gera diagnóstico com IA
- ✅ Plano de ação OKR (Objetivo + KRs)
- ✅ Contexto completo das competências

**Mas:**
- ❌ KRs são **apenas sugestões** (não persistidos)
- ❌ KRs não vinculados a Objective
- ❌ KRs não rastreados (sem check-in)

---

### **BFF - moversemais-store-graphql**

**Mutation CREATE_OBJECTIVE:**
- ✅ Existe e funciona
- ✅ Chama backend `POST /objectives`
- ✅ Retorna Objective criado

**Mutation SUGGEST_KEY_RESULTS:**
- ✅ Existe (para assessment)
- ✅ Chama IA para Key Results
- ✅ Mas não vincula a Objective

---

## 🎯 **COMPARAÇÃO: ATUAL vs PRD 001**

| Feature | Sistema Atual | PRD 001 (Goal Chunking) | Gap |
|---------|---------------|-------------------------|-----|
| **Criar Meta** | ✅ Sim | ✅ Sim | Compatível |
| **Título + Prazo** | ✅ Sim | ✅ Sim | Compatível |
| **Contexto (como)** | ⚠️ Descrição genérica | ✅ Campo específico | Adaptar |
| **Motivo (por que)** | ❌ Não | ✅ Sim | **ADICIONAR** |
| **Modo Auto/Manual** | ❌ Não | ✅ Sim | **ADICIONAR** |
| **IA Estrutura Plano** | ❌ Não | ✅ Sim (Etapas+Ações+KRs) | **ADICIONAR** |
| **Etapas** | ❌ Não | ✅ Sim | **CRIAR TABELA** |
| **Ações** | ❌ Não | ✅ Sim | **CRIAR TABELA** |
| **KRs Persistidos** | ❌ Não | ✅ Sim | **CRIAR TABELA** |
| **Vinculação Ações↔KRs** | ❌ Não | ✅ Sim | **CRIAR RELACIONAMENTO** |
| **Revisão de Plano** | ❌ Não | ✅ Sim | **ADICIONAR** |
| **Regeneração** | ❌ Não | ✅ Sim | **ADICIONAR** |

---

## 🎯 **COMPARAÇÃO: ATUAL vs PRD 002**

| Feature | Sistema Atual | PRD 002 (Check-in) | Gap |
|---------|---------------|-------------------|-----|
| **Atualizar Progresso** | ⚠️ Manual (completionPercentage) | ✅ Via KRs | **MUDAR LÓGICA** |
| **Selecionar KRs** | ❌ Não | ✅ Sim | **ADICIONAR** |
| **Tipos de Input** | ❌ Não | ✅ Numérico/Binário/Frequência | **ADICIONAR** |
| **Cálculo Automático** | ❌ Não | ✅ Média ponderada KRs | **ADICIONAR** |
| **Histórico Check-ins** | ❌ Não | ✅ Sim | **CRIAR TABELA** |
| **Feedback IA** | ❌ Não | ✅ Sim | **ADICIONAR** |

---

## 💡 **DECISÃO ARQUITETURAL**

## ✅ **RECOMENDAÇÃO: REAPROVEITAR E EVOLUIR**

### **Por quê REAPROVEITAR?**

1. ✅ **Entidade Objective já existe** - Base sólida
2. ✅ **CRUD completo funciona** - Não reinventar
3. ✅ **IA já integrada** (ChatGPT) - KeyResultsService pronto
4. ✅ **Frontend tem UX boa** - CreateGoalPage estruturado
5. ✅ **BFF orquestra** - Mutation CREATE_OBJECTIVE funciona
6. ✅ **Tipos de objetivo** - PROFESSIONAL, HEALTH, etc. (alinhado)
7. ✅ **Campos compatíveis** - originalInput, objectiveText, description, dates, priority

### **O que ADICIONAR (Não Recriar):**

**Backend (moversemais-objective):**
1. **Tabela `stages`** (Etapas)
2. **Tabela `actions`** (Ações)
3. **Tabela `key_results`** (KRs persistidos)
4. **Tabela `checkins`** (Histórico)
5. **Campo `motive`** em Objective (por que importa)
6. **Campo `context`** em Objective (como alcançar)
7. **Campo `mode`** em Objective (AUTO/MANUAL)
8. **Service GoalChunkingService** (IA estrutura plano)
9. **Endpoint POST /objectives/{id}/generate-plan** (Goal Chunking)
10. **Endpoint POST /objectives/{id}/checkin** (Atualizar KRs)

**BFF:**
1. **Mutation `generateGoalPlan`** (Goal Chunking)
2. **Mutation `checkinProgress`** (Atualizar KRs)
3. **Query `getGoalWithPlan`** (Meta + Etapas + Ações + KRs)

**Frontend:**
1. **Evoluir CreateGoalPage.tsx:**
   - Adicionar campo "Motivo"
   - Adicionar campo "Contexto" (renomear description)
   - Adicionar toggle "Modo: Automático/Manual"
   - Adicionar tela de revisão de plano (após IA gerar)
2. **Criar CheckinPage.tsx** (nova página)

---

## 🚫 **POR QUE NÃO RECRIAR DO ZERO?**

### **Contras de Recriar:**
1. ❌ **Perder CRUD funcional** (objetivos atuais funcionam)
2. ❌ **Perder integração IA** (KeyResultsService pronto)
3. ❌ **Perder UX** (CreateGoalPage tem boa estrutura)
4. ❌ **Duplicar código** (2 sistemas de metas paralelos)
5. ❌ **Confusão** (Objectives vs Goals - qual usar?)
6. ❌ **Tempo** (3-4 semanas vs 1-2 semanas evoluindo)

### **Risco de Manter 2 Sistemas:**
- ❌ Usuário confuso (criar Objective ou Goal?)
- ❌ Manutenção duplicada
- ❌ Dados fragmentados

---

## ✅ **PLANO DE EVOLUÇÃO (RECOMENDADO)**

### **Fase 1: Adicionar Estrutura de Dados (Backend)**

**Criar tabelas:**
```sql
-- V025__create_stages_table.sql
CREATE TABLE stages (
    id UUID PRIMARY KEY,
    objective_id UUID REFERENCES objectives(id),
    title VARCHAR(255) NOT NULL,
    description TEXT,
    order_index INT NOT NULL,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- V026__create_actions_table.sql
CREATE TABLE actions (
    id UUID PRIMARY KEY,
    stage_id UUID REFERENCES stages(id),
    title VARCHAR(255) NOT NULL,
    description TEXT,
    status VARCHAR(50) DEFAULT 'PENDING',
    order_index INT NOT NULL,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- V027__create_key_results_table.sql
CREATE TABLE key_results (
    id UUID PRIMARY KEY,
    objective_id UUID REFERENCES objectives(id),
    title VARCHAR(255) NOT NULL,
    description TEXT,
    type VARCHAR(50) NOT NULL, -- NUMERIC, PERCENT, BINARY, FREQUENCY
    target_value DECIMAL,
    current_value DECIMAL DEFAULT 0,
    unit VARCHAR(50),
    weight DECIMAL DEFAULT 1.0,
    progress_percentage INT DEFAULT 0,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- V028__create_action_kr_link_table.sql (many-to-many)
CREATE TABLE action_kr_links (
    action_id UUID REFERENCES actions(id),
    kr_id UUID REFERENCES key_results(id),
    PRIMARY KEY (action_id, kr_id)
);

-- V029__create_checkins_table.sql
CREATE TABLE checkins (
    id UUID PRIMARY KEY,
    objective_id UUID REFERENCES objectives(id),
    kr_id UUID REFERENCES key_results(id),
    previous_value DECIMAL,
    new_value DECIMAL,
    feedback TEXT,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- V030__add_goal_chunking_fields_to_objectives.sql
ALTER TABLE objectives
ADD COLUMN motive TEXT,
ADD COLUMN context TEXT,
ADD COLUMN mode VARCHAR(20) DEFAULT 'MANUAL'; -- AUTO, MANUAL
```

---

### **Fase 2: Adicionar Goal Chunking (Backend)**

**Criar Service:**
- `GoalChunkingService.kt` (usa ChatGPT para estruturar plano)

**Criar Endpoint:**
- `POST /objectives/{id}/generate-plan`
- Input: objectiveId
- Output: Stages + Actions + KRs (JSON estruturado)

**Reutilizar:**
- ✅ `ChatGPTService` (já existe)
- ✅ `KeyResultsService` (referência de prompt)

---

### **Fase 3: Evoluir Frontend**

**Atualizar CreateGoalPage.tsx:**
1. Adicionar campo "Motivo" (por que importa)
2. Renomear "Descrição" para "Contexto" (como alcançar)
3. Adicionar toggle "Modo: Automático/Manual"
4. Se Automático:
   - Chamar mutation `generateGoalPlan`
   - Mostrar tela de revisão (Etapas + Ações + KRs)
   - Permitir edição
   - Botão "Regenerar Plano"
   - Botão "Aprovar e Ativar"
5. Se Manual:
   - Criar etapas/ações/KRs manualmente
   - Interface de construção de plano

**Criar CheckinPage.tsx (PRD 002):**
- Lista de metas ativas
- Selecionar KRs para atualizar
- Input por tipo de KR
- Feedback de progresso

---

### **Fase 4: Atualizar BFF**

**Adicionar Mutations:**
- `generateGoalPlan(objectiveId)` - Goal Chunking
- `checkinProgress(objectiveId, krUpdates)` - Check-in

**Adicionar Queries:**
- `getGoalWithPlan(objectiveId)` - Meta + Etapas + Ações + KRs

---

## 📊 **COMPARAÇÃO DE ESFORÇO**

| Abordagem | Tempo Estimado | Risco | Reuso | Resultado |
|-----------|----------------|-------|-------|-----------|
| **REAPROVEITAR** | 2-3 semanas | Baixo | 70% | Evolução natural |
| **RECRIAR** | 4-6 semanas | Médio | 0% | Sistema novo |

---

## ✅ **DECISÃO FINAL: REAPROVEITAR**

### **Justificativa:**

1. **Base Sólida:**
   - Objective entity bem estruturada
   - CRUD funcional
   - IA já integrada (ChatGPT)
   - Frontend com boa UX

2. **Compatibilidade:**
   - Campos atuais (originalInput, objectiveText, dates, priority) → compatíveis com PRD 001
   - Apenas adicionar: motive, context, mode
   - Estrutura de dados estende (não quebra)

3. **Eficiência:**
   - 70% do código reutilizado
   - 2-3 semanas vs 4-6 semanas
   - Menor risco de bugs

4. **Continuidade:**
   - Objetivos atuais continuam funcionando
   - Migração suave (adicionar campos, não recriar)
   - Usuários não perdem dados

---

## 🚀 **ROADMAP DE IMPLEMENTAÇÃO**

### **Sprint 1 (Semana 1) - Estrutura de Dados:**
- Card 030: Backend - Criar tabelas (stages, actions, key_results, checkins)
- Card 031: Backend - Adicionar campos em Objective (motive, context, mode)
- Card 032: Backend - Entities JPA (Stage, Action, KeyResult, Checkin)

### **Sprint 2 (Semana 2) - Goal Chunking:**
- Card 033: Backend - GoalChunkingService (IA estrutura plano)
- Card 034: Backend - Endpoint /generate-plan
- Card 035: BFF - Mutation generateGoalPlan
- Card 036: Frontend - Evoluir CreateGoalPage (modo auto/manual + revisão)

### **Sprint 3 (Semana 3) - Check-in:**
- Card 037: Backend - CheckinService (atualizar KRs)
- Card 038: Backend - Endpoint /checkin
- Card 039: BFF - Mutation checkinProgress
- Card 040: Frontend - CheckinPage (nova página)

---

## ⚠️ **COMPONENTE A ELIMINAR**

### **SmartGoalCreator.tsx:**
- ❌ **ELIMINAR** (não integra com backend)
- ❌ Mock local (não funcional)
- ❌ Duplica CreateGoalPage.tsx
- ✅ **Manter apenas CreateGoalPage.tsx** (evoluir)

---

## 📋 **MODELO DE DADOS FINAL (Proposta)**

```
Objective (Meta)
├─ Campos atuais (manter)
├─ motive: String? (NOVO - por que importa)
├─ context: String? (NOVO - como alcançar)
├─ mode: String (NOVO - AUTO/MANUAL)
│
├─ Stages (Etapas) [1:N]
│   ├─ id, objectiveId, title, description, order
│   │
│   └─ Actions (Ações) [1:N]
│       ├─ id, stageId, title, description, status, order
│       └─ linkedKRs (many-to-many)
│
├─ KeyResults (KRs) [1:N]
│   ├─ id, objectiveId, title, description
│   ├─ type (NUMERIC, PERCENT, BINARY, FREQUENCY)
│   ├─ targetValue, currentValue, unit, weight
│   └─ progressPercentage (calculado)
│
└─ Checkins (Histórico) [1:N]
    ├─ id, objectiveId, krId, timestamp
    ├─ previousValue, newValue
    └─ feedback (IA)
```

---

## 🎯 **PRÓXIMOS PASSOS (AGUARDANDO SUA DECISÃO)**

### **Opção A: REAPROVEITAR (Recomendado)**
- Evoluir Objective entity
- Adicionar tabelas relacionadas
- Evoluir CreateGoalPage.tsx
- 2-3 semanas

### **Opção B: RECRIAR**
- Criar Goal entity (separado)
- Criar tudo do zero
- Novo frontend
- 4-6 semanas
- Manter Objectives e Goals paralelos (confuso)

---

## ✅ **MINHA RECOMENDAÇÃO COMO ARQUIOBALDO**

**REAPROVEITAR e EVOLUIR sistema atual.**

**Razões:**
1. Base sólida (70% pronto)
2. IA já integrada
3. UX boa
4. Menor tempo
5. Menor risco
6. Continuidade para usuários

**Eliminar:**
- SmartGoalCreator.tsx (mock não funcional)

**Evoluir:**
- CreateGoalPage.tsx (adicionar Goal Chunking)

**Adicionar:**
- Tabelas: stages, actions, key_results, checkins
- Services: GoalChunkingService, CheckinService
- Frontend: CheckinPage.tsx (nova)

---

**Status:** 🟡 **ANÁLISE COMPLETA - AGUARDANDO SUA DECISÃO**

**Eduardo, aprova reaproveitar e evoluir? Ou prefere recriar do zero?**

---

**Data:** 01/11/2025  
**Arquiteto:** Arquiobaldo  
**Versão:** 1.0

