# 📋 Guias Passo a Passo - MoverseMais

## 📋 **Visão Geral**

Este documento contém guias passo a passo específicos para implementar autenticação e segurança em cada serviço da plataforma MoverseMais.

---

## 🎯 **Ordem de Implementação Recomendada**

1. **BFF (moversemais-store-graphql)** - Semana 1
2. **User Service (moversemais-user)** - Semana 2  
3. **Objective Service (moversemais-objective)** - Semana 3
4. **Frontend (moversemais-store-front)** - Semana 4
5. **Auth Service (novo)** - Semana 5

---

## 🔧 **1. BFF (moversemais-store-graphql) - Semana 1**

### **Passo 1: Preparar Estrutura**
```bash
cd moversemais-store-graphql
mkdir -p middleware utils
```

### **Passo 2: Instalar Dependências**
```bash
npm install jsonwebtoken @types/jsonwebtoken
```

### **Passo 3: Criar Middleware de Autenticação**
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
      const payload = jwt.verify(token, JWT_SECRET) as JWTPayload
      request.headers.set('X-User-Id', payload.sub)
      request.headers.set('X-Provider', payload.provider)
    } catch (error) {
      console.warn('JWT inválido, continuando com usuário mock')
    }
  }
  
  return NextResponse.next()
}
```

### **Passo 4: Configurar Next.js**
```typescript
// next.config.js
/** @type {import('next').NextConfig} */
const nextConfig = {
  async middleware(request) {
    // Aplicar apenas em rotas da API
    if (request.nextUrl.pathname.startsWith('/api/')) {
      return authMiddleware(request)
    }
  }
}

module.exports = nextConfig
```

### **Passo 5: Atualizar Resolvers**
```typescript
// app/api/graphql/resolvers/objectives-resolver.ts
export const objectivesResolver = {
  Query: {
    objectives: async (parent: any, args: any, context: any) => {
      // Context já tem userId do middleware
      const userId = context.req.headers.get('X-User-Id')
      console.log('🔍 [BFF] Buscando objetivos para usuário:', userId)
      
      // Chamar backend com headers de segurança
      const response = await fetch(`${process.env.BACKEND_URL}/api/v1/objectives`, {
        headers: {
          'X-User-Id': userId,
          'X-Provider': context.req.headers.get('X-Provider'),
          'X-Request-Id': context.req.headers.get('X-Request-Id')
        }
      })
      
      return response.json()
    }
  }
}
```

### **Passo 6: Configurar Variáveis de Ambiente**
```bash
# .env.local
BACKEND_URL=http://localhost:8080
USER_SERVICE_URL=http://localhost:8083
JWT_SECRET=dev-secret-key
AUTH_ENABLED=false
```

### **Passo 7: Testar**
```bash
npm run dev
# Acessar: http://localhost:3001/api/graphql
```

---

## 🔧 **2. User Service (moversemais-user) - Semana 2**

### **Passo 1: Adicionar Dependências**
```kotlin
// build.gradle.kts
dependencies {
    // Adicionar dependências de segurança
    implementation("org.springframework.boot:spring-boot-starter-security")
    implementation("io.jsonwebtoken:jjwt-api:0.11.5")
    implementation("io.jsonwebtoken:jjwt-impl:0.11.5")
    implementation("io.jsonwebtoken:jjwt-jackson:0.11.5")
}
```

### **Passo 2: Criar Interceptor de Segurança**
```kotlin
// src/main/kotlin/com/moversemais/user/config/SecurityConfig.kt
@Configuration
@EnableWebSecurity
class SecurityConfig {
    
    @Bean
    fun securityFilterChain(http: HttpSecurity): SecurityFilterChain {
        return http
            .csrf { it.disable() }
            .sessionManagement { it.sessionCreationPolicy(SessionCreationPolicy.STATELESS) }
            .addFilterBefore(securityInterceptor(), UsernamePasswordAuthenticationFilter::class.java)
            .build()
    }
    
