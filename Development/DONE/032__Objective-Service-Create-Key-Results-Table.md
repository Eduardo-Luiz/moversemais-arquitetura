# 🎯 Card 032 - Objective Service: Criar Tabela Key Results (Indicadores)

**Agente Responsável:** Osvaldo  
**Microserviço:** moversemais-objective  
**Prioridade:** Alta  
**Status:** TODO  
**Estimativa:** 3 horas

---

## 📋 CONTEXTO

### **Situação Atual**
Cards 030 e 031 criaram estrutura de **Stages** (etapas) e **Actions** (ações) com sucesso. Agora temos:
- ✅ Tabela `objectives` (metas) - existente
- ✅ Tabela `stages` (etapas) - Card 030
- ✅ Tabela `actions` (ações) - Card 031
- ❌ Falta tabela `key_results` (indicadores de resultado)

### **Problema Identificado**
Para implementar **PRD 001 (Goal Chunking)** e **PRD 002 (Check-in de Progresso)**, precisamos de **Key Results (KRs)** - indicadores mensuráveis que mostram o progresso da meta.

**Atualmente:**
- ❌ Não existe tabela `key_results`
- ❌ Não há como medir progresso de metas
- ❌ IA não pode gerar indicadores mensuráveis
- ❌ Usuário não tem como fazer check-in de progresso
- ❌ Sistema não calcula % de conclusão da meta

**Exemplo do que queremos:**
```
Meta: Correr 5km em 30 minutos
├─ KR1: Tempo de corrida (30:00 min) - Peso: 40%
├─ KR2: Frequência cardíaca média (150 bpm) - Peso: 30%
└─ KR3: Treinos completados (12 treinos) - Peso: 30%

Progresso da Meta = (KR1 × 40%) + (KR2 × 30%) + (KR3 × 30%)
```

### **Solução Proposta**
Criar tabela `key_results` (indicadores) com relacionamento **1 Objective → N Key Results**, permitindo que cada meta tenha múltiplos indicadores mensuráveis com tipos diferentes (numérico, percentual, binário, etc.).

### **Onde se Encaixa na Arquitetura**
```
Módulo Goals
├─ objectives (existente) ✅
├─ stages (Card 030) ✅
├─ actions (Card 031) ✅
├─ key_results (este card) ← CRIAR
├─ action_kr_links (próximo card)
└─ checkins (próximo card)
```

### **Impacto se Não For Feito**
- Metas sem indicadores mensuráveis
- IA não pode gerar KRs
- Usuário não pode fazer check-in
- Sistema não calcula progresso
- PRD 002 (Check-in) impossível de implementar

---

## 🎯 REQUISITOS OBRIGATÓRIOS

### **1. Migration Flyway**

**Criar arquivo:** `src/main/resources/db/migration/V027__create_key_results_table.sql`

**Tabela `key_results` deve conter:**
- UUID como primary key (padrão do projeto)
- Foreign key para `objectives(id)` com ON DELETE CASCADE
- **Campos obrigatórios:**
  - `objective_id`: Referência para meta (FK)
  - `title`: Título do KR (ex: "Tempo de corrida")
  - `description`: Descrição detalhada (opcional)
  - `type`: Tipo do KR (NUMERIC, PERCENTAGE, BINARY, CURRENCY, TIME)
  - `target_value`: Valor alvo (ex: 30.00 para 30 minutos)
  - `current_value`: Valor atual (DEFAULT 0)
  - `unit`: Unidade de medida (ex: "min", "bpm", "km", "R$")
  - `weight`: Peso do KR no cálculo (DEFAULT 1.0, range 0.0-1.0)
  - `progress_percentage`: % de progresso (0-100, calculado)
  - Timestamps (created_at, updated_at)

**Campos opcionais (você decide):**
- `order_index`: Ordem de exibição dos KRs
- `is_active`: Se KR está ativo (soft delete)
- `completed_at`: Quando atingiu 100%
- `baseline_value`: Valor inicial (ponto de partida)

**Padrão do Projeto (seguir V026__create_actions_table.sql):**
- `CREATE TABLE IF NOT EXISTS` (idempotente)
- `gen_random_uuid()` para UUID
- `TIMESTAMP` + `DEFAULT NOW()`
- Índices com `IF NOT EXISTS`
- Nomenclatura snake_case
- Comentários SQL (COMMENT ON)
- Verificação de integridade (DO $$)

### **2. Relacionamento**

**Foreign Key obrigatória:**
- `objective_id` REFERENCES `objectives(id)` ON DELETE CASCADE
- Se meta deletada, KRs também são deletados (LGPD)

