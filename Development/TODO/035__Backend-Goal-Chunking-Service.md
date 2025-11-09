# 🎯 Card 035 - Backend: Goal Chunking Service (IA Gera Planos)

**Agente Responsável:** Osvaldo  
**Microserviço:** moversemais-objective  
**Prioridade:** Alta  
**Status:** TODO  
**Estimativa:** 6-8 horas

---

## 🔍 **ESTUDO PRÉVIO REALIZADO**

**Arquiobaldo estudou a aplicação antes de criar este card:**
- ✅ Leu PRD 001 (Goal Chunking) - 95 linhas
- ✅ Verificou `AssessmentDiagnosisService.kt` - Como chamar ChatGPT e parsear JSON
- ✅ Verificou `KeyResultsService.kt` - Exemplo de prompt estruturado
- ✅ Verificou `ChatGPTService.kt` - Serviço base de comunicação com IA
- ✅ Sprint 1 COMPLETA - Entities e Repositories criados (você criou!)

---

## 📋 CONTEXTO

### **Situação Atual**

**Sprint 1 COMPLETA** ✅ (você completou!)
- Tabelas criadas: stages, actions, key_results, action_kr_links, checkins
- Entities JPA: Stage, Action, KeyResult, ActionKrLink, Checkin
- Repositories: Todos criados com queries customizadas
- Campos em objectives: motive, context, mode

**Integração IA Existente** ✅
- `ChatGPTService.kt` - Comunicação com OpenAI
- `AssessmentDiagnosisService.kt` - Gera diagnóstico com IA
- `KeyResultsService.kt` - Gera Key Results com IA
- Padrão estabelecido: prompt estruturado → JSON response → parsing

**O que falta:**
- ❌ Backend não gera planos estruturados (Etapas + Ações + KRs)
- ❌ Não existe GoalChunkingService
- ❌ Não existe endpoint para gerar plano
- ❌ IA não é usada para Goal Chunking

### **Problema Identificado**

Para implementar **PRD 001 (Goal Chunking)**, precisamos que:

**Backend:**
- IA transforme meta em plano estruturado
- Plano contenha: Etapas → Ações → Key Results → Vinculações
- Plano seja salvo no banco usando entities da Sprint 1
- Endpoint permita BFF solicitar geração de plano

**Exemplo do que queremos:**
```
Meta: "Correr 5km em 30 minutos"
Motivo: "Melhorar saúde e disposição"
Contexto: "Treinar 3x por semana, 30min"

↓ IA GERA ↓

Etapa 1: Avaliar condicionamento físico atual
  ├─ Ação 1: Fazer corrida teste de 2km → impacta KR1, KR2
  └─ Ação 2: Registrar tempo e frequência → impacta KR1, KR2

Etapa 2: Criar rotina de treinos
  ├─ Ação 3: Planejar 3 treinos semanais → impacta KR3
  └─ Ação 4: Executar primeiro treino → impacta KR1, KR3

Key Results:
  KR1: Tempo de corrida (30 min) - Peso: 50%
  KR2: Frequência cardíaca (150 bpm) - Peso: 30%
  KR3: Treinos completados (12) - Peso: 20%
```

### **Solução Proposta**

Criar **GoalChunkingService** que:
1. Busca objective por ID
2. Monta prompt estruturado para ChatGPT
3. Chama IA solicitando JSON estruturado
4. Parseia resposta
5. Valida estrutura do plano
6. Salva no banco (stages → actions → key_results → action_kr_links)
7. Retorna plano completo

Criar **Endpoint** `POST /objectives/{id}/generate-plan`

### **Onde se Encaixa na Arquitetura**

```
Sprint 2: Goal Chunking - Backend (este card)
├─ GoalChunkingService (IA gera plano)
├─ POST /objectives/{id}/generate-plan
├─ DTOs (GoalPlanRequest, GoalPlanResponse)
└─ UseCase (se seguir Clean Architecture)

Próximo:
├─ Card 036: BFF (Gabriela) - Mutations GraphQL
└─ Card 037: Frontend (Lisa) - Interface
```

### **Impacto se Não For Feito**