    @Bean
    fun securityInterceptor(): SecurityInterceptor {
        return SecurityInterceptor()
    }
}
```

### **Passo 3: Implementar Interceptor**
```kotlin
// src/main/kotlin/com/moversemais/user/interceptor/SecurityInterceptor.kt
@Component
class SecurityInterceptor : HandlerInterceptor {
    
    private val logger = LoggerFactory.getLogger(SecurityInterceptor::class.java)
    
    override fun preHandle(request: HttpServletRequest, response: HttpServletResponse, handler: Any): Boolean {
        try {
            // Validar headers obrigatórios
            val userId = request.getHeader("X-User-Id")
            val provider = request.getHeader("X-Provider")
            val requestId = request.getHeader("X-Request-Id")
            
            if (userId.isNullOrBlank()) {
                throw SecurityException("Missing X-User-Id header")
            }
            
            // Em desenvolvimento, aceitar usuário mock
            if (userId == "dev-user-123") {
                logger.info("🔍 [DEV] Acesso permitido para usuário mock: $userId")
                return true
            }
            
            // Validar se usuário existe
            validateUser(userId)
            
            // Log de auditoria
            logger.info("🔍 [USER] Acesso: ${request.requestURI} - Usuário: $userId")
            
            return true
        } catch (e: Exception) {
            logger.warn("❌ [USER] Acesso negado: ${e.message}")
            response.status = HttpStatus.UNAUTHORIZED.value()
            response.writer.write("""{"error":"AUTH_001","message":"Access denied"}""")
            return false
        }
    }
    
    private fun validateUser(userId: String) {
        // Implementar validação de usuário
        // Por enquanto, apenas logar
        logger.info("🔍 [USER] Validando usuário: $userId")
    }
}
```

### **Passo 4: Atualizar Controller**
```kotlin
// src/main/kotlin/com/moversemais/user/controller/UserController.kt
@RestController
@RequestMapping("/api/v1/users")
class UserController {
    
    @GetMapping("/me")
    fun getCurrentUser(
        @RequestHeader("X-User-Id") userId: String,
        @RequestHeader("X-Provider") provider: String
    ): UserResponse {
        logger.info("🔍 [USER] Buscando dados do usuário: $userId")
        
        // Em desenvolvimento, retornar dados mock
        if (userId == "dev-user-123") {
            return UserResponse(
                id = userId,
                email = "dev@moversemais.com",
                name = "Usuário Desenvolvimento",
                provider = provider,
                createdAt = Instant.now(),
                lastAccessAt = Instant.now()
            )
        }
        
        // Implementar busca real do usuário
        return userService.findById(userId)
    }
    
    @PostMapping
    fun createUser(@RequestBody @Valid request: CreateUserRequest): UserResponse {
        logger.info("🔍 [USER] Criando usuário: ${request.email}")
        
        val user = User(
            id = UUID.randomUUID(),
            email = request.email,
            name = request.name,
            provider = request.provider,
            providerId = request.providerId
        )
        
        return userService.save(user)
    }
}
```

### **Passo 5: Configurar Perfil de Desenvolvimento**
```yaml
# src/main/resources/application-dev.yml
spring:
  profiles:
    active: dev
  datasource:
    url: jdbc:postgresql://localhost:5432/moversemais_dev
    username: developer
    password: dev123

# Configurações de desenvolvimento
dev:
  security:
    enabled: false
  mock:
    user-id: dev-user-123
```

### **Passo 6: Testar**
```bash
./gradlew bootRun --args='--spring.profiles.active=dev'
# Testar: http://localhost:8083/api/v1/users/me
```

---

## 🔧 **3. Objective Service (moversemais-objective) - Semana 3**

### **Passo 1: Adicionar Dependências**
```kotlin
// build.gradle.kts
dependencies {
    // Adicionar dependências de segurança
    implementation("org.springframework.boot:spring-boot-starter-security")
    implementation("io.jsonwebtoken:jjwt-api:0.11.5")
    implementation("io.jsonwebtoken:jjwt-impl:0.11.5")
    implementation("io.jsonwebtoken:jjwt-jackson:0.11.5")
}
```

### **Passo 2: Criar Interceptor de Segurança**
```kotlin
// src/main/kotlin/com/moversemais/objective/config/SecurityConfig.kt
@Configuration
@EnableWebSecurity
class SecurityConfig {
    
