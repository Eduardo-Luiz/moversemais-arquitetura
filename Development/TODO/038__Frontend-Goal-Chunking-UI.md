# 🎯 Card 038 - Frontend: Goal Chunking UI (Interface Completa)

**Agente Responsável:** Lisa  
**Microserviço:** moversemais-store-front  
**Prioridade:** Alta  
**Status:** TODO  
**Estimativa:** 8-10 horas

---

## 🔍 **ESTUDO PRÉVIO REALIZADO**

**Arquiobaldo estudou a aplicação antes de criar este card:**
- ✅ Leu `CreateGoalPage.tsx` (434 linhas - formulário de criação)
- ✅ Leu `Navigation.tsx` (237 linhas - menu lateral)
- ✅ Leu `App.tsx` (258 linhas - rotas)
- ✅ Leu `queries.ts` (mutations GraphQL existentes)
- ✅ Verificou estrutura de componentes
- ✅ Backend + BFF COMPLETOS (Cards 035-037)

---

## 📋 CONTEXTO

### **Situação Atual**

**Backend + BFF COMPLETOS** ✅
- Backend: Goal Chunking com IA funcional (Osvaldo)
- BFF: GraphQL expondo Goal Chunking (Gabriela)
- Mutations: createObjective, generateGoalPlan
- Query: getGoalWithPlan

**Frontend DESATUALIZADO** ❌
- CreateGoalPage **NÃO tem** campos motive, context, mode
- **NÃO existe** tela de revisão de plano (GoalPlanReviewPage)
- **NÃO chama** mutation generateGoalPlan
- Menu lateral tem funcionalidades não prontas (planning, checkin, retrospective)

**Usuário BLOQUEADO** ❌
- Não consegue criar metas com IA
- Não consegue revisar plano gerado
- Menu confuso (opções que não funcionam)

### **Problema Identificado**

Para que **Goal Chunking funcione end-to-end**, precisamos que:

**Frontend:**
- CreateGoalPage aceite motive, context, mode
- Fluxo AUTO: criar meta → gerar plano → revisar → aprovar
- Fluxo MANUAL: criar meta → pronto (sem plano)
- Tela de revisão mostre plano estruturado (etapas, ações, KRs)
- Menu simplificado (apenas funcionalidades prontas)

**Atualmente:**
- ❌ Usuário não consegue usar Goal Chunking
- ❌ Menu confuso (opções que não funcionam)
- ❌ Sprint 2 incompleta

### **Solução Proposta**

Implementar Goal Chunking no Frontend:
1. **Simplificar Menu** - Remover funcionalidades não prontas
2. **Evoluir CreateGoalPage** - Adicionar campos motive, context, mode
3. **Criar GoalPlanReviewPage** - Tela de revisão de plano
4. **Integrar com BFF** - Chamar mutations e queries

### **Onde se Encaixa na Arquitetura**

```
Sprint 2: Goal Chunking
├─ Card 035: Backend ✅ DONE
├─ Card 036: Segurança ✅ DONE
├─ Card 036.2: Endpoints ✅ DONE
├─ Card 037: BFF ✅ DONE
└─ Card 038: Frontend (Lisa) ← ESTE CARD

Próximo:
└─ Sprint 3: Check-in de Progresso
```

### **Impacto se Não For Feito**

- Usuário não consegue usar Goal Chunking
- Backend e BFF prontos mas inacessíveis
- Sprint 2 incompleta
- Menu confuso

---

## 📊 **DIAGRAMA DE SEQUÊNCIA (REFERÊNCIA OBRIGATÓRIA)**

**Leia o diagrama completo em:**
`../moversemais-arquitetura/SEQUENCE_DIAGRAMS__Goals-Module.md`

**Seu papel (Lisa) - Resumo do Fluxo:**

