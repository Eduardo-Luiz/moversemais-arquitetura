# 🎯 Card 033 - Objective Service: Criar Tabela Action-KR Links (Vinculação)

**Agente Responsável:** Osvaldo  
**Microserviço:** moversemais-objective  
**Prioridade:** Alta  
**Status:** TODO  
**Estimativa:** 2 horas

---

## 📋 CONTEXTO

### **Situação Atual**
Cards 030, 031 e 032 criaram estrutura completa de **Stages**, **Actions** e **Key Results** com sucesso. Agora temos:
- ✅ Tabela `objectives` (metas) - existente
- ✅ Tabela `stages` (etapas) - Card 030
- ✅ Tabela `actions` (ações) - Card 031
- ✅ Tabela `key_results` (indicadores) - Card 032
- ❌ Falta tabela `action_kr_links` (vinculação many-to-many)

### **Problema Identificado**
Para implementar **PRD 001 (Goal Chunking)**, precisamos vincular **Actions** (ações executáveis) com **Key Results** (indicadores mensuráveis). Uma ação pode impactar múltiplos KRs, e um KR pode ser impactado por múltiplas ações.

**Atualmente:**
- ❌ Não existe tabela `action_kr_links`
- ❌ Actions e KRs estão desconectados
- ❌ Sistema não sabe quais ações impactam quais KRs
- ❌ IA não pode gerar vinculações inteligentes
- ❌ Usuário não vê relação entre ações e progresso

**Exemplo do que queremos:**
```
Meta: Correr 5km em 30 minutos
├─ Etapa 1: Avaliar condicionamento físico
│   ├─ Ação 1: Fazer corrida teste de 2km
│   │   └─ Impacta: KR1 (Tempo de corrida), KR2 (Frequência cardíaca)
│   └─ Ação 2: Registrar tempo e frequência
│       └─ Impacta: KR1 (Tempo de corrida), KR2 (Frequência cardíaca)
└─ KR1: Tempo de corrida (30 min) - Peso: 50%
    KR2: Frequência cardíaca (150 bpm) - Peso: 30%
    KR3: Treinos completados (12) - Peso: 20%
```

### **Solução Proposta**
Criar tabela `action_kr_links` (vinculação) com relacionamento **Many-to-Many** entre `actions` e `key_results`, permitindo que:
- Uma ação possa impactar múltiplos KRs
- Um KR possa ser impactado por múltiplas ações
- IA gere vinculações inteligentes ao criar plano
- Sistema calcule impacto de ações no progresso

### **Onde se Encaixa na Arquitetura**
```
Módulo Goals
├─ objectives (existente) ✅
├─ stages (Card 030) ✅
├─ actions (Card 031) ✅
├─ key_results (Card 032) ✅
├─ action_kr_links (este card) ← CRIAR
└─ checkins (próximo card)
```

### **Impacto se Não For Feito**
- Actions e KRs desconectados
- IA não pode gerar vinculações
- Sistema não sabe impacto de ações
- Usuário não vê relação ação → progresso
- Goal Chunking incompleto

---

## 🎯 REQUISITOS OBRIGATÓRIOS

### **1. Migration Flyway**

**Criar arquivo:** `src/main/resources/db/migration/V028__create_action_kr_links_table.sql`

**Tabela `action_kr_links` deve conter:**
- **Composite Primary Key** (action_id, kr_id)
- Foreign key para `actions(id)` com ON DELETE CASCADE
- Foreign key para `key_results(id)` com ON DELETE CASCADE
- **Campos obrigatórios:**
  - `action_id`: Referência para ação (FK)
  - `kr_id`: Referência para key result (FK)
  - `created_at`: Timestamp de criação

**Campos opcionais (você decide):**
- `impact_level`: Nível de impacto (LOW, MEDIUM, HIGH)
- `notes`: Observações sobre a vinculação
- `created_by`: Quem criou (USER ou AI)

**Padrão do Projeto (seguir V027__create_key_results_table.sql):**
- `CREATE TABLE IF NOT EXISTS` (idempotente)
- `TIMESTAMP` + `DEFAULT NOW()`
- Índices com `IF NOT EXISTS`
- Nomenclatura snake_case
- Comentários SQL (COMMENT ON)
- Verificação de integridade (DO $$)