    @Bean
    fun securityFilterChain(http: HttpSecurity): SecurityFilterChain {
        return http
            .csrf { it.disable() }
            .sessionManagement { it.sessionCreationPolicy(SessionCreationPolicy.STATELESS) }
            .addFilterBefore(securityInterceptor(), UsernamePasswordAuthenticationFilter::class.java)
            .build()
    }
    
    @Bean
    fun securityInterceptor(): SecurityInterceptor {
        return SecurityInterceptor()
    }
}
```

### **Passo 3: Implementar Interceptor**
```kotlin
// src/main/kotlin/com/moversemais/objective/interceptor/SecurityInterceptor.kt
@Component
class SecurityInterceptor : HandlerInterceptor {
    
    private val logger = LoggerFactory.getLogger(SecurityInterceptor::class.java)
    
    override fun preHandle(request: HttpServletRequest, response: HttpServletResponse, handler: Any): Boolean {
        try {
            // Validar headers obrigatórios
            val userId = request.getHeader("X-User-Id")
            val provider = request.getHeader("X-Provider")
            val requestId = request.getHeader("X-Request-Id")
            
            if (userId.isNullOrBlank()) {
                throw SecurityException("Missing X-User-Id header")
            }
            
            // Em desenvolvimento, aceitar usuário mock
            if (userId == "dev-user-123") {
                logger.info("🔍 [DEV] Acesso permitido para usuário mock: $userId")
                return true
            }
            
            // Log de auditoria
            logger.info("🔍 [OBJECTIVE] Acesso: ${request.requestURI} - Usuário: $userId")
            
            return true
        } catch (e: Exception) {
            logger.warn("❌ [OBJECTIVE] Acesso negado: ${e.message}")
            response.status = HttpStatus.UNAUTHORIZED.value()
            response.writer.write("""{"error":"AUTH_001","message":"Access denied"}""")
            return false
        }
    }
}
```

### **Passo 4: Atualizar Controller**
```kotlin
// src/main/kotlin/com/moversemais/objective/controller/ObjectiveController.kt
@RestController
@RequestMapping("/api/v1/objectives")
class ObjectiveController {
    
    @GetMapping
    fun getObjectives(
        @RequestHeader("X-User-Id") userId: String,
        @RequestParam(required = false) status: String?,
        @RequestParam(required = false) type: String?
    ): List<Objective> {
        logger.info("🔍 [OBJECTIVE] Buscando objetivos para usuário: $userId")
        
        // Em desenvolvimento, retornar objetivos mock
        if (userId == "dev-user-123") {
            return listOf(
                Objective(
                    id = UUID.randomUUID(),
                    title = "Objetivo Mock 1",
                    description = "Descrição do objetivo mock",
                    status = ObjectiveStatus.ACTIVE,
                    userId = UUID.fromString(userId)
                )
            )
        }
        
        // Implementar busca real dos objetivos
        return objectiveService.findByUserId(userId, status, type)
    }
    
    @PostMapping
    fun createObjective(
        @RequestHeader("X-User-Id") userId: String,
        @RequestBody @Valid request: CreateObjectiveRequest
    ): Objective {
        logger.info("🔍 [OBJECTIVE] Criando objetivo para usuário: $userId")
        
        val objective = Objective(
            id = UUID.randomUUID(),
            title = request.title,
            description = request.description,
            status = ObjectiveStatus.ACTIVE,
            userId = UUID.fromString(userId)
        )
        
        return objectiveService.save(objective)
    }
}
```

### **Passo 5: Configurar Perfil de Desenvolvimento**
```yaml
# src/main/resources/application-dev.yml
spring:
  profiles:
    active: dev
  datasource:
    url: jdbc:postgresql://localhost:5432/moversemais_dev
    username: developer
    password: dev123

# Configurações de desenvolvimento
dev:
  security:
    enabled: false
  mock:
    user-id: dev-user-123
