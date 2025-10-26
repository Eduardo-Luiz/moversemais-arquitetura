# 🎯 Auth Service - Implementação SSO e Segurança

## 📋 **Card de Desenvolvimento**

**Serviço**: moversemais-auth (Novo)  
**Tipo**: SSO e Segurança  
**Prioridade**: Média  
**Estimativa**: 2-3 dias  
**Status**: TODO  

---

## 🎯 **Objetivo**

Criar novo serviço de autenticação para gerenciar SSO (Google, Facebook, LinkedIn) e geração de JWT tokens, servindo como fonte única de verdade para autenticação.

---

## 📚 **Documentação de Referência**

**Leia antes de implementar:**
- `../auth-security/SSO_SECURITY_ARCHITECTURE.md` - Arquitetura SSO completa
- `../auth-security/SECURITY_PATTERNS.md` - Padrões de segurança
- `../auth-security/IMPLEMENTATION_GUIDE.md` - Guia de implementação completo

---

## ✅ **Tarefas Obrigatórias**

### **1. Criar Estrutura do Projeto**
```bash
# Criar projeto Spring Boot
mkdir moversemais-auth
cd moversemais-auth
gradle init --type kotlin-application
```

### **2. Configurar build.gradle.kts**
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

### **3. Configuração OAuth**
Criar arquivo: `src/main/kotlin/com/moversemais/auth/config/OAuthConfig.kt`

**Funcionalidades obrigatórias:**
- ✅ Configuração Google OAuth
- ✅ Configuração Facebook OAuth
- ✅ Configuração LinkedIn OAuth
- ✅ Redirecionamento após login
- ✅ Geração de JWT após OAuth

### **4. JWT Service**
Criar arquivo: `src/main/kotlin/com/moversemais/auth/service/JWTService.kt`

**Funcionalidades obrigatórias:**
- ✅ Geração de JWT tokens
- ✅ Validação de JWT tokens
- ✅ Refresh tokens
- ✅ Claims personalizados

### **5. User Service**
Criar arquivo: `src/main/kotlin/com/moversemais/auth/service/UserService.kt`

**Funcionalidades obrigatórias:**
- ✅ Criar/atualizar usuário após OAuth
- ✅ Buscar usuário por ID
- ✅ Gerenciar dados mínimos (LGPD)
- ✅ Log de operações

### **6. Controllers**
Criar arquivos:
- `src/main/kotlin/com/moversemais/auth/controller/AuthController.kt`
- `src/main/kotlin/com/moversemais/auth/controller/UserController.kt`

**Funcionalidades obrigatórias:**
- ✅ Endpoints de OAuth
- ✅ Endpoints de JWT
- ✅ Endpoints de usuário
- ✅ Health check

### **7. Entidades e DTOs**
Criar arquivos:
- `src/main/kotlin/com/moversemais/auth/entity/User.kt`
- `src/main/kotlin/com/moversemais/auth/dto/*.kt`

**Funcionalidades obrigatórias:**
- ✅ Entidade User com campos LGPD
- ✅ DTOs de request/response
- ✅ Mapeamento OAuth → User

### **8. Configuração de Banco**
Criar arquivo: `src/main/resources/db/migration/V1__Create_auth_tables.sql`

**Funcionalidades obrigatórias:**
- ✅ Tabela users
- ✅ Tabela oauth_tokens
- ✅ Índices necessários

### **9. Testes Básicos**
**Testes obrigatórios:**
- ✅ OAuth flow funcionando
- ✅ JWT generation/validation
- ✅ Endpoints respondendo
- ✅ Banco de dados funcionando

---

## 🔧 **Implementação Detalhada**

### **Configuração OAuth**
```kotlin
// src/main/kotlin/com/moversemais/auth/config/OAuthConfig.kt
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
    
    @Bean
    fun oauth2AuthorizedClientService(): OAuth2AuthorizedClientService {
        return InMemoryOAuth2AuthorizedClientService(clientRegistrationRepository())
    }
}
```

