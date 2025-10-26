# 🚀 Guia de Implementação - MoverseMais

## 📋 **Visão Geral**

Este documento consolida os guias de implementação para cada serviço da plataforma MoverseMais, seguindo os padrões arquiteturais definidos para SSO, segurança e compliance LGPD.

---

## 🎯 **Roadmap de Implementação**

### **Fase 1: Infraestrutura de Segurança (Semanas 1-2)**
1. **Criar Auth Service**
2. **Configurar OAuth providers**
3. **Implementar JWT generation/validation**
4. **Configurar network isolation**

### **Fase 2: Frontend e BFF (Semanas 3-4)**
1. **Implementar OAuth flow no React**
2. **Adicionar middleware de segurança no BFF**
3. **Configurar logs de auditoria**
4. **Implementar rate limiting**

### **Fase 3: Microserviços (Semanas 5-6)**
1. **Adicionar interceptors de segurança**
2. **Implementar validação de headers**
3. **Configurar logs de acesso**
4. **Atualizar User Service com dados mínimos**

### **Fase 4: Compliance e Monitoramento (Semanas 7-8)**
1. **Implementar endpoints LGPD**
2. **Configurar métricas de segurança**
3. **Implementar alertas**
4. **Realizar testes de penetração**

---

## 🏗️ **Implementação por Serviço**

### **1. Auth Service (Novo Serviço)**

#### **Estrutura do Projeto**
```
moversemais-auth/
├── src/main/kotlin/com/moversemais/auth/
│   ├── AuthApplication.kt
│   ├── config/
│   │   ├── OAuthConfig.kt
│   │   ├── JWTConfig.kt
│   │   └── SecurityConfig.kt
│   ├── controller/
│   │   ├── AuthController.kt
│   │   └── HealthController.kt
│   ├── service/
│   │   ├── OAuthService.kt
│   │   ├── JWTService.kt
│   │   └── UserService.kt
│   ├── dto/
│   │   ├── OAuthRequest.kt
│   │   ├── OAuthResponse.kt
│   │   └── UserData.kt
│   └── exception/
│       ├── OAuthException.kt
│       └── JWTException.kt
├── src/main/resources/
│   ├── application.yml
│   └── db/migration/
│       └── V1__Create_auth_tables.sql
└── docker-compose.yml
```

#### **Dependências Gradle**
```kotlin
dependencies {
    // Spring Boot
    implementation("org.springframework.boot:spring-boot-starter-web")
    implementation("org.springframework.boot:spring-boot-starter-data-jpa")
    implementation("org.springframework.boot:spring-boot-starter-security")
    implementation("org.springframework.boot:spring-boot-starter-actuator")
    
    // OAuth
    implementation("org.springframework.security:spring-security-oauth2-client")
    implementation("org.springframework.security:spring-security-oauth2-jose")
    
    // JWT
    implementation("io.jsonwebtoken:jjwt-api:0.11.5")
    implementation("io.jsonwebtoken:jjwt-impl:0.11.5")
    implementation("io.jsonwebtoken:jjwt-jackson:0.11.5")
    
    // Database
    implementation("org.flywaydb:flyway-core")
    runtimeOnly("org.postgresql:postgresql")
    
    // Validation
    implementation("org.springframework.boot:spring-boot-starter-validation")
}
```

#### **Configuração OAuth**
```kotlin
@Configuration
@EnableWebSecurity
class OAuthConfig {
    
    @Bean
    fun clientRegistrationRepository(): ClientRegistrationRepository {
        return InMemoryClientRegistrationRepository(
            googleClientRegistration(),
            facebookClientRegistration(),
            linkedinClientRegistration()
        )
    }
    
    private fun googleClientRegistration(): ClientRegistration {
        return ClientRegistration.withRegistrationId("google")
            .clientId(env.getProperty("oauth.google.client-id"))
            .clientSecret(env.getProperty("oauth.google.client-secret"))
            .authorizationGrantType(AuthorizationGrantType.AUTHORIZATION_CODE)
            .redirectUri("{baseUrl}/login/oauth2/code/{registrationId}")
            .scope("openid", "profile", "email")
            .authorizationUri("https://accounts.google.com/o/oauth2/v2/auth")
            .tokenUri("https://www.googleapis.com/oauth2/v4/token")
            .userInfoUri("https://www.googleapis.com/oauth2/v3/userinfo")
            .userNameAttributeName("sub")
            .clientName("Google")
            .build()
    }
}
```