```

### **Passo 6: Testar**
```bash
./gradlew bootRun --args='--spring.profiles.active=dev'
# Testar: http://localhost:8080/api/v1/objectives
```

---

## 🔧 **4. Frontend (moversemais-store-front) - Semana 4**

### **Passo 1: Criar Hook de Autenticação Mock**
```typescript
// src/hooks/useAuth.ts
import { useState, useEffect } from 'react'

interface User {
  id: string
  email: string
  name: string
  provider: string
}

export const useAuth = () => {
  const [user, setUser] = useState<User | null>(null)
  const [loading, setLoading] = useState(true)
  
  useEffect(() => {
    // Em desenvolvimento, sempre considerar logado
    if (import.meta.env.VITE_AUTH_ENABLED === 'false') {
      setUser({
        id: 'dev-user-123',
        email: 'dev@moversemais.com',
        name: 'Usuário Desenvolvimento',
        provider: 'google'
      })
      setLoading(false)
    } else {
      // Implementação real de auth aqui
      setLoading(false)
    }
  }, [])
  
  return {
    user,
    loading,
    isAuthenticated: !!user
  }
}
```

### **Passo 2: Criar Componente de Proteção de Rotas**
```typescript
// src/components/ProtectedRoute.tsx
import { useAuth } from '../hooks/useAuth'
import { LoadingSpinner } from './LoadingSpinner'

interface ProtectedRouteProps {
  children: React.ReactNode
}

export const ProtectedRoute: React.FC<ProtectedRouteProps> = ({ children }) => {
  const { loading, isAuthenticated } = useAuth()
  
  if (loading) {
    return <LoadingSpinner />
  }
  
  if (!isAuthenticated) {
    return (
      <div className="flex items-center justify-center min-h-screen">
        <div className="text-center">
          <h2 className="text-2xl font-bold mb-4">Acesso Negado</h2>
          <p>Você precisa estar logado para acessar esta página.</p>
        </div>
      </div>
    )
  }
  
  return <>{children}</>
}
```

### **Passo 3: Atualizar Configuração do BFF**
```typescript
// src/utils/bffClient.ts
const BFF_URL = import.meta.env.VITE_BFF_URL || 'http://localhost:3001'

export const bffClient = {
  async query(query: string, variables?: any) {
    const response = await fetch(`${BFF_URL}/api/graphql`, {
      method: 'POST',
      headers: {
        'Content-Type': 'application/json',
        'X-User-Id': 'dev-user-123', // Mock para desenvolvimento
        'X-Provider': 'google'
      },
      body: JSON.stringify({ query, variables })
    })
    
    if (!response.ok) {
      throw new Error(`BFF Error: ${response.status}`)
    }
    
    return response.json()
  }
}
```

### **Passo 4: Configurar Variáveis de Ambiente**
```bash
# .env.development
VITE_BFF_URL=http://localhost:3001
VITE_AUTH_ENABLED=false
VITE_MOCK_USER_ID=dev-user-123
```

### **Passo 5: Atualizar Rotas**
```typescript
// src/App.tsx
import { ProtectedRoute } from './components/ProtectedRoute'

function App() {
  return (
    <Router>
      <Routes>
        {/* Rotas públicas */}
        <Route path="/" element={<HomePage />} />
        <Route path="/about" element={<AboutPage />} />
        
        {/* Rotas protegidas */}
        <Route path="/dashboard" element={
          <ProtectedRoute>
            <DashboardPage />
          </ProtectedRoute>
        } />
        <Route path="/objectives" element={
          <ProtectedRoute>
            <ObjectivesPage />
          </ProtectedRoute>
        } />
      </Routes>
    </Router>
  )
}
```

### **Passo 6: Testar**
```bash
npm run dev
# Acessar: http://localhost:5173
```

---

## 🔧 **5. Auth Service (Novo) - Semana 5**

### **Passo 1: Criar Estrutura do Projeto**
```bash
mkdir moversemais-auth
cd moversemais-auth
gradle init --type kotlin-application
```

### **Passo 2: Configurar build.gradle.kts**
```kotlin
// build.gradle.kts
plugins {
    kotlin("jvm") version "1.9.25"
    id("org.springframework.boot") version "3.2.1"
    id("io.spring.dependency-management") version "1.1.4"
}