**Cálculo de Progresso:**
- `progress_percentage` = (current_value / target_value) × 100
- Será calculado automaticamente no backend (não no banco)
- Armazenado para performance (evitar recálculo)

### **3. Tipos de Key Results**

**Enum Type (obrigatório):**

Conforme **PRD 002**, suportamos 5 tipos de KRs:

1. **NUMERIC** - Valor numérico simples
   - Ex: "12 treinos completados"
   - target_value: 12, current_value: 5
   - unit: "treinos"

2. **PERCENTAGE** - Percentual (0-100)
   - Ex: "80% de satisfação"
   - target_value: 80, current_value: 65
   - unit: "%"

3. **BINARY** - Sim/Não (0 ou 1)
   - Ex: "Certificação obtida"
   - target_value: 1, current_value: 0
   - unit: null

4. **CURRENCY** - Valor monetário
   - Ex: "R$ 5.000 economizados"
   - target_value: 5000, current_value: 2300
   - unit: "R$"

5. **TIME** - Tempo/duração
   - Ex: "30 minutos de corrida"
   - target_value: 30, current_value: 35
   - unit: "min"

**Implementação (você decide):**
- Constraint CHECK com enum values (recomendado)
- Ou VARCHAR sem constraint
- Seguir padrão de V026 (actions tem constraint CHECK)

### **4. Peso dos KRs (Weight)**

**Campo `weight` (obrigatório):**
- Tipo: DECIMAL(3,2) - Permite valores como 0.40 (40%)
- Range: 0.0 a 1.0
- DEFAULT: 1.0 (100% se for único KR)
- Constraint CHECK: `weight >= 0.0 AND weight <= 1.0`

**Lógica de Negócio:**
- Soma dos weights de todos KRs de uma meta DEVE ser 1.0 (100%)
- Validação será feita no backend (não no banco)
- Exemplo:
  - KR1: weight 0.40 (40%)
  - KR2: weight 0.30 (30%)
  - KR3: weight 0.30 (30%)
  - Total: 1.00 (100%) ✅

### **5. Valores (target_value, current_value)**

**Campos numéricos:**
- Tipo: DECIMAL(10,2) - Suporta valores grandes e decimais
- `target_value`: Valor alvo (obrigatório)
- `current_value`: Valor atual (DEFAULT 0)
- Permite negativos (ex: reduzir peso de 90kg para 80kg)

**Exemplos:**
- Tempo: 30.00 (30 minutos)
- Dinheiro: 5000.00 (R$ 5.000)
- Percentual: 80.00 (80%)
- Binário: 1.00 (sim) ou 0.00 (não)
- Treinos: 12.00 (12 treinos)

### **6. Progresso (progress_percentage)**

**Campo `progress_percentage`:**
- Tipo: INTEGER (0-100)
- DEFAULT: 0
- Constraint CHECK: `progress_percentage >= 0 AND progress_percentage <= 100`
- Calculado no backend e armazenado

**Fórmula:**
```
progress_percentage = MIN(100, (current_value / target_value) × 100)
```

**Casos especiais:**
- Se target_value = 0: progress = 0
- Se current_value >= target_value: progress = 100
- BINARY: 0% ou 100% (sem valores intermediários)

### **7. Índices de Performance**

**Obrigatórios:**
- `idx_kr_objective_id` - Buscar KRs de uma meta
- `idx_kr_type` - Filtrar por tipo
- `idx_kr_progress` - Ordenar por progresso

**Opcional (você decide):**
- Índice composto (objective_id, progress_percentage)
- Índice em weight (filtrar KRs principais)
- Índice em is_active (se adicionar)
- Índice em order_index (se adicionar)

---

## ⚠️ RESTRIÇÕES

### **O que NÃO PODE ser alterado:**

1. ❌ **NÃO alterar tabela `objectives`** (será alterada no Card 035)
2. ❌ **NÃO alterar tabela `stages`** (Card 030)
3. ❌ **NÃO alterar tabela `actions`** (Card 031)
4. ❌ **NÃO alterar tabela `assessments`**
5. ❌ **NÃO alterar migrations existentes** (V1-V026)
6. ❌ **NÃO criar entities JPA ainda** (apenas migration SQL)
7. ❌ **NÃO criar services ou controllers** (apenas tabela)
8. ❌ **NÃO implementar cálculo de progresso** (será no backend depois)

### **O que DEVE ser preservado:**

1. ✅ **Padrão de nomenclatura** (snake_case)
2. ✅ **Padrão de migrations** (V{numero}__descricao.sql)
3. ✅ **Padrão de UUID** (gen_random_uuid())
4. ✅ **Padrão de timestamps** (TIMESTAMP + NOW())
5. ✅ **Padrão de índices** (IF NOT EXISTS)
6. ✅ **Padrão de comentários** (COMMENT ON)
7. ✅ **Padrão de verificação** (DO $$ block)
8. ✅ **Padrão de V026** (actions - que você criou!)

