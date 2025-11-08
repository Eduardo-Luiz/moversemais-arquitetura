# 🎯 Card 034 - Objective Service: Completar Sprint 1 - Estrutura de Dados Completa

**Agente Responsável:** Osvaldo  
**Microserviço:** moversemais-objective  
**Prioridade:** Alta  
**Status:** TODO  
**Estimativa:** 4-6 horas

---

## 🔍 **ESTUDO PRÉVIO REALIZADO**

**Arquiobaldo estudou a aplicação antes de criar este card:**
- ✅ Leu `Objective.kt` (139 linhas - entity bem estruturada)
- ✅ Leu `ObjectiveRepository.kt` (372 linhas - muitas queries úteis)
- ✅ Leu `Assessment.kt` (padrão de entity com @GeneratedValue UUID)
- ✅ Verificou estrutura de pacotes (entity/, repository/, usecase/)
- ✅ Verificou migrations Flyway (V025-V028 criadas por você!)
- ✅ Leu AGENTS.md (Clean Architecture, Strategy Pattern)

---

## 📋 CONTEXTO

### **Situação Atual**

Você completou com **EXCELÊNCIA** os Cards 030-033:
- ✅ Card 030: Tabela `stages` (Score 5/5)
- ✅ Card 031: Tabela `actions` (Score 6/5)
- ✅ Card 032: Tabela `key_results` (Score 7/5)
- ✅ Card 033: Tabela `action_kr_links` (Score 8/5 + INOVAÇÃO!)

**Padrão estabelecido:**
- Migrations SQL exemplares (V025-V028)
- Verificação robusta (até quádrupla!)
- Comentários completos
- Testes exaustivos

### **Problema Identificado**

Para implementar **PRD 001 (Goal Chunking)** e **PRD 002 (Check-in de Progresso)**, ainda faltam:

1. ❌ **Tabela `checkins`** - Histórico de progresso dos Key Results
2. ❌ **Campos em `objectives`** - motive, context, mode (para Goal Chunking)
3. ❌ **Entities JPA** - Representação Kotlin das tabelas criadas
4. ❌ **Repositories** - Acesso aos dados via Spring Data JPA

**Sem isso:**
- Banco de dados incompleto (sem histórico de check-ins)
- Objective não tem campos para Goal Chunking
- Backend não consegue acessar os dados (sem entities/repositories)
- PRD 001 e PRD 002 impossíveis de implementar

### **Solução Proposta**

Completar a **Sprint 1 - Estrutura de Dados** criando:
1. Tabela `checkins` (histórico de atualizações de KRs)
2. Campos adicionais em `objectives` (suporte a Goal Chunking)
3. Entities JPA (Stage, Action, KeyResult, ActionKrLink, Checkin)
4. Repositories Spring Data JPA (acesso aos dados)

### **Onde se Encaixa na Arquitetura**

```
Sprint 1: Estrutura de Dados (FINALIZAR)
├─ Card 030: Tabela stages ✅ DONE
├─ Card 031: Tabela actions ✅ DONE
├─ Card 032: Tabela key_results ✅ DONE
├─ Card 033: Tabela action_kr_links ✅ DONE
└─ Card 034: Finalizar Sprint 1 ← ESTE CARD
    ├─ Tabela checkins
    ├─ Campos em objectives
    ├─ Entities JPA
    └─ Repositories
```

### **Impacto se Não For Feito**

- Sprint 1 incompleta
- Backend não acessa dados criados
- Sprint 2 (Goal Chunking) bloqueada
- Sprint 3 (Check-in) bloqueada

---

## 🎯 REQUISITOS OBRIGATÓRIOS

### **1. Tabela `checkins` - Histórico de Progresso**

**Função de Negócio:**
Registrar **TODAS as atualizações** de progresso dos Key Results ao longo do tempo. Quando um usuário faz check-in (atualiza um KR), o sistema deve:
- Registrar valor anterior e novo valor
- Armazenar timestamp da atualização
- Permitir análise de evolução temporal
- Suportar feedback/notas sobre o progresso

**Requisitos Funcionais:**
- Vincular check-in a um objetivo específico
- Vincular check-in a um Key Result específico
- Armazenar valor anterior (before)
- Armazenar novo valor (after)
- Permitir notas/observações do usuário
- Permitir feedback gerado por IA
- Timestamp automático de criação
- Suportar análise de histórico (ordenação temporal)

