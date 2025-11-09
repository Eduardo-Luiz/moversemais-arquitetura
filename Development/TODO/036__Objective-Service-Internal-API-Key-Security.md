# 🔐 Card 036 - Objective Service: Proteger com Internal API Key

**Agente Responsável:** Osvaldo  
**Microserviço:** moversemais-objective  
**Prioridade:** 🔴 CRÍTICA (Segurança)  
**Status:** TODO  
**Estimativa:** 3-4 horas

---

## 🔍 **ESTUDO PRÉVIO REALIZADO**

**Arquiobaldo estudou antes de criar este card:**
- ✅ Leu Card 019 (User Service - InternalApiKeyFilter implementado)
- ✅ Verificou SecurityFilter atual do moversemais-objective
- ✅ Identificou que NÃO valida X-Internal-Service-Key
- ✅ Verificou que moversemais-user e moversemais-auth JÁ usam esse padrão

---

## 📋 CONTEXTO

### **Situação Atual**

**moversemais-objective:**
- ✅ Tem `SecurityFilter` (valida X-User-Id, X-Provider)
- ❌ **NÃO valida X-Internal-Service-Key**
- ❌ **Frontend pode chamar diretamente** os endpoints

**Outros serviços (padrão estabelecido):**
- ✅ **moversemais-user:** Tem `InternalApiKeyFilter` (Card 019)
- ✅ **moversemais-auth:** Tem proteção similar (Card 013)
- ✅ **Apenas BFF** pode chamar endpoints

### **Problema Identificado**

**Inconsistência de Segurança:**
- moversemais-user e moversemais-auth: **Protegidos** (apenas BFF acessa)
- moversemais-objective: **Desprotegido** (Frontend pode acessar diretamente)

**Risco:**
- Frontend bypassa BFF
- Arquitetura quebrada (Frontend → Backend direto)
- Inconsistência de padrão de segurança
- Dificulta auditoria e monitoramento

**Endpoints Vulneráveis:**
- `POST /api/v1/objectives` (criar meta)
- `POST /api/v1/objectives/{id}/generate-plan` (Goal Chunking - NOVO!)
- `PUT /api/v1/objectives/{id}` (atualizar meta)
- `DELETE /api/v1/objectives/{id}` (deletar meta)
- `POST /api/v1/assessments` (criar assessment)
- Todos os endpoints de negócio

### **Solução Proposta**

Implementar `InternalApiKeyFilter` no moversemais-objective seguindo **EXATAMENTE** o padrão do moversemais-user (Card 019).

### **Onde se Encaixa na Arquitetura**

```
Arquitetura Correta:
Frontend → BFF → Backend (com X-Internal-Service-Key)

Antes (ERRADO):
Frontend ──┬──→ BFF → Backend
           └────────→ Backend (bypass!)

Depois (CORRETO):
Frontend ────→ BFF → Backend (X-Internal-Service-Key validado)
                      ↑
                      └─ Apenas BFF tem a chave
```

### **Impacto se Não For Feito**

- Inconsistência de segurança entre serviços
- Frontend pode bypassar BFF
- Dificulta auditoria centralizada
- Quebra arquitetura de microserviços

---

## 🎯 REQUISITOS OBRIGATÓRIOS

### **1. Implementar InternalApiKeyFilter**

**Função de Negócio:**
Garantir que **apenas o BFF** possa chamar endpoints de negócio do moversemais-objective, seguindo o mesmo padrão de moversemais-user e moversemais-auth.

**Requisitos Funcionais:**
- Criar filtro que valida header `X-Internal-Service-Key`
- Proteger TODOS os endpoints de negócio (objectives, assessments, AI, goal chunking)
- Manter endpoints públicos (health, swagger)
- Usar **constant-time comparison** (prevenir timing attacks)
- Headers **case-insensitive** (X-Internal-Service-Key ou x-internal-service-key)
- Logs de auditoria (INFO para válido, WARN para inválido)
- Mensagens de erro genéricas (não revelar detalhes)

**Paths Protegidos (requerem X-Internal-Service-Key):**
- `/api/v1/objectives/**` (TODOS os endpoints de objectives)
- `/api/v1/assessments/**` (TODOS os endpoints de assessments)
- `/api/v1/ai/**` (TODOS os endpoints de IA)

**Paths Públicos (NÃO requerem X-Internal-Service-Key):**
- `/health`
- `/api/v1/health`
- `/actuator/**`
- `/swagger-ui/**`
- `/api-docs/**`

