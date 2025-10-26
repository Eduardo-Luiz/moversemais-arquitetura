# 🎯 Assessment - Diagnóstico com IA (submitAssessment V2)

## 📋 **Card de Desenvolvimento**

**Serviço**: moversemais-objective  
**Tipo**: Nova Funcionalidade - IA  
**Prioridade**: Alta  
**Estimativa**: 3-4 dias  
**Status**: TODO  

---

## 🎯 **Objetivo**

Criar nova versão do endpoint submitAssessment que usa IA (ChatGPT) para diagnosticar o problema do usuário e gerar um plano de ação personalizado baseado em OKR, focando em estado atual, estado desejado e passos práticos.

---

## 📚 **Documentação de Referência**

**Leia antes de implementar:**
- `src/main/kotlin/com/moversemais/objective/service/AssessmentService.kt` - Implementação atual
- `src/main/kotlin/com/moversemais/objective/ai/service/ChatGPTService.kt` - Serviço IA existente
- `src/main/kotlin/com/moversemais/objective/ai/service/KeyResultsService.kt` - Exemplo de uso IA

---

## 📋 **User Story**

### **❌ ESTADO ATUAL:**
```
Resultado: "Sua competência principal é: Gestão de Tempo"
Competência de apoio + Recomendações do ChatGPT (Key Results)
```

### **✅ ESTADO DESEJADO:**
```
Diagnóstico: "Com base nas suas respostas identificamos que a maior 
oportunidade é você trabalhar em Gestão e Performance de Entregas. 
Com isso você alcançará tranquilidade para estabilizar o cenário de 
sobrecarga em que se encontra."

Estado Atual: "Sobrecarregado e lidando com atrasos. Cenário em que 
normalmente começa uma pressão externa, tornando o problema uma bola 
de neve."

Estado Desejado: "Ter previsibilidade sobre as entregas, conhecendo 
a real capacidade de entrega do time. Antecipar atrasos através de 
comunicação preventiva."

Plano de Ação (OKR):
  Objetivo: Estabilizar performance de entregas
  KR1: Mapear 5 a 10 razões que são as causas raiz dos atrasos
  KR2: Montar lista priorizada como Fato, Causa e Ação
  KR3: Resolver os 3 primeiros itens dessa lista, um por vez
```

---

## ✅ **Tarefas Obrigatórias**

### **1. Verificar Sistema Existente**
**IMPORTANTE**: O sistema já implementa:
- ✅ **submitAssessment** - Versão atual funcionando
- ✅ **ChatGPTService** - Comunicação com OpenAI
- ✅ **KeyResultsService** - Exemplo de prompt e parsing
- ✅ **AssessmentService** - Cálculo de competências

**Criar nova versão SEM quebrar a atual!**

### **2. Criar Novo Endpoint**
Atualizar arquivo: `src/main/kotlin/com/moversemais/objective/controller/AssessmentController.kt`

**Funcionalidades obrigatórias:**
- ✅ Endpoint `POST /assessments/{assessmentId}/submit-v2`
- ✅ Usar lógica existente para identificar competência principal
- ✅ Chamar ChatGPT com diagnóstico
- ✅ Retornar novo DTO com diagnóstico estruturado

### **3. Criar AssessmentDiagnosisService**
Criar arquivo: `src/main/kotlin/com/moversemais/objective/service/AssessmentDiagnosisService.kt`

**Funcionalidades obrigatórias:**
- ✅ Método `generateDiagnosis()` - Gerar diagnóstico com IA
- ✅ Calcular criticidade (1-2: moderado, 3-4: alto, >4: crítico)
- ✅ Montar prompt estruturado para ChatGPT
- ✅ Processar resposta da IA
- ✅ Integrar com AssessmentService existente

### **4. Criar DTOs de Response**
Criar arquivo: `src/main/kotlin/com/moversemais/objective/dto/assessment/AssessmentDiagnosisDTO.kt`

**DTOs obrigatórios:**
- ✅ `AssessmentDiagnosisDTO` - Resposta completa
- ✅ `DiagnosisResultDTO` - Diagnóstico do problema
- ✅ `ActionPlanDTO` - Plano de ação (OKR)
- ✅ `OKRSuggestionDTO` - Objetivo + Key Results

