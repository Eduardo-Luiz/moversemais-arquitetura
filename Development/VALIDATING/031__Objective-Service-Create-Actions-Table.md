# 🎯 Card 031 - Objective Service: Criar Tabela Actions (Ações)

**Agente Responsável:** Osvaldo  
**Microserviço:** moversemais-objective  
**Prioridade:** Alta  
**Status:** TODO  
**Estimativa:** 2 horas

---

## 📋 CONTEXTO

### **Situação Atual**
Card 030 criou tabela `stages` (etapas) com sucesso. Agora temos estrutura para quebrar metas em fases sequenciais. Próximo passo é criar tabela `actions` (ações executáveis) dentro de cada etapa.

### **Problema Identificado**
Para implementar **PRD 001 (Goal Chunking)**, precisamos que cada etapa contenha **ações executáveis concretas**. Atualmente:
- ✅ Tabela `stages` existe (Card 030)
- ❌ Não existe tabela `actions` (ações)
- ❌ Etapas estão vazias (sem tarefas)
- ❌ IA não pode gerar ações executáveis
- ❌ Usuário não tem checklist de tarefas

**Exemplo do que queremos:**
```
Etapa 1: Avaliar condicionamento físico atual
├─ Ação 1: Fazer corrida teste de 2km
├─ Ação 2: Registrar tempo e frequência cardíaca
└─ Ação 3: Avaliar nível de cansaço
```

### **Solução Proposta**
Criar tabela `actions` (ações) com relacionamento **1 Stage → N Actions**, permitindo que cada etapa contenha lista de tarefas executáveis geradas pela IA ou criadas manualmente.

### **Onde se Encaixa na Arquitetura**
```
Módulo Goals
├─ objectives (existente) ✅
├─ stages (Card 030) ✅
├─ actions (este card) ← CRIAR
├─ key_results (próximo card)
└─ checkins (próximo card)
```

### **Impacto se Não For Feito**
- Etapas sem ações executáveis
- IA não pode gerar checklist de tarefas
- Usuário não tem o que fazer (plano abstrato)
- Goal Chunking incompleto

---

## 🎯 REQUISITOS OBRIGATÓRIOS

### **1. Migration Flyway**

**Criar arquivo:** `src/main/resources/db/migration/V026__create_actions_table.sql`

**Tabela `actions` deve conter:**
- UUID como primary key (padrão do projeto)
- Foreign key para `stages(id)` com ON DELETE CASCADE
- Campos obrigatórios:
  - title: Título da ação (obrigatório)
  - description: Descrição detalhada (opcional)
  - status: Status da ação (PENDING, IN_PROGRESS, COMPLETED)
  - order_index: Ordem dentro da etapa (1, 2, 3...)
  - Timestamps (created_at, updated_at)
- Campos opcionais (você decide):
  - completed_at: Quando foi concluída
  - due_date: Prazo da ação
  - estimated_effort: Esforço estimado

**Padrão do Projeto (seguir V025__create_stages_table.sql):**
- `CREATE TABLE IF NOT EXISTS` (idempotente)
- `gen_random_uuid()` para UUID
- `TIMESTAMP` + `DEFAULT NOW()`
- Índices com `IF NOT EXISTS`
- Nomenclatura snake_case
- Comentários SQL (COMMENT ON)
- Verificação de integridade (DO $$)

### **2. Relacionamento**

**Foreign Key obrigatória:**
- `stage_id` REFERENCES `stages(id)` ON DELETE CASCADE
- Se etapa deletada, ações também são deletadas

**Ordem Sequencial:**
- `order_index` INT NOT NULL
- Permite reordenação de ações dentro da etapa
- IA define ordem ao gerar plano

### **3. Status da Ação**

**Enum Status (obrigatório):**
- PENDING (padrão - ainda não iniciada)
- IN_PROGRESS (em andamento)
- COMPLETED (concluída)

**Implementação (você decide):**
- Constraint CHECK com enum values
- Ou apenas VARCHAR sem constraint
- Seguir padrão de V1 (objectives tem constraint CHECK para status)

### **4. Índices de Performance**

**Obrigatórios:**
- `idx_actions_stage_id` - Buscar ações de uma etapa
- `idx_actions_status` - Filtrar por status
- `idx_actions_order` - Ordenar ações

