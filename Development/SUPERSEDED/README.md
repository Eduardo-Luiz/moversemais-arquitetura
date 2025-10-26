# ⚠️ SUPERSEDED - Cards Obsoletos

## 📋 **O que é esta pasta?**

Esta pasta contém **cards implementados mas substituídos** por versões com melhor arquitetura ou segurança.

---

## 🚨 **Cards Nesta Pasta**

### **Card 008 - Frontend Auth Integration**
### **Card 020 - BFF OAuth Endpoints**

**Data de Implementação:** Janeiro 2025  
**Data de Substituição:** Janeiro 2025  
**Motivo:** Violação de segurança - Auth Service exposto publicamente

---

## ❌ **Problema Arquitetural Identificado**

### **Falha de Segurança:**

A implementação dos Cards 008 e 020 criou um fluxo onde:

```
❌ IMPLEMENTADO (INSEGURO):
Frontend → BFF (proxy) → Auth Service (PÚBLICO!) → Google
```

**Problemas:**
1. ❌ Auth Service acessível publicamente
2. ❌ Navegador do usuário acessa Auth Service diretamente
3. ❌ CORS do Auth Service aceita requisições públicas
4. ❌ Superfície de ataque aumentada
5. ❌ Viola princípio: "Apenas BFF é público, backends são internos"

---

## ✅ **Solução Correta**

### **Substituído por:**

- **Card 021** - BFF OAuth Server-Side
- **Card 022** - Frontend OAuth Server-Side
- **Card 023** - Auth Service OAuth Refactor

### **Arquitetura Segura:**

```
✅ CORRETO (SEGURO):
Frontend → BFF (OAuth server-side) → Google
                ↓ (interno)
           Auth Service (100% INTERNO)
```

**Correções:**
1. ✅ BFF gerencia OAuth flow server-side
2. ✅ BFF tem credenciais Google (Client ID/Secret)
3. ✅ Callback vem para BFF (não Auth Service)
4. ✅ Auth Service 100% interno (VPC/rede privada)
5. ✅ Padrão de mercado (NextAuth.js, Auth0, Cognito)

---

## 📚 **Referências**

### **Padrão de Mercado:**
- NextAuth.js: OAuth server-side em Next.js
- Auth0, AWS Cognito, Okta: Backend gerencia OAuth
- OAuth 2.0 Best Practices (RFC 6749, RFC 8252)

### **Política Violada:**
> "Backends devem ser 100% internos e inacessíveis da internet. Apenas BFF é público."

---

## 🎓 **Lição Aprendida**

**Lição:** OAuth flow com redirects HTTP 302 pode parecer simples, mas expõe backends se não implementado server-side.

**Princípio:** BFF deve gerenciar OAuth **server-side**, não fazer proxy de redirects.

**Boa Prática:** Estudar padrões de mercado (NextAuth.js) antes de implementar OAuth.

---

## ⚠️ **NÃO USE ESTES CARDS**

Os cards nesta pasta estão **obsoletos** e **não devem ser usados como referência** para novas implementações.

Use os Cards 021, 022, 023 como referência correta.

---

**Data:** Janeiro 2025  
**Arquiteto:** MoverseMais  
**Motivo:** Segurança e Padrões de Mercado