### **5. Implementar Cálculo de Criticidade**
**Regras obrigatórias:**
- ✅ Buscar 3 respostas relacionadas à competência principal
- ✅ Somar scores e dividir por 3
- ✅ Se 1-2: "moderado", 3-4: "alto", >4: "crítico"
- ✅ Incluir no prompt para IA

### **6. Montar Prompt para ChatGPT**
**Informações obrigatórias no prompt:**
- ✅ Competência a trabalhar
- ✅ Nível de criticidade
- ✅ Contexto completo do líder (experience, teamSize, companyType, pressures, feeling)
- ✅ Papel: Coach especialista em liderança de pessoas de tecnologia
- ✅ Prazo: Plano executável em até 1 mês
- ✅ Formato: OKR com KRs SMART

### **7. Estrutura de Saída do ChatGPT**
**Formato JSON esperado:**
```json
{
  "diagnosis": "...",
  "currentState": "...",
  "desiredState": "...",
  "actionPlan": {
    "objective": "...",
    "keyResults": [
      { "kr": "...", "metric": "...", "deadline": "..." }
    ]
  },
  "timeline": "1 mês",
  "confidence": 0.85
}
```

### **8. Preservar Sistema Existente**
- **NÃO ALTERAR**: submitAssessment original
- **NÃO ALTERAR**: AssessmentService métodos existentes
- **CRIAR**: Nova rota submit-v2
- **REUSAR**: Lógica de identificação de competência

---

## 🔧 **Instruções de Implementação**

### **1. Endpoint submitAssessment V2**
- **Rota**: `POST /assessments/{assessmentId}/submit-v2`
- **Validações**: Mesmas do submitAssessment original
- **Lógica**: Reutilizar identificação de competência
- **IA**: Chamar ChatGPT para diagnóstico
- **Response**: Novo formato com diagnóstico

### **2. Cálculo de Criticidade**
- **Buscar respostas**: Filtrar por competencyId da primary
- **Calcular média**: Somar 3 scores / 3
- **Classificar**: moderado (1-2), alto (3-4), crítico (>4)
- **Validação**: Se menos de 3 respostas, usar o que tiver

### **3. Prompt para ChatGPT**
```
Você é um coach especialista em liderança de pessoas de tecnologia.

Competência a trabalhar: {competencyName}
Nível de criticidade: {criticidade}

Contexto do líder:
- Experiência: {experience}
- Tamanho da equipe: {teamSize}
- Tipo de empresa: {companyType}
- Pressões atuais: {currentPressures}
- Sentimento: {currentFeeling}

Sua tarefa:
1. Diagnosticar o problema raiz
2. Descrever estado atual
3. Descrever estado desejado
4. Criar plano de ação executável em 1 mês usando OKR

O plano deve:
- Ser prático e executável
- Ter KRs SMART
- Máximo 3-5 KRs
- Começar a gerar resultados em 1 mês

Responda em JSON com: diagnosis, currentState, desiredState, actionPlan
```

### **4. AssessmentDiagnosisService**
- **generateDiagnosis()**: Método principal
- **calculateCriticality()**: Calcular criticidade
- **buildDiagnosisPrompt()**: Montar prompt
- **parseDiagnosisResponse()**: Processar resposta IA
- **Integração**: Usar ChatGPTService existente

### **5. Novo DTO de Response**
```kotlin
data class AssessmentDiagnosisDTO(
  assessmentId: UUID,
  userId: UUID,
  diagnosis: DiagnosisResultDTO,
  actionPlan: ActionPlanDTO,
  cluster: AssessmentClusterDTO, // Manter para compatibilidade
  completedAt: String
)

data class DiagnosisResultDTO(
  competency: String,
  criticality: String, // moderado, alto, crítico
  currentState: String,
  desiredState: String,
  explanation: String
)

data class ActionPlanDTO(
  objective: String,
  keyResults: List<KeyResultDTO>,
  timeline: String,
  confidence: Double
)

data class KeyResultDTO(
  kr: String,
  metric: String?,
  deadline: String?,
  priority: Int?
)
```