**Restrições:**
- Check-in é **imutável** (não pode ser editado após criação)
- Relacionamento obrigatório com objective e key_result
- Se objective deletado → check-ins deletados (CASCADE)
- Se key_result deletado → check-ins deletados (CASCADE)

**Casos de Uso:**
- Usuário atualiza KR "Tempo de corrida" de 35min para 32min
- Sistema registra: previous=35, new=32, timestamp=now
- IA gera feedback: "Ótimo progresso! Você melhorou 8.5%"
- Usuário visualiza histórico de evolução do KR

---

### **2. Campos em `objectives` - Suporte a Goal Chunking**

**Função de Negócio:**
Adicionar campos que suportam **Goal Chunking (PRD 001)**:

**Campo `motive` (Motivo):**
- **Por que** essa meta importa para o usuário?
- Motivação pessoal/profissional
- Usado pela IA para gerar plano contextualizado
- Exemplo: "Quero correr 5km para melhorar minha saúde e ter mais energia no trabalho"

**Campo `context` (Contexto):**
- **Como** o usuário pretende alcançar a meta?
- Contexto de execução
- Recursos disponíveis
- Restrições conhecidas
- Exemplo: "Tenho 30min por dia, 3x por semana, treino no parque perto de casa"

**Campo `mode` (Modo):**
- **AUTO**: IA gera plano completo (Etapas + Ações + KRs)
- **MANUAL**: Usuário cria plano manualmente
- Determina fluxo de criação da meta

**Requisitos Funcionais:**
- `motive`: TEXT (opcional, mas recomendado para modo AUTO)
- `context`: TEXT (opcional, mas recomendado para modo AUTO)
- `mode`: ENUM ou VARCHAR (AUTO, MANUAL) - DEFAULT 'MANUAL'
- Campos devem ser **adicionados** à tabela existente (não recriar)
- Metas antigas continuam funcionando (campos nullable ou com default)

---

### **3. Entities JPA - Representação Kotlin**

**Função de Negócio:**
Permitir que o backend **acesse e manipule** os dados das tabelas criadas (stages, actions, key_results, action_kr_links, checkins) usando Kotlin.

**Requisitos Funcionais:**
Criar entities JPA para:
1. **Stage** (Etapa)
2. **Action** (Ação)
3. **KeyResult** (Indicador de Resultado)
4. **ActionKrLink** (Vinculação Many-to-Many)
5. **Checkin** (Histórico de Progresso)

**Expectativas:**
- Seguir padrão de `Objective.kt` existente (estudado - 139 linhas)
- Usar `@GeneratedValue(strategy = GenerationType.AUTO)` ou `UUID` (padrão do projeto)
- Usar `@CreationTimestamp` e `@UpdateTimestamp` (Hibernate - padrão do projeto)
- Data classes do Kotlin (padrão do projeto)
- Relacionamentos JPA corretos (@ManyToOne, @OneToMany, @ManyToMany)
- Cascade apropriado (LGPD - deletar dados relacionados)
- Validações Bean Validation onde apropriado (@Min, @Max, @NotNull)
- Métodos auxiliares úteis (isCompleted, calculateProgress, etc.)
- Você decide estrutura de classes, annotations, relacionamentos

**Padrão Identificado em Objective.kt:**
```kotlin
@Entity
@Table(name = "objectives", indexes = [...])
data class Objective(
    @Id
    @GeneratedValue(strategy = GenerationType.AUTO)
    val id: UUID? = null,
    
    @CreationTimestamp
    @Column(name = "created_at", nullable = false, updatable = false)
    val createdAt: LocalDateTime? = null,
    
    @UpdateTimestamp
    @Column(name = "updated_at", nullable = false)
    var updatedAt: LocalDateTime? = null,
    
    // ... outros campos
) {
    // Métodos auxiliares
    fun isActive(): Boolean = status == Status.ACTIVE
}
```

**Restrições:**
- NÃO alterar `Objective.kt` existente (apenas adicionar campos)
- Seguir nomenclatura do projeto (PascalCase para classes)
- Usar data classes do Kotlin quando apropriado

---