```
FASE 1: Criação da Meta
1. Usuário preenche formulário (título, prazo, tipo, motivo, contexto)
2. Usuário seleciona modo: AUTO ou MANUAL
3a. Se AUTO: Clica "Criar Meta com IA"
3b. Se MANUAL: Clica "Criar Meta Manual"

FASE 2: Geração do Plano (apenas AUTO)
10. Redireciona para tela de loading "Gerando plano estruturado..."
11. Chama mutation generateGoalPlan
27. Recebe plano completo (stages, actions, KRs)

FASE 4: Revisão e Aprovação (apenas AUTO)
28. Redireciona para GoalPlanReviewPage
29. Exibe plano estruturado
30a. Usuário clica "Regenerar Plano" → repete FASE 2
30b. Usuário clica "Aprovar e Ativar" → redireciona para lista
```

**Leia o diagrama completo para entender todos os detalhes!**

---

## 🎯 REQUISITOS OBRIGATÓRIOS

### **PARTE 1: Simplificar Menus (Navigation.tsx + NavigationBar.tsx)**

**Função de Negócio:**
Remover funcionalidades não prontas dos menus para não confundir o usuário.

**2 Componentes de Menu Identificados:**

**1. Navigation.tsx (Menu Lateral - Desktop) - linha 24-65:**
```typescript
menuItems = [
  { id: 'dashboard', icon: '🏠', path: '/dashboard' },  // ❌ NÃO PRONTO
  { id: 'assessment', icon: '📊', path: '/assessment' },
  { id: 'goals', icon: '🎯', path: '/goals' },
  { id: 'timeline', icon: '📈', path: '/timeline' },  // ❌ NÃO PRONTO
  { id: 'planning', icon: '📅', path: '/planning', submenu: [...] },  // ❌ NÃO PRONTO
  { id: 'checkin', icon: '✅', path: '/checkin' },  // ❌ NÃO PRONTO
]
```

**2. NavigationBar.tsx (Tab Bar - Mobile) - linha 24-88:**
```typescript
tabItems = [
  { id: 'dashboard', icon: <svg>...</svg>, label: 'Home', path: '/dashboard' },  // ❌ NÃO PRONTO
  { id: 'goals', icon: <svg>...</svg>, label: 'Objetivos', path: '/goals' },
  { id: 'timeline', icon: <svg>...</svg>, label: 'Timeline', path: '/timeline' },  // ❌ NÃO PRONTO
  { id: 'planning-weekly', icon: <svg>...</svg>, label: 'Planejar Semana', path: '/planning/weekly' },  // ❌ NÃO PRONTO
  { id: 'planning-daily', icon: <svg>...</svg>, label: 'Planejar Dia', path: '/planning/daily' },  // ❌ NÃO PRONTO
  { id: 'checkin', icon: <svg>...</svg>, label: 'Check-in', path: '/checkin' },  // ❌ NÃO PRONTO
]
```

**Menus Simplificados (ambos devem ficar):**
```typescript
// Navigation.tsx (Desktop)
menuItems = [
  { id: 'assessment', icon: '📊', path: '/assessment', tooltip: 'Assessment' },
  { id: 'goals', icon: '🎯', path: '/goals', tooltip: 'Metas' },
]

// NavigationBar.tsx (Mobile)
tabItems = [
  { id: 'assessment', icon: <svg>...</svg>, label: 'Assessment', path: '/assessment' },
  { id: 'goals', icon: <svg>...</svg>, label: 'Metas', path: '/goals' },
]
```

**Requisitos:**
- **Atualizar 2 arquivos:** Navigation.tsx (desktop) + NavigationBar.tsx (mobile)
- Manter apenas: Assessment e Metas (Goals)
- Remover: Dashboard, Timeline, Planning (weekly/daily), Check-in, Retrospective
- Menu limpo e focado

**Você decide:**
- Ordem dos itens (Assessment primeiro ou Goals primeiro?)
- Ícones e labels
- Se mantém algum item adicional