**Opcional (você decide):**
- Índice composto (stage_id, order_index)
- Índice composto (stage_id, status)
- Índice em completed_at (se adicionar)

---

## ⚠️ RESTRIÇÕES

### **O que NÃO PODE ser alterado:**

1. ❌ **NÃO alterar tabela `objectives`**
2. ❌ **NÃO alterar tabela `stages`** (Card 030)
3. ❌ **NÃO alterar tabela `assessments`**
4. ❌ **NÃO alterar migrations existentes** (V1-V025)
5. ❌ **NÃO criar entities JPA ainda** (apenas migration SQL)
6. ❌ **NÃO criar services ou controllers** (apenas tabela)

### **O que DEVE ser preservado:**

1. ✅ **Padrão de nomenclatura** (snake_case)
2. ✅ **Padrão de migrations** (V{numero}__descricao.sql)
3. ✅ **Padrão de UUID** (gen_random_uuid())
4. ✅ **Padrão de timestamps** (TIMESTAMP + NOW())
5. ✅ **Padrão de índices** (IF NOT EXISTS)
6. ✅ **Padrão de comentários** (COMMENT ON)
7. ✅ **Padrão de verificação** (DO $$ block)

---

## 📚 DOCUMENTAÇÃO DE REFERÊNCIA

### **Arquivos para Estudar (OBRIGATÓRIO):**

1. **Padrão de Migrations (CRÍTICO):**
   - `src/main/resources/db/migration/V025__create_stages_table.sql` - **REFERÊNCIA PRINCIPAL**
   - Card 030 acabou de criar - seguir EXATAMENTE esse padrão
   - Osvaldo já conhece o padrão (você criou V025!)

2. **Padrão de Status (Enum):**
   - `src/main/resources/db/migration/V1__create_objectives_table.sql`
   - Linhas 36-50: Como criar constraint CHECK para enum
   - Bloco `DO $$` para validar status

3. **Análise Arquitetural:**
   - `../moversemais-arquitetura/ANALYSIS__Goals-Module-Architecture.md`
   - Seção "TABELA 2: actions"

4. **Documentação:**
   - `../moversemais-objective/AGENTS.md`
   - `../moversemais-arquitetura/AGENTS.md`

### **Cards Relacionados:**
- Card 030: Tabela Stages ✅ (DONE - pré-requisito)
- Card 032-035: Próximos cards (dependem deste)

---

## 🔧 WORKFLOW

### **1. ESTUDAR (OBRIGATÓRIO - 10 minutos)**

```bash
cd moversemais-objective

# Estudar V025 que VOCÊ criou (Card 030)
cat src/main/resources/db/migration/V025__create_stages_table.sql

# Estudar padrão de enum/status em V1
cat src/main/resources/db/migration/V1__create_objectives_table.sql | grep -A 15 "chk_status"

# Ler análise arquitetural
cat ../moversemais-arquitetura/ANALYSIS__Goals-Module-Architecture.md | grep -A 40 "TABELA 2"

# Ler AGENTS.md
cat AGENTS.md
```

**Perguntas para Responder Antes de Implementar:**
- Migration V026 (próxima após V025)?
- Status: constraint CHECK ou apenas VARCHAR?
- Campos adicionais: completed_at? due_date? estimated_effort?
- Índices: quais além dos 3 obrigatórios?

### **2. CRIAR BRANCH**

```bash
git checkout -b feature/goals-actions-table
```

### **3. IMPLEMENTAR (VOCÊ DECIDE COMO)**

**Criar arquivo:**
- Nome: `V026__create_actions_table.sql`
- Localização: `src/main/resources/db/migration/`

**Você decide:**
- Se adiciona `completed_at` (quando foi concluída)
- Se adiciona `due_date` (prazo da ação)
- Se adiciona `estimated_effort` (esforço estimado)
- Constraint CHECK para status ou não
- Ordem dos campos
- Comentários SQL
- Validações adicionais

**Mas DEVE seguir:**
- ✅ Padrão de V025__create_stages_table.sql (que você criou!)
- ✅ UUID com gen_random_uuid()
- ✅ Foreign key com ON DELETE CASCADE
- ✅ Índices com IF NOT EXISTS
- ✅ Comentários SQL (COMMENT ON)
- ✅ Verificação de integridade (DO $$)

