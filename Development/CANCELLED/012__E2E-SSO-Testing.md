# 🎯 E2E Testing - SSO Completo

## 📋 **Card de Desenvolvimento**

**Serviço**: Todos (End-to-End Testing)  
**Tipo**: Testes de Integração Completa  
**Prioridade**: Alta  
**Estimativa**: 2-3 dias  
**Status**: TODO  

---

## 🎯 **Objetivo**

Implementar testes end-to-end completos do fluxo SSO, validando integração entre todos os serviços (Frontend → BFF → Auth → User → Objective), incluindo cenários de sucesso, erro e edge cases.

---

## 📚 **Documentação de Referência**

**Leia antes de implementar:**
- Cards 001-008, 013, 015-020 - Todas as implementações concluídas (pré-requisito)
- Cypress/Playwright Documentation
- Testes existentes nos projetos para referência

**⚠️ IMPORTANTE**: Apenas Google OAuth será testado (Facebook e LinkedIn não foram implementados)

---

## ✅ **Tarefas Obrigatórias**

### **1. Configurar Framework de Testes**
**Escolher e configurar:**
- ✅ Cypress ou Playwright para testes E2E
- ✅ Configurar ambiente de testes
- ✅ Configurar mocks para OAuth (testes automatizados)
- ✅ Configurar relatórios de testes

### **2. Testes de Autenticação (Apenas Google)**
**Cenários obrigatórios:**
- ✅ Login com Google bem-sucedido via BFF
- ✅ Callback OAuth processado corretamente
- ✅ JWT salvo no localStorage
- ✅ Dados do usuário salvos
- ✅ Redirecionamento para dashboard
- ✅ Arquitetura BFF respeitada (Frontend → BFF → Auth)

### **3. Testes de Navegação**
**Cenários obrigatórios:**
- ✅ Rotas públicas acessíveis sem login
- ✅ Rotas privadas bloqueadas sem login
- ✅ Rotas privadas acessíveis após login
- ✅ Redirecionamento correto após login
- ✅ Navegação entre páginas autenticadas

### **4. Testes de Headers**
**Cenários obrigatórios:**
- ✅ Headers enviados do Frontend para BFF
- ✅ Headers propagados do BFF para backends
- ✅ X-User-Id correto em todas as requisições
- ✅ X-Provider correto em todas as requisições
- ✅ X-Request-Id único em cada requisição

### **5. Testes de JWT**
**Cenários obrigatórios:**
- ✅ JWT válido aceito
- ✅ JWT inválido rejeitado com 401
- ✅ JWT expirado rejeitado com 401
- ✅ Refresh token funcionando
- ✅ Logout limpa JWT

### **6. Testes de Integração entre Serviços**
**Cenários obrigatórios:**
- ✅ Auth cria usuário no User Service
- ✅ Frontend busca dados via BFF
- ✅ BFF busca objetivos no Objective Service
- ✅ Headers propagados corretamente
- ✅ Logs de auditoria em todos os serviços

### **7. Testes de Erros e Edge Cases**
**Cenários obrigatórios:**
- ✅ OAuth cancelado pelo usuário
- ✅ OAuth com erro do provider
- ✅ Auth service offline
- ✅ User service offline
- ✅ BFF offline
- ✅ Timeout de requisições
- ✅ Múltiplos logins sequenciais

### **8. Testes de Segurança**
**Cenários obrigatórios:**
- ✅ Requisição sem JWT rejeitada
- ✅ Requisição sem headers rejeitada
- ✅ JWT de outro usuário rejeitado
- ✅ CORS funcionando corretamente
- ✅ HTTPS obrigatório em produção

---

## 🔧 **Instruções de Implementação**

### **1. Framework de Testes**
- **Escolher**: Cypress ou Playwright
- **Configurar**: Ambiente de testes isolado
- **Mocks**: OAuth providers para testes automatizados
- **CI/CD**: Integrar com GitHub Actions