---

## 📚 DOCUMENTAÇÃO DE REFERÊNCIA

### **Arquivos para Estudar (OBRIGATÓRIO):**

1. **Padrão de Migrations (CRÍTICO):**
   - `src/main/resources/db/migration/V026__create_actions_table.sql` - **REFERÊNCIA PRINCIPAL**
   - Card 031 acabou de criar - seguir EXATAMENTE esse padrão
   - Osvaldo, você criou V026 com EXCELÊNCIA!

2. **Padrão de Constraint CHECK:**
   - `src/main/resources/db/migration/V026__create_actions_table.sql`
   - Linhas 35-42: Como criar constraint CHECK com DO $$
   - Mesmo padrão para `type` e `weight`

3. **Padrão de DECIMAL:**
   - `src/main/resources/db/migration/V1__create_objectives_table.sql`
   - Verificar se há uso de DECIMAL no projeto

4. **PRD 002 - Check-in de Progresso:**
   - `../moversemais-arquitetura/PRD/prd_002_checkin_progress.md`
   - Seção "Tipos de KR" (linhas 30-80)
   - Entender lógica de negócio dos KRs

5. **Análise Arquitetural:**
   - `../moversemais-arquitetura/ANALYSIS__Goals-Module-Architecture.md`
   - Seção "TABELA 3: key_results"

6. **Documentação:**
   - `../moversemais-objective/AGENTS.md`
   - `../moversemais-arquitetura/AGENTS.md`

### **Cards Relacionados:**
- Card 030: Tabela Stages ✅ (DONE)
- Card 031: Tabela Actions ✅ (DONE - pré-requisito direto)
- Card 033: Tabela action_kr_links (próximo - depende deste)
- Card 034: Tabela checkins (próximo - depende deste)

---

## 🔧 WORKFLOW

### **1. ESTUDAR (OBRIGATÓRIO - 15 minutos)**

```bash
cd moversemais-objective

# Estudar V026 que VOCÊ criou (Card 031)
cat src/main/resources/db/migration/V026__create_actions_table.sql

# Estudar PRD 002 (tipos de KR)
cat ../moversemais-arquitetura/PRD/prd_002_checkin_progress.md | grep -A 50 "Tipos de KR"

# Estudar análise arquitetural
cat ../moversemais-arquitetura/ANALYSIS__Goals-Module-Architecture.md | grep -A 60 "TABELA 3"

# Ler AGENTS.md
cat AGENTS.md
```

**Perguntas para Responder Antes de Implementar:**
- Migration V027 (próxima após V026)?
- Type: constraint CHECK ou VARCHAR?
- Weight: DECIMAL(3,2) com CHECK (0.0-1.0)?
- Progress: INTEGER com CHECK (0-100)?
- Campos adicionais: order_index? is_active? baseline_value?
- Índices: quais além dos 3 obrigatórios?

### **2. CRIAR BRANCH**

```bash
git checkout -b feature/goals-key-results-table
```

### **3. IMPLEMENTAR (VOCÊ DECIDE COMO)**

**Criar arquivo:**
- Nome: `V027__create_key_results_table.sql`
- Localização: `src/main/resources/db/migration/`

**Você decide:**
- Se adiciona `order_index` (ordem de exibição)
- Se adiciona `is_active` (soft delete)
- Se adiciona `completed_at` (quando atingiu 100%)
- Se adiciona `baseline_value` (valor inicial)
- Constraint CHECK para type ou não
- Constraint CHECK para weight (recomendado)
- Constraint CHECK para progress_percentage (recomendado)
- Ordem dos campos
- Comentários SQL
- Validações adicionais

**Mas DEVE seguir:**
- ✅ Padrão de V026__create_actions_table.sql (que você criou!)
- ✅ UUID com gen_random_uuid()
- ✅ Foreign key com ON DELETE CASCADE
- ✅ Índices com IF NOT EXISTS
- ✅ Comentários SQL (COMMENT ON)
- ✅ Verificação de integridade (DO $$)
- ✅ Constraint CHECK para type (5 valores)
- ✅ Constraint CHECK para weight (0.0-1.0)
- ✅ Constraint CHECK para progress_percentage (0-100)

### **4. TESTAR**

**Testes Obrigatórios:**

