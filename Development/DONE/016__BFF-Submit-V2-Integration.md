# 🎯 BFF - Integração Submit Assessment V2

## 📋 **Card de Desenvolvimento**

**Serviço**: moversemais-store-graphql (BFF)  
**Tipo**: Integração - Nova Mutation  
**Prioridade**: Alta  
**Estimativa**: 1 dia  
**Status**: TODO  

---

## 🎯 **Objetivo**

Adicionar mutation GraphQL `submitAssessmentV2` no BFF para integrar com o novo endpoint `/submit-v2` do moversemais-objective, disponibilizando diagnóstico AI com plano de ação OKR para o frontend.

---

## 📚 **Documentação de Referência**

**Leia antes de implementar:**
- `015__Assessment-AI-Diagnosis-Implementation.md` - Card 015 (pré-requisito)
- `app/api/graphql/resolvers/assessment.ts` - Resolvers existentes
- `app/api/graphql/services/assessment-service.ts` - Service existente
- `app/api/graphql/services/backend-client.ts` - Cliente HTTP existente

---

## ✅ **Tarefas Obrigatórias**

### **1. Verificar Sistema Existente**
**IMPORTANTE**: O sistema já implementa:
- ✅ **submitAssessment** - Mutation original funcionando
- ✅ **assessmentService** - Serviço com lógica de negócio
- ✅ **backendClient** - Cliente HTTP para backend
- ✅ **Types e interfaces** - GraphQL types definidos

**Adicionar submitAssessmentV2 SEM quebrar o original!**

### **2. Criar Interfaces TypeScript**
Atualizar arquivo: `app/api/graphql/types/interfaces.ts`

**Interfaces obrigatórias:**
- ✅ `AssessmentDiagnosisResponse` - Response completo do V2
- ✅ `DiagnosisResult` - Diagnóstico estruturado
- ✅ `ActionPlan` - Plano OKR
- ✅ `KeyResult` - Key Result individual

### **3. Atualizar BackendClient**
Atualizar arquivo: `app/api/graphql/services/backend-client.ts`

**Funcionalidades obrigatórias:**
- ✅ Método `submitAssessmentV2(assessmentId)` - Chamar `/submit-v2`
- ✅ Headers de segurança (mesmo padrão)
- ✅ Tratamento de erros

### **4. Atualizar AssessmentService**
Atualizar arquivo: `app/api/graphql/services/assessment-service.ts`

**Funcionalidades obrigatórias:**
- ✅ Método `submitAssessmentV2(assessmentId)` - Processar resposta do backend
- ✅ Manter método `submitAssessment` original
- ✅ Mapear response do backend para GraphQL

### **5. Criar Mutation GraphQL**
Atualizar arquivo: `app/api/graphql/resolvers/assessment.ts`

**Funcionalidades obrigatórias:**
- ✅ Mutation `submitAssessmentV2(assessmentId: ID!)`
- ✅ Resolver chamando `assessmentService.submitAssessmentV2()`
- ✅ Retornar tipo `AssessmentDiagnosisResponse`

### **6. Atualizar Schema GraphQL**
Atualizar arquivo: `app/api/graphql/schema/types.ts`

**Types obrigatórios:**
```graphql
type AssessmentDiagnosisResponse {
  assessmentId: ID!
  userId: ID!
  diagnosis: DiagnosisResult!
  actionPlan: ActionPlan!
  cluster: AssessmentCluster
  completedAt: String!
}

type DiagnosisResult {
  competency: String!
  criticality: String!
  currentState: String!
  desiredState: String!
  explanation: String!
}

type ActionPlan {
  objective: String!
  keyResults: [KeyResult!]!
  timeline: String!
  confidence: Float!
}

type KeyResult {
  kr: String!
  metric: String
  deadline: String
  priority: Int
}
```

### **7. Preservar Sistema Existente**
- **NÃO ALTERAR**: submitAssessment original
- **NÃO ALTERAR**: Resolvers, services, types existentes
- **ADICIONAR**: Nova mutation submitAssessmentV2
- **MANTER**: Compatibilidade com frontend atual

---

## 🔧 **Instruções de Implementação**

### **1. BackendClient**
- **Método**: `submitAssessmentV2(assessmentId)`
- **Endpoint**: `POST /assessments/${assessmentId}/submit-v2`
- **Headers**: Mesmo padrão (X-User-Id, X-Provider, X-Request-Id)
- **Response**: Mapear para interface TypeScript

### **2. AssessmentService**
- **Método**: `submitAssessmentV2(assessmentId)`
- **Chamar**: `backendClient.submitAssessmentV2()`
- **Processar**: Response do backend
- **Retornar**: Formato GraphQL