**Restrições:**
- NÃO quebrar navegação existente
- NÃO remover rotas do App.tsx (apenas dos menus)
- Atualizar AMBOS os componentes (desktop + mobile)

---

### **PARTE 2: Evoluir CreateGoalPage.tsx**

**Função de Negócio:**
Adicionar campos de Goal Chunking (motive, context, mode) e implementar fluxos AUTO e MANUAL.

**Campos Novos:**

**1. Motivo (motive):**
- Label: "Por que essa meta importa para você?"
- Tipo: Textarea
- Placeholder: "Ex: Quero melhorar minha saúde e ter mais energia no trabalho"
- Opcional (mas recomendado para modo AUTO)
- Max: 2000 caracteres

**2. Contexto (context):**
- Label: "Como você pretende alcançar?"
- Tipo: Textarea
- Placeholder: "Ex: Vou treinar 3x por semana, 30 minutos por dia, no parque perto de casa"
- Opcional (mas recomendado para modo AUTO)
- Max: 5000 caracteres

**3. Modo (mode):**
- Label: "Como deseja criar o plano?"
- Tipo: Toggle ou Radio Buttons
- Opções:
  - 🤖 AUTO: "Deixar IA criar plano estruturado"
  - ✍️ MANUAL: "Criar plano manualmente depois"
- Default: MANUAL

**Layout Sugerido:**
```
┌─────────────────────────────────────────┐
│ Criar Nova Meta                         │
│                                         │
│ Tipo de Meta: [Dropdown]               │
│ Título: [Input]                         │
│ Prazo: [Date Picker]                    │
│ Prioridade: [1-5]                       │
│                                         │
│ ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━ │
│ 🤖 Goal Chunking com IA                │
│ ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━ │
│                                         │
│ Por que essa meta importa?              │
│ [Textarea - motive]                     │
│                                         │
│ Como você pretende alcançar?            │
│ [Textarea - context]                    │
│                                         │
│ Como deseja criar o plano?              │
│ ( ) 🤖 Deixar IA criar plano           │
│ (•) ✍️ Criar manualmente depois        │
│                                         │
│ [Criar Meta]                            │
└─────────────────────────────────────────┘
```

**Fluxo AUTO:**
1. Usuário preenche: título, prazo, tipo, motivo, contexto
2. Seleciona: modo AUTO
3. Clica: "Criar Meta com IA"
4. Loading: "Criando meta..."
5. Mutation: createObjective (mode=AUTO)
6. Recebe: objectiveId (status=DRAFT)
7. Loading: "Gerando plano estruturado com IA..."
8. Mutation: generateGoalPlan(objectiveId)
9. Recebe: plano completo
10. Redireciona: `/goals/plan-review/{objectiveId}`

**Fluxo MANUAL:**
1. Usuário preenche: título, prazo, tipo
2. Seleciona: modo MANUAL
3. Clica: "Criar Meta"
4. Loading: "Criando meta..."
5. Mutation: createObjective (mode=MANUAL)
6. Recebe: objectiveId (status=ACTIVE)
7. Redireciona: `/goals` (lista de metas)
8. Sucesso: "Meta criada! Você pode adicionar etapas depois."

**Validações:**
- Se mode=AUTO e motive vazio: warning "Recomendamos preencher o motivo para melhor resultado da IA"
- Se mode=AUTO e context vazio: warning "Recomendamos preencher o contexto para melhor resultado da IA"
- Não bloquear criação (apenas avisar)

**Você decide:**
- Layout exato dos campos
- Posição do toggle mode
- Mensagens de loading
- Validações adicionais
- UX/UI

**Restrições:**
- NÃO quebrar CreateGoalPage existente
- NÃO remover campos existentes
- Seguir padrão de design do projeto

---

### **PARTE 3: Criar GoalPlanReviewPage.tsx (Nova Página)**

**Função de Negócio:**
Mostrar plano gerado pela IA e permitir que usuário revise e aprove (ou regenere).