```bash
# 1. Rodar aplicação (Flyway executa migration)
./gradlew bootRun

# Verificar logs:
# "Migrating schema to version 027 - create key results table"
# "Successfully applied 1 migration"

# 2. Conectar ao PostgreSQL
docker exec moversemais-postgres psql -U developer -d moversemais_objective

# 3. Verificar tabela criada
\d key_results

# Esperado:
# - Colunas corretas (mínimo 12)
# - Foreign key para objectives
# - Índices criados
# - Constraints de type, weight, progress

# 4. Testar foreign key (inserir KR)
INSERT INTO key_results (objective_id, title, type, target_value, current_value, unit, weight)
VALUES (
  (SELECT id FROM objectives LIMIT 1),
  'Tempo de corrida',
  'TIME',
  30.00,
  0.00,
  'min',
  1.0
);

# Esperado: Sucesso (se objective existe)

# 5. Testar tipos válidos
INSERT INTO key_results (objective_id, title, type, target_value, unit, weight)
VALUES 
  ((SELECT id FROM objectives LIMIT 1), 'KR Numérico', 'NUMERIC', 12.00, 'treinos', 0.40),
  ((SELECT id FROM objectives LIMIT 1), 'KR Percentual', 'PERCENTAGE', 80.00, '%', 0.30),
  ((SELECT id FROM objectives LIMIT 1), 'KR Binário', 'BINARY', 1.00, null, 0.30);

# Esperado: 3 KRs inseridos

# 6. Testar tipo inválido (se constraint CHECK)
INSERT INTO key_results (objective_id, title, type, target_value, weight)
VALUES (
  (SELECT id FROM objectives LIMIT 1),
  'KR Inválido',
  'TIPO_INVALIDO',
  100.00,
  1.0
);

# Esperado: Erro (violates check constraint)

# 7. Testar weight fora do range (se constraint CHECK)
INSERT INTO key_results (objective_id, title, type, target_value, weight)
VALUES (
  (SELECT id FROM objectives LIMIT 1),
  'KR Weight Inválido',
  'NUMERIC',
  100.00,
  1.5  -- Maior que 1.0
);

# Esperado: Erro (violates check constraint)

# 8. Testar progress_percentage fora do range (se constraint CHECK)
UPDATE key_results 
SET progress_percentage = 150 
WHERE id = (SELECT id FROM key_results LIMIT 1);

# Esperado: Erro (violates check constraint)

# 9. Testar ON DELETE CASCADE
# Deletar objective e verificar se KRs foram deletados junto
DELETE FROM objectives WHERE id = (SELECT id FROM objectives LIMIT 1);

# Verificar se KRs foram deletados
SELECT COUNT(*) FROM key_results WHERE objective_id = (SELECT id FROM objectives LIMIT 1);
# Esperado: 0

# 10. Testar múltiplos KRs na mesma meta
INSERT INTO key_results (objective_id, title, type, target_value, unit, weight)
VALUES 
  ((SELECT id FROM objectives LIMIT 1), 'KR 1', 'NUMERIC', 10.00, 'km', 0.40),
  ((SELECT id FROM objectives LIMIT 1), 'KR 2', 'TIME', 30.00, 'min', 0.30),
  ((SELECT id FROM objectives LIMIT 1), 'KR 3', 'PERCENTAGE', 80.00, '%', 0.30);

# Verificar soma dos weights = 1.0
SELECT objective_id, SUM(weight) as total_weight 
FROM key_results 
GROUP BY objective_id;
# Esperado: 1.00

# 11. Testar cálculo manual de progresso
UPDATE key_results SET current_value = 15.00 WHERE title = 'KR 1';
-- current_value (15) / target_value (10) = 150% → progress = 100%

UPDATE key_results SET progress_percentage = 100 WHERE title = 'KR 1';

# 12. Limpeza
DELETE FROM key_results WHERE title LIKE 'KR %';
```

**Verificações:**
- [ ] Migration V027 executa sem erro
- [ ] Tabela `key_results` criada
- [ ] Foreign key para objectives funciona
- [ ] Índices criados (mínimo 3)
- [ ] Tipos válidos funcionam (NUMERIC, PERCENTAGE, BINARY, CURRENCY, TIME)
- [ ] Constraint CHECK valida type (se adicionou)
- [ ] Constraint CHECK valida weight (0.0-1.0)
- [ ] Constraint CHECK valida progress_percentage (0-100)
- [ ] ON DELETE CASCADE funciona
- [ ] Múltiplos KRs na mesma meta
- [ ] DECIMAL funciona (valores decimais)
- [ ] DEFAULT values funcionam

### **5. DOCUMENTAR DECISÕES**

Ao final do card, documente:
- Estrutura SQL final
- Campos adicionais escolhidos (order_index, is_active, etc.)
- Constraints CHECK implementadas (type, weight, progress)
- Índices adicionais (se houver)
- Testes realizados (12 cenários)
- Dificuldades encontradas
- Decisões técnicas tomadas