### **4. TESTAR**

**Testes Obrigatórios:**

```bash
# 1. Rodar aplicação (Flyway executa migration)
./gradlew bootRun

# Verificar logs:
# "Migrating schema to version 026 - create actions table"
# "Successfully applied 1 migration"

# 2. Conectar ao PostgreSQL
docker exec moversemais-postgres psql -U developer -d moversemais_objective

# 3. Verificar tabela criada
\d actions

# Esperado:
# - Colunas corretas
# - Foreign key para stages
# - Índices criados
# - Constraint de status (se adicionou)

# 4. Testar foreign key (inserir action)
INSERT INTO actions (stage_id, title, status, order_index)
VALUES (
  (SELECT id FROM stages LIMIT 1),
  'Ação Teste',
  'PENDING',
  1
);

# Esperado: Sucesso (se stage existe)

# 5. Testar status (se constraint CHECK)
INSERT INTO actions (stage_id, title, status, order_index)
VALUES (
  (SELECT id FROM stages LIMIT 1),
  'Ação Teste 2',
  'STATUS_INVALIDO',
  2
);

# Esperado: Erro (se constraint CHECK ativo)

# 6. Testar ON DELETE CASCADE
# Deletar stage e verificar se actions foram deletadas junto

# 7. Testar múltiplas ações na mesma etapa
INSERT INTO actions (stage_id, title, status, order_index)
VALUES 
  ((SELECT id FROM stages LIMIT 1), 'Ação 1', 'PENDING', 1),
  ((SELECT id FROM stages LIMIT 1), 'Ação 2', 'IN_PROGRESS', 2),
  ((SELECT id FROM stages LIMIT 1), 'Ação 3', 'COMPLETED', 3);

# Verificar ordenação por order_index
SELECT * FROM actions WHERE stage_id = (SELECT id FROM stages LIMIT 1) ORDER BY order_index;

# 8. Limpeza
DELETE FROM actions WHERE title LIKE 'Ação Teste%';
```

**Verificações:**
- [ ] Migration V026 executa sem erro
- [ ] Tabela `actions` criada
- [ ] Foreign key para stages funciona
- [ ] Índices criados
- [ ] Status PENDING/IN_PROGRESS/COMPLETED funcionam
- [ ] Constraint CHECK valida (se adicionou)
- [ ] ON DELETE CASCADE funciona
- [ ] order_index funciona
- [ ] Múltiplas ações na mesma etapa

### **5. DOCUMENTAR DECISÕES**

Ao final do card, documente:
- Estrutura SQL final
- Campos adicionais escolhidos (completed_at, due_date, etc.)
- Constraint CHECK para status (sim ou não)
- Índices adicionais (se houver)
- Testes realizados
- Dificuldades encontradas

### **6. COMMIT E PUSH**

```bash
git add src/main/resources/db/migration/V026__create_actions_table.sql
git commit -m "feat(objective-service): cria tabela actions para Goal Chunking

- Tabela actions (ações) para etapas de metas
- Relacionamento 1:N com stages
- Foreign key com ON DELETE CASCADE
- Status: PENDING, IN_PROGRESS, COMPLETED
- Índices para performance (stage_id, status, order_index)
- Suporta Goal Chunking (PRD 001)
- Ref: Card 031"

git push origin feature/goals-actions-table
```

### **7. MOVER PARA VALIDAÇÃO**

```bash
mv Development/TODO/031__Objective-Service-Create-Actions-Table.md \
   Development/VALIDATING/031__Objective-Service-Create-Actions-Table.md
```

---

## ✅ CRITÉRIOS DE VALIDAÇÃO

### **Funcionalidades:**
- [ ] Migration V026 criada
- [ ] Tabela `actions` existe no banco
- [ ] Colunas corretas (id, stage_id, title, description, status, order_index, timestamps)
- [ ] Foreign key para stages funciona
- [ ] ON DELETE CASCADE funciona
- [ ] Índices criados (mínimo 3)
- [ ] Status PENDING/IN_PROGRESS/COMPLETED funcionam

### **Padrão:**
- [ ] Seguiu estrutura de V025__create_stages_table.sql
- [ ] UUID com gen_random_uuid()
- [ ] Nomenclatura snake_case
- [ ] IF NOT EXISTS em índices
- [ ] Timestamps com padrão do projeto
- [ ] Comentários SQL (COMMENT ON)
- [ ] Verificação de integridade (DO $$)