### **2. Relacionamento Many-to-Many**

**Composite Primary Key obrigatória:**
- `PRIMARY KEY (action_id, kr_id)`
- Garante unicidade da vinculação
- Não permite duplicatas

**Foreign Keys obrigatórias:**
- `action_id` REFERENCES `actions(id)` ON DELETE CASCADE
- `kr_id` REFERENCES `key_results(id)` ON DELETE CASCADE
- Se ação deletada, vinculações também são deletadas
- Se KR deletado, vinculações também são deletadas

**Lógica de Negócio:**
- Uma ação pode ter 0 ou N vinculações com KRs
- Um KR pode ter 0 ou N vinculações com ações
- Vinculação é opcional (ações podem existir sem KRs)
- IA gera vinculações ao criar plano

### **3. Índices de Performance**

**Obrigatórios:**
- `idx_action_kr_action_id` - Buscar KRs de uma ação
- `idx_action_kr_kr_id` - Buscar ações de um KR

**Opcional (você decide):**
- Índice em `impact_level` (se adicionar)
- Índice em `created_by` (se adicionar)
- Índice em `created_at` (temporal)

**Observação:**
- Composite Primary Key já cria índice automático em (action_id, kr_id)
- Índices adicionais são para queries reversas

### **4. Campos Opcionais (Decisão Técnica)**

**Impact Level (opcional):**
- Enum: LOW, MEDIUM, HIGH
- Indica quanto a ação impacta o KR
- Útil para priorização e cálculos
- Se adicionar, usar constraint CHECK

**Notes (opcional):**
- TEXT para observações
- Útil para IA explicar vinculação
- Ex: "Esta ação impacta diretamente o tempo de corrida"

**Created By (opcional):**
- Enum: USER, AI
- Indica origem da vinculação
- Útil para analytics e auditoria
- Se adicionar, usar constraint CHECK

---

## ⚠️ RESTRIÇÕES

### **O que NÃO PODE ser alterado:**

1. ❌ **NÃO alterar tabela `objectives`** (será alterada no Card 035)
2. ❌ **NÃO alterar tabela `stages`** (Card 030)
3. ❌ **NÃO alterar tabela `actions`** (Card 031)
4. ❌ **NÃO alterar tabela `key_results`** (Card 032)
5. ❌ **NÃO alterar tabela `assessments`**
6. ❌ **NÃO alterar migrations existentes** (V1-V027)
7. ❌ **NÃO criar entities JPA ainda** (apenas migration SQL)
8. ❌ **NÃO criar services ou controllers** (apenas tabela)

### **O que DEVE ser preservado:**

1. ✅ **Padrão de nomenclatura** (snake_case)
2. ✅ **Padrão de migrations** (V{numero}__descricao.sql)
3. ✅ **Padrão de timestamps** (TIMESTAMP + NOW())
4. ✅ **Padrão de índices** (IF NOT EXISTS)
5. ✅ **Padrão de comentários** (COMMENT ON)
6. ✅ **Padrão de verificação** (DO $$ block)
7. ✅ **Padrão de V027** (key_results - que você criou!)

---

## 📚 DOCUMENTAÇÃO DE REFERÊNCIA

### **Arquivos para Estudar (OBRIGATÓRIO):**

1. **Padrão de Migrations (CRÍTICO):**
   - `src/main/resources/db/migration/V027__create_key_results_table.sql` - **REFERÊNCIA PRINCIPAL**
   - Card 032 acabou de criar - seguir EXATAMENTE esse padrão
   - Osvaldo, você criou V027 com EXCELÊNCIA (Score 7/5)!

2. **Padrão de Composite Primary Key:**
   - Verificar se existe no projeto
   - Sintaxe: `PRIMARY KEY (campo1, campo2)`
   - Garante unicidade da combinação

3. **Padrão de Constraint CHECK:**
   - `src/main/resources/db/migration/V027__create_key_results_table.sql`
   - Linhas 43-48: Como criar constraint CHECK com DO $$
   - Mesmo padrão para `impact_level` e `created_by` (se adicionar)