### **6. COMMIT E PUSH**

```bash
git add src/main/resources/db/migration/V027__create_key_results_table.sql
git commit -m "feat(objective-service): cria tabela key_results para Goal Chunking

- Tabela key_results (indicadores de resultado) para metas
- Relacionamento 1:N com objectives
- Foreign key com ON DELETE CASCADE
- 5 tipos: NUMERIC, PERCENTAGE, BINARY, CURRENCY, TIME
- Weight (peso) com range 0.0-1.0
- Progress percentage (0-100)
- Índices para performance
- Suporta Goal Chunking (PRD 001) e Check-in (PRD 002)
- Ref: Card 032"

git push origin feature/goals-key-results-table
```

### **7. MOVER PARA VALIDAÇÃO**

```bash
mv Development/TODO/032__Objective-Service-Create-Key-Results-Table.md \
   Development/VALIDATING/032__Objective-Service-Create-Key-Results-Table.md
```

---

## ✅ CRITÉRIOS DE VALIDAÇÃO

### **Funcionalidades:**
- [ ] Migration V027 criada
- [ ] Tabela `key_results` existe no banco
- [ ] Colunas obrigatórias (mínimo 12)
- [ ] Foreign key para objectives funciona
- [ ] ON DELETE CASCADE funciona
- [ ] Índices criados (mínimo 3)
- [ ] 5 tipos de KR funcionam (NUMERIC, PERCENTAGE, BINARY, CURRENCY, TIME)
- [ ] Weight range 0.0-1.0 validado
- [ ] Progress range 0-100 validado
- [ ] DECIMAL funciona para valores decimais

### **Padrão:**
- [ ] Seguiu estrutura de V026__create_actions_table.sql
- [ ] UUID com gen_random_uuid()
- [ ] Nomenclatura snake_case
- [ ] IF NOT EXISTS em índices
- [ ] Timestamps com padrão do projeto
- [ ] Comentários SQL (COMMENT ON)
- [ ] Verificação de integridade (DO $$)
- [ ] Constraints CHECK (type, weight, progress)

### **Testes:**
- [ ] Flyway executou migration
- [ ] Tabela visível no PostgreSQL (\d key_results)
- [ ] Insert funciona (5 tipos)
- [ ] Foreign key valida
- [ ] Cascade delete funciona
- [ ] Múltiplos KRs na mesma meta
- [ ] Constraint type valida (se adicionou)
- [ ] Constraint weight valida (0.0-1.0)
- [ ] Constraint progress valida (0-100)
- [ ] DECIMAL aceita valores decimais

---

## 🚨 TROUBLESHOOTING

### **Problema: Migration não executa**
**Solução:**
- Verificar numeração (V027)
- Verificar sintaxe SQL (DECIMAL, CHECK)
- Logs: `./gradlew bootRun | grep Flyway`

### **Problema: Foreign key error**
**Solução:**
- Verificar se tabela `objectives` existe
- Sintaxe: `REFERENCES objectives(id)`

### **Problema: Constraint CHECK error**
**Solução:**
- Verificar sintaxe: `CHECK (type IN (...))`
- Bloco `DO $$` para evitar duplicação
- Verificar nomes únicos de constraints

### **Problema: DECIMAL não aceita decimais**
**Solução:**
- Verificar sintaxe: `DECIMAL(10,2)`
- 10 dígitos totais, 2 decimais
- Exemplo: 9999999.99

### **Problema: Weight > 1.0 aceito**
**Solução:**
- Adicionar constraint CHECK: `weight >= 0.0 AND weight <= 1.0`
- Bloco `DO $$` para evitar duplicação

---

## 🎯 EXPECTATIVAS

### **Você é o Especialista em Backend**

**Osvaldo, você acabou de criar V026 (Card 031) com EXCELÊNCIA (Score 6/5)!** 🏆

Eu confio que você:

1. ✅ **Conhece o padrão** (você criou V026 com verificação tripla!)
2. ✅ **Sabe estruturar tabelas complexas** perfeitamente
3. ✅ **Domina PostgreSQL, DECIMAL e Constraints**

**Por isso:**
- Siga o MESMO padrão de V026 (que você criou)
- Estrutura similar (FK, índices, comentários, verificação)
- Tome decisões técnicas fundamentadas
- Adicione campos que julgar úteis (order_index, is_active, etc.)
- Implemente constraints CHECK robustas (type, weight, progress)

**Eu defini O QUE precisa ser feito. Você decide COMO fazer.**

**Campos obrigatórios:**
- id, objective_id, title, description, type, target_value, current_value, unit, weight, progress_percentage, timestamps

