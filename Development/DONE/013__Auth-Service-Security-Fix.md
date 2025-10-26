# 🚨 Auth Service - Correção de Segurança CRÍTICA

## 📋 **Card de Desenvolvimento**

**Serviço**: moversemais-auth  
**Tipo**: 🚨 SEGURANÇA CRÍTICA - ALTA PRIORIDADE  
**Prioridade**: 🔴 CRÍTICA  
**Estimativa**: 1 dia  
**Status**: TODO  

---

## 🎯 **Objetivo**

Corrigir vulnerabilidade crítica de segurança no moversemais-auth onde endpoints internos (`/api/v1/auth/token/validate`, `/api/v1/auth/token/refresh`, `/api/v1/auth/user`) estão abertos publicamente, permitindo que qualquer pessoa valide tokens, crie usuários ou renove tokens.

---

## 🚨 **PROBLEMA CRÍTICO**

### **❌ Situação Atual (INSEGURO):**
```kotlin
.requestMatchers("/api/v1/auth/**", "/api/v1/oauth/**", ...).permitAll()
```

**Vulnerabilidades:**
- ❌ Qualquer um pode chamar `/api/v1/auth/token/validate`
- ❌ Qualquer um pode chamar `/api/v1/auth/token/refresh`
- ❌ Qualquer um pode chamar `/api/v1/auth/user`
- ❌ Possível criação massiva de usuários falsos
- ❌ Possível validação de tokens sem autorização

---

## 📚 **Documentação de Referência**

**Leia antes de implementar:**
- `../auth-security/SECURITY_PATTERNS.md` - Padrões de segurança
- `002__User-Service-SSO-Implementation.md` - SecurityInterceptor exemplo
- `003__Objective-Service-SSO-Implementation.md` - SecurityFilter exemplo

---

## ✅ **Tarefas Obrigatórias**

### **1. Verificar Sistema Existente**
**IMPORTANTE**: O sistema já implementa:
- ✅ **OAuthConfig** - Configuração Spring Security
- ✅ **AuthController** - Endpoints críticos
- ✅ **OAuthController** - Endpoints OAuth
- ✅ **Banco de dados** - PostgreSQL configurado

**Sistema de segurança atual está INSEGURO!**

### **2. Criar SecurityFilter**
Criar arquivo: `src/main/kotlin/com/moversemais/auth/filter/SecurityFilter.kt`

**Funcionalidades obrigatórias:**
- ✅ Validar API Key em endpoints internos (`/api/v1/auth/**`)
- ✅ Permitir endpoints OAuth públicos (`/oauth2/**`, `/login/**`)
- ✅ Validar formato UUID da API Key
- ✅ Logs de auditoria de acesso
- ✅ Tratamento de erros com códigos padronizados

### **3. Atualizar OAuthConfig**
Atualizar arquivo: `src/main/kotlin/com/moversemais/auth/config/OAuthConfig.kt`

**Funcionalidades obrigatórias:**
- ✅ Separar endpoints públicos (OAuth) de internos (API)
- ✅ Adicionar SecurityFilter antes da cadeia
- ✅ Manter OAuth flow funcionando
- ✅ Proteger endpoints críticos

### **4. Configurar API Key**
Atualizar arquivo: `src/main/resources/application-docker.yml`

**Configurações obrigatórias:**
```yaml
app:
  security:
    internal-service-key: ${INTERNAL_SERVICE_KEY:dev-internal-key-change-in-production}
```

### **5. Atualizar BFF para Enviar API Key**
**IMPORTANTE**: BFF precisa enviar API Key ao chamar moversemais-auth

**Card relacionado**: Criar Card 014 para atualizar BFF

### **6. Testes de Segurança**
**Testes obrigatórios:**
- ✅ Endpoints OAuth públicos funcionando sem API Key
- ✅ Endpoints internos bloqueados sem API Key
- ✅ Endpoints internos funcionam com API Key válida
- ✅ API Key inválida retorna 401
- ✅ Logs de auditoria funcionando

---

## 🔧 **Instruções de Implementação**

### **1. SecurityFilter (Similar aos outros serviços)**
- **Objetivo**: Validar API Key em endpoints internos
- **Padrão**: Seguir SecurityFilter do moversemais-user e moversemais-objective
- **Validação**: Header `X-Internal-Service-Key`
- **Endpoints protegidos**: `/api/v1/auth/**` (exceto success/failure)
- **Endpoints públicos**: `/oauth2/**`, `/login/**`, `/actuator/**`, `/swagger-ui/**`

### **2. Diferenciação de Endpoints**
- **Públicos**: OAuth flow, Swagger, Actuator
- **Autenticados**: `/api/v1/oauth/user` (requer OAuth2User)
- **Internos**: `/api/v1/auth/**` (requer API Key)