- BFF não consegue solicitar geração de plano
- Frontend não consegue usar IA
- PRD 001 não implementado
- Sprint 2 bloqueada

---

## 🎯 REQUISITOS OBRIGATÓRIOS

### **1. GoalChunkingService - IA Gera Plano**

**Função de Negócio:**
Transformar uma meta (objective) em plano estruturado usando ChatGPT.

**Requisitos Funcionais:**
- Buscar objective por ID (usar ObjectiveRepository)
- Validar que objective existe e pertence ao usuário
- Validar que objective tem mode = 'AUTO' (Goal Chunking)
- Montar prompt estruturado para ChatGPT incluindo:
  - Título da meta (objectiveText)
  - Motivo (motive) - por que importa
  - Contexto (context) - como pretende alcançar
  - Prazo (startDate, endDate)
  - Tipo (objectiveType)
- Chamar ChatGPT solicitando JSON estruturado
- Parsear resposta da IA (JSON → objetos Kotlin)
- Validar estrutura do plano gerado
- Salvar no banco usando repositories da Sprint 1:
  - StageRepository
  - ActionRepository
  - KeyResultRepository
  - ActionKrLinkRepository
- Retornar plano completo

**Estrutura do JSON esperado da IA:**
```json
{
  "stages": [
    {
      "title": "Etapa 1: Avaliar condicionamento físico",
      "description": "Entender seu nível atual antes de começar",
      "order": 1,
      "actions": [
        {
          "title": "Fazer corrida teste de 2km",
          "description": "Correr 2km em ritmo confortável",
          "order": 1,
          "linkedKRs": [0, 1]
        },
        {
          "title": "Registrar tempo e frequência cardíaca",
          "description": "Anotar resultados para baseline",
          "order": 2,
          "linkedKRs": [0, 1]
        }
      ]
    },
    {
      "title": "Etapa 2: Criar rotina de treinos",
      "description": "Estabelecer frequência e intensidade",
      "order": 2,
      "actions": [
        {
          "title": "Planejar 3 treinos semanais",
          "description": "Definir dias e horários",
          "order": 1,
          "linkedKRs": [2]
        }
      ]
    }
  ],
  "keyResults": [
    {
      "title": "Tempo de corrida",
      "description": "Reduzir tempo para 30 minutos",
      "type": "TIME",
      "targetValue": 30.00,
      "unit": "min",
      "weight": 0.50,
      "orderIndex": 1
    },
    {
      "title": "Frequência cardíaca média",
      "description": "Manter abaixo de 150 bpm",
      "type": "NUMERIC",
      "targetValue": 150.00,
      "unit": "bpm",
      "weight": 0.30,
      "orderIndex": 2
    },
    {
      "title": "Treinos completados",
      "description": "Completar 12 treinos",
      "type": "NUMERIC",
      "targetValue": 12.00,
      "unit": "treinos",
      "weight": 0.20,
      "orderIndex": 3
    }
  ]
}
```

**Validações Obrigatórias:**
- Mínimo 1 etapa (stage)
- Mínimo 3 ações (actions) no total
- Mínimo 1 Key Result (KR)
- Soma dos weights dos KRs = 1.0 (ou próximo, ex: 0.99-1.01)
- Tipos de KR válidos: NUMERIC, PERCENTAGE, BINARY, CURRENCY, TIME
- linkedKRs válidos (índices existentes no array de KRs)
- order sequencial (1, 2, 3...)

**Prompt para ChatGPT (exemplo):**
```
Você é um assistente especializado em planejamento de metas.

OBJETIVO: Transformar a meta do usuário em um plano estruturado.

META DO USUÁRIO:
- Título: {objectiveText}
- Motivo: {motive}
- Contexto: {context}
- Prazo: {startDate} até {endDate}
- Tipo: {objectiveType}

INSTRUÇÕES:
1. Crie ETAPAS sequenciais (stages) para alcançar a meta
2. Para cada etapa, crie AÇÕES executáveis (actions)
3. Crie KEY RESULTS (KRs) mensuráveis para medir progresso
4. Vincule cada ação aos KRs que ela impacta (linkedKRs)

REGRAS:
- Mínimo 1 etapa, 3 ações, 1 KR
- Ações devem ser específicas e executáveis
- KRs devem ser mensuráveis (número, percentual, sim/não, moeda, tempo)
- Soma dos weights dos KRs = 1.0
- linkedKRs são índices do array de keyResults (0, 1, 2...)

TIPOS DE KR:
- NUMERIC: valor numérico (ex: 12 treinos)
- PERCENTAGE: percentual 0-100 (ex: 80% satisfação)
- BINARY: sim/não (ex: certificação obtida)
- CURRENCY: valor monetário (ex: R$ 5000)
- TIME: tempo/duração (ex: 30 minutos)

RESPONDA APENAS COM JSON VÁLIDO:
{
  "stages": [...],
  "keyResults": [...]
}
```