### **Testes:**
- [ ] Flyway executou migration
- [ ] Tabela visível no PostgreSQL (\d actions)
- [ ] Insert funciona
- [ ] Foreign key valida
- [ ] Cascade delete funciona
- [ ] Múltiplas ações na mesma etapa
- [ ] Ordenação por order_index funciona

---

## 🚨 TROUBLESHOOTING

### **Problema: Migration não executa**
**Solução:**
- Verificar numeração (V026)
- Verificar sintaxe SQL
- Logs: `./gradlew bootRun | grep Flyway`

### **Problema: Foreign key error**
**Solução:**
- Verificar se tabela `stages` existe (Card 030)
- Sintaxe: `REFERENCES stages(id)`

### **Problema: Status constraint error**
**Solução:**
- Se usar constraint CHECK, verificar sintaxe
- Bloco `DO $$` para evitar duplicação
- Ou deixar VARCHAR sem constraint (você decide)

### **Problema: Índice não criado**
**Solução:**
- Verificar `IF NOT EXISTS`
- Verificar nome único do índice

---

## 🎯 EXPECTATIVAS

### **Você é o Especialista em Backend**

**Osvaldo, você acabou de criar V025 (Card 030) com excelência.** Eu confio que você:

1. ✅ **Conhece o padrão** (você criou V025!)
2. ✅ **Sabe estruturar tabelas** perfeitamente
3. ✅ **Domina PostgreSQL e Flyway**

**Por isso:**
- Siga o MESMO padrão de V025 (que você criou)
- Estrutura similar (FK, índices, comentários, verificação)
- Tome decisões técnicas fundamentadas
- Adicione campos que julgar úteis (completed_at, due_date, etc.)

**Eu defini O QUE precisa ser feito. Você decide COMO fazer.**

**Campos obrigatórios:**
- id, stage_id, title, status, order_index, timestamps

**Campos opcionais (você decide se adiciona):**
- description, completed_at, due_date, estimated_effort, priority, etc.

---

## 📊 OUTPUT ESPERADO

Ao finalizar, documente aqui:

### **Decisões Técnicas Tomadas:**

1. **Migration V026** - Seguindo numeração sequencial após V025
2. **Padrão de V025** - Seguiu EXATAMENTE o padrão que criei no Card 030
3. **Campos Adicionais** - Adicionei 3 campos opcionais úteis:
   - `completed_at` - Timestamp de conclusão da ação
   - `due_date` - Prazo estimado para conclusão
   - `estimated_effort` - Esforço estimado (ex: "30min", "2h")
4. **Constraint CHECK** - Validação de status (PENDING, IN_PROGRESS, COMPLETED)
5. **8 Índices** - Performance otimizada:
   - Simples: stage_id, status, order_index, completed_at, due_date, created_at
   - Compostos: (stage_id, order_index), (stage_id, status)
6. **Status DEFAULT** - `PENDING` como padrão para novas ações
7. **Comentários SQL Completos** - Documentação inline para cada coluna
8. **Verificação de Integridade** - Bloco `DO $$` valida 11 colunas, 8 índices e 1 constraint

### **Estrutura SQL Final:**