### **2. Estrutura de Testes**
```
tests/
├── e2e/
│   ├── auth/
│   │   ├── google-login.spec.ts
│   │   ├── facebook-login.spec.ts
│   │   ├── linkedin-login.spec.ts
│   │   └── logout.spec.ts
│   ├── navigation/
│   │   ├── public-routes.spec.ts
│   │   ├── protected-routes.spec.ts
│   │   └── redirects.spec.ts
│   ├── integration/
│   │   ├── headers-propagation.spec.ts
│   │   ├── jwt-validation.spec.ts
│   │   └── service-communication.spec.ts
│   └── errors/
│       ├── oauth-errors.spec.ts
│       ├── service-offline.spec.ts
│       └── security-errors.spec.ts
```

### **3. Mocks para OAuth**
- **Google**: Mock de OAuth flow para testes automatizados
- **Facebook**: Mock de OAuth flow
- **LinkedIn**: Mock de OAuth flow
- **JWT**: Mock de tokens para testes

### **4. Métricas e Relatórios**
- **Coverage**: Cobertura de testes
- **Performance**: Tempo de resposta
- **Reliability**: Taxa de sucesso
- **Security**: Vulnerabilidades detectadas

### **5. Documentação de Testes**
- **README**: Como rodar testes
- **Cenários**: Documentar cada cenário testado
- **Troubleshooting**: Problemas comuns
- **CI/CD**: Como configurar pipeline

---

## 🧪 **Critérios de Validação**

### **Funcionalidades Obrigatórias**
- [ ] Framework de testes configurado
- [ ] Testes de autenticação implementados
- [ ] Testes de navegação implementados
- [ ] Testes de headers implementados
- [ ] Testes de JWT implementados
- [ ] Testes de integração implementados
- [ ] Testes de erros implementados
- [ ] Testes de segurança implementados

### **Testes Obrigatórios**
- [ ] Login Google E2E funcionando
- [ ] Login Facebook E2E funcionando
- [ ] Login LinkedIn E2E funcionando
- [ ] Headers propagados corretamente
- [ ] JWT validado corretamente
- [ ] Serviços integrados funcionando
- [ ] Erros tratados corretamente
- [ ] Segurança validada

### **Código Obrigatório**
- [ ] Testes E2E implementados
- [ ] Mocks configurados
- [ ] CI/CD configurado
- [ ] Documentação criada
- [ ] Relatórios funcionando
- [ ] Coverage > 80%

---

## 🚨 **Troubleshooting**

### **Problema: Testes falham localmente**
- Verificar se todos os serviços estão rodando
- Verificar se banco de dados está limpo
- Verificar se variáveis de ambiente estão configuradas
- Verificar logs dos serviços

### **Problema: Mocks OAuth não funcionam**
- Verificar configuração do framework de testes
- Verificar se endpoints mockados estão corretos
- Verificar se respostas mockadas estão no formato correto

### **Problema: Testes passam localmente mas falham em CI/CD**
- Verificar variáveis de ambiente no CI/CD
- Verificar se serviços estão acessíveis
- Verificar timeouts
- Verificar ordem de execução dos testes

---

## 📋 **Checklist de Implementação**

- [ ] Framework de testes configurado
- [ ] Testes de autenticação implementados (Google, Facebook, LinkedIn)
- [ ] Testes de navegação implementados
- [ ] Testes de headers implementados
- [ ] Testes de JWT implementados
- [ ] Testes de integração implementados
- [ ] Testes de erros implementados
- [ ] Testes de segurança implementados
- [ ] Mocks configurados
- [ ] CI/CD configurado
- [ ] Documentação criada
- [ ] Relatórios funcionando

---

## 🎯 **Próximo Passo**

Após completar este card, mover para `WIP/` e depois para `VALIDATING/` para análise.

**Este é o último card da implementação SSO!**

---

**Criado em**: Janeiro 2025  
**Responsável**: Equipe MoverseMais  
**Status**: TODO