**Referências Existentes (ESTUDAR):**
- `AssessmentDiagnosisService.kt` - Como chamar ChatGPT e parsear JSON
- `KeyResultsService.kt` - Exemplo de prompt estruturado
- `ChatGPTService.kt` - Serviço base

**Você decide:**
- Estrutura de classes (service, DTOs internos)
- Nomes de métodos
- Lógica de parsing
- Tratamento de erros
- Logs e auditoria

**Restrições:**
- NÃO alterar ChatGPTService base
- NÃO alterar entities da Sprint 1
- NÃO alterar repositories da Sprint 1
- Reutilizar ChatGPTService existente

---

### **2. Endpoint POST /objectives/{id}/generate-plan**

**Função de Negócio:**
Permitir que BFF solicite geração de plano para uma meta específica.

**Requisitos Funcionais:**
- **Path:** `POST /api/v1/objectives/{objectiveId}/generate-plan`
- **Headers obrigatórios:** 
  - `X-User-Id` (UUID do usuário)
  - `X-Internal-Service-Key` (segurança - validar)
- **Body (opcional):**
  ```json
  {
    "regenerate": false  // true = deletar plano antigo e gerar novo
  }
  ```
- **Response 200 (sucesso):**
  ```json
  {
    "success": true,
    "message": "Plano gerado com sucesso",
    "objective": {
      "id": "uuid",
      "title": "...",
      "stages": [...],
      "keyResults": [...]
    }
  }
  ```
- **Response 404:** Objective não encontrado
- **Response 403:** Objective não pertence ao usuário
- **Response 400:** 
  - Objective não está em modo AUTO
  - Objective sem motive/context (recomendar preencher)
  - Plano já existe e regenerate=false
- **Response 500:** Erro ao chamar IA ou salvar dados

**Comportamento:**
- Se objective já tem plano (stages existentes):
  - Se `regenerate=false`: retornar erro 400 "Plano já existe"
  - Se `regenerate=true`: deletar plano antigo (cascade) e gerar novo
- Se objective não tem plano: gerar novo
- Validar X-Internal-Service-Key (segurança)
- Validar X-User-Id (ownership)

**Você decide:**
- Criar Controller ou adicionar método em ObjectiveController
- Criar UseCase (se seguir Clean Architecture)
- Estrutura de DTOs (request, response)
- Tratamento de erros
- Logs e auditoria

**Restrições:**
- NÃO alterar endpoints existentes
- Seguir padrão de segurança do projeto (X-Internal-Service-Key)
- Seguir padrão de Clean Architecture (se usado)

---

### **3. DTOs (Data Transfer Objects)**

**Criar DTOs para Request/Response:**

**GoalPlanRequest (opcional):**
```kotlin
data class GoalPlanRequest(
    val regenerate: Boolean = false
)
```

**GoalPlanResponse:**
```kotlin
data class GoalPlanResponse(
    val success: Boolean,
    val message: String,
    val objective: ObjectiveWithPlanDTO? = null,
    val errors: List<String>? = null
)

data class ObjectiveWithPlanDTO(
    val id: UUID,
    val title: String,
    val motive: String?,
    val context: String?,
    val mode: String,
    val startDate: LocalDateTime,
    val endDate: LocalDateTime,
    val stages: List<StageDTO>,
    val keyResults: List<KeyResultDTO>
)

data class StageDTO(
    val id: UUID,
    val title: String,
    val description: String?,
    val orderIndex: Int,
    val actions: List<ActionDTO>
)

data class ActionDTO(
    val id: UUID,
    val title: String,
    val description: String?,
    val status: String,
    val orderIndex: Int,
    val linkedKeyResults: List<KeyResultDTO>
)

data class KeyResultDTO(
    val id: UUID,
    val title: String,
    val description: String?,
    val type: String,
    val targetValue: BigDecimal,
    val currentValue: BigDecimal,
    val unit: String?,
    val weight: BigDecimal,
    val progressPercentage: Int,
    val orderIndex: Int?
)
```