### **3. API Key**
- **Desenvolvimento**: `dev-internal-key-change-in-production`
- **Produção**: UUID forte via variável de ambiente
- **Rotação**: Documentar processo de rotação
- **Compartilhamento**: Apenas BFF e serviços autorizados

### **4. Logs de Auditoria**
- **Acesso autorizado**: `✅ [AUTH] API Key válida: /api/v1/auth/token/validate`
- **Acesso negado**: `❌ [AUTH] API Key inválida ou ausente: /api/v1/auth/token/validate`
- **OAuth flow**: `🔍 [AUTH] OAuth login iniciado: google`

### **5. Preservar Sistema Existente**
- **NÃO ALTERAR**: OAuth flow (deve continuar público)
- **NÃO ALTERAR**: Controllers, Services, DTOs
- **ADICIONAR**: SecurityFilter para endpoints internos
- **MANTER**: Funcionalidade OAuth funcionando

---

## 🧪 **Critérios de Validação**

### **Funcionalidades Obrigatórias**
- [ ] SecurityFilter implementado
- [ ] API Key configurada
- [ ] Endpoints OAuth públicos funcionando
- [ ] Endpoints internos protegidos
- [ ] Logs de auditoria funcionando
- [ ] OAuth flow não quebrado

### **Testes de Segurança Obrigatórios**
- [ ] `GET /oauth2/authorization/google` funciona SEM API Key (público)
- [ ] `POST /api/v1/auth/token/validate` BLOQUEIA sem API Key
- [ ] `POST /api/v1/auth/token/validate` FUNCIONA com API Key válida
- [ ] `POST /api/v1/auth/token/refresh` BLOQUEIA sem API Key
- [ ] `POST /api/v1/auth/user` BLOQUEIA sem API Key
- [ ] `GET /api/v1/oauth/user` funciona com OAuth2User (pós-login)

### **Código Obrigatório**
- [ ] SecurityFilter implementado
- [ ] OAuthConfig atualizado com filter
- [ ] Configuração de API Key
- [ ] Logs de auditoria
- [ ] Testes de segurança funcionando
- [ ] OAuth flow preservado (NÃO quebrar)

---

## 🚨 **Troubleshooting**

### **Problema: OAuth parou de funcionar**
- Verificar se endpoints OAuth estão em .permitAll()
- Verificar se SecurityFilter não está bloqueando OAuth
- Verificar ordem dos filters
- Verificar logs de Spring Security

### **Problema: API Key não é validada**
- Verificar se SecurityFilter está registrado
- Verificar se filter está antes do OAuth filter
- Verificar regex de paths internos
- Verificar logs do SecurityFilter

### **Problema: Endpoints ainda estão abertos**
- Verificar se SecurityFilter está executando
- Verificar logs de requisições
- Verificar configuração de API Key
- Testar com curl sem API Key

---

## 📋 **Checklist de Implementação**

- [ ] SecurityFilter implementado (padrão dos outros serviços)
- [ ] OAuthConfig atualizado com SecurityFilter
- [ ] API Key configurada em application-docker.yml
- [ ] API Key configurada em application-render.yml
- [ ] Endpoints OAuth públicos funcionando
- [ ] Endpoints internos protegidos com API Key
- [ ] Logs de auditoria funcionando
- [ ] Testes de segurança funcionando
- [ ] OAuth flow preservado (não quebrado)
- [ ] Documentação atualizada

---

## 🔧 **Exemplo de Teste:**

### **✅ OAuth Flow (Deve Funcionar SEM API Key):**
```bash
# Deve funcionar
curl http://localhost:8084/oauth2/authorization/google
```

### **❌ Endpoints Internos (Deve BLOQUEAR sem API Key):**
```bash
# Deve retornar 401
curl -X POST http://localhost:8084/api/v1/auth/token/validate \
  -H "Content-Type: application/json" \
  -d '{"token":"..."}'
```

### **✅ Endpoints Internos (Deve FUNCIONAR com API Key):**
```bash
# Deve funcionar
curl -X POST http://localhost:8084/api/v1/auth/token/validate \
  -H "Content-Type: application/json" \
  -H "X-Internal-Service-Key: dev-internal-key-change-in-production" \
  -d '{"token":"..."}'
```

---

## 🎯 **Próximo Passo**

🚨 **ESTE CARD TEM PRIORIDADE MÁXIMA!**

Após completar este card, criar Card 014 para atualizar BFF com API Key.

**Este é um problema de SEGURANÇA CRÍTICO que deve ser resolvido IMEDIATAMENTE!**

---

**Criado em**: Janeiro 2025  
**Responsável**: Equipe MoverseMais  
**Status**: 🚨 TODO - PRIORIDADE MÁXIMA