4. **Análise Arquitetural:**
   - `../moversemais-arquitetura/ANALYSIS__Goals-Module-Architecture.md`
   - Seção "TABELA 4: action_kr_links"

5. **Documentação:**
   - `../moversemais-objective/AGENTS.md`
   - `../moversemais-arquitetura/AGENTS.md`

### **Cards Relacionados:**
- Card 030: Tabela Stages ✅ (DONE)
- Card 031: Tabela Actions ✅ (DONE)
- Card 032: Tabela Key Results ✅ (DONE - pré-requisito direto)
- Card 034: Tabela checkins (próximo)

---

## 🔧 WORKFLOW

### **1. ESTUDAR (OBRIGATÓRIO - 10 minutos)**

```bash
cd moversemais-objective

# Estudar V027 que VOCÊ criou (Card 032)
cat src/main/resources/db/migration/V027__create_key_results_table.sql

# Estudar análise arquitetural
cat ../moversemais-arquitetura/ANALYSIS__Goals-Module-Architecture.md | grep -A 30 "TABELA 4"

# Verificar se existe composite PK no projeto
grep -r "PRIMARY KEY (" src/main/resources/db/migration/

# Ler AGENTS.md
cat AGENTS.md
```

**Perguntas para Responder Antes de Implementar:**
- Migration V028 (próxima após V027)?
- Composite Primary Key: `PRIMARY KEY (action_id, kr_id)`?
- Campos adicionais: impact_level? notes? created_by?
- Constraint CHECK para impact_level ou created_by?
- Índices: quais além dos 2 obrigatórios?

### **2. CRIAR BRANCH**

```bash
git checkout -b feature/goals-action-kr-links-table
```

### **3. IMPLEMENTAR (VOCÊ DECIDE COMO)**

**Criar arquivo:**
- Nome: `V028__create_action_kr_links_table.sql`
- Localização: `src/main/resources/db/migration/`

**Você decide:**
- Se adiciona `impact_level` (LOW, MEDIUM, HIGH)
- Se adiciona `notes` (TEXT)
- Se adiciona `created_by` (USER, AI)
- Constraint CHECK para impact_level ou created_by
- Ordem dos campos
- Comentários SQL
- Validações adicionais

**Mas DEVE seguir:**
- ✅ Padrão de V027__create_key_results_table.sql (que você criou!)
- ✅ Composite Primary Key (action_id, kr_id)
- ✅ 2 Foreign keys com ON DELETE CASCADE
- ✅ Índices com IF NOT EXISTS
- ✅ Comentários SQL (COMMENT ON)
- ✅ Verificação de integridade (DO $$)

### **4. TESTAR**

**Testes Obrigatórios:**