dependencies {
    implementation("org.springframework.boot:spring-boot-starter-web")
    implementation("org.springframework.boot:spring-boot-starter-data-jpa")
    implementation("org.springframework.boot:spring-boot-starter-security")
    implementation("org.springframework.security:spring-security-oauth2-client")
    implementation("io.jsonwebtoken:jjwt-api:0.11.5")
    implementation("io.jsonwebtoken:jjwt-impl:0.11.5")
    implementation("io.jsonwebtoken:jjwt-jackson:0.11.5")
    implementation("org.flywaydb:flyway-core")
    runtimeOnly("org.postgresql:postgresql")
}
```

### **Passo 3: Implementar Controller Básico**
```kotlin
// src/main/kotlin/com/moversemais/auth/controller/AuthController.kt
@RestController
@RequestMapping("/api/v1/auth")
class AuthController {
    
    @PostMapping("/user")
    fun createOrUpdateUser(@RequestBody request: CreateUserRequest): UserResponse {
        // Implementar criação/atualização de usuário
        return UserResponse(
            id = UUID.randomUUID().toString(),
            email = request.email,
            name = request.name,
            provider = request.provider
        )
    }
    
    @PostMapping("/token")
    fun generateToken(@RequestBody request: TokenRequest): TokenResponse {
        // Implementar geração de JWT
        return TokenResponse(
            token = "mock-jwt-token",
            expiresIn = 3600
        )
    }
}
```

### **Passo 4: Configurar Aplicação**
```kotlin
// src/main/kotlin/com/moversemais/auth/AuthApplication.kt
@SpringBootApplication
class AuthApplication

fun main(args: Array<String>) {
    runApplication<AuthApplication>(*args)
}
```

### **Passo 5: Testar**
```bash
./gradlew bootRun
# Testar: http://localhost:8084/api/v1/auth/user
```

---

## 📋 **Checklist de Implementação**

### **Semana 1: BFF**
- [ ] Instalar dependências JWT
- [ ] Criar middleware de autenticação
- [ ] Configurar Next.js
- [ ] Atualizar resolvers
- [ ] Testar comunicação

### **Semana 2: User Service**
- [ ] Adicionar dependências de segurança
- [ ] Criar interceptor de segurança
- [ ] Atualizar controller
- [ ] Configurar perfil de desenvolvimento
- [ ] Testar endpoints

### **Semana 3: Objective Service**
- [ ] Adicionar dependências de segurança
- [ ] Criar interceptor de segurança
- [ ] Atualizar controller
- [ ] Configurar perfil de desenvolvimento
- [ ] Testar endpoints

### **Semana 4: Frontend**
- [ ] Criar hook de autenticação mock
- [ ] Implementar proteção de rotas
- [ ] Atualizar cliente BFF
- [ ] Configurar variáveis de ambiente
- [ ] Testar fluxo completo

### **Semana 5: Auth Service**
- [ ] Criar estrutura do projeto
- [ ] Configurar dependências
- [ ] Implementar controller básico
- [ ] Configurar aplicação
- [ ] Testar integração

---

## 🚨 **Troubleshooting**

### **Problema: BFF não conecta com backend**
```bash
# Verificar se backend está rodando
curl http://localhost:8080/api/v1/health

# Verificar logs do BFF
tail -f logs/bff.log
```

### **Problema: Frontend não carrega dados**
```bash
# Verificar console do navegador
# Verificar se BFF está respondendo
curl http://localhost:3001/api/graphql
```

### **Problema: Headers não chegam no backend**
```bash
# Verificar se middleware está sendo aplicado
# Verificar logs do BFF
# Verificar logs do backend
```

---

**Última Atualização**: Outubro 2025  
**Versão**: 1.0  
**Mantenedor**: Equipe MoverseMais

---

*Este documento fornece passos específicos para implementar segurança em cada serviço.*
