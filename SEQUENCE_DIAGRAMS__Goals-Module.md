# 📊 Diagramas de Sequência - Módulo Goals

**Arquiteto:** Arquiobaldo  
**Data:** 08/11/2025  
**Contexto:** PRD 001 (Goal Chunking) + PRD 002 (Check-in de Progresso)

---

## 🎯 Diagrama 1: Criação de Meta com IA (Goal Chunking)

### **Fluxo Completo: PRD 001**

```mermaid
sequenceDiagram
    actor User as Usuário
    participant FE as Frontend<br/>(CreateGoalPage)
    participant BFF as BFF GraphQL<br/>(Next.js)
    participant BE as Backend<br/>(Objective Service)
    participant DB as PostgreSQL
    participant AI as ChatGPT API

    Note over User,AI: FASE 1: CRIAÇÃO DA META

    User->>FE: 1. Preenche formulário<br/>(título, prazo, tipo, motivo, contexto)
    User->>FE: 2. Seleciona modo:<br/>[ ] AUTO (IA gera plano)<br/>[ ] MANUAL (criar depois)
    
    alt Modo AUTO (IA)
        User->>FE: 3a. Clica "Criar Meta com IA"
    
    FE->>BFF: 4. Mutation: createObjective<br/>{title, dates, type, motive, context, mode: AUTO}
    
    BFF->>BE: 5. POST /objectives<br/>Headers: X-User-Id, X-Internal-Service-Key
    
    BE->>DB: 6. INSERT INTO objectives<br/>(title, motive, context, mode=AUTO, status=DRAFT)
    DB-->>BE: 7. Objective created (id: uuid)
    
    BE-->>BFF: 8. Response: {objectiveId, status: DRAFT}
    BFF-->>FE: 9. Response: {objectiveId}
    
    FE->>FE: 10. Redireciona para tela de loading<br/>"Gerando plano estruturado..."

    Note over User,AI: FASE 2: GERAÇÃO DO PLANO PELA IA

    FE->>BFF: 11. Mutation: generateGoalPlan<br/>{objectiveId}
    
    BFF->>BE: 12. POST /objectives/{id}/generate-plan<br/>Headers: X-User-Id, X-Internal-Service-Key
    
    BE->>DB: 13. SELECT * FROM objectives WHERE id=?
    DB-->>BE: 14. Objective data (title, motive, context, dates)
    
    BE->>BE: 15. GoalChunkingService<br/>Monta prompt estruturado
    
    BE->>AI: 16. POST /chat/completions<br/>Prompt: "Transforme meta em plano estruturado<br/>JSON: {stages, actions, keyResults}"
    
    AI-->>BE: 17. JSON Response:<br/>{stages: [...], keyResults: [...]}
    
    BE->>BE: 18. Valida JSON<br/>(mínimo 1 etapa, 3 ações, 1 KR)
    
    Note over BE,DB: FASE 3: SALVAR PLANO NO BANCO

    BE->>DB: 19. BEGIN TRANSACTION
    
    loop Para cada Stage
        BE->>DB: 20. INSERT INTO stages<br/>(objective_id, title, description, order_index)
        DB-->>BE: stage_id
        
        loop Para cada Action
            BE->>DB: 21. INSERT INTO actions<br/>(stage_id, title, description, order_index, status=PENDING)
            DB-->>BE: action_id
        end
    end
    
    loop Para cada KeyResult
        BE->>DB: 22. INSERT INTO key_results<br/>(objective_id, title, type, target_value, weight, progress=0)
        DB-->>BE: kr_id
    end
    
    loop Para cada Vinculação (Action ↔ KR)
        BE->>DB: 23. INSERT INTO action_kr_links<br/>(action_id, kr_id, impact_level, created_by=AI)
    end
    
    BE->>DB: 24. UPDATE objectives SET status=ACTIVE
    BE->>DB: 25. COMMIT TRANSACTION
    
    BE-->>BFF: 26. Response: GoalPlanResponse<br/>{objective, stages, actions, keyResults}
    BFF-->>FE: 27. Response: ObjectiveWithPlan
    
    Note over User,AI: FASE 4: REVISÃO E APROVAÇÃO

    FE->>FE: 28. Redireciona para GoalPlanReviewPage
    FE->>User: 29. Exibe plano estruturado:<br/>- Etapas com ações<br/>- Key Results com pesos<br/>- Botões: Regenerar | Aprovar
    
    alt Usuário quer regenerar
        User->>FE: 30a. Clica "Regenerar Plano"
        FE->>BFF: 31a. Mutation: generateGoalPlan<br/>{objectiveId, regenerate: true}
        Note over BFF,DB: Repete FASE 2 e 3<br/>(deleta plano antigo via CASCADE)
    else Usuário aprova
        User->>FE: 30b. Clica "Aprovar e Ativar"
        FE->>FE: 31b. Redireciona para lista de metas
        FE->>User: 32b. Sucesso: "Meta criada com plano estruturado!"
    end
    
    else Modo MANUAL (Usuário cria)
        User->>FE: 3b. Clica "Criar Meta Manual"
        
        FE->>BFF: 4b. Mutation: createObjective<br/>{title, dates, type, motive, context, mode: MANUAL}
        
        BFF->>BE: 5b. POST /objectives<br/>Headers: X-User-Id, X-Internal-Service-Key
        
        BE->>DB: 6b. INSERT INTO objectives<br/>(title, motive, context, mode=MANUAL, status=ACTIVE)
        DB-->>BE: 7b. Objective created (id: uuid)
        
        BE-->>BFF: 8b. Response: {objectiveId, status: ACTIVE}
        BFF-->>FE: 9b. Response: {objectiveId}
        
        FE->>FE: 10b. Redireciona para lista de metas
        FE->>User: 11b. Sucesso: "Meta criada!<br/>Você pode adicionar etapas e ações depois."
        
        Note over User,FE: FUTURO: Interface para criar<br/>etapas/ações manualmente<br/>(não implementado nesta Sprint)
    end
```