```bash
# 1. Rodar aplicação (Flyway executa migration)
./gradlew bootRun

# Verificar logs:
# "Migrating schema to version 028 - create action kr links table"
# "Successfully applied 1 migration"

# 2. Conectar ao PostgreSQL
docker exec moversemais-postgres psql -U developer -d moversemais_objective

# 3. Verificar tabela criada
\d action_kr_links

# Esperado:
# - Colunas corretas
# - Composite Primary Key (action_id, kr_id)
# - 2 Foreign keys (actions, key_results)
# - Índices criados

# 4. Testar vinculação simples
INSERT INTO action_kr_links (action_id, kr_id)
VALUES (
  (SELECT id FROM actions LIMIT 1),
  (SELECT id FROM key_results LIMIT 1)
);

# Esperado: Sucesso (se action e KR existem)

# 5. Testar duplicata (composite PK)
INSERT INTO action_kr_links (action_id, kr_id)
VALUES (
  (SELECT id FROM actions LIMIT 1),
  (SELECT id FROM key_results LIMIT 1)
);

# Esperado: Erro (violates unique constraint - duplicate key)

# 6. Testar múltiplas vinculações (1 action → N KRs)
INSERT INTO action_kr_links (action_id, kr_id)
VALUES 
  ((SELECT id FROM actions LIMIT 1), (SELECT id FROM key_results OFFSET 0 LIMIT 1)),
  ((SELECT id FROM actions LIMIT 1), (SELECT id FROM key_results OFFSET 1 LIMIT 1)),
  ((SELECT id FROM actions LIMIT 1), (SELECT id FROM key_results OFFSET 2 LIMIT 1));

# Esperado: 3 vinculações criadas (mesma action, 3 KRs diferentes)

# 7. Testar vinculação reversa (N actions → 1 KR)
INSERT INTO action_kr_links (action_id, kr_id)
VALUES 
  ((SELECT id FROM actions OFFSET 0 LIMIT 1), (SELECT id FROM key_results LIMIT 1)),
  ((SELECT id FROM actions OFFSET 1 LIMIT 1), (SELECT id FROM key_results LIMIT 1)),
  ((SELECT id FROM actions OFFSET 2 LIMIT 1), (SELECT id FROM key_results LIMIT 1));

# Esperado: 3 vinculações criadas (3 actions, mesmo KR)

# 8. Testar foreign key (action inválida)
INSERT INTO action_kr_links (action_id, kr_id)
VALUES (
  'aaaaaaaa-aaaa-aaaa-aaaa-aaaaaaaaaaaa',
  (SELECT id FROM key_results LIMIT 1)
);

# Esperado: Erro (violates foreign key constraint)

# 9. Testar ON DELETE CASCADE (deletar action)
-- Criar vinculação
INSERT INTO action_kr_links (action_id, kr_id)
VALUES (
  (SELECT id FROM actions LIMIT 1),
  (SELECT id FROM key_results LIMIT 1)
);

-- Deletar action
DELETE FROM actions WHERE id = (SELECT id FROM actions LIMIT 1);

-- Verificar se vinculação foi deletada
SELECT COUNT(*) FROM action_kr_links WHERE action_id = (SELECT id FROM actions LIMIT 1);
# Esperado: 0

# 10. Testar ON DELETE CASCADE (deletar KR)
-- Criar vinculação
INSERT INTO action_kr_links (action_id, kr_id)
VALUES (
  (SELECT id FROM actions LIMIT 1),
  (SELECT id FROM key_results LIMIT 1)
);

-- Deletar KR
DELETE FROM key_results WHERE id = (SELECT id FROM key_results LIMIT 1);

# Verificar se vinculação foi deletada
SELECT COUNT(*) FROM action_kr_links WHERE kr_id = (SELECT id FROM key_results LIMIT 1);
# Esperado: 0

# 11. Testar índices (buscar KRs de uma action)
SELECT kr.* 
FROM key_results kr
JOIN action_kr_links akl ON kr.id = akl.kr_id
WHERE akl.action_id = (SELECT id FROM actions LIMIT 1);

# Esperado: Lista de KRs vinculados à action

# 12. Testar índices (buscar actions de um KR)
SELECT a.* 
FROM actions a
JOIN action_kr_links akl ON a.id = akl.action_id
WHERE akl.kr_id = (SELECT id FROM key_results LIMIT 1);

# Esperado: Lista de actions vinculadas ao KR

# 13. Limpeza
DELETE FROM action_kr_links;
```

**Verificações:**
- [ ] Migration V028 executa sem erro
- [ ] Tabela `action_kr_links` criada
- [ ] Composite Primary Key funciona
- [ ] 2 Foreign keys funcionam
- [ ] ON DELETE CASCADE funciona (action)
- [ ] ON DELETE CASCADE funciona (KR)
- [ ] Índices criados (mínimo 2)
- [ ] Duplicata rejeitada (composite PK)
- [ ] 1 action → N KRs funciona
- [ ] N actions → 1 KR funciona
- [ ] Queries com JOIN funcionam

### **5. DOCUMENTAR DECISÕES**

Ao final do card, documente:
- Estrutura SQL final
- Campos adicionais escolhidos (impact_level, notes, created_by)
- Constraints CHECK implementadas (se houver)
- Índices adicionais (se houver)
- Testes realizados (13 cenários)
- Dificuldades encontradas
- Decisões técnicas tomadas

### **6. COMMIT E PUSH**

