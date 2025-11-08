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
(Você preenche)

### **Estrutura SQL Final:**
(Cole o SQL completo)

### **Testes Realizados:**
(Liste cenários testados - mínimo 12)

### **Dificuldades Encontradas:**
(Se houver)

### **Melhorias Implementadas:**
(Além do requisitado)

---

**Data de Criação:** 08/11/2025  
**Criado por:** Arquiobaldo (Arquiteto MoverseMais)  
**Contexto:** Módulo Goals - PRD 001 (Goal Chunking) + PRD 002 (Check-in)  
**Dependência:** Card 031 ✅ (DONE)  
**Versão:** 1.0

