# 🚀 Desenvolvimento Local Simplificado - MoverseMais

## 📋 **Visão Geral**

Este documento define uma versão **simplificada** da arquitetura de segurança para desenvolvimento local, mantendo a facilidade de execução que você já tem, mas preparando para produção.

---

## 🎯 **Filosofia: Desenvolvimento vs Produção**

### **Desenvolvimento Local (Agora)**
- ✅ **Simplicidade**: Um comando para subir tudo
- ✅ **Velocidade**: Sem complexidade desnecessária
- ✅ **Debugging**: Fácil de debugar e testar
- ✅ **Flexibilidade**: Mudanças rápidas

### **Produção (Futuro)**
- 🔒 **Segurança**: Todos os padrões implementados
- 🔒 **Monitoramento**: Logs e métricas completas
- 🔒 **Compliance**: LGPD e auditoria
- 🔒 **Escalabilidade**: Preparado para crescimento

---

## 🏗️ **Arquitetura Simplificada para Desenvolvimento**

### **Fluxo Simplificado**
```
┌─────────────┐    ┌─────────────┐    ┌─────────────┐
│   Frontend  │    │   BFF       │    │   Backend   │
│   (React)   │◄──►│   (Next.js) │◄──►│   (Spring)  │
│   Porta: 5173│    │   Porta: 3001│    │   Porta: 8080│
└─────────────┘    └─────────────┘    └─────────────┘
```

### **Segurança Mínima para Desenvolvimento**
1. **JWT simples** (sem refresh token)
2. **Validação básica** de headers
3. **Logs simples** (console.log)
4. **Sem rate limiting** (desenvolvimento)
5. **CORS permissivo** (localhost)

---

## 🚀 **Setup Super Simples**

### **1. Docker Compose Unificado**
```yaml
# docker-compose.dev.yml
version: '3.8'
services:
  # Banco de dados
  postgres:
    image: postgres:15
    environment:
      - POSTGRES_DB=moversemais_dev
      - POSTGRES_USER=developer
      - POSTGRES_PASSWORD=dev123
    ports:
      - "5432:5432"
    volumes:
      - postgres_data:/var/lib/postgresql/data

  # Frontend
  frontend:
    build: ./moversemais-store-front
    ports:
      - "5173:5173"
    environment:
      - VITE_BFF_URL=http://localhost:3001
    depends_on:
      - bff

  # BFF
  bff:
    build: ./moversemais-store-graphql
    ports:
      - "3001:3001"
    environment:
      - BACKEND_URL=http://objective:8080
      - USER_SERVICE_URL=http://user:8083
    depends_on:
      - objective
      - user

  # Objective Service
  objective:
    build: ./moversemais-objective
    ports:
      - "8080:8080"
    environment:
      - SPRING_PROFILES_ACTIVE=docker
      - DATABASE_URL=postgresql://developer:dev123@postgres:5432/moversemais_dev
    depends_on:
      - postgres

  # User Service
  user:
    build: ./moversemais-user
    ports:
      - "8083:8083"
    environment:
      - SPRING_PROFILES_ACTIVE=docker
      - DATABASE_URL=postgresql://developer:dev123@postgres:5432/moversemais_dev
    depends_on:
      - postgres

volumes:
  postgres_data:
```

### **2. Comando Único para Subir Tudo**
```bash
# Criar script simples
# scripts/dev-up.sh
#!/bin/bash
echo "🚀 Subindo MoverseMais em modo desenvolvimento..."
docker-compose -f docker-compose.dev.yml up --build
```

```bash
# Tornar executável
chmod +x scripts/dev-up.sh

# Executar
./scripts/dev-up.sh
```

---

## 🔧 **Implementação Gradual por Serviço**

### **Fase 1: BFF com JWT Simples (Semana 1)**

#### **BFF - Middleware Básico**
```typescript
// middleware/auth-simple.ts
import jwt from 'jsonwebtoken'

const JWT_SECRET = 'dev-secret-key' // Apenas para desenvolvimento

export function simpleAuthMiddleware(request: NextRequest) {
  // Para desenvolvimento, aceitar header X-User-Id diretamente
  const userId = request.headers.get('X-User-Id')
  
  if (!userId) {
    // Em desenvolvimento, criar usuário mock
    const mockUserId = 'dev-user-123'
    request.headers.set('X-User-Id', mockUserId)
    request.headers.set('X-Provider', 'google')
    request.headers.set('X-Request-Id', crypto.randomUUID())
    return NextResponse.next()
  }
  
  // Se tiver JWT, validar
  const token = request.headers.get('authorization')?.replace('Bearer ', '')
  if (token) {
    try {
      const payload = jwt.verify(token, JWT_SECRET)
      request.headers.set('X-User-Id', payload.sub)
      request.headers.set('X-Provider', payload.provider)
    } catch (error) {
      // Em desenvolvimento, continuar mesmo com JWT inválido
      console.warn('JWT inválido, continuando com usuário mock')
    }
  }
  
  return NextResponse.next()
}
```