---

## 📊 Diagrama 2: Check-in de Progresso (Atualização de KRs)

### **Fluxo Completo: PRD 002**

```mermaid
sequenceDiagram
    actor User as Usuário
    participant FE as Frontend<br/>(CheckinPage)
    participant BFF as BFF GraphQL<br/>(Next.js)
    participant BE as Backend<br/>(Objective Service)
    participant DB as PostgreSQL
    participant AI as ChatGPT API<br/>(opcional)

    Note over User,AI: FASE 1: LISTAR METAS ATIVAS E KRs

    User->>FE: 1. Acessa tela de Check-in
    
    FE->>BFF: 2. Query: getActiveObjectivesWithKRs<br/>{userId}
    
    BFF->>BE: 3. GET /objectives?status=ACTIVE&userId={id}
    
    BE->>DB: 4. SELECT o.*, kr.* FROM objectives o<br/>LEFT JOIN key_results kr ON kr.objective_id = o.id<br/>WHERE o.user_id = ? AND o.status = 'ACTIVE'
    
    DB-->>BE: 5. Lista de objectives com KRs
    BE-->>BFF: 6. Response: [{objective, keyResults: [...]}]
    BFF-->>FE: 7. Response: ActiveObjectives
    
    FE->>User: 8. Exibe:<br/>- Metas ativas<br/>- KRs de cada meta<br/>- Progresso atual de cada KR

    Note over User,AI: FASE 2: SELECIONAR KRs PARA ATUALIZAR

    User->>FE: 9. Seleciona meta
    FE->>User: 10. Exibe KRs da meta:<br/>☐ KR1: Tempo de corrida (30 min) - 60%<br/>☐ KR2: Frequência cardíaca (150 bpm) - 80%<br/>☐ KR3: Treinos completados (12) - 50%
    
    User->>FE: 11. Marca checkboxes:<br/>☑ KR1: Tempo de corrida<br/>☑ KR3: Treinos completados

    Note over User,AI: FASE 3: REGISTRAR PROGRESSO

    loop Para cada KR selecionado
        FE->>User: 12. Exibe input apropriado<br/>conforme tipo de KR
        
        alt KR tipo NUMERIC
            FE->>User: Input numérico<br/>"Valor atual: [___]"
            User->>FE: Digite: 28 (minutos)
        else KR tipo PERCENTAGE
            FE->>User: Slider 0-100%<br/>"Progresso: [====] 75%"
            User->>FE: Ajusta slider
        else KR tipo BINARY
            FE->>User: Toggle Sim/Não<br/>"Concluído? [Sim] [Não]"
            User->>FE: Seleciona: Sim
        else KR tipo CURRENCY
            FE->>User: Input monetário<br/>"Valor: R$ [___]"
            User->>FE: Digite: 3500
        else KR tipo TIME
            FE->>User: Input tempo<br/>"Tempo: [__]:[__]"
            User->>FE: Digite: 28:30
        end
        
        opt Observações opcionais
            User->>FE: 13. Adiciona nota:<br/>"Melhorei 2 minutos!"
        end
    end
    
    User->>FE: 14. Clica "Registrar Check-in"

    Note over User,AI: FASE 4: PROCESSAR CHECK-IN NO BACKEND

    FE->>BFF: 15. Mutation: checkinProgress<br/>{objectiveId, krUpdates: [{krId, newValue, notes}]}
    
    BFF->>BE: 16. POST /objectives/{id}/checkin<br/>Headers: X-User-Id, X-Internal-Service-Key<br/>Body: {krUpdates: [...]}
    
    BE->>DB: 17. BEGIN TRANSACTION
    
    loop Para cada KR atualizado
        BE->>DB: 18. SELECT * FROM key_results WHERE id = ?
        DB-->>BE: 19. KR atual (current_value, progress_percentage)
        
        BE->>BE: 20. Calcula novo progresso:<br/>progress = (newValue / targetValue) × 100
        
        BE->>DB: 21. INSERT INTO checkins<br/>(objective_id, kr_id, previous_value, new_value,<br/>previous_progress, new_progress, user_notes, created_by_user_id)
        
        BE->>DB: 22. UPDATE key_results<br/>SET current_value = ?, progress_percentage = ?,<br/>updated_at = NOW()<br/>WHERE id = ?
        
        opt Se progresso = 100%
            BE->>DB: 23. UPDATE key_results<br/>SET completed_at = NOW()
        end
    end
    
    Note over BE,DB: FASE 5: CALCULAR PROGRESSO DA META

    BE->>DB: 24. SELECT * FROM key_results<br/>WHERE objective_id = ?
    DB-->>BE: 25. Todos os KRs da meta
    
    BE->>BE: 26. Calcula progresso ponderado:<br/>total = Σ(progress × weight)<br/>Ex: (60% × 0.5) + (50% × 0.2) = 40%
    
    BE->>DB: 27. UPDATE objectives<br/>SET completion_percentage = ?,<br/>progress_status = ?,<br/>updated_at = NOW()<br/>WHERE id = ?
    
    opt Se completion_percentage = 100%
        BE->>DB: 28. UPDATE objectives<br/>SET status = CONCLUDED,<br/>completed_at = NOW()
    end
    
    BE->>DB: 29. COMMIT TRANSACTION

    Note over BE,AI: FASE 6: FEEDBACK DA IA (OPCIONAL)

    opt Gerar feedback com IA
        BE->>BE: 30. Monta contexto:<br/>- KRs atualizados<br/>- Progresso anterior vs novo<br/>- Histórico de check-ins
        
        BE->>AI: 31. POST /chat/completions<br/>Prompt: "Gere feedback motivacional<br/>sobre progresso do usuário"
        
        AI-->>BE: 32. Feedback: "Excelente! Você melhorou<br/>2 minutos no tempo de corrida.<br/>Continue assim!"
        
        BE->>DB: 33. UPDATE checkins<br/>SET ai_feedback = ?<br/>WHERE id IN (...)
    end
    
    BE-->>BFF: 34. Response: CheckinResponse<br/>{success, newProgress, feedback}
    BFF-->>FE: 35. Response: CheckinResult
    
    Note over User,AI: FASE 7: EXIBIR RESULTADO

    FE->>User: 36. Exibe feedback:<br/>✅ Check-in registrado!<br/>📊 Progresso da meta: 40% → 45%<br/>💬 "Excelente! Continue assim!"
    
    FE->>User: 37. Atualiza visualização:<br/>- Barra de progresso da meta<br/>- Progresso individual dos KRs<br/>- Histórico de check-ins
```