**Rota:**
- Path: `/goals/plan-review/:objectiveId`
- Protegida (requer login)

**Estrutura da Página:**
```
┌─────────────────────────────────────────┐
│ ← Voltar    🎯 Revise seu Plano         │
│                                         │
│ Meta: Correr 5km em 30 minutos         │
│ Prazo: 3 meses (até 09/02/2026)        │
│ Motivo: Melhorar saúde e disposição    │
│                                         │
│ ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━ │
│ 📋 Plano Estruturado                   │
│ ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━ │
│                                         │
│ 📍 Etapa 1: Preparação física          │
│   ✓ Contratar personal trainer         │
│   ✓ Adquirir equipamento adequado      │
│   ✓ Fazer avaliação física             │
│                                         │
│ 📍 Etapa 2: Rotina de treinos          │
│   ✓ Correr 3x por semana               │
│   ✓ Aumentar distância gradualmente    │
│                                         │
│ ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━ │
│ 📊 Indicadores de Progresso (KRs)     │
│ ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━ │
│                                         │
│ • Treinos completados: 36 (100%)       │
│   Meta: 36 treinos | Atual: 0          │
│                                         │
│ [🔄 Regenerar Plano] [✅ Aprovar]      │
└─────────────────────────────────────────┘
```

**Funcionalidades:**
- Query `getGoalWithPlan` ao carregar página
- Mostrar meta (título, prazo, motivo)
- Mostrar etapas com ações (hierárquico)
- Mostrar KRs com pesos (%)
- Botão "Regenerar Plano": 
  - Loading: "Gerando novo plano..."
  - Mutation: generateGoalPlan(objectiveId, regenerate=true)
  - Atualiza página com novo plano
- Botão "Aprovar e Ativar":
  - Sucesso: "Meta criada com plano estruturado!"
  - Redireciona: `/goals` (lista de metas)
- Loading states (carregando plano, regenerando)
- Error states (plano não encontrado, erro ao gerar)

**Você decide:**
- Layout exato (design)
- Componentes (criar componentes separados?)
- Animações e transições
- Mensagens de feedback
- UX/UI

**Restrições:**
- Seguir padrão de design do projeto
- Responsivo (mobile/desktop)
- Acessibilidade

---

### **PARTE 4: Atualizar Mutations GraphQL (queries.ts)**

**Atualizar CREATE_OBJECTIVE:**
```typescript
export const CREATE_OBJECTIVE = gql`
  mutation CreateObjective($input: CreateObjectiveInput!) {
    createObjective(input: $input) {
      id
      objectiveText
      status
      mode  # NOVO
      motive  # NOVO
      context  # NOVO
      # ... outros campos
    }
  }
`;
```

**Criar GENERATE_GOAL_PLAN:**
```typescript
export const GENERATE_GOAL_PLAN = gql`
  mutation GenerateGoalPlan($input: GenerateGoalPlanInput!) {
    generateGoalPlan(input: $input) {
      success
      message
      objective {
        id
        title
        status
        stages {
          id
          title
          description
          orderIndex
          actions {
            id
            title
            description
            status
            orderIndex
          }
        }
        keyResults {
          id
          title
          description
          type
          targetValue
          currentValue
          unit
          weight
          progressPercentage
        }
      }
      errors
    }
  }
`;
```

**Criar GET_GOAL_WITH_PLAN:**
```typescript
export const GET_GOAL_WITH_PLAN = gql`
  query GetGoalWithPlan($id: ID!) {
    getGoalWithPlan(id: $id) {
      id
      title
      motive
      context
      mode
      status
      startDate
      endDate
      stages {
        id
        title
        description
        orderIndex
        actions {
          id
          title
          description
          status
          orderIndex
        }
      }
      keyResults {
        id
        title
        description
        type
        targetValue
        currentValue
        unit
        weight
        progressPercentage
      }
    }
  }
`;
```

---

## ⚠️ RESTRIÇÕES