**Você decide:**
- Estrutura exata dos DTOs
- Campos adicionais
- Validações (@NotNull, @NotBlank, etc.)
- Mappers (Entity → DTO)

---

## ⚠️ RESTRIÇÕES

### **O que NÃO PODE ser alterado:**

1. ❌ **NÃO alterar ChatGPTService base**
2. ❌ **NÃO alterar entities da Sprint 1** (Stage, Action, KeyResult, etc.)
3. ❌ **NÃO alterar repositories da Sprint 1**
4. ❌ **NÃO alterar migrations V025-V030**
5. ❌ **NÃO alterar Objective entity** (apenas usar)
6. ❌ **NÃO alterar endpoints existentes** de ObjectiveController
7. ❌ **NÃO criar novos endpoints** além do especificado

### **O que DEVE ser preservado:**

1. ✅ **Padrão de Clean Architecture** (Controller → UseCase → Service → Repository)
2. ✅ **Padrão de segurança** (X-Internal-Service-Key)
3. ✅ **Padrão de DTOs** (seguir ObjectiveResponse, etc.)
4. ✅ **Padrão de Exception Handling** (GlobalExceptionHandler)
5. ✅ **Padrão de integração IA** (ChatGPTService)

---

## 📚 DOCUMENTAÇÃO DE REFERÊNCIA

### **Arquivos para Estudar (OBRIGATÓRIO):**

1. **Integração IA (CRÍTICO):**
   - `src/main/kotlin/com/moversemais/objective/service/AssessmentDiagnosisService.kt`
   - Como chamar ChatGPT, montar prompt, parsear JSON
   - `src/main/kotlin/com/moversemais/objective/ai/service/KeyResultsService.kt`
   - Exemplo de prompt estruturado
   - `src/main/kotlin/com/moversemais/objective/ai/service/ChatGPTService.kt`
   - Serviço base de comunicação

2. **Entities da Sprint 1 (você criou!):**
   - `Stage.kt`, `Action.kt`, `KeyResult.kt`, `ActionKrLink.kt`
   - Relacionamentos, métodos auxiliares

3. **Repositories da Sprint 1 (você criou!):**
   - `StageRepository.kt`, `ActionRepository.kt`, etc.
   - Queries disponíveis

4. **PRD 001:**
   - `../moversemais-arquitetura/PRD/prd_001_goal_chunking.md`
   - Entender lógica de negócio

5. **Padrões do Projeto:**
   - `ObjectiveController.kt` - Padrão de endpoints
   - `CreateObjectiveUseCase.kt` - Padrão de UseCases
   - DTOs existentes (ObjectiveResponse, etc.)

---

## 🔧 WORKFLOW

### **1. ESTUDAR (OBRIGATÓRIO - 1 hora)**

```bash
cd moversemais-objective

# Estudar integração IA (CRÍTICO)
cat src/main/kotlin/com/moversemais/objective/service/AssessmentDiagnosisService.kt
cat src/main/kotlin/com/moversemais/objective/ai/service/KeyResultsService.kt
cat src/main/kotlin/com/moversemais/objective/ai/service/ChatGPTService.kt

# Estudar entities da Sprint 1 (você criou!)
cat src/main/kotlin/com/moversemais/objective/entity/Stage.kt
cat src/main/kotlin/com/moversemais/objective/entity/Action.kt
cat src/main/kotlin/com/moversemais/objective/entity/KeyResult.kt

# Estudar repositories da Sprint 1 (você criou!)
cat src/main/kotlin/com/moversemais/objective/repository/StageRepository.kt

# Ler PRD 001
cat ../moversemais-arquitetura/PRD/prd_001_goal_chunking.md

# Ler AGENTS.md
cat AGENTS.md
```