### **4. Repositories - Acesso aos Dados**

**Função de Negócio:**
Permitir que **Services** acessem os dados usando Spring Data JPA (queries automáticas e customizadas).

**Requisitos Funcionais:**
Criar repositories para:
1. **StageRepository**
2. **ActionRepository**
3. **KeyResultRepository**
4. **ActionKrLinkRepository**
5. **CheckinRepository**

**Queries Necessárias (exemplos):**
- Buscar stages de um objetivo (por objectiveId)
- Buscar actions de uma stage (por stageId)
- Buscar key results de um objetivo (por objectiveId)
- Buscar check-ins de um KR (por krId, ordenado por data)
- Buscar actions vinculadas a um KR (via action_kr_links)
- Buscar KRs vinculados a uma action (via action_kr_links)

**Expectativas:**
- Seguir padrão de `ObjectiveRepository.kt` existente (estudado - 372 linhas!)
- Usar Spring Data JPA (interface extends JpaRepository)
- Queries derivadas de nomes de métodos (findByObjectiveId, findAllByStageId, etc.)
- @Query customizadas quando necessário (para queries complexas)
- Você decide quais queries criar (baseado em casos de uso)

**Padrão Identificado em ObjectiveRepository.kt:**
```kotlin
@Repository
interface ObjectiveRepository : JpaRepository<Objective, UUID> {
    // Queries derivadas (Spring Data JPA)
    fun findAllByUserId(userId: UUID): List<Objective>
    fun findAllByUserIdAndStatus(userId: UUID, status: Status): List<Objective>
    
    // Queries customizadas (@Query)
    @Query("SELECT o FROM Objective o WHERE o.userId = :userId AND o.status = 'ACTIVE'")
    fun findActiveObjectivesByUserId(@Param("userId") userId: UUID): List<Objective>
    
    // Contagens
    fun countByUserIdAndStatus(userId: UUID, status: Status): Long
}
```

**Queries Úteis Identificadas:**
- `findAllByObjectiveId` (buscar stages/actions/KRs de um objetivo)
- `findAllByStageId` (buscar actions de uma stage)
- `findAllByKrId` (buscar check-ins de um KR)
- Ordenação por `order_index` (stages, actions)
- Ordenação por `created_at` (check-ins - histórico temporal)

**Restrições:**
- NÃO alterar `ObjectiveRepository.kt` existente

---

## ⚠️ RESTRIÇÕES

### **O que NÃO PODE ser alterado:**

1. ❌ **NÃO alterar migrations V025-V028** (já aplicadas e validadas)
2. ❌ **NÃO alterar `Objective.kt`** (apenas adicionar campos motive, context, mode)
3. ❌ **NÃO alterar `ObjectiveRepository.kt`** (apenas criar novos repositories)
4. ❌ **NÃO alterar `ObjectiveService.kt`** (será evoluído em Sprint 2)
5. ❌ **NÃO alterar `ObjectiveController.kt`** (será evoluído em Sprint 2)
6. ❌ **NÃO criar Services ainda** (GoalChunkingService será Sprint 2)
7. ❌ **NÃO criar Controllers ainda** (endpoints serão Sprint 2)
8. ❌ **NÃO criar DTOs ainda** (serão criados conforme necessidade em Sprint 2)

### **O que DEVE ser preservado:**

1. ✅ **Padrão de migrations** (V{numero}__descricao.sql)
2. ✅ **Padrão de entities** (seguir Objective.kt)
3. ✅ **Padrão de repositories** (seguir ObjectiveRepository.kt)
4. ✅ **Nomenclatura do projeto** (PascalCase, camelCase, snake_case)
5. ✅ **Annotations JPA** (padrão do projeto)
6. ✅ **Estrutura de pacotes** (entity, repository, etc.)

---

## 📚 DOCUMENTAÇÃO DE REFERÊNCIA

### **O QUE ARQUIOBALDO JÁ ESTUDOU (para você):**

**Padrões Identificados:**
1. ✅ **Entity Pattern**: `@GeneratedValue(UUID)`, `@CreationTimestamp`, data classes
2. ✅ **Repository Pattern**: Queries derivadas + @Query customizadas (372 linhas em ObjectiveRepository!)
3. ✅ **Estrutura de Pacotes**: entity/, repository/, usecase/, controller/
4. ✅ **Validações**: @Min, @Max, @NotNull (Bean Validation)
5. ✅ **Métodos Auxiliares**: isActive(), markAsCompleted(), etc.
6. ✅ **Índices**: @Table(indexes = [...]) para performance

