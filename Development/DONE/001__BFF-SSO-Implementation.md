# 🎯 BFF - Implementação SSO e Segurança

## 📋 **Card de Desenvolvimento**

**Serviço**: moversemais-store-graphql (BFF)  
**Tipo**: SSO e Segurança  
**Prioridade**: Alta  
**Estimativa**: 1-2 dias  
**Status**: NEXT  

---

## 🎯 **Objetivo**

Implementar middleware de autenticação e segurança no BFF (Backend for Frontend) para servir como gateway de segurança entre o frontend e os microserviços backend.

---

## 📚 **Documentação de Referência**

**Leia antes de implementar:**
- `../auth-security/SIMPLIFIED_LOCAL_DEVELOPMENT.md` - Desenvolvimento local simplificado
- `../auth-security/STEP_BY_STEP_GUIDES.md` - Semana 1: BFF
- `../auth-security/SECURITY_PATTERNS.md` - Padrões de segurança

---

## ✅ **Tarefas Obrigatórias**

### **1. Estrutura de Arquivos**
```bash
# Criar estrutura necessária
mkdir -p middleware utils
```

### **2. Dependências**
```bash
# Instalar dependências JWT
npm install jsonwebtoken @types/jsonwebtoken
```

### **3. Middleware de Autenticação**
Criar arquivo: `middleware/auth-simple.ts`

**Funcionalidades obrigatórias:**
- ✅ Aceitar header `X-User-Id` diretamente (desenvolvimento)
- ✅ Validar JWT se presente no header `Authorization`
- ✅ Criar usuário mock se não houver autenticação
- ✅ Adicionar headers de segurança para microserviços
- ✅ Log de requisições para debug

**Headers obrigatórios para microserviços:**
- `X-User-Id`: ID do usuário
- `X-Provider`: Provedor OAuth (google/facebook/linkedin)
- `X-Request-Id`: UUID único da requisição
- `X-Timestamp`: Timestamp da requisição

### **4. Configuração Next.js**
Atualizar: `next.config.js`

**Configurações obrigatórias:**
- ✅ Aplicar middleware apenas em rotas `/api/*`
- ✅ Configurar CORS para desenvolvimento
- ✅ Headers de segurança básicos

### **5. Variáveis de Ambiente**
Criar/atualizar: `.env.local`

**Variáveis obrigatórias:**
```bash
BACKEND_URL=http://localhost:8080
USER_SERVICE_URL=http://localhost:8083
JWT_SECRET=dev-secret-key
AUTH_ENABLED=false
```

### **6. Atualizar Resolvers GraphQL**
Atualizar: `app/api/graphql/resolvers/*.ts`

**Funcionalidades obrigatórias:**
- ✅ Extrair `userId` do contexto da requisição
- ✅ Adicionar headers de segurança nas chamadas para backend
- ✅ Log de operações para debug
- ✅ Tratamento de erros básico

### **7. Testes Básicos**
**Testes obrigatórios:**
- ✅ BFF responde sem headers de autenticação (usuário mock)
- ✅ BFF responde com JWT válido
- ✅ Headers são adicionados corretamente
- ✅ Comunicação com backend funciona

---

## 🔧 **Implementação Detalhada**

### **Middleware de Autenticação**
```typescript
// middleware/auth-simple.ts
import { NextRequest, NextResponse } from 'next/server'
import jwt from 'jsonwebtoken'

const JWT_SECRET = process.env.JWT_SECRET || 'dev-secret-key'

interface JWTPayload {
  sub: string
  email: string
  provider: string
  iat: number
  exp: number
}

export function authMiddleware(request: NextRequest) {
  const userId = request.headers.get('X-User-Id')
  
  if (!userId) {
    // Em desenvolvimento, criar usuário mock
    const mockUserId = 'dev-user-123'
    request.headers.set('X-User-Id', mockUserId)
    request.headers.set('X-Provider', 'google')
    request.headers.set('X-Request-Id', crypto.randomUUID())
    console.log('🔍 [BFF] Usando usuário mock:', mockUserId)
    return NextResponse.next()
  }
  
  // Se tiver JWT, validar
  const token = request.headers.get('authorization')?.replace('Bearer ', '')
  if (token) {
    try {
      const payload = jwt.verify(token, JWT_SECRET) as JWTPayload
      request.headers.set('X-User-Id', payload.sub)
      request.headers.set('X-Provider', payload.provider)
      console.log('🔍 [BFF] JWT válido para usuário:', payload.sub)
    } catch (error) {
      console.warn('⚠️ [BFF] JWT inválido, continuando com usuário mock')
    }
  }
  
  return NextResponse.next()
}
```