### **O que NÃO PODE ser alterado:**

1. ❌ **NÃO quebrar CreateGoalPage existente** (apenas evoluir)
2. ❌ **NÃO remover campos existentes** (adicionar novos)
3. ❌ **NÃO quebrar rotas existentes** (App.tsx)
4. ❌ **NÃO alterar BFF ou Backend** (já estão prontos)
5. ❌ **NÃO remover rotas** do App.tsx (apenas do menu)

### **O que DEVE ser preservado:**

1. ✅ **Padrão de design** do projeto (Tailwind CSS)
2. ✅ **Padrão de componentes** (estrutura existente)
3. ✅ **Padrão de mutations** (useMutation do Apollo)
4. ✅ **Padrão de navegação** (useNavigate)
5. ✅ **Responsividade** (mobile-first)
6. ✅ **Error handling** (ErrorModal, NotificationToast)

---

## 📚 DOCUMENTAÇÃO DE REFERÊNCIA

### **Arquivos para Estudar (OBRIGATÓRIO):**

1. **Componente Principal:**
   - `src/components/CreateGoalPage.tsx` (434 linhas)
   - Formulário atual, estrutura, validações

2. **Menus (2 componentes):**
   - `src/components/Navigation.tsx` (237 linhas - menu lateral desktop)
   - `src/components/NavigationBar.tsx` (150 linhas - tab bar mobile)
   - menuItems e tabItems (ambos precisam ser simplificados)

3. **Rotas:**
   - `src/App.tsx` (258 linhas)
   - Rotas existentes

4. **Mutations GraphQL:**
   - `src/graphql/queries.ts`
   - CREATE_OBJECTIVE atual

5. **Hooks:**
   - `src/hooks/useErrorHandler.ts`
   - Padrão de error handling

6. **Diagrama de Sequência (IMPORTANTE!):**
   - `../moversemais-arquitetura/SEQUENCE_DIAGRAMS__Goals-Module.md`
   - **Leia o Diagrama 1 completo!**

7. **BFF GraphQL (referência):**
   - Card 037 (DONE) - Gabriela implementou
   - Mutations e queries disponíveis

---

## 🔧 WORKFLOW

### **1. ESTUDAR (OBRIGATÓRIO - 1 hora)**

```bash
cd moversemais-store-front

# Estudar CreateGoalPage
cat src/components/CreateGoalPage.tsx

# Estudar Menus (2 componentes)
cat src/components/Navigation.tsx
cat src/components/NavigationBar.tsx

# Estudar App.tsx (rotas)
cat src/App.tsx

# Estudar queries GraphQL
cat src/graphql/queries.ts

# Ler diagrama de sequência (IMPORTANTE!)
cat ../moversemais-arquitetura/SEQUENCE_DIAGRAMS__Goals-Module.md

# Ler AGENTS.md
cat AGENTS.md
```

**Perguntas para Responder:**
- Como CreateGoalPage está estruturado?
- Como adicionar campos novos sem quebrar?
- Como implementar toggle mode (AUTO vs MANUAL)?
- Como criar nova página (GoalPlanReviewPage)?
- Como chamar mutations em sequência (create → generate)?

### **2. CRIAR BRANCH**

```bash
git checkout -b feature/goals-chunking-frontend
```

### **3. IMPLEMENTAR (VOCÊ DECIDE COMO)**

**Você é a especialista em React + TypeScript!**

**Ordem sugerida:**
1. Simplificar Navigation.tsx (remover itens não prontos)
2. Atualizar queries.ts (adicionar mutations e query)
3. Atualizar CreateGoalPage.tsx (campos motive, context, mode)
4. Criar GoalPlanReviewPage.tsx (nova página)
5. Atualizar App.tsx (adicionar rota /goals/plan-review/:id)
6. Testar

**Decisões técnicas que você toma:**
- Layout dos campos novos
- Design do toggle mode
- Estrutura do GoalPlanReviewPage
- Componentes auxiliares (StageCard, ActionItem, KRCard?)
- Estados (loading, error, success)
- Animações e transições
- Mensagens de feedback