**Campos opcionais (você decide se adiciona):**
- order_index, is_active, completed_at, baseline_value, etc.

**Constraints obrigatórias:**
- CHECK para type (5 valores)
- CHECK para weight (0.0-1.0)
- CHECK para progress_percentage (0-100)

---

## 📊 OUTPUT ESPERADO

Ao finalizar, documente aqui:

### **Decisões Técnicas Tomadas:**

1. **Migration V027** - Seguindo numeração sequencial após V026
2. **Padrão de V026** - Seguiu EXATAMENTE o padrão que criei no Card 031
3. **16 Colunas** - Todos os campos obrigatórios + 4 opcionais úteis:
   - `order_index` - Ordem de exibição dos KRs
   - `is_active` - Soft delete (DEFAULT TRUE)
   - `completed_at` - Timestamp quando atingiu 100%
   - `baseline_value` - Valor inicial/ponto de partida
4. **5 Tipos de KR** - Conforme PRD 002:
   - NUMERIC, PERCENTAGE, BINARY, CURRENCY, TIME
   - Constraint CHECK com enum values
5. **3 Constraints CHECK** - Validações robustas:
   - `chk_kr_type` - Valida 5 tipos de KR
   - `weight` CHECK (0.0-1.0) - Range de peso
   - `progress_percentage` CHECK (0-100) - Range de progresso
6. **DECIMAL(10,2)** - Para valores numéricos (target, current, baseline)
7. **DECIMAL(3,2)** - Para weight (permite 0.40, 0.30, etc.)
8. **10 Índices Customizados** - Performance otimizada:
   - Simples: objective_id, type, progress, is_active, order, completed_at, created_at
   - Compostos: (objective_id, progress), (objective_id, weight), (objective_id, order)
9. **Comentários SQL Completos** - Documentação inline para cada coluna
10. **Verificação Tripla** - Bloco `DO $$` valida colunas, índices E constraints

### **Estrutura SQL Final:**