**Referência (ESTUDAR):**
- Card 019: `moversemais-user/filter/InternalApiKeyFilter.kt` (128 linhas)
- Seguir **EXATAMENTE** o mesmo padrão

**Você decide:**
- Nome da classe (InternalApiKeyFilter recomendado)
- Estrutura interna
- Ordem de execução dos filtros
- Mensagens de log

**Restrições:**
- NÃO alterar SecurityFilter existente (pode coexistir)
- NÃO quebrar endpoints existentes
- NÃO alterar SecurityConfig drasticamente (apenas adicionar filtro)

---

### **2. Configuração de API Key**

**Função de Negócio:**
Configurar API Key por ambiente (desenvolvimento vs produção).

**Arquivos para Atualizar:**

**application.yml:**
```yaml
security:
  internal-service-key: ${INTERNAL_SERVICE_KEY:dev-internal-key-change-in-production}
```

**application-render.yml:**
```yaml
security:
  internal-service-key: ${INTERNAL_SERVICE_KEY}
```

**Você decide:**
- Estrutura exata da configuração
- Nome da propriedade (recomendado: security.internal-service-key)
- Valor default para desenvolvimento

---

### **3. Integração com SecurityConfig**

**Função de Negócio:**
Integrar InternalApiKeyFilter na cadeia de filtros do Spring Security.

**Requisitos Funcionais:**
- Adicionar filtro ANTES de outros filtros
- Garantir ordem de execução correta
- Não quebrar configuração existente

**Exemplo (você decide estrutura exata):**
```kotlin
@Configuration
@EnableWebSecurity
class SecurityConfig(
    private val internalApiKeyFilter: InternalApiKeyFilter  // Injetar
) {
    @Bean
    fun securityFilterChain(http: HttpSecurity): SecurityFilterChain {
        http
            .addFilterBefore(internalApiKeyFilter, UsernamePasswordAuthenticationFilter::class.java)
            // ... resto da configuração
    }
}
```

---

### **4. Logs de Auditoria**

**Requisitos Funcionais:**
- Log INFO: API Key válida (acesso autorizado)
- Log WARN: API Key inválida ou ausente
- Log DEBUG: Path público acessado
- **NÃO logar** o valor da API Key (segurança)

**Código de Erro:**
- `INTERNAL_API_001`: Invalid or missing internal API key

---

## ⚠️ RESTRIÇÕES

### **O que NÃO PODE ser alterado:**

1. ❌ **NÃO alterar SecurityFilter existente** (pode coexistir)
2. ❌ **NÃO alterar endpoints existentes** (controllers)
3. ❌ **NÃO alterar entities/repositories**
4. ❌ **NÃO alterar GoalChunkingService** (Card 035)
5. ❌ **NÃO quebrar health checks** (devem continuar públicos)
6. ❌ **NÃO quebrar swagger** (deve continuar público)

### **O que DEVE ser preservado:**

1. ✅ **Padrão do moversemais-user** (Card 019 - InternalApiKeyFilter)
2. ✅ **Constant-time comparison** (MessageDigest.isEqual)
3. ✅ **Case-insensitive headers**
4. ✅ **Logs de auditoria**
5. ✅ **Mensagens de erro genéricas**

---

## 📚 DOCUMENTAÇÃO DE REFERÊNCIA

### **Arquivos para Estudar (OBRIGATÓRIO):**

1. **Referência Principal (CRÍTICO):**
   - Card 019: `../moversemais-arquitetura/Development/DONE/019__User-Service-Security-Fix.md`
   - **Seguir EXATAMENTE o mesmo padrão!**
   - InternalApiKeyFilter: 128 linhas
   - Constant-time comparison
   - Case-insensitive headers

2. **Código Atual do Projeto:**
   - `src/main/kotlin/com/moversemais/objective/filter/SecurityFilter.kt` (atual)
   - `src/main/kotlin/com/moversemais/objective/config/SecurityConfig.kt` (atual)
   - Entender o que já existe

3. **Configurações:**
   - `src/main/resources/application.yml`
   - `src/main/resources/application-render.yml`

---

## 🔧 WORKFLOW

### **1. ESTUDAR (OBRIGATÓRIO - 30 minutos)**