```bash
git add src/main/resources/db/migration/V028__create_action_kr_links_table.sql
git commit -m "feat(objective-service): cria tabela action_kr_links para Goal Chunking

- Tabela action_kr_links (vinculação many-to-many)
- Relacionamento entre actions e key_results
- Composite Primary Key (action_id, kr_id)
- 2 Foreign keys com ON DELETE CASCADE
- Índices para queries bidirecionais
- Suporta Goal Chunking (PRD 001)
- Ref: Card 033"

git push origin feature/goals-action-kr-links-table
```

### **7. MOVER PARA VALIDAÇÃO**

```bash
mv Development/TODO/033__Objective-Service-Create-Action-KR-Links-Table.md \
   Development/VALIDATING/033__Objective-Service-Create-Action-KR-Links-Table.md
```

---

## ✅ CRITÉRIOS DE VALIDAÇÃO

### **Funcionalidades:**
- [ ] Migration V028 criada
- [ ] Tabela `action_kr_links` existe no banco
- [ ] Composite Primary Key (action_id, kr_id)
- [ ] 2 Foreign keys funcionam (actions, key_results)
- [ ] ON DELETE CASCADE funciona (ambas FKs)
- [ ] Índices criados (mínimo 2)
- [ ] Duplicata rejeitada (composite PK)
- [ ] 1 action → N KRs funciona
- [ ] N actions → 1 KR funciona

### **Padrão:**
- [ ] Seguiu estrutura de V027__create_key_results_table.sql
- [ ] Nomenclatura snake_case
- [ ] IF NOT EXISTS em índices
- [ ] Timestamps com padrão do projeto
- [ ] Comentários SQL (COMMENT ON)
- [ ] Verificação de integridade (DO $$)

### **Testes:**
- [ ] Flyway executou migration
- [ ] Tabela visível no PostgreSQL (\d action_kr_links)
- [ ] Insert funciona
- [ ] 2 Foreign keys validam
- [ ] Cascade delete funciona (action)
- [ ] Cascade delete funciona (KR)
- [ ] Duplicata rejeitada
- [ ] Queries com JOIN funcionam

---

## 🚨 TROUBLESHOOTING

### **Problema: Migration não executa**
**Solução:**
- Verificar numeração (V028)
- Verificar sintaxe SQL (composite PK)
- Logs: `./gradlew bootRun | grep Flyway`

### **Problema: Foreign key error**
**Solução:**
- Verificar se tabelas `actions` e `key_results` existem
- Sintaxe: `REFERENCES actions(id)` e `REFERENCES key_results(id)`

### **Problema: Composite PK não funciona**
**Solução:**
- Sintaxe correta: `PRIMARY KEY (action_id, kr_id)`
- Não usar `id` separado (composite PK substitui)

### **Problema: Duplicata aceita**
**Solução:**
- Verificar se composite PK foi criado
- `\d action_kr_links` deve mostrar PK em (action_id, kr_id)

### **Problema: CASCADE não funciona**
**Solução:**
- Verificar `ON DELETE CASCADE` em ambas FKs
- Testar deletando action e KR separadamente

---

## 🎯 EXPECTATIVAS

### **Você é o Especialista em Backend**

**Osvaldo, você acabou de criar V027 (Card 032) com EXCELÊNCIA MÁXIMA (Score 7/5)!** 🏆🏆🏆

Eu confio que você:

1. ✅ **Conhece o padrão** (você criou V027 com 16 colunas, 10 índices!)
2. ✅ **Sabe estruturar tabelas complexas** perfeitamente
3. ✅ **Domina PostgreSQL, Composite PK e Many-to-Many**

**Por isso:**
- Siga o MESMO padrão de V027 (que você criou)
- Estrutura similar (FKs, índices, comentários, verificação)
- Tome decisões técnicas fundamentadas
- Adicione campos que julgar úteis (impact_level, notes, created_by)

**Eu defini O QUE precisa ser feito. Você decide COMO fazer.**

**Campos obrigatórios:**
- action_id, kr_id, created_at

**Campos opcionais (você decide se adiciona):**
- impact_level (LOW, MEDIUM, HIGH)
- notes (TEXT)
- created_by (USER, AI)

**Constraints obrigatórias:**
- Composite Primary Key (action_id, kr_id)
- 2 Foreign keys com ON DELETE CASCADE