### **JWT Service**
```kotlin
// src/main/kotlin/com/moversemais/auth/service/JWTService.kt
@Service
class JWTService {
    
    private val jwtSecret = env.getProperty("jwt.secret") ?: "dev-secret-key"
    private val jwtExpiration = env.getProperty("jwt.expiration", Long::class.java) ?: 3600L
    
    private val jwtParser = Jwts.parserBuilder()
        .setSigningKey(jwtSecret)
        .build()
    
    fun generateToken(user: User): String {
        val now = Date()
        val expiration = Date(now.time + jwtExpiration * 1000)
        
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
    
    fun generateRefreshToken(user: User): String {
        val now = Date()
        val expiration = Date(now.time + 7 * 24 * 60 * 60 * 1000) // 7 dias
        
        return Jwts.builder()
            .setSubject(user.id.toString())
            .claim("type", "refresh")
            .setIssuedAt(now)
            .setExpiration(expiration)
            .signWith(jwtSecret, SignatureAlgorithm.HS256)
            .compact()
    }
}
```

### **User Service**
```kotlin
// src/main/kotlin/com/moversemais/auth/service/UserService.kt
@Service
class UserService {
    
    private val logger = LoggerFactory.getLogger(UserService::class.java)
    
    fun createOrUpdateUser(profile: OAuthProfile, account: OAuthAccount): User {
        logger.info("🔍 [AUTH] Criando/atualizando usuário: ${profile.email}")
        
        val existingUser = userRepository.findByProviderAndProviderId(account.provider, account.providerId)
        
        val user = existingUser?.copy(
            email = profile.email,
            name = profile.name,
            lastAccessAt = Instant.now()
        ) ?: User(
            id = UUID.randomUUID(),
            email = profile.email,
            name = profile.name,
            provider = account.provider,
            providerId = account.providerId,
            createdAt = Instant.now(),
            lastAccessAt = Instant.now()
        )
        
        val savedUser = userRepository.save(user)
        logger.info("✅ [AUTH] Usuário salvo: ${savedUser.id}")
        
        return savedUser
    }
    
    fun findById(userId: String): User {
        return userRepository.findById(UUID.fromString(userId))
            .orElseThrow { UserNotFoundException("User not found") }
    }
}
```

### **Auth Controller**
```kotlin
// src/main/kotlin/com/moversemais/auth/controller/AuthController.kt
@RestController
@RequestMapping("/api/v1/auth")
class AuthController {
    
    @PostMapping("/user")
    fun createOrUpdateUser(@RequestBody request: CreateUserRequest): UserResponse {
        logger.info("🔍 [AUTH] Criando/atualizando usuário: ${request.email}")
        
        val user = userService.createOrUpdateUser(
            OAuthProfile(request.email, request.name),
            OAuthAccount(request.provider, request.providerId)
        )
        
        val token = jwtService.generateToken(user)
        val refreshToken = jwtService.generateRefreshToken(user)
        
        return UserResponse(
            id = user.id.toString(),
            email = user.email,
            name = user.name,
            provider = user.provider,
            token = token,
            refreshToken = refreshToken,
            expiresIn = 3600
        )
    }
    
    @PostMapping("/token/refresh")
    fun refreshToken(@RequestBody request: RefreshTokenRequest): TokenResponse {
        logger.info("🔍 [AUTH] Refresh token para usuário: ${request.userId}")
        
        val user = userService.findById(request.userId)
        val token = jwtService.generateToken(user)
        val refreshToken = jwtService.generateRefreshToken(user)
        
        return TokenResponse(
            token = token,
            refreshToken = refreshToken,
            expiresIn = 3600
        )
    }
    
    @PostMapping("/token/validate")
    fun validateToken(@RequestBody request: ValidateTokenRequest): TokenValidationResponse {
        logger.info("🔍 [AUTH] Validando token")
        
        try {
            val claims = jwtService.validateToken(request.token)
            return TokenValidationResponse(
                valid = true,
                userId = claims.subject,
                email = claims.get("email", String::class.java),
                provider = claims.get("provider", String::class.java)
            )
        } catch (e: Exception) {
            return TokenValidationResponse(valid = false)
        }
    }
}
```