```bash
cd moversemais-objective

# Estudar Card 019 (REFERÊNCIA PRINCIPAL)
cat ../moversemais-arquitetura/Development/DONE/019__User-Service-Security-Fix.md

# Estudar SecurityFilter atual
cat src/main/kotlin/com/moversemais/objective/filter/SecurityFilter.kt

# Estudar SecurityConfig atual
cat src/main/kotlin/com/moversemais/objective/config/SecurityConfig.kt

# Ler AGENTS.md
cat AGENTS.md
```

**Perguntas para Responder:**
- Como InternalApiKeyFilter funciona no moversemais-user?
- Como integrar com SecurityConfig?
- Quais paths proteger?
- Como coexistir com SecurityFilter atual?

### **2. CRIAR BRANCH**

```bash
git checkout -b feature/objective-service-internal-api-key
```

### **3. IMPLEMENTAR (VOCÊ DECIDE COMO)**

**Você é o especialista em Spring Security!**

**Ordem sugerida:**
1. Criar InternalApiKeyFilter (seguir padrão Card 019)
2. Atualizar application.yml (adicionar security.internal-service-key)
3. Atualizar SecurityConfig (adicionar filtro)
4. Testar

**Decisões técnicas que você toma:**
- Nome exato da classe
- Estrutura interna
- Ordem dos filtros
- Mensagens de log
- Paths protegidos vs públicos

**Mas DEVE seguir:**
- ✅ Padrão do Card 019 (InternalApiKeyFilter)
- ✅ Constant-time comparison
- ✅ Case-insensitive headers
- ✅ Logs de auditoria

### **4. TESTAR**

**Testes Obrigatórios:**

```bash
# 1. Rodar aplicação
./gradlew bootRun

# 2. Testar endpoint SEM API Key (deve bloquear)
curl -X POST http://localhost:8080/api/v1/objectives \
  -H "X-User-Id: 550e8400-e29b-41d4-a716-446655440000" \
  -H "Content-Type: application/json" \
  -d '{...}'

# Esperado: 401 Unauthorized

# 3. Testar endpoint COM API Key válida (deve permitir)
curl -X POST http://localhost:8080/api/v1/objectives \
  -H "X-User-Id: 550e8400-e29b-41d4-a716-446655440000" \
  -H "X-Internal-Service-Key: dev-internal-key-change-in-production" \
  -H "Content-Type: application/json" \
  -d '{...}'

# Esperado: 200/201 OK

# 4. Testar endpoint COM API Key inválida (deve bloquear)
curl -X POST http://localhost:8080/api/v1/objectives \
  -H "X-Internal-Service-Key: chave-errada" \
  -d '{...}'

# Esperado: 401 Unauthorized

# 5. Testar health check SEM API Key (deve permitir)
curl http://localhost:8080/api/v1/health

# Esperado: 200 OK

# 6. Testar swagger SEM API Key (deve permitir)
curl http://localhost:8080/swagger-ui/index.html

# Esperado: 200 OK

# 7. Testar Goal Chunking COM API Key
curl -X POST http://localhost:8080/api/v1/objectives/{id}/generate-plan \
  -H "X-User-Id: {userId}" \
  -H "X-Internal-Service-Key: dev-internal-key-change-in-production"

# Esperado: 200 OK (se objective em DRAFT)
```

**Verificações:**
- [ ] InternalApiKeyFilter criado
- [ ] API Key configurada (application.yml)
- [ ] Integrado com SecurityConfig
- [ ] Endpoints protegidos (objectives, assessments, AI)
- [ ] Health checks públicos
- [ ] Swagger público
- [ ] Constant-time comparison implementado
- [ ] Logs de auditoria funcionando
- [ ] Build compilado
- [ ] Aplicação iniciou

### **5. DOCUMENTAR DECISÕES**

Ao final do card, documente:
- Estrutura do InternalApiKeyFilter
- Paths protegidos vs públicos
- Integração com SecurityConfig
- Testes realizados
- Dificuldades encontradas

### **6. COMMIT E PUSH**

```bash
git add .
git commit -m "feat(objective-service): adiciona proteção com Internal API Key

- InternalApiKeyFilter implementado (padrão Card 019)
- Protege: /objectives/**, /assessments/**, /ai/**
- Público: /health, /actuator/**, /swagger-ui/**
- Constant-time comparison (previne timing attacks)
- Case-insensitive headers
- Logs de auditoria
- Configuração por ambiente
- Apenas BFF pode chamar endpoints
- Ref: Card 036"

git push origin feature/objective-service-internal-api-key
```