**Perguntas para Responder Antes de Implementar:**
- Como AssessmentDiagnosisService chama ChatGPT?
- Como parsear JSON response?
- Como salvar entities relacionadas (cascade)?
- Criar UseCase ou direto no Service?
- Estrutura de DTOs?

### **2. CRIAR BRANCH**

```bash
git checkout -b feature/goals-chunking-backend
```

### **3. IMPLEMENTAR (VOCÊ DECIDE COMO)**

**Você é o especialista em Spring Boot + Kotlin + IA.**

**Ordem sugerida (você pode mudar):**
1. GoalChunkingService (lógica principal)
2. DTOs (request, response)
3. UseCase (se seguir Clean Architecture)
4. Controller/Endpoint
5. Exception handling
6. Testes

**Decisões técnicas que você toma:**
- Estrutura de classes e métodos
- Prompt exato para ChatGPT
- Lógica de parsing JSON
- Validações e tratamento de erros
- Logs e auditoria
- Nomes de variáveis, métodos, classes

**Mas DEVE seguir:**
- ✅ Padrão de Clean Architecture
- ✅ Padrão de segurança (X-Internal-Service-Key)
- ✅ Padrão de DTOs
- ✅ Reutilizar ChatGPTService

### **4. TESTAR**

**Testes Obrigatórios:**

```bash
# 1. Rodar aplicação
./gradlew bootRun

# 2. Testar endpoint via curl/Postman
curl -X POST http://localhost:8080/api/v1/objectives/{id}/generate-plan \
  -H "X-User-Id: {userId}" \
  -H "X-Internal-Service-Key: {key}" \
  -H "Content-Type: application/json" \
  -d '{"regenerate": false}'

# Esperado: Response 200 com plano gerado

# 3. Verificar dados no banco
docker exec moversemais-postgres psql -U developer -d moversemais_objective

SELECT * FROM stages WHERE objective_id = '{id}';
SELECT * FROM actions WHERE stage_id IN (SELECT id FROM stages WHERE objective_id = '{id}');
SELECT * FROM key_results WHERE objective_id = '{id}';
SELECT * FROM action_kr_links;

# 4. Testar validações
# - Objective não encontrado (404)
# - Objective de outro usuário (403)
# - Objective sem motive/context (400)
# - Plano já existe (400)
# - Regenerar plano (200)

# 5. Testar resposta da IA
# - JSON válido?
# - Estrutura correta?
# - Validações passam?
```

**Verificações:**
- [ ] Endpoint responde 200
- [ ] Plano gerado pela IA é válido
- [ ] Dados salvos no banco (stages, actions, key_results, action_kr_links)
- [ ] Validações funcionam (mínimo 1 etapa, 3 ações, 1 KR)
- [ ] Soma dos weights = 1.0 (ou próximo)
- [ ] linkedKRs corretos
- [ ] Erros tratados (404, 403, 400, 500)
- [ ] Build compilado
- [ ] Aplicação iniciou

### **5. DOCUMENTAR DECISÕES**

Ao final do card, documente:
- Prompt usado para ChatGPT
- Estrutura de classes criadas
- Lógica de parsing JSON
- Validações implementadas
- Testes realizados
- Dificuldades encontradas
- Decisões técnicas tomadas

### **6. COMMIT E PUSH**

```bash
git add .
git commit -m "feat(objective-service): implementa Goal Chunking com IA

- GoalChunkingService (IA gera plano estruturado)
- Endpoint POST /objectives/{id}/generate-plan
- DTOs (GoalPlanRequest, GoalPlanResponse)
- UseCase (se aplicável)
- Integração com ChatGPT
- Validações (mínimo 1 etapa, 3 ações, 1 KR)
- Salva no banco (stages, actions, key_results, action_kr_links)
- Tratamento de erros
- Testes realizados
- Ref: Card 035"

git push origin feature/goals-chunking-backend
```

### **7. MOVER PARA VALIDAÇÃO**