### **Configuração de Aplicação**
```kotlin
// src/main/kotlin/com/moversemais/auth/AuthApplication.kt
@SpringBootApplication
class AuthApplication

fun main(args: Array<String>) {
    runApplication<AuthApplication>(*args)
}
```

### **Configuração de Banco**
```sql
-- src/main/resources/db/migration/V1__Create_auth_tables.sql
CREATE TABLE users (
    id UUID PRIMARY KEY,
    email_hash VARCHAR(255) NOT NULL UNIQUE,
    email VARCHAR(255) NOT NULL,
    name VARCHAR(255),
    provider VARCHAR(50) NOT NULL,
    provider_id VARCHAR(255) NOT NULL,
    created_at TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP,
    last_access_at TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP,
    status VARCHAR(20) DEFAULT 'ACTIVE',
    timezone VARCHAR(50) DEFAULT 'America/Sao_Paulo',
    preferred_language VARCHAR(10) DEFAULT 'pt-BR'
);

CREATE TABLE oauth_tokens (
    id UUID PRIMARY KEY,
    user_id UUID NOT NULL REFERENCES users(id),
    token_type VARCHAR(20) NOT NULL,
    token_value TEXT NOT NULL,
    expires_at TIMESTAMP WITH TIME ZONE NOT NULL,
    created_at TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP
);

CREATE INDEX idx_users_email_hash ON users(email_hash);
CREATE INDEX idx_users_provider ON users(provider, provider_id);
CREATE INDEX idx_oauth_tokens_user_id ON oauth_tokens(user_id);
```

---

## 🧪 **Critérios de Validação**

### **Funcionalidades Obrigatórias**
- [ ] OAuth flow funcionando (Google, Facebook, LinkedIn)
- [ ] JWT generation/validation funcionando
- [ ] User service funcionando com banco real
- [ ] Banco de dados configurado
- [ ] Endpoints respondendo

### **Testes Obrigatórios**
- [ ] `curl http://localhost:8084/api/v1/auth/health` funciona
- [ ] OAuth flow completo funcionando
- [ ] JWT tokens sendo gerados
- [ ] Usuários sendo criados/atualizados no banco real

### **Código Obrigatório**
- [ ] OAuthConfig implementado
- [ ] JWTService implementado
- [ ] UserService implementado
- [ ] Controllers implementados
- [ ] Entidades e DTOs implementados
- [ ] Migrações de banco criadas

---

## 🚨 **Troubleshooting**

### **Problema: OAuth não funciona**
- Verificar configuração dos providers
- Verificar client-id e client-secret
- Verificar URLs de redirecionamento

### **Problema: JWT não é gerado**
- Verificar configuração JWT
- Verificar se usuário está sendo criado
- Verificar logs de erro

### **Problema: Banco não conecta**
- Verificar configuração de banco
- Verificar se migrações foram executadas
- Verificar logs de conexão

---

## 📋 **Checklist de Implementação**

- [ ] Estrutura do projeto criada
- [ ] Dependências configuradas
- [ ] OAuthConfig implementado
- [ ] JWTService implementado
- [ ] UserService implementado
- [ ] Controllers implementados
- [ ] Entidades e DTOs implementados
- [ ] Migrações de banco criadas
- [ ] Testes básicos funcionando

---

## 🎯 **Próximo Passo**

Após completar este card, mover para `WIP/` e depois para `VALIDATING/` para análise.

**Próximo card**: Integração completa entre todos os serviços

---

**Criado em**: Outubro 2025  
**Responsável**: Equipe MoverseMais  
**Status**: TODO