#### **BFF - Configuração Simplificada**
```typescript
// next.config.js
/** @type {import('next').NextConfig} */
const nextConfig = {
  async middleware(request) {
    // Aplicar apenas em rotas da API
    if (request.nextUrl.pathname.startsWith('/api/')) {
      return simpleAuthMiddleware(request)
    }
  }
}

module.exports = nextConfig
```

### **Fase 2: Microserviços com Validação Básica (Semana 2)**

#### **User Service - Interceptor Simples**
```kotlin
// Interceptor simplificado para desenvolvimento
@Component
class DevSecurityInterceptor : HandlerInterceptor {
    
    override fun preHandle(request: HttpServletRequest, response: HttpServletResponse, handler: Any): Boolean {
        // Em desenvolvimento, apenas logar a requisição
        val userId = request.getHeader("X-User-Id") ?: "dev-user-123"
        val endpoint = request.requestURI
        
        println("🔍 [DEV] Acesso: $endpoint - Usuário: $userId")
        
        // Adicionar usuário mock se não existir
        if (request.getHeader("X-User-Id").isNullOrBlank()) {
            request.setAttribute("userId", "dev-user-123")
            request.setAttribute("provider", "google")
        }
        
        return true
    }
}
```

#### **Objective Service - Validação Básica**
```kotlin
// Validação simplificada para desenvolvimento
@Service
class DevObjectiveSecurityService {
    
    fun validateUserAccess(objectiveId: UUID, userId: String): Objective {
        val objective = objectiveRepository.findById(objectiveId)
            .orElseThrow { ObjectiveNotFoundException("Objective not found") }
        
        // Em desenvolvimento, permitir acesso se for usuário mock
        if (userId == "dev-user-123" || objective.userId == UUID.fromString(userId)) {
            return objective
        }
        
        throw AuthorizationException("Access denied")
    }
}
```

### **Fase 3: Frontend com Autenticação Mock (Semana 3)**

#### **Frontend - Auth Mock para Desenvolvimento**
```typescript
// hooks/useAuth-dev.ts
export const useAuth = () => {
  const [user, setUser] = useState(null)
  const [loading, setLoading] = useState(false)
  
  useEffect(() => {
    // Em desenvolvimento, sempre considerar logado
    setUser({
      id: 'dev-user-123',
      email: 'dev@moversemais.com',
      name: 'Usuário Desenvolvimento',
      provider: 'google'
    })
    setLoading(false)
  }, [])
  
  return {
    user,
    loading,
    isAuthenticated: true
  }
}
```

#### **Frontend - Configuração de Desenvolvimento**
```typescript
// config/dev.ts
export const DEV_CONFIG = {
  AUTH_ENABLED: false, // Desabilitar auth em desenvolvimento
  MOCK_USER: {
    id: 'dev-user-123',
    email: 'dev@moversemais.com',
    name: 'Usuário Desenvolvimento'
  },
  BFF_URL: 'http://localhost:3001'
}
```

---

## 📋 **Guia Passo a Passo por Serviço**

### **1. Frontend (moversemais-store-front)**

#### **Passo 1: Instalar dependências**
```bash
cd moversemais-store-front
npm install
```

#### **Passo 2: Configurar variáveis de ambiente**
```bash
# .env.development
VITE_BFF_URL=http://localhost:3001
VITE_AUTH_ENABLED=false
VITE_MOCK_USER_ID=dev-user-123
```

#### **Passo 3: Implementar auth mock**
```typescript
// src/hooks/useAuth.ts
export const useAuth = () => {
  if (import.meta.env.VITE_AUTH_ENABLED === 'false') {
    return {
      user: { id: 'dev-user-123', email: 'dev@moversemais.com' },
      loading: false,
      isAuthenticated: true
    }
  }
  
  // Implementação real de auth aqui
}
```

#### **Passo 4: Executar**
```bash
npm run dev
```

### **2. BFF (moversemais-store-graphql)**

#### **Passo 1: Instalar dependências**
```bash
cd moversemais-store-graphql
npm install
```

#### **Passo 2: Configurar variáveis de ambiente**
```bash
# .env.local
BACKEND_URL=http://localhost:8080
USER_SERVICE_URL=http://localhost:8083
JWT_SECRET=dev-secret-key
AUTH_ENABLED=false
```

#### **Passo 3: Implementar middleware simples**
```typescript
// middleware/auth-simple.ts (criar arquivo)
export function simpleAuthMiddleware(request: NextRequest) {
  // Implementação simplificada acima
}
```

#### **Passo 4: Executar**
```bash
npm run dev
```

### **3. Objective Service (moversemais-objective)**

#### **Passo 1: Configurar perfil de desenvolvimento**
```yaml
# src/main/resources/application-dev.yml
spring:
  profiles:
    active: dev
  datasource:
    url: jdbc:postgresql://localhost:5432/moversemais_dev
    username: developer
    password: dev123
  jpa:
    hibernate:
      ddl-auto: update

# Configurações de desenvolvimento
dev:
  security:
    enabled: false
  mock:
    user-id: dev-user-123
```