---

## 📊 OUTPUT ESPERADO

Ao finalizar, documente aqui:

### **Decisões Técnicas Tomadas:**

1. **Migration V028** - Seguindo numeração sequencial após V027
2. **Padrão de V027** - Seguiu EXATAMENTE o padrão que criei no Card 032
3. **Composite Primary Key** - Primeira no projeto! `PRIMARY KEY (action_id, kr_id)`
4. **3 Campos Opcionais Adicionados** - Todos úteis para o sistema:
   - `impact_level` - Nível de impacto (LOW, MEDIUM, HIGH)
   - `notes` - Observações sobre a vinculação
   - `created_by` - Origem (USER ou AI)
5. **2 Constraints CHECK** - Validações robustas:
   - `chk_action_kr_impact_level` - Valida 3 níveis de impacto
   - `chk_action_kr_created_by` - Valida origem (USER ou AI)
6. **2 Foreign Keys com CASCADE** - Limpeza automática bidirecional
7. **5 Índices** - Performance para queries bidirecionais:
   - `idx_action_kr_action_id` - Buscar KRs de uma action
   - `idx_action_kr_kr_id` - Buscar actions de um KR
   - `idx_action_kr_impact_level` - Filtrar por impacto
   - `idx_action_kr_created_by` - Filtrar por origem
   - `idx_action_kr_created_at` - Ordenação temporal
8. **Defaults Inteligentes** - MEDIUM para impact, USER para created_by
9. **Comentários SQL Completos** - Documentação inline detalhada
10. **Verificação Quádrupla** - Bloco `DO $$` valida colunas, índices, constraints CHECK E FKs

### **Estrutura SQL Final:**