```sql
-- V027__create_key_results_table.sql
-- Criar tabela key_results (indicadores) para Goal Chunking e Check-in de Progresso
-- Card 032 - Objective Service: Criar Tabela Key Results
-- Data: 08/11/2025
-- Referência: PRD 002 (Check-in de Progresso)

-- Tabela de Key Results (indicadores mensuráveis) para medir progresso de metas
CREATE TABLE IF NOT EXISTS key_results (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    objective_id UUID NOT NULL,
    title VARCHAR(500) NOT NULL,
    description TEXT,
    type VARCHAR(20) NOT NULL,
    target_value DECIMAL(10,2) NOT NULL,
    current_value DECIMAL(10,2) NOT NULL DEFAULT 0.00,
    unit VARCHAR(50),
    weight DECIMAL(3,2) NOT NULL DEFAULT 1.00 CHECK (weight >= 0.00 AND weight <= 1.00),
    progress_percentage INTEGER NOT NULL DEFAULT 0 CHECK (progress_percentage >= 0 AND progress_percentage <= 100),
    order_index INTEGER CHECK (order_index >= 1),
    is_active BOOLEAN NOT NULL DEFAULT TRUE,
    completed_at TIMESTAMP,
    baseline_value DECIMAL(10,2),
    created_at TIMESTAMP DEFAULT NOW(),
    updated_at TIMESTAMP DEFAULT NOW(),
    
    -- Foreign key com CASCADE para LGPD e limpeza automática
    CONSTRAINT fk_key_results_objective 
        FOREIGN KEY (objective_id) 
        REFERENCES objectives(id) 
        ON DELETE CASCADE
);

-- Índices para performance
CREATE INDEX IF NOT EXISTS idx_kr_objective_id ON key_results(objective_id);
CREATE INDEX IF NOT EXISTS idx_kr_type ON key_results(type);
CREATE INDEX IF NOT EXISTS idx_kr_progress ON key_results(progress_percentage);
CREATE INDEX IF NOT EXISTS idx_kr_objective_progress ON key_results(objective_id, progress_percentage);
CREATE INDEX IF NOT EXISTS idx_kr_objective_weight ON key_results(objective_id, weight);
CREATE INDEX IF NOT EXISTS idx_kr_is_active ON key_results(is_active);
CREATE INDEX IF NOT EXISTS idx_kr_order ON key_results(order_index);
CREATE INDEX IF NOT EXISTS idx_kr_objective_order ON key_results(objective_id, order_index);
CREATE INDEX IF NOT EXISTS idx_kr_completed_at ON key_results(completed_at);
CREATE INDEX IF NOT EXISTS idx_kr_created_at ON key_results(created_at);

-- Constraint para validar tipo de KR (conforme PRD 002)
DO $$ 
BEGIN
    IF NOT EXISTS (SELECT 1 FROM pg_constraint WHERE conname = 'chk_kr_type') THEN
        ALTER TABLE key_results ADD CONSTRAINT chk_kr_type
            CHECK (type IN ('NUMERIC', 'PERCENTAGE', 'BINARY', 'CURRENCY', 'TIME'));
    END IF;
END $$;

-- Comentários para documentação
COMMENT ON TABLE key_results IS 'Key Results (indicadores mensuráveis) para medir progresso de metas (PRD 002)';
COMMENT ON COLUMN key_results.id IS 'Identificador único do Key Result';
COMMENT ON COLUMN key_results.objective_id IS 'Referência para a meta (objective) pai';
COMMENT ON COLUMN key_results.title IS 'Título do KR (ex: "Tempo de corrida")';
COMMENT ON COLUMN key_results.description IS 'Descrição detalhada do KR (opcional)';
COMMENT ON COLUMN key_results.type IS 'Tipo do KR: NUMERIC, PERCENTAGE, BINARY, CURRENCY, TIME';
COMMENT ON COLUMN key_results.target_value IS 'Valor alvo a ser atingido (obrigatório)';
COMMENT ON COLUMN key_results.current_value IS 'Valor atual (atualizado via check-ins)';
COMMENT ON COLUMN key_results.unit IS 'Unidade de medida (ex: "min", "km", "R$", "%")';
COMMENT ON COLUMN key_results.weight IS 'Peso do KR no cálculo da meta (0.0-1.0, soma deve ser 1.0)';
COMMENT ON COLUMN key_results.progress_percentage IS 'Percentual de progresso (0-100%, calculado)';
COMMENT ON COLUMN key_results.order_index IS 'Ordem de exibição dos KRs (1, 2, 3...)';
COMMENT ON COLUMN key_results.is_active IS 'Se KR está ativo (soft delete)';
COMMENT ON COLUMN key_results.completed_at IS 'Data e hora em que atingiu 100% de progresso';
COMMENT ON COLUMN key_results.baseline_value IS 'Valor inicial/baseline (ponto de partida)';
COMMENT ON COLUMN key_results.created_at IS 'Data de criação do KR';
COMMENT ON COLUMN key_results.updated_at IS 'Data da última atualização do KR';

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
    WHERE table_name = 'key_results' 
    AND column_name IN ('id', 'objective_id', 'title', 'description', 'type', 'target_value', 
                        'current_value', 'unit', 'weight', 'progress_percentage', 'order_index',
                        'is_active', 'completed_at', 'baseline_value', 'created_at', 'updated_at');
    
    IF column_count != 16 THEN
        RAISE EXCEPTION 'Falha ao criar tabela key_results: % colunas criadas (esperado: 16)', column_count;
    END IF;
    
    -- Verificar índices criados
    SELECT COUNT(*) INTO index_count 
    FROM pg_indexes 
    WHERE tablename = 'key_results' 
    AND indexname LIKE 'idx_kr_%';
    
    IF index_count < 10 THEN
        RAISE EXCEPTION 'Falha ao criar índices: % índices criados (esperado: >= 10)', index_count;
    END IF;
    
    -- Verificar constraints
    SELECT COUNT(*) INTO constraint_count
    FROM pg_constraint
    WHERE conname IN ('chk_kr_type', 'key_results_weight_check', 'key_results_progress_percentage_check');
    
    IF constraint_count < 3 THEN
        RAISE EXCEPTION 'Falha ao criar constraints: % constraints criados (esperado: >= 3)', constraint_count;
    END IF;
    
    RAISE NOTICE 'Tabela key_results criada com sucesso: % colunas, % índices, % constraints', column_count, index_count, constraint_count;
END $$;
```

### **Testes Realizados:**

✅ **1. Migration Executada**
- Flyway aplicou V027 com sucesso
- Tabela `key_results` criada no banco
- Comando: `./gradlew bootRun`

✅ **2. Estrutura Verificada**
- 16 colunas criadas corretamente
- 11 índices criados (10 + primary key)
- Foreign key configurada
- 4 constraints ativas (type, weight, progress, order_index)
- Comando: `\d key_results` no PostgreSQL

✅ **3. Inserção dos 5 Tipos de KR**
- NUMERIC: "Treinos completados" (12 treinos)
- PERCENTAGE: "Satisfação da equipe" (80%)
- BINARY: "Certificação obtida" (sim/não)
- CURRENCY: "Economia gerada" (R$ 5.000)
- TIME: "Tempo de corrida" (30 min)
- Todos inseridos com sucesso

