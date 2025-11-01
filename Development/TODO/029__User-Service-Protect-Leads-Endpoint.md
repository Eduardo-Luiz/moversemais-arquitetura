# 🔴 Card 029 - User Service: Proteger Endpoint de Leads

**Agente Responsável:** Alex  
**Microserviço:** moversemais-user  
**Prioridade:** 🔴 CRÍTICA  
**Status:** TODO  
**Estimativa:** 1 hora

---

## 📋 CONTEXTO

### **Situação Atual - VULNERABILIDADE CRÍTICA**

O endpoint `/api/v1/leads` foi implementado (Card 026) mas está **DESPROTEGIDO**. Qualquer pessoa na internet pode registrar leads falsos, poluir a base de dados e gerar spam.

### **Problema Identificado - FALHA DO ARQUITETO**

**Arquiobaldo (eu) FALHOU** ao criar Card 026:
- ❌ Não estudou código existente do User Service antes de criar o card
- ❌ Ignorou que Card 019 já implementou `InternalApiKeyFilter`
- ❌ Não incluiu requisito de segurança no Card 026
- ❌ Alex implementou sem proteção (porque card não exigiu)

**Resultado:** Endpoint vulnerável em produção! 🚨

### **Padrão de Segurança Existente**

O User Service **JÁ TEM** proteção implementada (Card 019):

**Arquivo:** `src/main/kotlin/com/moversemais/user/filter/InternalApiKeyFilter.kt`

**Como funciona:**
- Valida header `X-Internal-Service-Key` para endpoints internos
- Apenas BFF (ou outros serviços) com a chave podem chamar
- Constant-time comparison (previne timing attacks)
- Já protege `/api/v1/users/sync`

**Atualmente protegido:**
```kotlin
protectedPaths = ["/api/v1/users/sync"]
```

**FALTA ADICIONAR:**
```kotlin
protectedPaths = ["/api/v1/users/sync", "/api/v1/leads"]  // ← ADICIONAR
```

### **Solução Proposta**

Adicionar `/api/v1/leads` à lista de paths protegidos no `InternalApiKeyFilter` existente.

### **Onde se Encaixa**
```
Internet → User Service (público mas protegido)
    ↓
InternalApiKeyFilter ← VOCÊ (adicionar /leads aqui)
    ↓
    Se X-Internal-Service-Key válido → Permite
    Se inválido/ausente → 401 Unauthorized
```

### **Impacto se Não For Feito**
- 🔴 Spam de leads falsos
- 🔴 Poluição da base de dados
- 🔴 Impossível distinguir leads reais de fake
- 🔴 Campanhas futuras comprometidas
- 🔴 VULNERABILIDADE CRÍTICA EM PRODUÇÃO

---

## 🎯 REQUISITOS OBRIGATÓRIOS

### **1. Atualizar InternalApiKeyFilter**

**Arquivo:** `src/main/kotlin/com/moversemais/user/filter/InternalApiKeyFilter.kt`

**Mudança Mínima:**

Adicionar `/api/v1/leads` à lista de paths protegidos.

**Onde está:**
```kotlin
private val protectedPaths = listOf(
    "/api/v1/users/sync"
)
```

**Como deve ficar:**
```kotlin
private val protectedPaths = listOf(
    "/api/v1/users/sync",
    "/api/v1/leads"  // ← ADICIONAR ESTA LINHA
)
```

**Só isso!** O resto já funciona.

### **2. Validar que Funciona**

**Teste 1: Sem API Key (deve bloquear)**
```bash
curl -X POST http://localhost:8083/api/v1/leads \
  -H "Content-Type: application/json" \
  -d '{"email":"teste@moversemais.com", "optInPlatformNews":true, "optInBlogNews":true, "source":"ASSESSMENT"}'

# Esperado: 401 Unauthorized
# Error: INTERNAL_API_001
# Message: "Invalid or missing internal API key"
```

**Teste 2: Com API Key válido (deve permitir)**
```bash
curl -X POST http://localhost:8083/api/v1/leads \
  -H "Content-Type: application/json" \
  -H "X-Internal-Service-Key: dev-internal-key-change-in-production" \
  -d '{"email":"teste@moversemais.com", "optInPlatformNews":true, "optInBlogNews":true, "source":"ASSESSMENT"}'

# Esperado: 201 Created
# Response: {"id":"uuid", "message":"Lead registrado com sucesso", "isNewLead":true}
```

**Teste 3: Swagger continua público**
```bash
curl http://localhost:8083/swagger-ui/index.html

# Esperado: 200 OK (Swagger acessível)
```

**Teste 4: Endpoints LGPD não afetados**
```bash
curl http://localhost:8083/api/v1/users/me/data \
  -H "X-User-Id: 550e8400-e29b-41d4-a716-446655440000"

# Esperado: 200 OK (não afetado)
```

### **3. Logs Verificados**

**Log quando bloqueado:**
```
WARN [SECURITY-FILTER] API Key inválida ou ausente para: /api/v1/leads
```

**Log quando permitido:**
```
INFO [SECURITY-FILTER] API Key válida - acesso autorizado: /api/v1/leads
```

---

## ⚠️ RESTRIÇÕES

### **O que NÃO PODE ser alterado:**