```sql
-- V028__create_action_kr_links_table.sql
-- Criar tabela action_kr_links (vinculação many-to-many) para Goal Chunking
-- Card 033 - Objective Service: Criar Tabela Action-KR Links
-- Data: 08/11/2025

-- Tabela de vinculação many-to-many entre actions e key_results
CREATE TABLE IF NOT EXISTS action_kr_links (
    action_id UUID NOT NULL,
    kr_id UUID NOT NULL,
    impact_level VARCHAR(20) NOT NULL DEFAULT 'MEDIUM',
    notes TEXT,
    created_by VARCHAR(20) NOT NULL DEFAULT 'USER',
    created_at TIMESTAMP DEFAULT NOW(),
    
    -- Composite Primary Key (garante unicidade da vinculação)
    PRIMARY KEY (action_id, kr_id),
    
    -- Foreign keys com CASCADE para limpeza automática
    CONSTRAINT fk_action_kr_links_action 
        FOREIGN KEY (action_id) 
        REFERENCES actions(id) 
        ON DELETE CASCADE,
    
    CONSTRAINT fk_action_kr_links_kr 
        FOREIGN KEY (kr_id) 
        REFERENCES key_results(id) 
        ON DELETE CASCADE
);

-- Índices para performance (queries bidirecionais)
CREATE INDEX IF NOT EXISTS idx_action_kr_action_id ON action_kr_links(action_id);
CREATE INDEX IF NOT EXISTS idx_action_kr_kr_id ON action_kr_links(kr_id);
CREATE INDEX IF NOT EXISTS idx_action_kr_impact_level ON action_kr_links(impact_level);
CREATE INDEX IF NOT EXISTS idx_action_kr_created_by ON action_kr_links(created_by);
CREATE INDEX IF NOT EXISTS idx_action_kr_created_at ON action_kr_links(created_at);

-- Constraint para validar impact_level
DO $$ 
BEGIN
    IF NOT EXISTS (SELECT 1 FROM pg_constraint WHERE conname = 'chk_action_kr_impact_level') THEN
        ALTER TABLE action_kr_links ADD CONSTRAINT chk_action_kr_impact_level
            CHECK (impact_level IN ('LOW', 'MEDIUM', 'HIGH'));
    END IF;
END $$;

-- Constraint para validar created_by
DO $$ 
BEGIN
    IF NOT EXISTS (SELECT 1 FROM pg_constraint WHERE conname = 'chk_action_kr_created_by') THEN
        ALTER TABLE action_kr_links ADD CONSTRAINT chk_action_kr_created_by
            CHECK (created_by IN ('USER', 'AI'));
    END IF;
END $$;

-- Comentários para documentação
COMMENT ON TABLE action_kr_links IS 'Vinculação many-to-many entre actions e key_results (Goal Chunking - PRD 001)';
COMMENT ON COLUMN action_kr_links.action_id IS 'Referência para a ação (action)';
COMMENT ON COLUMN action_kr_links.kr_id IS 'Referência para o Key Result';
COMMENT ON COLUMN action_kr_links.impact_level IS 'Nível de impacto da ação no KR: LOW, MEDIUM (padrão), HIGH';
COMMENT ON COLUMN action_kr_links.notes IS 'Observações sobre a vinculação (opcional)';
COMMENT ON COLUMN action_kr_links.created_by IS 'Origem da vinculação: USER (manual) ou AI (gerada pela IA)';
COMMENT ON COLUMN action_kr_links.created_at IS 'Data de criação da vinculação';

-- Verificação de integridade
DO $$
DECLARE
    column_count INTEGER;
    index_count INTEGER;
    constraint_count INTEGER;
    fk_count INTEGER;
BEGIN
    -- Verificar colunas criadas
    SELECT COUNT(*) INTO column_count 
    FROM information_schema.columns 
    WHERE table_name = 'action_kr_links' 
    AND column_name IN ('action_id', 'kr_id', 'impact_level', 'notes', 'created_by', 'created_at');
    
    IF column_count != 6 THEN
        RAISE EXCEPTION 'Falha ao criar tabela action_kr_links: % colunas criadas (esperado: 6)', column_count;
    END IF;
    
    -- Verificar índices criados
    SELECT COUNT(*) INTO index_count 
    FROM pg_indexes 
    WHERE tablename = 'action_kr_links' 
    AND indexname LIKE 'idx_action_kr_%';
    
    IF index_count < 5 THEN
        RAISE EXCEPTION 'Falha ao criar índices: % índices criados (esperado: >= 5)', index_count;
    END IF;
    
    -- Verificar constraints CHECK
    SELECT COUNT(*) INTO constraint_count
    FROM pg_constraint
    WHERE conname IN ('chk_action_kr_impact_level', 'chk_action_kr_created_by');
    
    IF constraint_count < 2 THEN
        RAISE EXCEPTION 'Falha ao criar constraints CHECK: % constraints criados (esperado: >= 2)', constraint_count;
    END IF;
    
    -- Verificar foreign keys
    SELECT COUNT(*) INTO fk_count
    FROM pg_constraint
    WHERE conname IN ('fk_action_kr_links_action', 'fk_action_kr_links_kr');
    
    IF fk_count != 2 THEN
        RAISE EXCEPTION 'Falha ao criar foreign keys: % FKs criadas (esperado: 2)', fk_count;
    END IF;
    
    RAISE NOTICE 'Tabela action_kr_links criada com sucesso: % colunas, % índices, % constraints CHECK, % FKs', column_count, index_count, constraint_count, fk_count;
END $$;
```

### **Testes Realizados:**

✅ **1. Migration Executada**
- Flyway aplicou V028 com sucesso
- Tabela `action_kr_links` criada no banco
- Comando: `./gradlew bootRun`

✅ **2. Estrutura Verificada**
- 6 colunas criadas corretamente
- Composite Primary Key (action_id, kr_id)
- 6 índices criados (5 + composite PK)
- 2 Foreign keys configuradas
- 2 Constraints CHECK ativos
- Comando: `\d action_kr_links` no PostgreSQL

✅ **3. Vinculação Simples (1 action → 2 KRs)**
- Ação 1 vinculada a KR1 (HIGH) e KR2 (MEDIUM)
- 2 vinculações criadas com sucesso
- impact_level e created_by funcionando

✅ **4. Composite PK Bloqueando Duplicatas**
- Tentativa de inserir duplicata rejeitada
- Mensagem: "duplicate key value violates unique constraint"
- Composite PK funcionando perfeitamente