---

## 📋 **ANÁLISE DOS DIAGRAMAS**

### **Diagrama 1 - Goal Chunking (PRD 001)**

**2 Fluxos Identificados:**

#### **Fluxo A: Modo AUTO (IA gera plano)**
1. ✅ **Criação da Meta** - Frontend → BFF → Backend → DB (mode=AUTO, status=DRAFT)
2. ✅ **Geração do Plano pela IA** - Backend → ChatGPT → Parsing
3. ✅ **Salvar Plano no Banco** - Transaction com stages, actions, key_results, action_kr_links
4. ✅ **Revisão e Aprovação** - Frontend exibe plano, usuário aprova ou regenera
5. ✅ **Ativação** - status=DRAFT → status=ACTIVE

#### **Fluxo B: Modo MANUAL (Usuário cria depois)**
1. ✅ **Criação da Meta** - Frontend → BFF → Backend → DB (mode=MANUAL, status=ACTIVE)
2. ✅ **Meta criada SEM plano** - Apenas objective, sem stages/actions/KRs
3. ⏸️ **Interface Manual (FUTURO)** - Usuário cria etapas/ações depois
4. ⏸️ **Não implementado nesta Sprint** - Foco em modo AUTO

**Pontos Críticos:**
- **Modo AUTO:** Meta criada com `status=DRAFT` → gera plano → aprova → `status=ACTIVE`
- **Modo MANUAL:** Meta criada com `status=ACTIVE` → SEM plano (stages/actions/KRs vazios)
- Plano gerado em transação (tudo ou nada)
- Vinculações action ↔ KR criadas automaticamente pela IA
- Regenerar = deletar plano antigo (CASCADE) + gerar novo

