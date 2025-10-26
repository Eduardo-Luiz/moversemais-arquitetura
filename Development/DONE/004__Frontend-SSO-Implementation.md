# 🎯 Frontend - Implementação SSO e Segurança

## 📋 **Card de Desenvolvimento**

**Serviço**: moversemais-store-front  
**Tipo**: SSO e Segurança  
**Prioridade**: Alta  
**Estimativa**: 1-2 dias  
**Status**: TODO  

---

## 🎯 **Objetivo**

Atualizar Apollo Client para incluir headers de segurança obrigatórios (`X-User-Id`, `X-Provider`, `X-Request-Id`) e preparar para integração com SSO real. O sistema de autenticação e rotas já existe e funciona.

---

## 📚 **Documentação de Referência**

**Leia antes de implementar:**
- `../auth-security/SIMPLIFIED_LOCAL_DEVELOPMENT.md` - Desenvolvimento local simplificado
- `../auth-security/STEP_BY_STEP_GUIDES.md` - Semana 4: Frontend
- `../auth-security/SECURITY_PATTERNS.md` - Padrões de segurança

---

## ✅ **Tarefas Obrigatórias**

### **1. Verificar Sistema Existente**
**IMPORTANTE**: O sistema já implementa:
- ✅ **Autenticação** - localStorage com `authToken` e `userData`
- ✅ **ProtectedRoute** - Componente já existe e funciona
- ✅ **Rotas públicas/privadas** - Já configuradas no App.tsx
- ✅ **Apollo Client** - Já configurado para GraphQL
- ✅ **Navegação** - NavigationBar já implementada

**Não é necessário alterar estes componentes!**

### **2. Atualizar Apollo Client**
Atualizar arquivo: `src/lib/apollo-client.ts`

**Funcionalidades obrigatórias:**
- ✅ Adicionar headers de segurança obrigatórios (`X-User-Id`, `X-Provider`, `X-Request-Id`)
- ✅ Extrair dados do usuário do localStorage
- ✅ Gerar Request ID único para cada requisição
- ✅ Configuração via variáveis de ambiente

### **3. Atualizar Variáveis de Ambiente**
Atualizar arquivo: `.env.development`

**Variáveis obrigatórias:**
```bash
VITE_GRAPHQL_URL=http://localhost:3001/api/graphql
VITE_FIXED_USER_ID=550e8400-e29b-41d4-a716-446655440000
VITE_MOCK_PROVIDER=google
VITE_DEV_MODE=true
```

### **4. Criar Hook useAuth (Opcional)**
Criar arquivo: `src/hooks/useAuth.ts`

**Funcionalidades obrigatórias:**
- ✅ Centralizar lógica de autenticação existente
- ✅ Estado de loading
- ✅ Estado de autenticação
- ✅ Funções de login/logout

### **5. Testes Básicos**
**Testes obrigatórios:**
- ✅ Headers obrigatórios sendo enviados para BFF
- ✅ Comunicação com BFF funcionando
- ✅ Rotas públicas/privadas funcionando
- ✅ Sistema de autenticação funcionando

---

## 🔧 **Instruções de Implementação**

### **1. Atualizar Apollo Client**
- **Objetivo**: Adicionar headers de segurança obrigatórios
- **Headers necessários**: `X-User-Id`, `X-Provider`, `X-Request-Id`, `X-Timestamp`
- **Fonte de dados**: localStorage (`userData`, `authToken`) + variáveis de ambiente
- **Fallback**: Usar variáveis de ambiente se localStorage estiver vazio
- **Geração**: Request ID único para cada requisição

### **2. Atualizar Variáveis de Ambiente**
- **Arquivo**: `.env.development`
- **Adicionar**: `VITE_MOCK_PROVIDER=google`
- **Manter**: Variáveis existentes (`VITE_GRAPHQL_URL`, `VITE_FIXED_USER_ID`)

### **3. Hook useAuth (Opcional)**
- **Objetivo**: Centralizar lógica de autenticação existente
- **Funcionalidades**: Estado do usuário, loading, login/logout
- **Fonte**: localStorage existente
- **Integração**: Com sistema de rotas existente

### **4. Preservar Sistema Existente**
- **NÃO ALTERAR**: App.tsx, ProtectedRoute.tsx, NavigationBar.tsx
- **NÃO ALTERAR**: Sistema de rotas existente
- **NÃO ALTERAR**: Sistema de autenticação existente
- **FOCO**: Apenas adicionar headers ao Apollo Client

### **5. Sistema Existente (NÃO ALTERAR)**
- **App.tsx**: Rotas públicas/privadas já configuradas
- **ProtectedRoute.tsx**: Verificação de autenticação já implementada
- **NavigationBar.tsx**: Menu de navegação já implementado
- **Sistema de autenticação**: localStorage já funciona
- **Apollo Client**: Já configurado para GraphQL


---

## 🧪 **Critérios de Validação**

### **Funcionalidades Obrigatórias**
- [ ] Apollo Client atualizado com headers obrigatórios
- [ ] Headers de segurança sendo enviados para BFF (`X-User-Id`, `X-Provider`, `X-Request-Id`)
- [ ] Sistema de autenticação existente funcionando
- [ ] Rotas públicas/privadas funcionando (já existem)

### **Testes Obrigatórios**
- [ ] Headers obrigatórios sendo enviados para BFF
- [ ] Comunicação com BFF funcionando
- [ ] Rotas públicas acessíveis sem autenticação
- [ ] Rotas protegidas redirecionam se não autenticado
- [ ] Sistema de autenticação existente funcionando

### **Código Obrigatório**
- [ ] Apollo Client atualizado com headers
- [ ] Variáveis de ambiente configuradas
- [ ] Hook useAuth implementado (opcional)
- [ ] Sistema existente preservado (NÃO ALTERAR)

---

## 🚨 **Troubleshooting**

### **Problema: Headers não são enviados para BFF**
- Verificar se Apollo Client está configurado corretamente
- Verificar se localStorage tem dados do usuário
- Verificar console do navegador para erros
- Verificar se variáveis de ambiente estão configuradas

### **Problema: BFF não responde**
- Verificar se BFF está rodando na porta 3001
- Verificar se headers estão sendo enviados
- Verificar console do navegador para erros
- Verificar se BFF está configurado para aceitar headers

### **Problema: Sistema de autenticação não funciona**
- Verificar se localStorage tem `authToken` e `userData`
- Verificar se ProtectedRoute está funcionando
- Verificar se rotas estão configuradas corretamente
- Verificar console do navegador para erros

---

## 📋 **Checklist de Implementação**

- [ ] Apollo Client atualizado com headers obrigatórios
- [ ] Variáveis de ambiente configuradas
- [ ] Hook useAuth implementado (opcional)
- [ ] Headers sendo enviados para BFF
- [ ] Testes básicos funcionando
- [ ] Comunicação com BFF funcionando
- [ ] Sistema existente preservado (NÃO ALTERAR)

---

## 🎯 **Próximo Passo**

Após completar este card, mover para `WIP/` e depois para `VALIDATING/` para análise.

**Próximo card**: Auth Service - Implementação SSO e Segurança

---

**Criado em**: Outubro 2025  
**Responsável**: Equipe MoverseMais  
**Status**: TODO
