# 🎯 Production - Deploy e Configuração

## 📋 **Card de Desenvolvimento**

**Serviço**: Todos (moversemais-auth, moversemais-user, moversemais-objective, moversemais-store-graphql, moversemais-store-front)  
**Tipo**: Deploy e Configuração de Produção  
**Prioridade**: Alta  
**Estimativa**: 2-3 dias  
**Status**: TODO  

---

## 🎯 **Objetivo**

Configurar SSO (Google OAuth) em produção no Render.com, incluindo variáveis de ambiente OAuth, secrets, URLs de produção e configurações de segurança. Todos os serviços já estão deployados no Render.com.

---

## 📚 **Documentação de Referência**

**Leia antes de implementar:**
- Cards 001-008, 013, 015-020 - Implementações base concluídas
- Render.com Documentation
- `src/main/resources/application-render.yml` - Configurações de produção existentes

**⚠️ IMPORTANTE**: Apenas Google OAuth será configurado (Facebook e LinkedIn não serão implementados no MVP)

---

## ✅ **Tarefas Obrigatórias**

### **1. Verificar Sistema Existente**
**IMPORTANTE**: Os serviços JÁ ESTÃO deployados:
- ✅ **Render.com** - Todos os serviços backend deployados
- ✅ **PostgreSQL** - Bancos em produção funcionando
- ✅ **Frontend** - Já deployado
- ✅ **BFF** - Já deployado

**Faltam apenas configurações SSO (Google OAuth)!**

### **2. moversemais-auth (Render.com)**
**Configurações SSO a adicionar:**
- ✅ OAUTH_GOOGLE_CLIENT_ID
- ✅ OAUTH_GOOGLE_CLIENT_SECRET
- ✅ JWT_SECRET (se ainda não configurado)
- ✅ INTERNAL_SERVICE_KEY (para User Service)
- ✅ Configurar Google Cloud Console redirect URIs de produção
- ✅ Testar OAuth Google em produção

### **3. moversemais-user (Render.com)**
**Configurações SSO a adicionar:**
- ✅ INTERNAL_SERVICE_KEY (mesmo valor do Auth Service)
- ✅ Testar integração Auth → User em produção
- ✅ Verificar API Key funcionando

### **4. moversemais-objective (Render.com)**
**Verificações:**
- ✅ Headers de segurança funcionando
- ✅ Integração com BFF funcionando
- ✅ Sem mudanças necessárias para SSO

### **5. moversemais-store-graphql (BFF)**
**Configurações SSO a adicionar:**
- ✅ AUTH_SERVICE_URL (URL produção do Auth Service)
- ✅ JWT_SECRET (mesmo do Auth Service)
- ✅ BACKEND_OBJECTIVE_URL, BACKEND_USER_URL, etc (URLs produção)
- ✅ CORS para frontend de produção
- ✅ Testar OAuth flow via BFF em produção

### **6. moversemais-store-front (Frontend)**
**Configurações SSO a adicionar:**
- ✅ VITE_GRAPHQL_URL (URL produção do BFF)
- ❌ NÃO configurar VITE_AUTH_SERVICE_URL (Frontend não chama Auth direto!)
- ✅ Testar OAuth flow via BFF em produção

### **7. OAuth Redirect URIs (Apenas Google)**
**Configurações obrigatórias:**
- ✅ Google: Adicionar URLs de produção no Google Cloud Console
- ✅ Testar redirect Google funcionando

### **8. Secrets e Segurança**
**Configurações obrigatórias:**
- ✅ JWT_SECRET forte (32+ caracteres, mesmo em todos os serviços)
- ✅ OAUTH_GOOGLE_CLIENT_ID e OAUTH_GOOGLE_CLIENT_SECRET
- ✅ INTERNAL_SERVICE_KEY (Auth ↔ User)
- ✅ Database passwords seguros
- ✅ HTTPS forçado
- ✅ CORS restrito (Frontend → BFF, BFF → Backends)

---

## 🔧 **Instruções de Implementação**