```bash
mv Development/TODO/035__Backend-Goal-Chunking-Service.md \
   Development/VALIDATING/035__Backend-Goal-Chunking-Service.md
```

---

## ✅ CRITÉRIOS DE VALIDAÇÃO

### **Funcionalidades:**
- [ ] GoalChunkingService criado
- [ ] Endpoint POST /objectives/{id}/generate-plan funciona
- [ ] Plano gerado pela IA é válido (JSON estruturado)
- [ ] Dados salvos no banco (stages, actions, key_results, action_kr_links)
- [ ] Validações funcionam (mínimo 1 etapa, 3 ações, 1 KR)
- [ ] Soma dos weights = 1.0 (ou próximo: 0.99-1.01)
- [ ] linkedKRs corretos (vinculações action ↔ KR)
- [ ] Erros tratados (404, 403, 400, 500)
- [ ] Regenerate funciona (deleta plano antigo)

### **Padrão:**
- [ ] Seguiu Clean Architecture
- [ ] Seguiu padrão de segurança (X-Internal-Service-Key)
- [ ] Seguiu padrão de DTOs
- [ ] Reutilizou ChatGPTService
- [ ] Código limpo e documentado
- [ ] Build compilado
- [ ] Aplicação iniciou

### **Qualidade:**
- [ ] Prompt estruturado e claro
- [ ] Parsing JSON robusto
- [ ] Validações completas
- [ ] Tratamento de erros adequado
- [ ] Logs úteis
- [ ] Testes realizados

---

## 🚨 TROUBLESHOOTING

### **Problema: ChatGPT não retorna JSON válido**
**Solução:**
- Melhorar prompt (ser mais específico)
- Adicionar "RESPONDA APENAS COM JSON VÁLIDO"
- Tratar parsing errors (try/catch)

### **Problema: Soma dos weights != 1.0**
**Solução:**
- Aceitar range 0.99-1.01
- Ou normalizar weights automaticamente
- Ou pedir IA ajustar

### **Problema: linkedKRs inválidos**
**Solução:**
- Validar índices antes de salvar
- Índices devem estar entre 0 e keyResults.size-1

### **Problema: Cascade delete não funciona**
**Solução:**
- Verificar relacionamentos JPA nas entities
- Verificar cascade type (ALL, REMOVE)

---

## 🎯 EXPECTATIVAS

### **Você é o Especialista em Backend + IA**

**Osvaldo, você completou Sprint 1 com Score 10/5!** 🏆

**Você criou:**
- 5 entities JPA
- 5 repositories
- 2 migrations
- 1.177 linhas de código
- 30+ métodos auxiliares

**Agora, você vai integrar IA para Goal Chunking!**

**Você tem acesso a:**
- AssessmentDiagnosisService (como chamar ChatGPT)
- Entities da Sprint 1 (você criou!)
- Repositories da Sprint 1 (você criou!)

**Eu confio que você:**
- Sabe chamar ChatGPT e parsear JSON
- Sabe salvar entities relacionadas
- Sabe criar endpoints robustos
- Sabe tratar erros adequadamente

**Eu defini O QUE precisa ser feito. Você decide COMO fazer.**

**Este é um card desafiador, mas você já provou que consegue!** 💪

---

## 📊 OUTPUT ESPERADO

Ao finalizar, documente aqui:

### **Decisões Técnicas Tomadas:**
(Você preenche)

### **Prompt Usado para ChatGPT:**
(Cole o prompt completo)

### **Estrutura de Classes Criadas:**
(Liste classes, métodos principais)

### **Lógica de Parsing JSON:**
(Descreva como parseou)

### **Testes Realizados:**
(Liste cenários testados)

### **Dificuldades Encontradas:**
(Se houver)

### **Melhorias Implementadas:**
(Além do requisitado)

---

**Data de Criação:** 08/11/2025  
**Criado por:** Arquiobaldo (Arquiteto MoverseMais)  
**Contexto:** Módulo Goals - Sprint 2 (Goal Chunking) - Backend  
**Dependência:** Card 034 ✅ (DONE - Sprint 1 Completa)  
**Próximo:** Card 036 (BFF - Gabriela)  
**Versão:** 1.0