---

### **Diagrama 2 - Check-in (PRD 002)**

**Fases Identificadas:**
1. ✅ **Listar Metas Ativas e KRs** - Query com JOIN
2. ✅ **Selecionar KRs para Atualizar** - Usuário escolhe quais KRs
3. ✅ **Registrar Progresso** - Input apropriado por tipo de KR
4. ✅ **Processar Check-in no Backend** - Transaction com checkins + update key_results
5. ✅ **Calcular Progresso da Meta** - Progresso ponderado (Σ progress × weight)
6. ✅ **Feedback da IA (Opcional)** - ChatGPT gera feedback motivacional
7. ✅ **Exibir Resultado** - Frontend atualiza visualização

**Pontos Críticos:**
- Check-in registra histórico imutável (tabela `checkins`)
- Progresso calculado: `(newValue / targetValue) × 100`
- Progresso da meta: média ponderada dos KRs
- Feedback da IA é opcional (pode ser assíncrono)

---

## 🚨 **DESCOBERTAS IMPORTANTES**

### **1. Status da Meta (Objective)**

**PRD 001 sugere:**
- Meta criada com `status=DRAFT` (antes de gerar plano)
- Após gerar plano: `status=ACTIVE`

**Problema:**
- Tabela `objectives` atual tem `status`: ACTIVE, CONCLUDED, ARCHIVED
- **NÃO TEM `DRAFT`!**

**Opções:**

**Opção A: Adicionar DRAFT ao enum Status**
```sql
-- Migration V031
ALTER TABLE objectives DROP CONSTRAINT IF EXISTS chk_status;
ALTER TABLE objectives ADD CONSTRAINT chk_status 
    CHECK (status IN ('DRAFT', 'ACTIVE', 'CONCLUDED', 'ARCHIVED'));
```
- ✅ Segue PRD 001 fielmente
- ✅ Separa meta em criação (DRAFT) de meta ativa (ACTIVE)
- ❌ Altera enum existente (pode impactar código)

**Opção B: Usar ACTIVE para ambos**
- Modo AUTO: criar como ACTIVE, gerar plano, manter ACTIVE
- Modo MANUAL: criar como ACTIVE, sem plano
- ✅ Não altera enum existente
- ✅ Mais simples
- ❌ Não diferencia meta em criação de meta ativa

**Opção C: Usar campo separado (has_plan)**
- Adicionar campo `has_plan: BOOLEAN` em objectives
- Modo AUTO: has_plan=false → gera plano → has_plan=true
- Modo MANUAL: has_plan=false (sem plano)
- ✅ Não altera enum
- ✅ Indica se meta tem plano gerado
- ❌ Campo adicional