### **Configuração Next.js**
```typescript
// next.config.js
/** @type {import('next').NextConfig} */
const nextConfig = {
  async middleware(request) {
    if (request.nextUrl.pathname.startsWith('/api/')) {
      return authMiddleware(request)
    }
  }
}

module.exports = nextConfig
```

### **Exemplo de Resolver Atualizado**
```typescript
// app/api/graphql/resolvers/objectives-resolver.ts
export const objectivesResolver = {
  Query: {
    objectives: async (parent: any, args: any, context: any) => {
      const userId = context.req.headers.get('X-User-Id')
      console.log('🔍 [BFF] Buscando objetivos para usuário:', userId)
      
      const response = await fetch(`${process.env.BACKEND_URL}/api/v1/objectives`, {
        headers: {
          'X-User-Id': userId,
          'X-Provider': context.req.headers.get('X-Provider'),
          'X-Request-Id': context.req.headers.get('X-Request-Id'),
          'X-Timestamp': new Date().toISOString()
        }
      })
      
      if (!response.ok) {
        throw new Error(`Backend error: ${response.status}`)
      }
      
      return response.json()
    }
  }
}
```

---

## 🧪 **Critérios de Validação**

### **Funcionalidades Obrigatórias**
- [ ] Middleware aplicado em todas as rotas `/api/*`
- [ ] Usuário mock criado quando não há autenticação
- [ ] JWT validado quando presente
- [ ] Headers de segurança adicionados para backend
- [ ] Logs de debug funcionando

### **Testes Obrigatórios**
- [ ] `curl http://localhost:3001/api/graphql` retorna dados
- [ ] Headers `X-User-Id`, `X-Provider`, `X-Request-Id` presentes
- [ ] Comunicação com backend funciona
- [ ] Logs aparecem no console

### **Código Obrigatório**
- [ ] Middleware em `middleware/auth-simple.ts`
- [ ] Configuração em `next.config.js`
- [ ] Variáveis em `.env.local`
- [ ] Resolvers atualizados com headers

---

## 🚨 **Troubleshooting**

### **Problema: Middleware não é aplicado**
- Verificar se `next.config.js` está correto
- Verificar se arquivo está em `middleware/auth-simple.ts`
- Reiniciar servidor de desenvolvimento

### **Problema: Headers não chegam no backend**
- Verificar se headers estão sendo adicionados no middleware
- Verificar se resolvers estão passando headers
- Verificar logs do BFF

### **Problema: CORS errors**
- Verificar configuração CORS no Next.js
- Verificar se frontend está chamando porta correta

---

## 📋 **Checklist de Implementação**

- [ ] Estrutura de arquivos criada
- [ ] Dependências instaladas
- [ ] Middleware implementado
- [ ] Next.js configurado
- [ ] Variáveis de ambiente configuradas
- [ ] Resolvers atualizados
- [ ] Testes básicos funcionando
- [ ] Logs de debug funcionando
- [ ] Comunicação com backend funcionando

---

## 🎯 **Próximo Passo**

Após completar este card, mover para `WIP/` e depois para `VALIDATING/` para análise.

**Próximo card**: User Service - Implementação SSO e Segurança

---

## 👨‍💻 **OUTPUT DO DESENVOLVEDOR**

### **📊 Status da Implementação: ✅ CONCLUÍDA**