```sql
-- V026__create_actions_table.sql
-- Criar tabela actions (ações) para Goal Chunking (PRD 001)
-- Card 031 - Objective Service: Criar Tabela Actions
-- Data: 08/11/2025

-- Tabela de ações (actions) para checklist executável dentro de cada etapa
CREATE TABLE IF NOT EXISTS actions (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    stage_id UUID NOT NULL,
    title VARCHAR(500) NOT NULL,
    description TEXT,
    status VARCHAR(20) NOT NULL DEFAULT 'PENDING',
    order_index INTEGER NOT NULL CHECK (order_index >= 1),
    completed_at TIMESTAMP,
    due_date TIMESTAMP,
    estimated_effort VARCHAR(50),
    created_at TIMESTAMP DEFAULT NOW(),
    updated_at TIMESTAMP DEFAULT NOW(),
    
    -- Foreign key com CASCADE para LGPD e limpeza automática
    CONSTRAINT fk_actions_stage 
        FOREIGN KEY (stage_id) 
        REFERENCES stages(id) 
        ON DELETE CASCADE
);

-- Índices para performance
CREATE INDEX IF NOT EXISTS idx_actions_stage_id ON actions(stage_id);
CREATE INDEX IF NOT EXISTS idx_actions_status ON actions(status);
CREATE INDEX IF NOT EXISTS idx_actions_order ON actions(order_index);
CREATE INDEX IF NOT EXISTS idx_actions_stage_order ON actions(stage_id, order_index);
CREATE INDEX IF NOT EXISTS idx_actions_stage_status ON actions(stage_id, status);
CREATE INDEX IF NOT EXISTS idx_actions_completed_at ON actions(completed_at);
CREATE INDEX IF NOT EXISTS idx_actions_due_date ON actions(due_date);
CREATE INDEX IF NOT EXISTS idx_actions_created_at ON actions(created_at);

-- Constraint para validar status
DO $$ 
BEGIN
    IF NOT EXISTS (SELECT 1 FROM pg_constraint WHERE conname = 'chk_actions_status') THEN
        ALTER TABLE actions ADD CONSTRAINT chk_actions_status
            CHECK (status IN ('PENDING', 'IN_PROGRESS', 'COMPLETED'));
    END IF;
END $$;

-- Comentários para documentação
COMMENT ON TABLE actions IS 'Ações executáveis dentro de cada etapa (Goal Chunking - PRD 001)';
COMMENT ON COLUMN actions.id IS 'Identificador único da ação';
COMMENT ON COLUMN actions.stage_id IS 'Referência para a etapa (stage) pai';
COMMENT ON COLUMN actions.title IS 'Título da ação (obrigatório)';
COMMENT ON COLUMN actions.description IS 'Descrição detalhada da ação (opcional)';
COMMENT ON COLUMN actions.status IS 'Status da ação: PENDING (padrão), IN_PROGRESS, COMPLETED';
COMMENT ON COLUMN actions.order_index IS 'Ordem de execução da ação dentro da etapa (1, 2, 3...)';
COMMENT ON COLUMN actions.completed_at IS 'Data e hora em que a ação foi concluída';
COMMENT ON COLUMN actions.due_date IS 'Prazo estimado para conclusão da ação';
COMMENT ON COLUMN actions.estimated_effort IS 'Esforço estimado (ex: "30min", "2h", "1 dia")';
COMMENT ON COLUMN actions.created_at IS 'Data de criação da ação';
COMMENT ON COLUMN actions.updated_at IS 'Data da última atualização da ação';

-- Verificação de integridade
DO $$
DECLARE
    column_count INTEGER;
    index_count INTEGER;
    constraint_count INTEGER;
BEGIN
    -- Verificar colunas criadas
    SELECT COUNT(*) INTO column_count 
    FROM information_schema.columns 
    WHERE table_name = 'actions' 
    AND column_name IN ('id', 'stage_id', 'title', 'description', 'status', 'order_index', 
                        'completed_at', 'due_date', 'estimated_effort', 'created_at', 'updated_at');
    
    IF column_count != 11 THEN
        RAISE EXCEPTION 'Falha ao criar tabela actions: % colunas criadas (esperado: 11)', column_count;
    END IF;
    
    -- Verificar índices criados
    SELECT COUNT(*) INTO index_count 
    FROM pg_indexes 
    WHERE tablename = 'actions' 
    AND indexname LIKE 'idx_actions_%';
    
    IF index_count < 8 THEN
        RAISE EXCEPTION 'Falha ao criar índices: % índices criados (esperado: >= 8)', index_count;
    END IF;
    
    -- Verificar constraint de status
    SELECT COUNT(*) INTO constraint_count
    FROM pg_constraint
    WHERE conname = 'chk_actions_status';
    
    IF constraint_count != 1 THEN
        RAISE EXCEPTION 'Falha ao criar constraint de status: % constraints criados (esperado: 1)', constraint_count;
    END IF;
    
    RAISE NOTICE 'Tabela actions criada com sucesso: % colunas, % índices, % constraints', column_count, index_count, constraint_count;
END $$;
```

### **Testes Realizados:**

✅ **1. Migration Executada**
- Flyway aplicou V026 com sucesso
- Tabela `actions` criada no banco
- Comando: `./gradlew bootRun`