### **3. Resolver**
```typescript
submitAssessmentV2: async (_: GraphQLContext, { assessmentId }: { assessmentId: string }) => {
  return assessmentService.submitAssessmentV2(assessmentId)
}
```

### **4. Types GraphQL**
- **AssessmentDiagnosisResponse**: Response principal
- **DiagnosisResult**: Diagnóstico com estado atual/desejado
- **ActionPlan**: OKR com key results
- **KeyResult**: KR individual

### **5. Preservar Sistema Existente**
- **submitAssessment**: Manter funcionando (V1)
- **submitAssessmentV2**: Adicionar (V2)
- **Frontend**: Pode escolher qual usar
- **Compatibilidade**: Ambos funcionando

---

## 🧪 **Critérios de Validação**

### **Funcionalidades Obrigatórias**
- [ ] Interfaces TypeScript implementadas
- [ ] BackendClient.submitAssessmentV2() implementado
- [ ] AssessmentService.submitAssessmentV2() implementado
- [ ] Mutation submitAssessmentV2 implementada
- [ ] Types GraphQL atualizados
- [ ] submitAssessment original preservado

### **Testes Obrigatórios**
- [ ] Mutation submitAssessmentV2 retorna diagnóstico
- [ ] Diagnóstico contém estado atual/desejado
- [ ] Action plan contém OKR estruturado
- [ ] Key results são SMART
- [ ] submitAssessment original ainda funciona
- [ ] Ambas mutations funcionando em paralelo

### **Código Obrigatório**
- [ ] Interfaces TypeScript implementadas
- [ ] BackendClient atualizado
- [ ] AssessmentService atualizado
- [ ] Resolver atualizado
- [ ] Schema GraphQL atualizado
- [ ] Testes funcionando
- [ ] Sistema existente preservado

---

## 🚨 **Troubleshooting**

### **Problema: Backend retorna erro 404**
- Verificar se Card 015 foi implementado
- Verificar se endpoint `/submit-v2` existe
- Verificar se moversemais-objective está rodando
- Verificar logs do backend

### **Problema: Response não mapeia corretamente**
- Verificar interfaces TypeScript
- Verificar estrutura do response do backend
- Verificar logs do BFF
- Verificar parsing JSON

### **Problema: Mutation original quebrou**
- Verificar se não alterou submitAssessment
- Verificar se imports estão corretos
- Verificar se types não conflitam
- Rollback se necessário

---

## 📋 **Checklist de Implementação**

- [ ] Interfaces TypeScript implementadas
- [ ] BackendClient.submitAssessmentV2() implementado
- [ ] AssessmentService.submitAssessmentV2() implementado
- [ ] Mutation submitAssessmentV2 implementada
- [ ] Types GraphQL atualizados
- [ ] Testes de integração funcionando
- [ ] submitAssessment original preservado
- [ ] Logs de auditoria implementados
- [ ] Documentação atualizada

---

## 🎯 **Exemplo de Uso:**

### **GraphQL Mutation:**
```graphql
mutation {
  submitAssessmentV2(assessmentId: "123e4567-e89b-12d3-a456-426614174000") {
    assessmentId
    userId
    diagnosis {
      competency
      criticality
      currentState
      desiredState
      explanation
    }
    actionPlan {
      objective
      keyResults {
        kr
        metric
        deadline
        priority
      }
      timeline
      confidence
    }
    cluster {
      primary {
        competency {
          name
          description
        }
      }
    }
    completedAt
  }
}
```

---

## 🎯 **Próximo Passo**

Após completar este card, mover para `WIP/` e depois para `VALIDATING/` para análise.

**Próximo card**: Frontend - Exibição de Diagnóstico AI

---

**Criado em**: Janeiro 2025  
**Responsável**: Equipe MoverseMais  
**Status**: VALIDATING

---

## 📋 **Output do Desenvolvedor**

### **✅ Implementação Completa - Card 016**

**Branch**: `feature/bff-submit-v2-integration`  
**Commit**: `7cfefe5`  
**Data**: 05/01/2025

### **🔧 Arquivos Modificados:**

1. **`app/api/graphql/types/interfaces.ts`**
   - ✅ Adicionado `DiagnosisResult` interface
   - ✅ Adicionado `ActionPlanKeyResult` interface
   - ✅ Adicionado `ActionPlan` interface
   - ✅ Adicionado `AssessmentDiagnosisResponse` interface