**Mas DEVE seguir:**
- ✅ Padrão de design do projeto
- ✅ Responsividade
- ✅ Error handling (ErrorModal, NotificationToast)
- ✅ Apollo Client (useMutation, useQuery)

### **4. TESTAR**

**Testes Obrigatórios:**

```bash
# 1. Rodar aplicação
npm run dev

# 2. Fazer login
# http://localhost:5173

# 3. Verificar menus simplificados (desktop + mobile)
# Navigation.tsx (desktop): Deve mostrar apenas Assessment, Metas
# NavigationBar.tsx (mobile): Deve mostrar apenas Assessment, Metas
# NÃO deve mostrar: Dashboard, Timeline, Planning, Check-in, Retrospective

# 4. Testar criação modo MANUAL
# - Acessar /goals/create
# - Preencher: título, prazo, tipo
# - Selecionar: modo MANUAL
# - Clicar: "Criar Meta"
# - Verificar: redireciona para /goals
# - Verificar: meta aparece na lista

# 5. Testar criação modo AUTO
# - Acessar /goals/create
# - Preencher: título, prazo, tipo, motivo, contexto
# - Selecionar: modo AUTO
# - Clicar: "Criar Meta com IA"
# - Verificar: loading "Criando meta..."
# - Verificar: loading "Gerando plano estruturado..."
# - Verificar: redireciona para /goals/plan-review/{id}

# 6. Testar tela de revisão
# - Verificar: plano exibido (etapas, ações, KRs)
# - Verificar: botão "Regenerar Plano" funciona
# - Verificar: loading "Gerando novo plano..."
# - Verificar: plano atualizado (diferente do anterior)
# - Verificar: botão "Aprovar" funciona
# - Verificar: redireciona para /goals
# - Verificar: meta aparece na lista (status=ACTIVE)

# 7. Testar responsividade
# - Mobile (< 768px)
# - Tablet (768px - 1024px)
# - Desktop (> 1024px)

# 8. Testar error handling
# - Criar meta sem preencher campos obrigatórios
# - Tentar gerar plano para meta inexistente
# - Simular erro de rede
```

**Verificações:**
- [ ] Navigation.tsx simplificado (apenas Assessment e Metas)
- [ ] NavigationBar.tsx simplificado (apenas Assessment e Metas)
- [ ] CreateGoalPage tem campos motive, context, mode
- [ ] Modo MANUAL funciona (cria meta ACTIVE)
- [ ] Modo AUTO funciona (cria DRAFT → gera plano → ACTIVE)
- [ ] GoalPlanReviewPage criada e funcional
- [ ] Plano exibido corretamente (etapas, ações, KRs)
- [ ] Regenerar plano funciona
- [ ] Aprovar plano funciona
- [ ] Loading states implementados
- [ ] Error states implementados
- [ ] Responsivo (mobile/desktop)
- [ ] Navegação funciona

### **5. DOCUMENTAR DECISÕES**

Ao finalizar, documente:
- Componentes criados/atualizados
- Layout escolhido
- Estados gerenciados
- Testes realizados
- Dificuldades encontradas

### **6. COMMIT E PUSH**

```bash
git add .
git commit -m "feat(frontend): implementa Goal Chunking UI

- Navigation: menu simplificado (Assessment, Metas)
- CreateGoalPage: campos motive, context, mode
- Fluxo AUTO: criar → gerar plano → revisar → aprovar
- Fluxo MANUAL: criar → pronto
- GoalPlanReviewPage: tela de revisão (nova)
- Mutations: CREATE_OBJECTIVE, GENERATE_GOAL_PLAN
- Query: GET_GOAL_WITH_PLAN
- Loading states (criando, gerando plano)
- Error handling
- Responsivo
- Ref: Card 038"

git push origin feature/goals-chunking-frontend
```