#### **JWT Service**
```kotlin
@Service
class JWTService {
    
    private val jwtParser = Jwts.parserBuilder()
        .setSigningKey(jwtSecret)
        .build()
    
    fun generateToken(user: User): String {
        val now = Date()
        val expiration = Date(now.time + jwtExpiration)
        
        return Jwts.builder()
            .setSubject(user.id.toString())
            .claim("email", user.email)
            .claim("provider", user.provider)
            .setIssuedAt(now)
            .setExpiration(expiration)
            .setIssuer("moversemais-auth")
            .setAudience("moversemais-platform")
            .signWith(jwtSecret, SignatureAlgorithm.HS256)
            .compact()
    }
    
    fun validateToken(token: String): Claims {
        return try {
            jwtParser.parseClaimsJws(token).body
        } catch (e: JwtException) {
            throw InvalidTokenException("Invalid JWT token")
        }
    }
}
```

### **2. Frontend (moversemais-store-front)**

#### **Configuração NextAuth**
```typescript
// pages/api/auth/[...nextauth].ts
import NextAuth from 'next-auth'
import GoogleProvider from 'next-auth/providers/google'
import FacebookProvider from 'next-auth/providers/facebook'
import LinkedInProvider from 'next-auth/providers/linkedin'

export default NextAuth({
  providers: [
    GoogleProvider({
      clientId: process.env.GOOGLE_CLIENT_ID!,
      clientSecret: process.env.GOOGLE_CLIENT_SECRET!,
    }),
    FacebookProvider({
      clientId: process.env.FACEBOOK_CLIENT_ID!,
      clientSecret: process.env.FACEBOOK_CLIENT_SECRET!,
    }),
    LinkedInProvider({
      clientId: process.env.LINKEDIN_CLIENT_ID!,
      clientSecret: process.env.LINKEDIN_CLIENT_SECRET!,
    })
  ],
  callbacks: {
    async jwt({ token, account, profile }) {
      if (account && profile) {
        // Enviar dados para Auth Service
        const response = await fetch(`${process.env.AUTH_SERVICE_URL}/api/v1/auth/user`, {
          method: 'POST',
          headers: { 'Content-Type': 'application/json' },
          body: JSON.stringify({
            email: profile.email,
            name: profile.name,
            provider: account.provider,
            providerId: account.providerAccountId
          })
        })
        
        const userData = await response.json()
        token.userId = userData.id
        token.provider = account.provider
      }
      return token
    },
    async session({ session, token }) {
      session.user.id = token.userId
      session.user.provider = token.provider
      return session
    }
  },
  pages: {
    signIn: '/auth/signin',
    error: '/auth/error'
  },
  session: {
    strategy: 'jwt',
    maxAge: 24 * 60 * 60 // 24 horas
  }
})
```

#### **Hook de Autenticação**
```typescript
// hooks/useAuth.ts
import { useSession } from 'next-auth/react'
import { useRouter } from 'next/router'
import { useEffect } from 'react'

export const useAuth = () => {
  const { data: session, status } = useSession()
  const router = useRouter()
  
  useEffect(() => {
    if (status === 'loading') return
    
    if (!session) {
      router.push('/auth/signin')
    }
  }, [session, status, router])
  
  return {
    user: session?.user,
    loading: status === 'loading',
    isAuthenticated: !!session
  }
}
```

#### **Proteção de Rotas**
```typescript
// components/ProtectedRoute.tsx
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
    return null // useAuth já redireciona
  }
  
  return <>{children}</>
}
```

### **3. BFF (moversemais-store-graphql)**