**Você não precisa estudar tudo novamente, mas pode consultar se tiver dúvidas.**

### **Arquivos para Estudar (SE NECESSÁRIO):**

1. **Suas Migrations (REFERÊNCIA PRINCIPAL):**
   - `V025__create_stages_table.sql` (você criou - Card 030)
   - `V026__create_actions_table.sql` (você criou - Card 031)
   - `V027__create_key_results_table.sql` (você criou - Card 032)
   - `V028__create_action_kr_links_table.sql` (você criou - Card 033)
   - **Você conhece essas tabelas melhor que ninguém!**

2. **Padrão de Entity JPA:**
   - `src/main/kotlin/com/moversemais/objective/entity/Objective.kt`
   - Estudar: annotations, relacionamentos, métodos auxiliares

3. **Padrão de Repository:**
   - `src/main/kotlin/com/moversemais/objective/repository/ObjectiveRepository.kt`
   - Estudar: queries derivadas, @Query customizadas

4. **PRD 002 - Check-in de Progresso:**
   - `../moversemais-arquitetura/PRD/prd_002_checkin_progress.md`
   - Entender lógica de check-in e histórico

5. **Análise Arquitetural:**
   - `../moversemais-arquitetura/ANALYSIS__Goals-Module-Architecture.md`
   - Seção "TABELA 5: checkins" (linhas 300-320)
   - Seção "Campos em objectives" (linhas 313-318)

6. **Documentação:**
   - `../moversemais-objective/AGENTS.md`
   - `../moversemais-arquitetura/AGENTS.md`

### **Cards Relacionados:**
- Card 030-033: Tabelas criadas ✅ (DONE - você criou!)
- Card 035: Sprint 2 - Goal Chunking (próximo - depende deste)
- Card 036: Sprint 3 - Check-in (próximo - depende deste)

---

## 🔧 WORKFLOW

### **1. ESTUDAR (OBRIGATÓRIO - 30 minutos)**

```bash
cd moversemais-objective

# Estudar suas próprias migrations (V025-V028)
cat src/main/resources/db/migration/V025__create_stages_table.sql
cat src/main/resources/db/migration/V026__create_actions_table.sql
cat src/main/resources/db/migration/V027__create_key_results_table.sql
cat src/main/resources/db/migration/V028__create_action_kr_links_table.sql

# Estudar padrão de Entity
cat src/main/kotlin/com/moversemais/objective/entity/Objective.kt

# Estudar padrão de Repository
cat src/main/kotlin/com/moversemais/objective/repository/ObjectiveRepository.kt

# Ler PRD 002 (Check-in)
cat ../moversemais-arquitetura/PRD/prd_002_checkin_progress.md

# Ler análise arquitetural
cat ../moversemais-arquitetura/ANALYSIS__Goals-Module-Architecture.md | grep -A 30 "checkins"

# Ler AGENTS.md
cat AGENTS.md
```

**Perguntas para Responder Antes de Implementar:**
- Migration V029 para `checkins`?
- Migration V030 para campos em `objectives`?
- Estrutura de entities: relacionamentos bidirecionais ou unidirecionais?
- Cascade types: ALL, REMOVE, PERSIST?
- Repositories: quais queries customizadas criar?
- Validações: @NotNull, @Size, @Min, @Max?

### **2. CRIAR BRANCH**

```bash
git checkout -b feature/goals-complete-sprint-1
```

### **3. IMPLEMENTAR (VOCÊ DECIDE COMO)**

**Você é o especialista em Spring Boot + Kotlin + JPA.**

**Eu defino O QUE precisa ser feito. Você decide COMO fazer.**

**Ordem sugerida (você pode mudar):**
1. Migration V029 (tabela checkins)
2. Migration V030 (campos em objectives)
3. Entities JPA (Stage, Action, KeyResult, ActionKrLink, Checkin)
4. Atualizar Objective.kt (adicionar campos)
5. Repositories (Stage, Action, KeyResult, ActionKrLink, Checkin)
6. Testes (se julgar necessário)