✅ **2. Estrutura Verificada**
- 11 colunas criadas corretamente
- 9 índices criados (8 + primary key)
- Foreign key configurada
- Constraint de status ativa
- Comando: `\d actions` no PostgreSQL

✅ **3. Insert Funcional**
- Inserção de 3 actions com sucesso
- UUID gerado automaticamente
- Status DEFAULT 'PENDING' funcionando
- Timestamps populados automaticamente
- Campos opcionais (due_date, estimated_effort) funcionando

✅ **4. Foreign Key Validada**
- Relacionamento com stages funciona
- Não permite stage_id inválido

✅ **5. Ordenação por order_index**
- 3 ações criadas com order_index 1, 2, 3
- Busca ordenada funciona perfeitamente
- SQL: `ORDER BY order_index`

✅ **6. Constraint CHECK de Status**
- Status válidos funcionam: PENDING, IN_PROGRESS, COMPLETED
- Status inválido rejeitado com erro
- Mensagem: "violates check constraint chk_actions_status"

✅ **7. Filtro por Status**
- Índice `idx_actions_status` funciona
- Busca por status COMPLETED retornou 1 resultado

✅ **8. ON DELETE CASCADE**
- Stage deletado → 3 actions deletadas automaticamente
- Limpeza automática funcionando (LGPD)

✅ **9. Índices Verificados**
- 9 índices listados no PostgreSQL
- Todos com `IF NOT EXISTS`
- Índices compostos criados corretamente

### **Dificuldades Encontradas:**

1. **Erro de validação Flyway - Migration V025 não encontrada**
   - Problema: Branch `feature/goals-actions-table` criada antes do merge de V025
   - Solução: Merge correto de `feature/goals-stages-table` → `main` → nova branch
   - Lição: Sempre mergear branches anteriores na main ANTES de criar novas branches

2. **Teste inicial falhou - Sem stages no banco**
   - Problema: Tentei inserir action sem stage existente
   - Solução: Criar stage de teste primeiro
   - Validou: Foreign key está funcionando corretamente

### **Melhorias Implementadas:**

1. **3 Campos Adicionais Úteis** - `completed_at`, `due_date`, `estimated_effort` para UX melhor
2. **8 Índices de Performance** - Além dos 3 obrigatórios, adicionei 5 extras:
   - `idx_actions_completed_at` - Filtrar ações concluídas por data
   - `idx_actions_due_date` - Ordenar por prazo
   - `idx_actions_created_at` - Ordenação temporal
   - `idx_actions_stage_order` - Busca ordenada (composto)
   - `idx_actions_stage_status` - Filtro por etapa e status (composto)
3. **Status DEFAULT** - `PENDING` como padrão para melhor UX
4. **Constraint CHECK** - Validação robusta de status
5. **Comentários SQL Completos** - Documentação inline para cada coluna
6. **Verificação Tripla** - Bloco `DO $$` valida colunas, índices E constraints

---

**Data de Criação:** 08/11/2025  
**Criado por:** Arquiobaldo (Arquiteto MoverseMais)  
**Contexto:** Módulo Goals - PRD 001 (Goal Chunking)  
**Dependência:** Card 030 ✅ (DONE)  
**Versão:** 1.0

---

## 🚀 **STATUS FINAL DA IMPLEMENTAÇÃO**

**Implementado por:** Osvaldo  
**Data de Implementação:** 08/11/2025  
**Branch:** `feature/goals-actions-table`  
**Commit:** `b65fe06`  
**Status:** ✅ **CONCLUÍDO** - Aguardando validação arquitetural

### **Arquivos Criados:**
- `src/main/resources/db/migration/V026__create_actions_table.sql`

### **Resultado dos Testes:**
- ✅ Migration executada com sucesso
- ✅ Tabela criada com 11 colunas
- ✅ 9 índices criados (8 + primary key)
- ✅ Foreign key funcional
- ✅ ON DELETE CASCADE testado
- ✅ Constraint CHECK de status validado
- ✅ Insert/Update/Delete funcionando
- ✅ Ordenação por order_index funcional
- ✅ Filtros por status funcionando

### **Próximos Passos:**
1. Validação arquitetural
2. Merge para main
3. Implementar Cards 032-035 (dependem desta tabela)