#### **Middleware de Autenticação**
```typescript
// middleware/auth.ts
import { NextRequest, NextResponse } from 'next/server'
import jwt from 'jsonwebtoken'

interface JWTPayload {
  sub: string
  email: string
  provider: string
  iat: number
  exp: number
  iss: string
  aud: string
}

export async function authMiddleware(request: NextRequest) {
  const token = request.headers.get('authorization')?.replace('Bearer ', '')
  
  if (!token) {
    return NextResponse.json(
      { error: 'AUTH_001', message: 'Missing authentication token' },
      { status: 401 }
    )
  }
  
  try {
    const payload = jwt.verify(token, process.env.JWT_SECRET!) as JWTPayload
    
    // Validar claims obrigatórios
    if (!payload.sub || !payload.email || !payload.provider) {
      throw new Error('Invalid token claims')
    }
    
    // Adicionar headers para microserviços
    request.headers.set('X-User-Id', payload.sub)
    request.headers.set('X-User-Email', payload.email)
    request.headers.set('X-Provider', payload.provider)
    request.headers.set('X-Request-Id', crypto.randomUUID())
    request.headers.set('X-Timestamp', new Date().toISOString())
    
    return NextResponse.next()
  } catch (error) {
    return NextResponse.json(
      { error: 'AUTH_003', message: 'Invalid or expired token' },
      { status: 401 }
    )
  }
}

// Aplicar middleware em todas as rotas GraphQL
export const config = {
  matcher: '/api/graphql/:path*'
}
```

#### **Rate Limiting**
```typescript
// utils/rateLimiter.ts
import { NextRequest } from 'next/server'

interface RateLimitInfo {
  count: number
  windowStart: number
  blocked: boolean
}

const rateLimiter = new Map<string, RateLimitInfo>()

export function checkRateLimit(request: NextRequest): boolean {
  const userId = request.headers.get('X-User-Id')
  if (!userId) return false
  
  const now = Date.now()
  const windowMs = 60 * 1000 // 1 minuto
  const maxRequests = 100 // 100 requests por minuto
  
  const userLimit = rateLimiter.get(userId) || {
    count: 0,
    windowStart: now,
    blocked: false
  }
  
  // Reset window se necessário
  if (now - userLimit.windowStart > windowMs) {
    userLimit.count = 0
    userLimit.windowStart = now
    userLimit.blocked = false
  }
  
  // Verificar limite
  if (userLimit.count >= maxRequests) {
    userLimit.blocked = true
    return false
  }
  
  userLimit.count++
  rateLimiter.set(userId, userLimit)
  return true
}
```

### **4. User Service (moversemais-user)**

#### **Entidade User Atualizada**
```kotlin
@Entity
@Table(name = "users")
data class User(
    @Id
    val id: UUID = UUID.randomUUID(),
    
    @Column(name = "email_hash", nullable = false, unique = true)
    val emailHash: String, // SHA-256 hash para busca
    
    @Convert(converter = EncryptedStringConverter::class)
    @Column(name = "email", nullable = false)
    val email: String, // Email original criptografado
    
    @Column(name = "name")
    val name: String?,
    
    @Column(name = "provider", nullable = false)
    val provider: String, // google, facebook, linkedin
    
    @Column(name = "provider_id", nullable = false)
    val providerId: String,
    
    @Column(name = "created_at", nullable = false)
    val createdAt: Instant = Instant.now(),
    
    @Column(name = "last_access_at", nullable = false)
    val lastAccessAt: Instant = Instant.now(),
    
    @Enumerated(EnumType.STRING)
    @Column(name = "status", nullable = false)
    val status: UserStatus = UserStatus.ACTIVE,
    
    @Column(name = "timezone")
    val timezone: String? = "America/Sao_Paulo",
    
    @Column(name = "preferred_language")
    val preferredLanguage: String? = "pt-BR"
)

enum class UserStatus {
    ACTIVE, INACTIVE, ANONYMIZED, DELETED
}
```