### **7. MOVER PARA VALIDAÇÃO**

```bash
mv Development/TODO/036__Objective-Service-Internal-API-Key-Security.md \
   Development/VALIDATING/036__Objective-Service-Internal-API-Key-Security.md
```

---

## ✅ CRITÉRIOS DE VALIDAÇÃO

### **Funcionalidades:**
- [ ] InternalApiKeyFilter criado
- [ ] API Key configurada (application.yml, application-render.yml)
- [ ] Integrado com SecurityConfig
- [ ] Paths protegidos: /objectives/**, /assessments/**, /ai/**
- [ ] Paths públicos: /health, /actuator/**, /swagger-ui/**
- [ ] Constant-time comparison implementado
- [ ] Case-insensitive headers
- [ ] Logs de auditoria funcionando
- [ ] Código de erro: INTERNAL_API_001

### **Testes:**
- [ ] Endpoint protegido SEM API Key → 401
- [ ] Endpoint protegido COM API Key válida → 200
- [ ] Endpoint protegido COM API Key inválida → 401
- [ ] Health check SEM API Key → 200 (público)
- [ ] Swagger SEM API Key → 200 (público)
- [ ] Goal Chunking COM API Key → 200
- [ ] Logs de auditoria corretos

### **Segurança:**
- [ ] API Key nunca aparece em logs
- [ ] Mensagens de erro genéricas
- [ ] Constant-time comparison (MessageDigest.isEqual)
- [ ] Headers case-insensitive

### **Padrão:**
- [ ] Seguiu Card 019 (moversemais-user)
- [ ] Código limpo e documentado
- [ ] Build compilado
- [ ] Aplicação iniciou

---

## 🚨 TROUBLESHOOTING

### **Problema: API Key sempre bloqueado**
**Solução:**
- Verificar valor da variável de ambiente
- Verificar nome do header (case-insensitive?)
- Verificar comparação de strings (constant-time?)
- Verificar logs

### **Problema: SecurityFilter atual conflita**
**Solução:**
- InternalApiKeyFilter executa ANTES
- SecurityFilter executa DEPOIS (se passar)
- Ordem: InternalApiKeyFilter → SecurityFilter
- Ambos podem coexistir

### **Problema: Health check bloqueado**
**Solução:**
- Path /actuator/health deve ser público
- InternalApiKeyFilter deve ignorar esse path

### **Problema: Swagger bloqueado**
**Solução:**
- Paths /swagger-ui/**, /api-docs/** devem ser públicos
- InternalApiKeyFilter deve ignorar esses paths

---

## 🎯 EXPECTATIVAS

### **Você é o Especialista em Spring Security**

**Osvaldo, você completou 6 cards com EXCELÊNCIA (Scores 5-10/5)!** 🏆

**Agora, você vai implementar segurança crítica!**

**Eu confio que você:**
- Sabe implementar filtros Spring Security
- Sabe usar constant-time comparison
- Sabe integrar com SecurityConfig
- Sabe testar segurança adequadamente

**Referência:**
- Card 019 (moversemais-user) - **Seguir EXATAMENTE o mesmo padrão!**

**Eu defini O QUE precisa ser feito. Você decide COMO fazer.**

**Este é um card de SEGURANÇA CRÍTICA!** 🔐

---

## 📊 OUTPUT ESPERADO

Ao finalizar, documente aqui:

### **Decisões Técnicas Tomadas:**
(Você preenche)

### **Estrutura do InternalApiKeyFilter:**
(Descreva implementação)

### **Paths Protegidos vs Públicos:**
(Liste paths)

### **Integração com SecurityConfig:**
(Descreva como integrou)

### **Testes Realizados:**
(Liste cenários testados)

### **Dificuldades Encontradas:**
(Se houver)

### **Melhorias Implementadas:**
(Além do requisitado)

---

**Data de Criação:** 08/11/2025  
**Criado por:** Arquiobaldo (Arquiteto MoverseMais)  
**Contexto:** Segurança - Padronizar proteção com X-Internal-Service-Key  
**Dependência:** Card 035 ✅ (DONE - Goal Chunking)  
**Referência:** Card 019 (moversemais-user - InternalApiKeyFilter)  
**Próximo:** Card 037 (BFF - Gabriela)  
**Versão:** 1.0

