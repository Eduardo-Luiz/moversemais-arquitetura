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
(Você preenche)

### **Estrutura SQL Final:**
(Cole o SQL completo)

### **Testes Realizados:**
(Liste cenários testados)

### **Dificuldades Encontradas:**
(Se houver)

### **Melhorias Implementadas:**
(Além do requisitado)

---

**Data de Criação:** 08/11/2025  
**Criado por:** Arquiobaldo (Arquiteto MoverseMais)  
**Contexto:** Módulo Goals - PRD 001 (Goal Chunking)  
**Dependência:** Card 030 ✅ (DONE)  
**Versão:** 1.0