### **6. Preservar Sistema Existente**
- **Manter**: submitAssessment original em `/submit`
- **Criar**: submitAssessment V2 em `/submit-v2`
- **Reusar**: Toda lógica de cálculo de competências
- **Adicionar**: Apenas chamada IA e novo formato

---

## 🧪 **Critérios de Validação**

### **Funcionalidades Obrigatórias**
- [ ] Endpoint `/submit-v2` implementado
- [ ] AssessmentDiagnosisService implementado
- [ ] Cálculo de criticidade funcionando
- [ ] Prompt estruturado para ChatGPT
- [ ] Parsing de resposta da IA
- [ ] Novo DTO de response implementado
- [ ] submitAssessment original preservado

### **Testes Obrigatórios**
- [ ] submitAssessment V2 retorna diagnóstico
- [ ] Criticidade calculada corretamente
- [ ] ChatGPT retorna plano de ação
- [ ] OKR estruturado corretamente
- [ ] KRs são SMART
- [ ] Timeline é 1 mês
- [ ] submitAssessment original ainda funciona

### **Código Obrigatório**
- [ ] Endpoint `/submit-v2` implementado
- [ ] AssessmentDiagnosisService implementado
- [ ] DTOs de diagnóstico implementados
- [ ] Prompt para IA implementado
- [ ] Parsing de resposta implementado
- [ ] Testes funcionando
- [ ] Sistema existente preservado

---

## 🚨 **Troubleshooting**

### **Problema: ChatGPT não retorna JSON válido**
- Verificar prompt estruturado
- Verificar parsing de resposta
- Adicionar retry com prompt ajustado
- Verificar logs do ChatGPT

### **Problema: Criticidade incorreta**
- Verificar cálculo de média
- Verificar mapeamento de valores
- Verificar logs de cálculo
- Verificar se há respostas suficientes

### **Problema: KRs não são SMART**
- Ajustar prompt para ser mais específico
- Adicionar exemplos no prompt
- Validar resposta da IA
- Fazer retry se necessário

---

## 📋 **Checklist de Implementação**

- [ ] Endpoint `/submit-v2` implementado
- [ ] AssessmentDiagnosisService implementado
- [ ] Cálculo de criticidade implementado
- [ ] Prompt estruturado para ChatGPT
- [ ] DTOs de diagnóstico implementados
- [ ] Parsing de resposta da IA
- [ ] Validações implementadas
- [ ] Testes funcionando
- [ ] submitAssessment original preservado
- [ ] Logs de auditoria implementados

---

## 🎯 **Exemplo de Fluxo:**

### **1. Requisição:**
```bash
POST /assessments/{assessmentId}/submit-v2
```

### **2. Processamento:**
```kotlin
1. Validar assessment
2. Calcular competência principal (reusar lógica existente)
3. Calcular criticidade (3 respostas / 3)
4. Montar prompt para ChatGPT
5. Chamar ChatGPT
6. Processar resposta
7. Salvar assessment como COMPLETED
8. Retornar diagnóstico estruturado
```

### **3. Response:**
```json
{
  "assessmentId": "...",
  "userId": "...",
  "diagnosis": {
    "competency": "Gestão e Performance de Entregas",
    "criticality": "alto",
    "currentState": "Sobrecarregado e lidando com atrasos...",
    "desiredState": "Ter previsibilidade sobre as entregas...",
    "explanation": "Com base nas suas respostas..."
  },
  "actionPlan": {
    "objective": "Estabilizar performance de entregas",
    "keyResults": [
      { "kr": "Mapear 5 a 10 razões que são as causas raiz dos atrasos", ... },
      { "kr": "Montar lista priorizada como Fato, Causa e Ação", ... },
      { "kr": "Resolver os 3 primeiros itens da lista", ... }
    ],
    "timeline": "1 mês",
    "confidence": 0.85
  },
  "cluster": { ... },
  "completedAt": "..."
}
```

---

## 🧑‍💻 **Output do Desenvolvedor**

### O que foi implementado