### **7. MOVER PARA VALIDAÇÃO**

```bash
mv Development/TODO/038__Frontend-Goal-Chunking-UI.md \
   Development/VALIDATING/038__Frontend-Goal-Chunking-UI.md
```

---

## ✅ CRITÉRIOS DE VALIDAÇÃO

### **Funcionalidades:**
- [ ] Menu simplificado (apenas Assessment e Metas)
- [ ] CreateGoalPage com campos motive, context, mode
- [ ] Toggle mode (AUTO vs MANUAL) funcional
- [ ] Fluxo MANUAL completo (criar → lista)
- [ ] Fluxo AUTO completo (criar → gerar → revisar → aprovar)
- [ ] GoalPlanReviewPage criada
- [ ] Plano exibido (etapas, ações, KRs)
- [ ] Regenerar plano funciona
- [ ] Aprovar plano funciona
- [ ] Rota /goals/plan-review/:id adicionada

### **UX/UI:**
- [ ] Design consistente com projeto
- [ ] Loading states (criando, gerando)
- [ ] Error states (modal, toast)
- [ ] Success states (notificações)
- [ ] Responsivo (mobile/desktop)
- [ ] Animações suaves

### **Integração:**
- [ ] Mutations GraphQL funcionam
- [ ] Query GraphQL funciona
- [ ] BFF responde corretamente
- [ ] Navegação funciona
- [ ] Backward compatibility (não quebra)

---

## 🚨 TROUBLESHOOTING

### **Problema: Mutation não funciona**
**Solução:**
- Verificar se BFF está rodando (porta 3001)
- Verificar GraphQL Playground (testar mutation)
- Verificar console do navegador (erros)

### **Problema: Plano não carrega**
**Solução:**
- Verificar query GET_GOAL_WITH_PLAN
- Verificar objectiveId na URL
- Verificar console (erros de GraphQL)

### **Problema: Loading infinito**
**Solução:**
- Verificar estados (isLoading, isGenerating)
- Verificar callbacks (onCompleted, onError)
- Verificar logs do BFF

---

## 🎯 EXPECTATIVAS

### **Você é a Especialista em Frontend**

**Lisa, você já implementou várias funcionalidades com sucesso!**

**Agora, você vai criar a interface de Goal Chunking!**

**Eu confio que você:**
- Conhece o Frontend (você mantém essa aplicação!)
- Sabe criar formulários e páginas
- Sabe integrar com GraphQL (Apollo Client)
- Sabe criar UX/UI bonita e funcional

**Referências:**
- CreateGoalPage.tsx (você conhece!)
- Diagrama de sequência (leia!)
- BFF pronto (Gabriela completou!)

**Eu defini O QUE precisa ser feito. Você decide COMO fazer.**

**Este card completa Goal Chunking end-to-end!** 🚀

---

## 📊 OUTPUT ESPERADO

Ao finalizar, documente aqui:

### **Decisões Técnicas Tomadas:**
(Você preenche)

### **Componentes Criados/Atualizados:**
(Liste)

### **Layout Escolhido:**
(Descreva ou screenshot)

### **Estados Gerenciados:**
(Liste estados React)

### **Testes Realizados:**
(Liste cenários testados)

### **Dificuldades Encontradas:**
(Se houver)

### **Melhorias Implementadas:**
(Além do requisitado)

---

**Data de Criação:** 09/11/2025  
**Criado por:** Arquiobaldo (Arquiteto MoverseMais)  
**Contexto:** Módulo Goals - Sprint 2 (Goal Chunking) - Frontend  
**Dependência:** Cards 035, 036, 036.2, 037 ✅ (DONE - Backend + BFF Completos)  
**Diagrama:** SEQUENCE_DIAGRAMS__Goals-Module.md (Diagrama 1 - LEIA!)  
**Próximo:** Sprint 3 - Check-in de Progresso  
**Versão:** 1.0