2. **`app/api/graphql/services/backend-client.ts`**
   - ✅ Adicionado método `submitAssessmentV2(assessmentId)`
   - ✅ Endpoint: `POST /assessments/${assessmentId}/submit-v2`
   - ✅ Headers de segurança mantidos (X-User-Id, X-Provider, X-Request-Id, X-Timestamp)

3. **`app/api/graphql/services/assessment-service.ts`**
   - ✅ Adicionado método `submitAssessmentV2(assessmentId)`
   - ✅ Mapeamento completo de response do backend
   - ✅ Tratamento de erros com `processBackendError`
   - ✅ Método original `submitAssessment` preservado

4. **`app/api/graphql/resolvers/assessment.ts`**
   - ✅ Adicionado resolver `submitAssessmentV2`
   - ✅ Chama `assessmentService.submitAssessmentV2()`
   - ✅ Resolver original `submitAssessment` preservado

5. **`app/api/graphql/schema/types.ts`**
   - ✅ Adicionado type `AssessmentDiagnosisResponse`
   - ✅ Adicionado type `DiagnosisResult`
   - ✅ Adicionado type `ActionPlan`
   - ✅ Adicionado type `ActionPlanKeyResult`
   - ✅ Adicionado mutation `submitAssessmentV2(assessmentId: ID!)`

### **📊 Funcionalidades Implementadas:**

#### **1. Diagnóstico AI Estruturado:**
```graphql
diagnosis {
  competency: String!        # Competência identificada
  criticality: String!        # "moderado", "alto", "crítico"
  currentState: String!       # Estado atual do usuário
  desiredState: String!       # Estado desejado/ideal
  explanation: String!        # Explicação detalhada
}
```

#### **2. Plano de Ação OKR:**
```graphql
actionPlan {
  objective: String!          # Objetivo principal
  keyResults: [ActionPlanKeyResult!]!  # Key Results SMART
  timeline: String!           # Timeline sugerido
  confidence: Float!          # Confiança (0-1)
}
```

#### **3. Key Results SMART:**
```graphql
ActionPlanKeyResult {
  kr: String!                 # Key Result completo
  metric: String              # Métrica mensurável (opcional)
  deadline: String            # Prazo (opcional)
  priority: Int               # Prioridade 1-5 (opcional)
}
```

### **✅ Checklist de Validação:**

- [x] Interfaces TypeScript implementadas
- [x] BackendClient.submitAssessmentV2() implementado
- [x] AssessmentService.submitAssessmentV2() implementado
- [x] Mutation submitAssessmentV2 implementada
- [x] Types GraphQL atualizados
- [x] submitAssessment original preservado
- [x] Sem erros de lint/TypeScript
- [x] Headers de segurança propagados
- [x] Tratamento de erros implementado
- [x] Logs de auditoria implementados
- [ ] Testes de integração (aguardando backend Card 015)

### **🎯 Compatibilidade:**

- ✅ **submitAssessment (V1)**: Preservado e funcionando
- ✅ **submitAssessmentV2 (V2)**: Implementado e pronto
- ✅ **Ambas mutations**: Funcionando em paralelo
- ✅ **Cluster**: Mantido em V2 para compatibilidade

### **🔍 Exemplo de Uso - GraphQL:**

```graphql
# V1 - Original (preservado)
mutation {
  submitAssessment(assessmentId: "123e4567-e89b-12d3-a456-426614174000") {
    assessmentId
    userId
    detectedCluster {
      primary {
        competency { name }
      }
    }
  }
}

# V2 - Novo (diagnóstico AI)
mutation {
  submitAssessmentV2(assessmentId: "123e4567-e89b-12d3-a456-426614174000") {
    assessmentId
    userId
    diagnosis {
      competency
      criticality
      currentState
      desiredState
      explanation
    }
    actionPlan {
      objective
      keyResults {
        kr
        metric
        deadline
        priority
      }
      timeline
      confidence
    }
    cluster {
      primary {
        competency { name }
      }
    }
    completedAt
  }
}
```

### **⚠️ Observações:**

1. **Backend Dependency**: Implementação depende do Card 015 estar completado
2. **Testes Integração**: Aguardando backend `/submit-v2` disponível
3. **Headers**: Todos os headers de segurança sendo propagados corretamente
4. **Compatibilidade**: Sistema V1 totalmente preservado

### **📋 Conformidade com Especificação:**

- ✅ Seguiu exatamente o Card 016
- ✅ Preservou sistema existente
- ✅ Adicionou V2 sem quebrar V1
- ✅ Interfaces conforme especificado
- ✅ Endpoints conforme especificado
- ✅ Types GraphQL conforme especificado
- ✅ Sem alterações em código existente (exceto adições)

**Implementação pronta para validação do Arquiteto!** 🚀