**Decisões técnicas que você toma:**
- Estrutura exata das entities (campos, tipos, annotations)
- Relacionamentos JPA (@ManyToOne, @OneToMany, fetch types)
- Cascade types (ALL, REMOVE, PERSIST, MERGE)
- Validações Bean Validation
- Métodos auxiliares nas entities
- Queries customizadas nos repositories
- Índices adicionais (se julgar necessário)
- Nomes de classes, métodos, variáveis
- Organização de código

**Mas DEVE seguir:**
- ✅ Padrão de migrations (V{numero}__descricao.sql)
- ✅ Padrão de Objective.kt (annotations, estrutura)
- ✅ Padrão de ObjectiveRepository.kt (Spring Data JPA)
- ✅ Nomenclatura do projeto
- ✅ Estrutura de pacotes

### **4. TESTAR**

**Testes Obrigatórios:**

```bash
# 1. Rodar aplicação (Flyway executa migrations)
./gradlew bootRun

# Verificar logs:
# "Migrating schema to version 029 - create checkins table"
# "Migrating schema to version 030 - add goal chunking fields to objectives"
# "Successfully applied 2 migrations"

# 2. Conectar ao PostgreSQL
docker exec moversemais-postgres psql -U developer -d moversemais_objective

# 3. Verificar tabela checkins
\d checkins

# 4. Verificar campos em objectives
\d objectives
# Deve mostrar: motive, context, mode

# 5. Testar entities (criar instâncias)
# Você decide como testar (unit tests, integration tests, ou manual)

# 6. Testar repositories (queries)
# Você decide como testar

# 7. Testar relacionamentos JPA
# Criar objective → stages → actions → key_results
# Verificar cascade, fetch, lazy/eager loading
```

**Você decide:**
- Criar testes automatizados (JUnit + Spring Boot Test)?
- Testar manualmente via PostgreSQL?
- Criar script de teste?
- Nível de cobertura de testes?

### **5. DOCUMENTAR DECISÕES**

Ao final do card, documente:
- Estrutura SQL das migrations (V029, V030)
- Estrutura das entities JPA
- Relacionamentos JPA escolhidos (e por quê)
- Cascade types escolhidos (e por quê)
- Queries customizadas criadas (e por quê)
- Testes realizados
- Dificuldades encontradas
- Decisões técnicas tomadas

### **6. COMMIT E PUSH**

```bash
git add .
git commit -m "feat(objective-service): completa Sprint 1 - estrutura de dados

- Migration V029: tabela checkins (histórico de progresso)
- Migration V030: campos em objectives (motive, context, mode)
- Entities JPA: Stage, Action, KeyResult, ActionKrLink, Checkin
- Atualização: Objective.kt (campos adicionais)
- Repositories: Stage, Action, KeyResult, ActionKrLink, Checkin
- Relacionamentos JPA configurados
- Testes realizados
- Sprint 1 COMPLETA
- Ref: Card 034"

git push origin feature/goals-complete-sprint-1
```

### **7. MOVER PARA VALIDAÇÃO**

```bash
mv Development/TODO/034__Objective-Service-Complete-Sprint-1-Data-Structure.md \
   Development/VALIDATING/034__Objective-Service-Complete-Sprint-1-Data-Structure.md
```

---

## ✅ CRITÉRIOS DE VALIDAÇÃO

### **Funcionalidades:**
- [ ] Migration V029 criada e executada (tabela checkins)
- [ ] Migration V030 criada e executada (campos em objectives)
- [ ] Tabela `checkins` existe no banco
- [ ] Campos `motive`, `context`, `mode` existem em `objectives`
- [ ] Entities JPA criadas (Stage, Action, KeyResult, ActionKrLink, Checkin)
- [ ] Objective.kt atualizado (campos adicionais)
- [ ] Repositories criados (5 novos)
- [ ] Relacionamentos JPA funcionam
- [ ] Cascade funciona (deletar objective → deletar relacionados)
- [ ] Aplicação inicia sem erros

### **Padrão:**
- [ ] Seguiu padrão de migrations (V025-V028)
- [ ] Seguiu padrão de Objective.kt
- [ ] Seguiu padrão de ObjectiveRepository.kt
- [ ] Nomenclatura consistente
- [ ] Estrutura de pacotes correta
- [ ] Comentários SQL (se aplicável)
- [ ] Documentação de decisões técnicas