#### **Interceptor de Segurança**
```kotlin
@Component
class SecurityInterceptor : HandlerInterceptor {
    
    private val logger = LoggerFactory.getLogger(SecurityInterceptor::class.java)
    
    override fun preHandle(request: HttpServletRequest, response: HttpServletResponse, handler: Any): Boolean {
        try {
            // Validar headers obrigatórios
            validateRequiredHeaders(request)
            
            // Validar origem da requisição
            validateRequestOrigin(request)
            
            // Validar timestamp
            validateTimestamp(request)
            
            // Log de auditoria
            auditLog(request)
            
            return true
        } catch (e: SecurityException) {
            logger.warn("Security violation: ${e.message}", e)
            response.status = HttpStatus.UNAUTHORIZED.value()
            response.writer.write("""{"error":"SEC_001","message":"Security violation"}""")
            return false
        }
    }
    
    private fun validateRequiredHeaders(request: HttpServletRequest) {
        val requiredHeaders = listOf("X-User-Id", "X-Provider", "X-Request-Id")
        requiredHeaders.forEach { header ->
            if (request.getHeader(header).isNullOrBlank()) {
                throw SecurityException("Missing required header: $header")
            }
        }
    }
    
    private fun validateRequestOrigin(request: HttpServletRequest) {
        val bffIp = request.getHeader("X-Forwarded-For") ?: request.remoteAddr
        val allowedBffIps = listOf("127.0.0.1", "::1", "10.0.0.0/8")
        
        if (!isAllowedBffIp(bffIp, allowedBffIps)) {
            throw SecurityException("Request not from authorized BFF")
        }
    }
    
    private fun auditLog(request: HttpServletRequest) {
        val userId = request.getHeader("X-User-Id")
        val endpoint = request.requestURI
        val method = request.method
        val ip = request.remoteAddr
        
        logger.info("API access", mapOf(
            "event" to "API_ACCESS",
            "userId" to userId,
            "endpoint" to endpoint,
            "method" to method,
            "ip" to ip
        ))
    }
}
```

#### **Endpoints LGPD**
```kotlin
@RestController
@RequestMapping("/api/v1/users")
class UserController {
    
    @GetMapping("/me/data")
    fun getUserData(@RequestHeader("X-User-Id") userId: String): UserDataExport {
        val user = userService.findById(UUID.fromString(userId))
        return UserDataExport(
            personalData = user.toPersonalData(),
            processedData = user.toProcessedData(),
            consentHistory = consentService.getConsentHistory(userId),
            dataSources = listOf("oauth_${user.provider}")
        )
    }
    
    @PostMapping("/me/anonymize")
    fun anonymizeUserData(@RequestHeader("X-User-Id") userId: String): AnonymizationResult {
        val user = userService.findById(UUID.fromString(userId))
        
        // Anonimizar dados pessoais
        val anonymizedUser = user.copy(
            email = "anonymized_${user.id}@deleted.local",
            name = "Usuário Anonimizado",
            status = UserStatus.ANONYMIZED,
            anonymizedAt = Instant.now()
        )
        
        // Salvar dados anonimizados
        anonymizedDataService.save(anonymizedUser.toAnonymizedData())
        
        // Remover dados pessoais
        userService.delete(user)
        
        return AnonymizationResult(
            success = true,
            anonymizedDataId = anonymizedUser.id,
            message = "Dados anonimizados com sucesso"
        )
    }
    
    @DeleteMapping("/me")
    fun deleteUserData(@RequestHeader("X-User-Id") userId: String): DeletionResult {
        userService.deleteAllUserData(userId)
        return DeletionResult(
            success = true,
            message = "Todos os dados foram excluídos com sucesso"
        )
    }
}
```

### **5. Objective Service (moversemais-objective)**

#### **Interceptor de Segurança**
```kotlin
@Component
class SecurityInterceptor : HandlerInterceptor {
    
    override fun preHandle(request: HttpServletRequest, response: HttpServletResponse, handler: Any): Boolean {
        // Aplicar mesma validação do User Service
        validateRequiredHeaders(request)
        validateRequestOrigin(request)
        validateTimestamp(request)
        auditLog(request)
        return true
    }
    
    private fun auditLog(request: HttpServletRequest) {
        val userId = request.getHeader("X-User-Id")
        val endpoint = request.requestURI
        val method = request.method
        
        logger.info("Objective service access", mapOf(
            "event" to "OBJECTIVE_ACCESS",
            "userId" to userId,
            "endpoint" to endpoint,
            "method" to method
        ))
    }
}
```

#### **Validação de Propriedade**
```kotlin
@Service
class ObjectiveSecurityService {
    
    fun validateObjectiveOwnership(objectiveId: UUID, userId: String): Objective {
        val objective = objectiveRepository.findById(objectiveId)
            .orElseThrow { ObjectiveNotFoundException("Objective not found") }
        
        if (objective.userId != UUID.fromString(userId)) {
            throw AuthorizationException("Access denied to objective")
        }
        
        return objective
    }
    
    fun validateAssessmentOwnership(assessmentId: UUID, userId: String): Assessment {
        val assessment = assessmentRepository.findById(assessmentId)
            .orElseThrow { AssessmentNotFoundException("Assessment not found") }
        
        if (assessment.userId != UUID.fromString(userId)) {
            throw AuthorizationException("Access denied to assessment")
        }
        
        return assessment
    }
}
```