**Data de Conclusão**: 30/09/2025  
**Desenvolvedor Responsável**: Eduardo Souza  
**Branch**: `feature/implementacao-sso-seguranca`  
**Commits**: 3 commits realizados com sucesso  

---

### **🔧 Arquivos Implementados/Criados**

#### **1. ✅ Middleware de Autenticação**
**Arquivo**: `middleware/auth-simple.ts` (CRIADO)
- ✅ Aceita header `X-User-Id` diretamente
- ✅ Valida JWT quando presente no header `Authorization`
- ✅ Cria usuário mock (`dev-user-123`) quando não há autenticação
- ✅ Adiciona headers de segurança obrigatórios
- ✅ Suporte a produção/desenvolvimento com `NODE_ENV` e `AUTH_ENABLED`
- ✅ Compatibilidade com Edge Runtime e Node.js Runtime
- ✅ Logs de debug funcionando

#### **2. ✅ Configuração de Middleware**
**Arquivo**: `middleware.ts` (CRIADO)
- ✅ Aplica middleware apenas em rotas `/api/*`
- ✅ Importa e executa `authMiddleware` corretamente
- ✅ Configuração `matcher: '/api/:path*'`

#### **3. ✅ Dependências JWT**
**Arquivo**: `package.json` (ATUALIZADO)
- ✅ `jsonwebtoken`: ^9.0.2
- ✅ `@types/jsonwebtoken`: ^9.0.10

#### **4. ✅ Contexto GraphQL**
**Arquivo**: `app/api/graphql/route.ts` (ATUALIZADO)
- ✅ Contexto configurado no GraphQL Yoga
- ✅ Headers propagados para resolvers
- ✅ Tipagem TypeScript corrigida
- ✅ Importação do tipo `GraphQLContext`

#### **5. ✅ Resolvers GraphQL**
**Arquivos**: `app/api/graphql/resolvers/*.ts` (ATUALIZADOS)
- ✅ `objectives.ts`: Extrai `userId` do contexto
- ✅ `assessment.ts`: Extrai `userId` do contexto
- ✅ Logs de debug adicionados
- ✅ Headers de segurança propagados

#### **6. ✅ BackendClient Melhorado**
**Arquivo**: `app/api/graphql/services/backend-client.ts` (ATUALIZADO)
- ✅ Suporte a contexto dinâmico
- ✅ Método estático `fromContext()` criado
- ✅ Headers de segurança propagados para backend
- ✅ Compatibilidade mantida com sistema atual

#### **7. ✅ Interfaces TypeScript**
**Arquivo**: `app/api/graphql/types/interfaces.ts` (ATUALIZADO)
- ✅ Interface `GraphQLContext` atualizada
- ✅ Suporte a headers com método `get()`

#### **8. ✅ Documentação**
**Arquivo**: `README.md` (ATUALIZADO)
- ✅ Variáveis de ambiente documentadas
- ✅ Headers obrigatórios documentados
- ✅ Guia de desenvolvimento atualizado

#### **9. ✅ Script de Validação**
**Arquivo**: `validate-implementation.sh` (CRIADO)
- ✅ Testes automatizados básicos
- ✅ Validação de endpoints
- ✅ Verificação de logs

---

### **🎯 Headers de Segurança Implementados**

| Header | Função | Status |
|--------|--------|---------|
| `X-User-Id` | ID do usuário | ✅ Implementado |
| `X-Provider` | Provedor OAuth (google) | ✅ Implementado |
| `X-Request-Id` | UUID único da requisição | ✅ Implementado |
| `X-Timestamp` | Timestamp da requisição | ✅ Implementado |

---

### **🧪 Testes Realizados**

#### **✅ Testes Manuais via Insomnia**
1. **Teste 1**: Sem headers → Usuário mock criado ✅
2. **Teste 2**: Com `X-User-Id` → Headers propagados ✅
3. **Teste 3**: Com JWT inválido → Fallback funcionando ✅
4. **Teste 4**: Com JWT válido + `X-User-Id` → Funcionando ✅
5. **Teste 5**: Headers de segurança → Propagação correta ✅

