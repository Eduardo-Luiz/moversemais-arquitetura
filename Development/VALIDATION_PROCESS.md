# 🔍 Processo de Validação - MoverseMais

## 📋 **Visão Geral**

Este documento define o processo de validação para cards na pasta `VALIDATING/`, garantindo que implementações estejam dentro dos padrões arquiteturais definidos.

---

## 🎯 **Objetivo**

Validar se implementações de SSO e segurança estão:
- ✅ Funcionando corretamente
- ✅ Seguindo padrões arquiteturais
- ✅ Atendendo critérios de qualidade
- ✅ Prontas para produção

---

## 📋 **Processo de Validação**

### **1. Card Movido para VALIDATING**
Quando um card é movido para `VALIDATING/`, significa que:
- ✅ Implementação foi concluída
- ✅ Desenvolvedor testou localmente
- ✅ Pronto para análise arquitetural

### **2. Análise Arquitetural**
Verificar se implementação está seguindo:

#### **Padrões de Segurança**
- [ ] Headers de segurança implementados
- [ ] Validação de autenticação funcionando
- [ ] Logs de auditoria implementados
- [ ] Tratamento de erros adequado

#### **Padrões de Código**
- [ ] Estrutura de arquivos correta
- [ ] Dependências instaladas
- [ ] Configurações de ambiente
- [ ] Testes básicos funcionando

#### **Padrões de LGPD**
- [ ] Dados mínimos coletados
- [ ] Endpoints LGPD implementados
- [ ] Criptografia de dados sensíveis
- [ ] Logs de acesso implementados

### **3. Critérios de Aprovação**

#### **✅ APROVADO - Mover para DONE**
- [ ] Todos os critérios obrigatórios atendidos
- [ ] Código seguindo padrões arquiteturais
- [ ] Testes funcionando
- [ ] Documentação atualizada

#### **❌ REJEITADO - Mover para WIP**
- [ ] Critérios obrigatórios não atendidos
- [ ] Padrões arquiteturais não seguidos
- [ ] Testes não funcionando
- [ ] Documentação incompleta

### **4. Feedback Detalhado**
Para cards rejeitados, fornecer:
- **Problemas identificados**: Lista específica
- **Soluções sugeridas**: Como corrigir
- **Referências**: Documentos relevantes
- **Próximos passos**: O que fazer

---

## 🔧 **Ferramentas de Validação**

### **1. Validação de Código**
```bash
# Frontend
npm run build
npm run test

# Backend
./gradlew build
./gradlew test

# BFF
npm run build
npm run test
```

### **2. Validação de Segurança**
```bash
# Testar headers de segurança
curl -H "X-User-Id: dev-user-123" http://localhost:8080/api/v1/objectives

# Testar autenticação
curl -H "Authorization: Bearer token" http://localhost:3001/api/graphql

# Testar endpoints LGPD
curl -H "X-User-Id: dev-user-123" http://localhost:8083/api/v1/users/me/data
```

### **3. Validação de Funcionalidade**
```bash
# Testar fluxo completo
1. Frontend carrega usuário mock
2. BFF recebe headers de segurança
3. Backend valida headers
4. Dados retornados corretamente
```

---

## 📊 **Checklist de Validação por Serviço**

### **BFF (moversemais-store-graphql)**
- [ ] Middleware de autenticação funcionando
- [ ] Headers de segurança adicionados
- [ ] Comunicação com backend funcionando
- [ ] Logs de debug funcionando
- [ ] CORS configurado corretamente

### **User Service (moversemais-user)**
- [ ] Interceptor de segurança funcionando
- [ ] Endpoints LGPD implementados
- [ ] Validação de headers funcionando
- [ ] Logs de auditoria funcionando
- [ ] Dados mínimos implementados

### **Objective Service (moversemais-objective)**
- [ ] Interceptor de segurança funcionando
- [ ] Validação de propriedade implementada
- [ ] Headers de segurança validados
- [ ] Logs de auditoria funcionando
- [ ] Controllers atualizados

### **Frontend (moversemais-store-front)**
- [ ] Hook de autenticação mock funcionando
- [ ] Proteção de rotas implementada
- [ ] Cliente BFF com headers funcionando
- [ ] Navegação condicional funcionando
- [ ] Variáveis de ambiente configuradas

### **Auth Service (moversemais-auth)**
- [ ] OAuth flow funcionando
- [ ] JWT generation/validation funcionando
- [ ] User service funcionando
- [ ] Banco de dados configurado
- [ ] Endpoints respondendo

---

## 🚨 **Problemas Comuns e Soluções**

### **Problema: Headers não chegam no backend**
**Solução**: Verificar se BFF está adicionando headers corretamente
```typescript
// Verificar no BFF
request.headers.set('X-User-Id', userId)
request.headers.set('X-Provider', provider)
```

### **Problema: CORS errors no frontend**
**Solução**: Verificar configuração CORS no BFF
```typescript
// Verificar no BFF
const corsOptions = {
  origin: ['http://localhost:5173'],
  credentials: true
}
```

### **Problema: Interceptor não é aplicado**
**Solução**: Verificar configuração Spring Security
```kotlin
// Verificar no backend
@Bean
fun securityFilterChain(http: HttpSecurity): SecurityFilterChain {
    return http
        .addFilterBefore(securityInterceptor(), UsernamePasswordAuthenticationFilter::class.java)
        .build()
}
```

### **Problema: Usuário mock não carrega**
**Solução**: Verificar variáveis de ambiente
```bash
# Verificar no frontend
VITE_AUTH_ENABLED=false
VITE_MOCK_USER_ID=dev-user-123
```

---

## 📋 **Template de Feedback**

### **Para Cards Aprovados**
```
✅ APROVADO - Card movido para DONE

Implementação está seguindo todos os padrões arquiteturais:
- ✅ Segurança implementada corretamente
- ✅ Código seguindo padrões
- ✅ Testes funcionando
- ✅ Documentação atualizada

Próximo passo: Implementar próximo card da fila.
```

### **Para Cards Rejeitados**
```
❌ REJEITADO - Card movido para WIP

Problemas identificados:
1. [Problema específico]
2. [Problema específico]
3. [Problema específico]

Soluções sugeridas:
1. [Solução específica]
2. [Solução específica]
3. [Solução específica]

Referências:
- [Documento relevante]
- [Padrão específico]

Próximo passo: Corrigir problemas e mover para VALIDATING novamente.
```

---

## 🎯 **Próximos Passos**

1. **Implementar card**: Seguir guia específico
2. **Testar localmente**: Verificar funcionamento
3. **Mover para VALIDATING**: Solicitar análise
4. **Receber feedback**: Aprovação ou rejeição
5. **Corrigir se necessário**: Implementar melhorias
6. **Mover para DONE**: Card concluído

---

**Última Atualização**: Outubro 2025  
**Versão**: 1.0  
**Mantenedor**: Equipe MoverseMais

---

*Este processo garante qualidade e consistência nas implementações de SSO e segurança.*