#### **Passo 2: Implementar interceptor de desenvolvimento**
```kotlin
// src/main/kotlin/com/moversemais/objective/config/DevSecurityConfig.kt
@Configuration
@Profile("dev")
class DevSecurityConfig {
    
    @Bean
    fun devSecurityInterceptor(): DevSecurityInterceptor {
        return DevSecurityInterceptor()
    }
}
```

#### **Passo 3: Executar**
```bash
./gradlew bootRun --args='--spring.profiles.active=dev'
```

### **4. User Service (moversemais-user)**

#### **Passo 1: Configurar perfil de desenvolvimento**
```yaml
# src/main/resources/application-dev.yml
spring:
  profiles:
    active: dev
  datasource:
    url: jdbc:postgresql://localhost:5432/moversemais_dev
    username: developer
    password: dev123

dev:
  security:
    enabled: false
```

#### **Passo 2: Implementar interceptor de desenvolvimento**
```kotlin
// Mesmo padrão do Objective Service
```

#### **Passo 3: Executar**
```bash
./gradlew bootRun --args='--spring.profiles.active=dev'
```

---

## 🎯 **Scripts de Desenvolvimento**

### **1. Script para Subir Tudo**
```bash
#!/bin/bash
# scripts/dev-up.sh

echo "🚀 Subindo MoverseMais em modo desenvolvimento..."

# Subir banco
echo "📊 Subindo PostgreSQL..."
docker-compose -f docker-compose.dev.yml up -d postgres

# Aguardar banco
echo "⏳ Aguardando banco..."
sleep 10

# Subir serviços
echo "🔧 Subindo serviços..."
cd moversemais-objective && ./gradlew bootRun --args='--spring.profiles.active=dev' &
cd moversemais-user && ./gradlew bootRun --args='--spring.profiles.active=dev' &
cd moversemais-store-graphql && npm run dev &
cd moversemais-store-front && npm run dev &

echo "✅ Todos os serviços subindo!"
echo "🌐 Frontend: http://localhost:5173"
echo "🔗 BFF: http://localhost:3001"
echo "🎯 Objective: http://localhost:8080"
echo "👤 User: http://localhost:8083"
```

### **2. Script para Parar Tudo**
```bash
#!/bin/bash
# scripts/dev-down.sh

echo "🛑 Parando MoverseMais..."

# Parar processos
pkill -f "gradlew bootRun"
pkill -f "npm run dev"

# Parar containers
docker-compose -f docker-compose.dev.yml down

echo "✅ Todos os serviços parados!"
```

### **3. Script para Reset do Banco**
```bash
#!/bin/bash
# scripts/dev-reset-db.sh

echo "🗑️ Resetando banco de dados..."

# Parar serviços
./scripts/dev-down.sh

# Remover volume do banco
docker volume rm moversemais_postgres_data

# Subir novamente
./scripts/dev-up.sh

echo "✅ Banco resetado!"
```

---

## 🔄 **Migração Gradual para Produção**

### **Fase 1: Desenvolvimento (Agora)**
- ✅ Auth mock
- ✅ Validação básica
- ✅ Logs simples
- ✅ CORS permissivo

### **Fase 2: Staging (Próximo)**
- 🔄 JWT real
- 🔄 Validação de headers
- 🔄 Logs estruturados
- 🔄 CORS restritivo

### **Fase 3: Produção (Futuro)**
- 🔒 OAuth completo
- 🔒 Rate limiting
- 🔒 Monitoramento
- 🔒 Compliance LGPD

---

## 📋 **Checklist de Implementação Simplificada**

### **Semana 1: BFF**
- [ ] Configurar middleware simples
- [ ] Adicionar perfil de desenvolvimento
- [ ] Testar comunicação com frontend

### **Semana 2: Microserviços**
- [ ] Adicionar interceptors de desenvolvimento
- [ ] Configurar perfis de desenvolvimento
- [ ] Testar comunicação BFF → Backend

### **Semana 3: Frontend**
- [ ] Implementar auth mock
- [ ] Configurar variáveis de ambiente
- [ ] Testar fluxo completo

### **Semana 4: Scripts e Documentação**
- [ ] Criar scripts de desenvolvimento
- [ ] Documentar comandos
- [ ] Testar setup completo

---

## 🚨 **Troubleshooting para Desenvolvimento**

### **Problema: Serviço não sobe**
```bash
# Verificar se porta está em uso
lsof -ti:8080
kill -9 $(lsof -ti:8080)

# Verificar logs
tail -f logs/application.log
```

### **Problema: Banco não conecta**
```bash
# Verificar se PostgreSQL está rodando
docker ps | grep postgres

# Resetar banco
./scripts/dev-reset-db.sh
```

### **Problema: CORS no frontend**
```typescript
// Adicionar no BFF para desenvolvimento
const corsOptions = {
  origin: ['http://localhost:5173'],
  credentials: true
}
```

---

**Última Atualização**: Outubro 2025  
**Versão**: 1.0  
**Mantenedor**: Equipe MoverseMais

---

*Este documento foca na simplicidade para desenvolvimento local, mantendo a base para evolução para produção.*