✅ **4. Soma dos Weights Validada**
- 5 KRs com weight 0.20 cada
- Soma total: 1.00 (100%) ✅
- SQL: `SUM(weight) GROUP BY objective_id`

✅ **5. Constraint CHECK de Tipo**
- Tipos válidos funcionam: NUMERIC, PERCENTAGE, BINARY, CURRENCY, TIME
- Tipo inválido rejeitado com erro
- Mensagem: "violates check constraint chk_kr_type"

✅ **6. Constraint CHECK de Weight**
- Weight válido funciona: 0.0 a 1.0
- Weight > 1.0 rejeitado com erro
- Mensagem: "violates check constraint key_results_weight_check"

✅ **7. Constraint CHECK de Progress**
- Progress válido funciona: 0 a 100
- Progress > 100 rejeitado com erro
- Mensagem: "violates check constraint key_results_progress_percentage_check"

✅ **8. Atualização de Valores**
- current_value atualizado: 5.00 → 10.00
- progress_percentage atualizado: 0 → 83
- UPDATE funciona corretamente

✅ **9. ON DELETE CASCADE**
- Objective deletado → 5 KRs deletados automaticamente
- Limpeza automática funcionando (LGPD)
- Cascade em toda a hierarquia: objectives → key_results

✅ **10. Índices Verificados**
- 11 índices listados no PostgreSQL
- Todos com `IF NOT EXISTS`
- Índices compostos criados corretamente

✅ **11. Foreign Key Validada**
- Relacionamento com objectives funciona
- Não permite objective_id inválido

✅ **12. Campos Opcionais Funcionando**
- order_index: NULL permitido, CHECK >= 1 quando preenchido
- is_active: DEFAULT TRUE funcionando
- completed_at: NULL permitido
- baseline_value: NULL permitido

### **Dificuldades Encontradas:**

Nenhuma! A implementação seguiu perfeitamente o padrão de V026 que criei no Card 031.

### **Melhorias Implementadas:**

1. **4 Campos Opcionais Úteis** - `order_index`, `is_active`, `completed_at`, `baseline_value`
2. **10 Índices de Performance** - Além dos 3 obrigatórios, adicionei 7 extras:
   - `idx_kr_objective_progress` - Busca por meta e progresso (composto)
   - `idx_kr_objective_weight` - Busca por meta e peso (composto)
   - `idx_kr_objective_order` - Busca ordenada por meta (composto)
   - `idx_kr_is_active` - Filtrar KRs ativos
   - `idx_kr_order` - Ordenação por order_index
   - `idx_kr_completed_at` - Filtrar KRs concluídos
   - `idx_kr_created_at` - Ordenação temporal
3. **Soft Delete** - Campo `is_active` para desativar KRs sem deletar
4. **Baseline Value** - Campo para armazenar valor inicial (útil para cálculos)
5. **Comentários SQL Completos** - Documentação inline detalhada
6. **Verificação Tripla** - Bloco `DO $$` valida colunas, índices E constraints
7. **Referência ao PRD 002** - Comentários SQL mencionam PRD de Check-in

---

**Data de Criação:** 08/11/2025  
**Criado por:** Arquiobaldo (Arquiteto MoverseMais)  
**Contexto:** Módulo Goals - PRD 001 (Goal Chunking) + PRD 002 (Check-in)  
**Dependência:** Card 031 ✅ (DONE)  
**Versão:** 1.0

---

## 🚀 **STATUS FINAL DA IMPLEMENTAÇÃO**

**Implementado por:** Osvaldo  
**Data de Implementação:** 08/11/2025  
**Branch:** `feature/goals-key-results-table`  
**Commit:** `e61f008`  
**Status:** ✅ **CONCLUÍDO** - Aguardando validação arquitetural

### **Arquivos Criados:**
- `src/main/resources/db/migration/V027__create_key_results_table.sql`

### **Resultado dos Testes:**
- ✅ Migration executada com sucesso
- ✅ Tabela criada com 16 colunas
- ✅ 11 índices criados (10 + primary key)
- ✅ Foreign key funcional
- ✅ ON DELETE CASCADE testado
- ✅ 4 Constraints CHECK validados (type, weight, progress, order_index)
- ✅ 5 tipos de KR testados (NUMERIC, PERCENTAGE, BINARY, CURRENCY, TIME)
- ✅ Soma de weights = 1.00 validada
- ✅ Insert/Update/Delete funcionando
- ✅ Campos opcionais funcionando
- ✅ Soft delete (is_active) funcional
- ✅ DECIMAL(10,2) e DECIMAL(3,2) funcionando

### **Próximos Passos:**
1. Validação arquitetural
2. Merge para main
3. Implementar Cards 033-035 (dependem desta tabela)