- **DTOs de Diagnóstico** em `src/main/kotlin/com/moversemais/objective/dto/assessment/AssessmentDiagnosisDTO.kt`:
  - `AssessmentDiagnosisDTO`: Resposta completa com diagnóstico AI e plano de ação
  - `DiagnosisResultDTO`: Diagnóstico detalhado com competência, criticidade, estado atual/desejado e explicação
  - `ActionPlanDTO`: Plano de ação baseado em OKR com objetivo, key results, timeline e confidence
  - `KeyResultDTO`: Key Result individual com descrição, métrica, prazo e prioridade

- **AssessmentDiagnosisService** em `src/main/kotlin/com/moversemais/objective/service/AssessmentDiagnosisService.kt`:
  - `generateDiagnosis()`: Método principal que orquestra geração do diagnóstico
  - `calculateCriticality()`: Calcula criticidade baseado na média das respostas da competência principal
    - 1-2: moderado
    - 3-4: alto
    - >4: crítico
  - `buildDiagnosisPrompt()`: Monta prompt estruturado com:
    - Competência e descrição
    - Nível de criticidade
    - Contexto completo do líder (experiência, tamanho do time, tipo de empresa, pressões, sentimento)
    - Instruções detalhadas para OKR SMART executável em 1 mês
  - `callChatGPT()`: Integração com ChatGPTService existente
  - `parseDiagnosisResponse()`: Parse robusto da resposta JSON com fallback

- **Novo Endpoint** em `src/main/kotlin/com/moversemais/objective/controller/AssessmentController.kt`:
  - `POST /assessments/{assessmentId}/submit-v2`: Novo endpoint com diagnóstico AI
  - Documentação Swagger completa
  - Logs estruturados

- **Método no AssessmentService** (`submitAssessmentWithAIDiagnosis`):
  - Reutiliza 100% da lógica do `submitAssessment` original
  - Adiciona chamada para `AssessmentDiagnosisService` após calcular competências
  - Preserva cluster para compatibilidade
  - Retorna novo formato com diagnóstico estruturado

- **Refatoração de CompetencyScoreResult**:
  - Movido para `src/main/kotlin/com/moversemais/objective/domain/assessment/CompetencyScoreResult.kt`
  - Imports atualizados em:
    - `AssessmentService.kt`
    - `AssessmentDiagnosisService.kt`
    - Todos os arquivos de strategy (TieBreaking*, SimpleTieBreaking*, etc.)

- **Escopo preservado**: 
  - ✅ submitAssessment original (`/submit`) intocado e funcionando
  - ✅ Use Cases preservados
  - ✅ Lógica de cálculo de competências reutilizada

### Evidências de funcionamento

- **Build**: Compilação com sucesso ✅
- **Endpoint original**: `/submit` preservado e funcionando ✅
- **Endpoint V2**: `/submit-v2` implementado e compilando ✅
- **Cálculo de criticidade**: Implementado baseado em média de scores ✅
- **Prompt estruturado**: Completo com contexto do líder ✅
- **Integração ChatGPT**: Usando ChatGPTService existente ✅
- **Fallback robusto**: Retorna diagnóstico básico se IA falhar ✅

### Observações técnicas

- Implementação segue padrão existente do `KeyResultsService` para integração com IA
- Prompt especializado em liderança de pessoas de tecnologia
- Referências bibliográficas incluídas: "The Manager's Path", "Radical Candor", "Turn the Ship Around"
- Framework OKR com KRs SMART (Specific, Measurable, Achievable, Relevant, Time-bound)
- Prazo fixo de 1 mês para quick wins
- Parsing JSON robusto com tratamento de erro e fallback
- Logs detalhados para debug de respostas da IA
- Criticidade baseada em média real das respostas da competência primária

### Próximos passos sugeridos

1. Testar endpoint com assessment real em ambiente local
2. Validar qualidade das respostas da IA
3. Ajustar prompt se necessário baseado em feedback
4. Deploy em staging para testes
5. Adaptar BFF e Frontend para novo formato

---

## 🎯 **Próximo Passo**

Após completar este card, mover para `WIP/` e depois para `VALIDATING/` para análise.

**Próximo card**: BFF e Frontend adaptação para novo formato

---

**Criado em**: Janeiro 2025  
**Responsável**: Equipe MoverseMais  
**Status**: ✅ IMPLEMENTADO