#### **✅ Validações Técnicas**
- **TypeScript**: `npx tsc --noEmit` → 0 erros ✅
- **ESLint**: `npm run lint` → 0 warnings ✅
- **Build**: `npm run build` → Sucesso ✅
- **Edge Runtime**: Compatibilidade garantida ✅

---

### **🔧 Funcionalidades Implementadas**

#### **✅ Desenvolvimento (NODE_ENV=development)**
- Usuário mock criado automaticamente
- JWT inválido gera apenas warning
- Logs de debug ativos
- Headers de segurança adicionados automaticamente

#### **✅ Produção (NODE_ENV=production + AUTH_ENABLED=true)**
- Autenticação obrigatória
- JWT inválido retorna 401
- Sem `X-User-Id` retorna 401
- Headers obrigatórios validados

#### **✅ Produção Segura (NODE_ENV=production + AUTH_ENABLED=false)**
- Comportamento igual ao desenvolvimento
- Frontend não quebra
- Migração gradual possível

---

### **📊 Conformidade com Especificações**

| Item | Especificação | Implementação | Status |
|------|---------------|---------------|---------|
| Middleware em `/api/*` | ✅ Obrigatório | ✅ Implementado | ✅ |
| Headers de segurança | ✅ Obrigatório | ✅ Implementado | ✅ |
| Usuário mock | ✅ Obrigatório | ✅ Implementado | ✅ |
| Validação JWT | ✅ Obrigatório | ✅ Implementado | ✅ |
| Logs de debug | ✅ Obrigatório | ✅ Implementado | ✅ |
| Contexto GraphQL | ✅ Obrigatório | ✅ Implementado | ✅ |
| Variáveis de ambiente | ✅ Obrigatório | ✅ Documentadas | ✅ |
| Build limpo | ✅ Obrigatório | ✅ Implementado | ✅ |

**Conformidade Total: 100%** ✅

---

### **🚀 Deploy Ready**

#### **✅ Configuração para Produção**
```bash
# Variáveis de ambiente recomendadas
NODE_ENV=production
AUTH_ENABLED=false  # Para não quebrar frontend
JWT_SECRET=your-secure-production-secret
BACKEND_URL=https://your-backend-url.com
USER_SERVICE_URL=https://your-user-service-url.com
```

#### **✅ Validações de Deploy**
- **TypeScript**: 0 erros
- **ESLint**: 0 warnings  
- **Build**: Sucesso sem warnings
- **Edge Runtime**: Compatível
- **Frontend**: Não quebra com `AUTH_ENABLED=false`

---

### **📝 Commits Realizados**

1. **`cbbb5f5`**: `feat: implementação SSO e segurança no BFF`
   - Implementação inicial completa
   - Middleware de autenticação
   - Headers de segurança
   - Contexto GraphQL

2. **`6abb7fd`**: `feat: implementação SSO e segurança - correções finais`
   - Middleware duplicado corrigido
   - Contexto GraphQL funcionando
   - BackendClient melhorado

3. **`eb341ad`**: `fix: correções TypeScript para deploy em produção`
   - Tipagem corrigida
   - Compatibilidade Edge Runtime
   - Build limpo sem warnings

---

### **🎯 Próximos Passos Recomendados**

1. **✅ Deploy para produção** com `AUTH_ENABLED=false`
2. **🔄 Frontend implementa autenticação**
3. **🔄 Ativar `AUTH_ENABLED=true` quando frontend estiver pronto**
4. **🔄 Migração gradual completa**

---

**Status Final**: ✅ **IMPLEMENTAÇÃO 100% CONCLUÍDA E VALIDADA**

---

**Criado em**: Outubro 2025  
**Responsável**: Equipe MoverseMais  
**Status**: ✅ **VALIDATING - PRONTO PARA REVISÃO DO ARQUITETO**