✅ **5. Constraint CHECK de Impact Level**
- Níveis válidos funcionam: LOW, MEDIUM, HIGH
- Nível inválido rejeitado com erro
- Mensagem: "violates check constraint chk_action_kr_impact_level"

✅ **6. Constraint CHECK de Created By**
- Origens válidas: USER, AI
- DEFAULT 'USER' funcionando

✅ **7. Vinculação Reversa (3 actions → 1 KR)**
- KR3 vinculado a 3 actions diferentes
- Impact levels diferentes: HIGH, MEDIUM, LOW
- Notes funcionando

✅ **8. Query com JOIN (buscar KRs de action)**
- JOIN entre key_results e action_kr_links funciona
- Índice `idx_action_kr_action_id` otimizando query
- Retornou 3 KRs vinculados à Ação 1

✅ **9. Query Reversa (buscar actions de KR)**
- JOIN entre actions e action_kr_links funciona
- Índice `idx_action_kr_kr_id` otimizando query
- Retornou 3 actions vinculadas ao KR3

✅ **10. ON DELETE CASCADE (deletar action)**
- Action deletada → 3 vinculações deletadas automaticamente
- Cascade funcionando para FK de actions

✅ **11. ON DELETE CASCADE (deletar KR)**
- KR deletado → 2 vinculações deletadas automaticamente
- Cascade funcionando para FK de key_results

✅ **12. Limpeza em Cascata**
- Stage deletado → Actions deletadas → Vinculações deletadas
- Cascade em toda a hierarquia funcionando

✅ **13. Índices Verificados**
- 6 índices listados no PostgreSQL (5 + composite PK)
- Todos com `IF NOT EXISTS`
- Queries bidirecionais otimizadas

### **Dificuldades Encontradas:**

Nenhuma! Composite Primary Key foi implementado com sucesso na primeira tentativa. O padrão de V027 facilitou muito a implementação.

### **Melhorias Implementadas:**

1. **3 Campos Opcionais Úteis** - `impact_level`, `notes`, `created_by` para contexto rico
2. **2 Constraints CHECK** - Validação de impact_level e created_by
3. **5 Índices de Performance** - Queries bidirecionais otimizadas
4. **Defaults Inteligentes** - MEDIUM para impact, USER para created_by
5. **Comentários SQL Completos** - Documentação inline detalhada
6. **Verificação Quádrupla** - Bloco `DO $$` valida colunas, índices, constraints CHECK E FKs
7. **Primeira Composite PK do Projeto** - Implementação de referência para futuras tabelas many-to-many

---

**Data de Criação:** 08/11/2025  
**Criado por:** Arquiobaldo (Arquiteto MoverseMais)  
**Contexto:** Módulo Goals - PRD 001 (Goal Chunking)  
**Dependência:** Card 032 ✅ (DONE)  
**Versão:** 1.0

---

## 🚀 **STATUS FINAL DA IMPLEMENTAÇÃO**

**Implementado por:** Osvaldo  
**Data de Implementação:** 08/11/2025  
**Branch:** `feature/goals-action-kr-links-table`  
**Commit:** `ff19401`  
**Status:** ✅ **CONCLUÍDO** - Aguardando validação arquitetural

### **Arquivos Criados:**
- `src/main/resources/db/migration/V028__create_action_kr_links_table.sql`

### **Resultado dos Testes:**
- ✅ Migration executada com sucesso
- ✅ Tabela criada com 6 colunas
- ✅ Composite Primary Key funcional (primeira do projeto!)
- ✅ 6 índices criados (5 + composite PK)
- ✅ 2 Foreign keys funcionais
- ✅ ON DELETE CASCADE testado (ambas FKs)
- ✅ 2 Constraints CHECK validados
- ✅ Vinculação 1:N testada (1 action → 2 KRs)
- ✅ Vinculação N:1 testada (3 actions → 1 KR)
- ✅ Queries com JOIN funcionando
- ✅ Duplicatas bloqueadas
- ✅ Impact levels validados
- ✅ Cascade em toda hierarquia

### **Próximos Passos:**
1. Validação arquitetural
2. Merge para main e limpeza de branch
3. Implementar Card 034 (checkins)