---

## 🔧 **Configurações de Ambiente**

### **1. Variáveis de Ambiente**

#### **Auth Service**
```bash
# Database
DATABASE_URL=postgresql://user:password@localhost:5432/moversemais_auth
DB_USERNAME=developer
DB_PASSWORD=@@moversemais@@

# JWT
JWT_SECRET=your-super-secret-jwt-key-here
JWT_EXPIRATION=3600
JWT_ISSUER=moversemais-auth
JWT_AUDIENCE=moversemais-platform

# OAuth
OAUTH_GOOGLE_CLIENT_ID=your-google-client-id
OAUTH_GOOGLE_CLIENT_SECRET=your-google-client-secret
OAUTH_FACEBOOK_CLIENT_ID=your-facebook-client-id
OAUTH_FACEBOOK_CLIENT_SECRET=your-facebook-client-secret
OAUTH_LINKEDIN_CLIENT_ID=your-linkedin-client-id
OAUTH_LINKEDIN_CLIENT_SECRET=your-linkedin-client-secret

# Security
ALLOWED_ORIGINS=http://localhost:5173,https://moversemais.com
RATE_LIMIT_WINDOW_MS=60000
RATE_LIMIT_MAX_REQUESTS=100
```

#### **BFF**
```bash
# Auth Service
AUTH_SERVICE_URL=http://localhost:8084
JWT_SECRET=your-super-secret-jwt-key-here

# Backend Services
OBJECTIVE_SERVICE_URL=http://localhost:8080
USER_SERVICE_URL=http://localhost:8083

# Rate Limiting
RATE_LIMIT_WINDOW_MS=60000
RATE_LIMIT_MAX_REQUESTS=100

# CORS
ALLOWED_ORIGINS=http://localhost:5173,https://moversemais.com
```

#### **Frontend**
```bash
# NextAuth
NEXTAUTH_URL=http://localhost:5173
NEXTAUTH_SECRET=your-nextauth-secret

# OAuth Providers
GOOGLE_CLIENT_ID=your-google-client-id
GOOGLE_CLIENT_SECRET=your-google-client-secret
FACEBOOK_CLIENT_ID=your-facebook-client-id
FACEBOOK_CLIENT_SECRET=your-facebook-client-secret
LINKEDIN_CLIENT_ID=your-linkedin-client-id
LINKEDIN_CLIENT_SECRET=your-linkedin-client-secret

# BFF
BFF_URL=http://localhost:3001
```

### **2. Docker Compose**

#### **Auth Service**
```yaml
version: '3.8'
services:
  auth-service:
    build: ./moversemais-auth
    ports:
      - "8084:8080"
    environment:
      - SPRING_PROFILES_ACTIVE=docker
      - DATABASE_URL=postgresql://developer:@@moversemais@@@postgres:5432/moversemais_auth
    depends_on:
      - postgres
    networks:
      - moversemais-network

  postgres:
    image: postgres:15
    environment:
      - POSTGRES_DB=moversemais_auth
      - POSTGRES_USER=developer
      - POSTGRES_PASSWORD=@@moversemais@@
    ports:
      - "5434:5432"
    networks:
      - moversemais-network

networks:
  moversemais-network:
    driver: bridge
```

---

## 🧪 **Testes de Implementação**

### **1. Testes de Autenticação**
```typescript
// tests/auth.test.ts
import { render, screen, fireEvent } from '@testing-library/react'
import { useAuth } from '../hooks/useAuth'

describe('Authentication', () => {
  test('should redirect to login when not authenticated', () => {
    // Mock não autenticado
    jest.mocked(useAuth).mockReturnValue({
      user: null,
      loading: false,
      isAuthenticated: false
    })
    
    render(<ProtectedRoute><div>Protected Content</div></ProtectedRoute>)
    
    // Verificar redirecionamento
    expect(mockRouter.push).toHaveBeenCalledWith('/auth/signin')
  })
  
  test('should display content when authenticated', () => {
    // Mock autenticado
    jest.mocked(useAuth).mockReturnValue({
      user: { id: 'user-123', email: 'test@example.com' },
      loading: false,
      isAuthenticated: true
    })
    
    render(<ProtectedRoute><div>Protected Content</div></ProtectedRoute>)
    
    expect(screen.getByText('Protected Content')).toBeInTheDocument()
  })
})
```