### **1. Render.com - moversemais-auth**
- **Nome**: moversemais-auth
- **Tipo**: Web Service
- **Build**: `./gradlew build`
- **Start**: `java -jar build/libs/moversemais-auth-0.0.1-SNAPSHOT.jar`
- **Environment**: Production
- **Health Check**: `/actuator/health`

### **2. Variáveis de Ambiente - moversemais-auth**
```bash
DATABASE_URL=postgresql://...
JWT_SECRET=sua-chave-secreta-producao-32-chars
JWT_EXPIRATION=3600
JWT_REFRESH_EXPIRATION=604800
OAUTH_GOOGLE_CLIENT_ID=...
OAUTH_GOOGLE_CLIENT_SECRET=...
OAUTH_FACEBOOK_CLIENT_ID=...
OAUTH_FACEBOOK_CLIENT_SECRET=...
OAUTH_LINKEDIN_CLIENT_ID=...
OAUTH_LINKEDIN_CLIENT_SECRET=...
SPRING_PROFILES_ACTIVE=render
```

### **3. Redirect URIs de Produção**
- **Google**: `https://moversemais-auth.onrender.com/login/oauth2/code/google`
- **Facebook**: `https://moversemais-auth.onrender.com/login/oauth2/code/facebook`
- **LinkedIn**: `https://moversemais-auth.onrender.com/login/oauth2/code/linkedin`

### **4. Configurar CORS**
- **BFF**: Permitir apenas frontend de produção
- **Auth**: Permitir apenas BFF e frontend de produção
- **Backends**: Permitir apenas BFF

### **5. Testar Fluxo Completo**
- **Frontend** → **Auth** → **Google** → **Auth** → **User** → **Frontend**
- Verificar logs em todos os serviços
- Verificar headers sendo propagados
- Verificar JWT sendo gerado e validado

---

## 🧪 **Critérios de Validação**

### **Funcionalidades Obrigatórias**
- [ ] moversemais-auth deployado no Render.com
- [ ] Variáveis de ambiente configuradas
- [ ] OAuth redirect URIs configurados
- [ ] HTTPS funcionando
- [ ] CORS configurado corretamente
- [ ] Todos os serviços integrados

### **Testes Obrigatórios**
- [ ] Login com Google em produção funcionando
- [ ] Login com Facebook em produção funcionando
- [ ] Login com LinkedIn em produção funcionando
- [ ] JWT sendo gerado em produção
- [ ] Headers propagados em produção
- [ ] Integração completa funcionando

### **Código Obrigatório**
- [ ] Todos os serviços deployados
- [ ] Variáveis de ambiente configuradas
- [ ] OAuth configurado em produção
- [ ] CORS configurado
- [ ] HTTPS configurado
- [ ] Health checks funcionando

---

## 🚨 **Troubleshooting**

### **Problema: OAuth falha em produção**
- Verificar redirect URIs configurados
- Verificar HTTPS configurado
- Verificar secrets OAuth
- Verificar logs do Render.com

### **Problema: JWT não valida em produção**
- Verificar JWT_SECRET igual em todos os serviços
- Verificar CORS configurado
- Verificar HTTPS
- Verificar logs

### **Problema: Serviços não se comunicam**
- Verificar URLs de produção
- Verificar CORS
- Verificar headers sendo enviados
- Verificar firewall/network

---

## 📋 **Checklist de Implementação**

- [ ] moversemais-auth deployado
- [ ] moversemais-user atualizado
- [ ] moversemais-objective verificado
- [ ] moversemais-store-graphql configurado
- [ ] moversemais-store-front configurado
- [ ] OAuth redirect URIs configurados
- [ ] Secrets configurados
- [ ] CORS configurado
- [ ] HTTPS configurado
- [ ] Testes end-to-end em produção

---

## 🎯 **Próximo Passo**

Após completar este card, mover para `WIP/` e depois para `VALIDATING/` para análise.

**Próximo card**: E2E Testing

---

**Criado em**: Janeiro 2025  
**Responsável**: Equipe MoverseMais  
**Status**: TODO