### **Qualidade:**
- [ ] Código limpo e legível
- [ ] Relacionamentos JPA corretos
- [ ] Validações apropriadas
- [ ] Testes realizados (nível decidido por você)
- [ ] Sem warnings/erros de compilação

---

## 🚨 TROUBLESHOOTING

### **Problema: Migration não executa**
**Solução:**
- Verificar numeração (V029, V030)
- Verificar sintaxe SQL
- Logs: `./gradlew bootRun | grep Flyway`

### **Problema: Entity não mapeia tabela**
**Solução:**
- Verificar @Entity, @Table(name = "...")
- Verificar nomes de colunas (@Column(name = "..."))
- Verificar tipos de dados (Kotlin ↔ PostgreSQL)

### **Problema: Relacionamento JPA não funciona**
**Solução:**
- Verificar @ManyToOne, @OneToMany, @ManyToMany
- Verificar mappedBy (relacionamento bidirecional)
- Verificar fetch type (LAZY vs EAGER)
- Verificar cascade type

### **Problema: Repository não encontra dados**
**Solução:**
- Verificar nome do método (Spring Data JPA conventions)
- Verificar @Query (se customizada)
- Verificar relacionamento na entity

---

## 🎯 EXPECTATIVAS

### **Você é o Especialista em Backend**

**Osvaldo, você criou V025-V028 com EXCELÊNCIA CRESCENTE:**
- Card 030: Score 5/5 ⭐⭐⭐⭐⭐
- Card 031: Score 6/5 ⭐⭐⭐⭐⭐⭐
- Card 032: Score 7/5 ⭐⭐⭐⭐⭐⭐⭐
- Card 033: Score 8/5 ⭐⭐⭐⭐⭐⭐⭐⭐ + INOVAÇÃO!

**Você estabeleceu o padrão de qualidade do projeto.**

**Por isso, eu confio TOTALMENTE em você para:**

1. ✅ **Criar migrations V029 e V030** (você domina SQL)
2. ✅ **Criar entities JPA** (você conhece Spring Boot + Kotlin)
3. ✅ **Criar repositories** (você sabe Spring Data JPA)
4. ✅ **Definir relacionamentos** (você entende JPA)
5. ✅ **Tomar decisões técnicas** (você é o especialista)

**Eu defini:**
- ✅ **O QUE** precisa ser feito (requisitos de negócio)
- ✅ **POR QUE** precisa ser feito (contexto)
- ✅ **RESTRIÇÕES** (o que não pode ser alterado)
- ✅ **CRITÉRIOS DE VALIDAÇÃO** (como saber se está correto)

**Você decide:**
- ✅ **COMO** implementar (estrutura de código)
- ✅ **Relacionamentos JPA** (bidirecionais? cascade?)
- ✅ **Queries customizadas** (quais criar?)
- ✅ **Validações** (quais adicionar?)
- ✅ **Testes** (qual nível de cobertura?)
- ✅ **Métodos auxiliares** (quais são úteis?)

**Este é um card CONSOLIDADO e AMBICIOSO.**

**Você já provou que consegue entregar com EXCELÊNCIA.**

**Agora, mostre que consegue entregar um SPRINT COMPLETO!** 🚀

---

## 📊 OUTPUT ESPERADO

Ao finalizar, documente aqui:

### **Decisões Técnicas Tomadas:**
(Você preenche)

### **Estrutura SQL Final (V029 e V030):**
(Cole o SQL completo)

### **Estrutura das Entities JPA:**
(Descreva relacionamentos, cascade, validações)

### **Repositories Criados:**
(Liste queries customizadas, se houver)

### **Testes Realizados:**
(Liste cenários testados)

### **Dificuldades Encontradas:**
(Se houver)

### **Melhorias Implementadas:**
(Além do requisitado)

---

**Data de Criação:** 08/11/2025  
**Criado por:** Arquiobaldo (Arquiteto MoverseMais)  
**Contexto:** Módulo Goals - Finalizar Sprint 1 (Estrutura de Dados)  
**Dependência:** Cards 030-033 ✅ (DONE)  
**Versão:** 1.0