1. ❌ **NÃO alterar lógica de validação** do InternalApiKeyFilter (já funciona)
2. ❌ **NÃO alterar paths públicos** existentes
3. ❌ **NÃO alterar SecurityInterceptor** (endpoints /me/*)
4. ❌ **NÃO alterar LeadController** (já implementado)
5. ❌ **NÃO alterar LeadService** (já implementado)

### **O que DEVE ser feito:**

1. ✅ **Adicionar** `/api/v1/leads` à lista `protectedPaths`
2. ✅ **Testar** os 4 cenários acima
3. ✅ **Verificar logs** de auditoria
4. ✅ **Commit** com mensagem clara

---

## 📚 DOCUMENTAÇÃO DE REFERÊNCIA

### **Arquivos para Estudar (OBRIGATÓRIO):**

1. **Segurança Existente:**
   - `src/main/kotlin/com/moversemais/user/filter/InternalApiKeyFilter.kt` - **LEIA TODO**
   - `src/main/kotlin/com/moversemais/user/config/SecurityConfig.kt` - Como está configurado
   - `Development/DONE/019__User-Service-Security-Fix.md` - Card que implementou proteção

2. **Endpoint de Leads:**
   - `src/main/kotlin/com/moversemais/user/controller/LeadController.kt` - Endpoint a proteger

3. **Configuração:**
   - `src/main/resources/application-docker.yml` - Onde API Key está configurada
   - `src/main/resources/application-render.yml` - Config produção

### **Cards Relacionados:**
- Card 019: User Service Security Fix (já implementado - **ESTUDAR**)
- Card 026: Lead Management (criou endpoint vulnerável)

---

## 🔧 WORKFLOW

### **1. ESTUDAR (15 minutos)**

```bash
cd moversemais-user

# OBRIGATÓRIO: Ler InternalApiKeyFilter completo
cat src/main/kotlin/com/moversemais/user/filter/InternalApiKeyFilter.kt

# Entender Card 019
cat ../moversemais-arquitetura/Development/DONE/019__User-Service-Security-Fix.md

# Ver onde protectedPaths está definido
grep -n "protectedPaths" src/main/kotlin/com/moversemais/user/filter/InternalApiKeyFilter.kt
```

### **2. CRIAR BRANCH**

```bash
git checkout -b fix/protect-leads-endpoint
```

### **3. IMPLEMENTAR (10 minutos)**

**Mudança Única:**
```kotlin
// InternalApiKeyFilter.kt
private val protectedPaths = listOf(
    "/api/v1/users/sync",
    "/api/v1/leads"  // ← ADICIONAR
)
```

### **4. TESTAR (20 minutos)**

Executar os 4 testes especificados acima.

### **5. COMMIT E PUSH**

```bash
git add src/main/kotlin/com/moversemais/user/filter/InternalApiKeyFilter.kt
git commit -m "fix(user-service): protege endpoint /api/v1/leads com API Key

- Adiciona /api/v1/leads à lista de protectedPaths
- Endpoint agora requer X-Internal-Service-Key
- Apenas BFF pode registrar leads
- Corrige vulnerabilidade crítica de segurança
- Ref: Card 029"

git push origin fix/protect-leads-endpoint
```

### **6. MOVER PARA VALIDATING**

```bash
mv Development/TODO/029__User-Service-Protect-Leads-Endpoint.md \
   Development/VALIDATING/029__User-Service-Protect-Leads-Endpoint.md
```

---

## ✅ CRITÉRIOS DE VALIDAÇÃO

### **Funcionalidades:**
- [ ] `/api/v1/leads` sem API Key → 401 Unauthorized
- [ ] `/api/v1/leads` com API Key válido → 201 Created
- [ ] `/api/v1/leads` com API Key inválido → 401 Unauthorized
- [ ] `/swagger-ui/index.html` sem API Key → 200 OK (público)
- [ ] `/api/v1/users/me/data` com X-User-Id → 200 OK (não afetado)
- [ ] Logs de auditoria aparecendo

### **Segurança:**
- [ ] Endpoint protegido
- [ ] Apenas BFF pode chamar
- [ ] InternalApiKeyFilter funcionando
- [ ] Sem quebra de endpoints existentes

---

## 🚨 TROUBLESHOOTING

### **Problema: Ainda aceita sem API Key**
**Solução:**
- Verificar se adicionou `/api/v1/leads` corretamente
- Reiniciar aplicação
- Verificar logs de inicialização

### **Problema: Swagger bloqueado**
**Solução:**
- Verificar se `/swagger-ui/**` está em `publicPaths`
- Não deve estar em `protectedPaths`

---

## 🎯 EXPECTATIVAS

**Alex, essa é uma correção CRÍTICA.**

A falha foi **minha** (Arquiobaldo), não sua. Eu deveria ter incluído requisito de segurança no Card 026.

**Agora preciso que você:**
1. Leia o `InternalApiKeyFilter.kt` existente
2. Adicione `/api/v1/leads` aos `protectedPaths`
3. Teste que funciona
4. Commit e push

**Mudança de 1 linha, mas crítica para segurança.**

---

## 📊 OUTPUT ESPERADO

### **Decisões Técnicas:**
(Documentar se fez algo além de adicionar path)

### **Testes Realizados:**
- [ ] Teste 1: Sem API Key bloqueado
- [ ] Teste 2: Com API Key permitido
- [ ] Teste 3: Swagger público
- [ ] Teste 4: LGPD não afetado

### **Dificuldades:**
(Se houver)

---

**Data de Criação:** 01/11/2025  
**Criado por:** Arquiobaldo (corrigindo falha arquitetural)  
**Severidade:** 🔴 CRÍTICA  
**Versão:** 1.0