**DECISÃO NECESSÁRIA: Eduardo, qual opção você prefere?**

---

### **2. Ordem de Criação (Goal Chunking)**

**PRD 001 sugere:**
1. Criar Objective (DRAFT)
2. Gerar plano com IA
3. Salvar stages, actions, key_results, action_kr_links
4. Ativar meta (ACTIVE)

**Card 035 atual propõe:**
- Endpoint `POST /objectives/{id}/generate-plan`
- Assume que Objective já existe

**Isso está correto!** ✅

---

### **3. Tipos de Input por KR (Check-in)**

**PRD 002 especifica:**
- NUMERIC: input numérico
- PERCENTAGE: slider 0-100%
- BINARY: toggle Sim/Não
- CURRENCY: input monetário
- TIME: input tempo

**Frontend precisa:**
- Renderizar input apropriado por tipo
- Validar valores (ex: PERCENTAGE 0-100)

---

### **4. Cálculo de Progresso (Check-in)**

**PRD 002 especifica:**
- Progresso do KR: `(currentValue / targetValue) × 100`
- Progresso da meta: `Σ(progressKR × weight)`

**Exemplo:**
```
KR1: 60% × 0.50 = 30%
KR2: 80% × 0.30 = 24%
KR3: 50% × 0.20 = 10%
Total: 30% + 24% + 10% = 64%
```

**Backend precisa:**
- Calcular progresso de cada KR atualizado
- Recalcular progresso total da meta
- Atualizar `completion_percentage` em objectives

---

## ✅ **CARDS PRECISAM SER AJUSTADOS?**

### **Card 035 (Backend - Goal Chunking):**

**Ajustes Necessários:**
1. ✅ Adicionar discussão sobre `status=DRAFT` vs `ACTIVE`
2. ✅ Clarificar que endpoint assume Objective já existe
3. ✅ Especificar transação (tudo ou nada)
4. ✅ Especificar que regenerate deleta plano antigo (CASCADE)

**Está 90% correto!** Apenas precisa de clarificações.

---

### **Card 036 (BFF - Goal Chunking):**

**Ainda não criado.**

**Deve incluir:**
- Mutation `createObjective` (criar meta)
- Mutation `generateGoalPlan` (gerar plano)
- Query `getGoalWithPlan` (buscar meta com plano)
- Tratamento de status DRAFT vs ACTIVE

---

### **Card 037 (Frontend - Goal Chunking):**

**Ainda não criado.**

**Deve incluir:**
- CreateGoalPage (campos motive, context, mode)
- Fluxo: criar meta → gerar plano → revisar → aprovar
- GoalPlanReviewPage (visualização + aprovação)

---

### **Cards Futuros (Check-in):**

**Card 038 (Backend - Check-in):**
- Endpoint `POST /objectives/{id}/checkin`
- Salvar em `checkins` (histórico)
- Atualizar `key_results` (current_value, progress_percentage)
- Calcular progresso da meta (ponderado)
- Feedback da IA (opcional)

**Card 039 (BFF - Check-in):**
- Mutation `checkinProgress`
- Query `getActiveObjectivesWithKRs`

**Card 040 (Frontend - Check-in):**
- CheckinPage (listar metas + KRs)
- Inputs apropriados por tipo de KR
- Feedback visual de progresso

---

## 🎯 **PRÓXIMOS PASSOS**

**Eduardo, baseado nos diagramas:**

1. ✅ **Revisar Card 035** (Backend - Goal Chunking)?
   - Adicionar clarificações sobre status, transação, regenerate

2. ✅ **Criar Card 036** (BFF - Goal Chunking)?
   - Mutations: createObjective, generateGoalPlan
   - Query: getGoalWithPlan

3. ✅ **Criar Card 037** (Frontend - Goal Chunking)?
   - CreateGoalPage + GoalPlanReviewPage

4. ⏸️ **Deixar Check-in para depois?**
   - Cards 038-040 (Backend + BFF + Frontend)

**O que você prefere fazer agora?** 🤔

---

**Data de Criação:** 08/11/2025  
**Criado por:** Arquiobaldo (Arquiteto MoverseMais)  
**Contexto:** Análise de PRD 001 e PRD 002  
**Versão:** 1.0