### **2. Testes de Segurança**
```kotlin
// SecurityInterceptorTest.kt
@SpringBootTest
class SecurityInterceptorTest {
    
    @Test
    fun `should reject request without required headers`() {
        val request = MockHttpServletRequest()
        val response = MockHttpServletResponse()
        
        val result = securityInterceptor.preHandle(request, response, null)
        
        assertFalse(result)
        assertEquals(HttpStatus.UNAUTHORIZED.value(), response.status)
    }
    
    @Test
    fun `should accept request with valid headers`() {
        val request = MockHttpServletRequest()
        request.addHeader("X-User-Id", "user-123")
        request.addHeader("X-Provider", "google")
        request.addHeader("X-Request-Id", "req-123")
        request.remoteAddr = "127.0.0.1"
        
        val response = MockHttpServletResponse()
        
        val result = securityInterceptor.preHandle(request, response, null)
        
        assertTrue(result)
    }
}
```

### **3. Testes de LGPD**
```kotlin
// LGPDComplianceTest.kt
@SpringBootTest
class LGPDComplianceTest {
    
    @Test
    fun `should return user data for data access request`() {
        val userId = "user-123"
        val user = createTestUser(userId)
        
        val response = userController.getUserData(userId)
        
        assertEquals(userId, response.personalData.id)
        assertNotNull(response.processedData)
        assertNotNull(response.consentHistory)
    }
    
    @Test
    fun `should anonymize user data on request`() {
        val userId = "user-123"
        val user = createTestUser(userId)
        
        val response = userController.anonymizeUserData(userId)
        
        assertTrue(response.success)
        assertNotNull(response.anonymizedDataId)
        
        // Verificar se dados foram anonimizados
        val anonymizedUser = userService.findById(userId)
        assertEquals("Usuário Anonimizado", anonymizedUser.name)
        assertEquals(UserStatus.ANONYMIZED, anonymizedUser.status)
    }
}
```

---

## 📋 **Checklist de Implementação**

### **Fase 1: Infraestrutura**
- [ ] Criar Auth Service
- [ ] Configurar OAuth providers
- [ ] Implementar JWT generation/validation
- [ ] Configurar network isolation
- [ ] Implementar logs de auditoria

### **Fase 2: Frontend e BFF**
- [ ] Implementar OAuth flow no React
- [ ] Adicionar middleware de segurança no BFF
- [ ] Configurar rate limiting
- [ ] Implementar proteção de rotas
- [ ] Adicionar validação de tokens

### **Fase 3: Microserviços**
- [ ] Adicionar interceptors de segurança
- [ ] Implementar validação de headers
- [ ] Configurar logs de acesso
- [ ] Atualizar User Service com dados mínimos
- [ ] Implementar validação de propriedade

### **Fase 4: Compliance**
- [ ] Implementar endpoints LGPD
- [ ] Configurar métricas de segurança
- [ ] Implementar alertas
- [ ] Realizar testes de penetração
- [ ] Documentar procedimentos

---

## 🚨 **Troubleshooting Comum**

### **1. Problemas de CORS**
```typescript
// Solução: Configurar CORS corretamente no BFF
const corsOptions = {
  origin: ['http://localhost:5173', 'https://moversemais.com'],
  credentials: true,
  optionsSuccessStatus: 200
}
```

### **2. Problemas de JWT**
```kotlin
// Solução: Verificar configuração JWT
@Configuration
class JWTConfig {
    @Bean
    fun jwtDecoder(): JwtDecoder {
        return NimbusJwtDecoder.withSecretKey(secretKey).build()
    }
}
```

### **3. Problemas de Rate Limiting**
```typescript
// Solução: Ajustar configurações de rate limiting
const rateLimitConfig = {
  windowMs: 60 * 1000, // 1 minuto
  maxRequests: 100, // 100 requests por minuto
  message: 'Rate limit exceeded'
}
```

---

**Última Atualização**: Outubro 2025  
**Versão**: 1.0  
**Mantenedor**: Equipe MoverseMais

---

*Este documento deve ser atualizado sempre que houver mudanças na implementação.*
