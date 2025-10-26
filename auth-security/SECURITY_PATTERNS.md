# 🛡️ Padrões de Segurança - MoverseMais

## 📋 **Visão Geral**

Este documento define os padrões de segurança que todos os serviços da plataforma MoverseMais devem seguir, garantindo consistência e proteção contra vulnerabilidades comuns.

---

## 🔒 **Princípios de Segurança**

### **1. Defense in Depth**
- **Múltiplas camadas** de proteção
- **Validação em cada nível** da aplicação
- **Fail-safe**: Falha de forma segura

### **2. Least Privilege**
- **Acesso mínimo necessário** para cada componente
- **Princípio do menor privilégio** para usuários e serviços
- **Separação de responsabilidades** clara

### **3. Zero Trust**
- **Nunca confiar, sempre verificar**
- **Validação contínua** de identidade e autorização
- **Monitoramento constante** de atividades

---

## 🚪 **Controle de Acesso**

### **1. Estratégia de Proteção de APIs**

#### **Nível 1: BFF como Gateway de Segurança**
```typescript
// Padrão obrigatório no BFF
interface SecurityHeaders {
  'X-User-Id': string;           // UUID do usuário
  'X-Provider': string;          // google|facebook|linkedin
  'X-Request-Id': string;        // UUID único da requisição
  'X-Timestamp': string;         // Timestamp da requisição
  'X-Signature': string;         // Assinatura HMAC (opcional)
}
```

#### **Nível 2: Microserviços Protegidos**
```kotlin
// Padrão obrigatório em todos os microserviços
@Component
class SecurityValidationInterceptor : HandlerInterceptor {
    
    override fun preHandle(request: HttpServletRequest, response: HttpServletResponse, handler: Any): Boolean {
        // 1. Validar headers obrigatórios
        validateRequiredHeaders(request)
        
        // 2. Validar origem da requisição (BFF)
        validateRequestOrigin(request)
        
        // 3. Validar timestamp (prevent replay attacks)
        validateTimestamp(request)
        
        // 4. Log de auditoria
        auditLog(request)
        
        return true
    }
    
    private fun validateRequiredHeaders(request: HttpServletRequest) {
        val requiredHeaders = listOf("X-User-Id", "X-Provider", "X-Request-Id")
        requiredHeaders.forEach { header ->
            if (request.getHeader(header).isNullOrBlank()) {
                throw SecurityException("Missing required header: $header")
            }
        }
    }
}
```

### **2. Validação de Origem**

#### **BFF (moversemais-store-graphql)**
```typescript
// Middleware de validação de origem
const validateOrigin = (req: NextRequest) => {
  const allowedOrigins = [
    'http://localhost:5173',  // Frontend local
    'https://moversemais.com', // Frontend produção
  ];
  
  const origin = req.headers.get('origin');
  if (!allowedOrigins.includes(origin)) {
    throw new ForbiddenError('Invalid origin');
  }
};
```

#### **Microserviços**
```kotlin
// Validação de origem em microserviços
private fun validateRequestOrigin(request: HttpServletRequest) {
    val bffIp = request.getHeader("X-Forwarded-For") ?: request.remoteAddr
    val allowedBffIps = listOf("127.0.0.1", "::1", "10.0.0.0/8")
    
    if (!isAllowedBffIp(bffIp, allowedBffIps)) {
        throw SecurityException("Request not from authorized BFF")
    }
}
```

---

## 🔐 **Autenticação e Autorização**

### **1. JWT Validation Pattern**

#### **BFF (moversemais-store-graphql)**
```typescript
// Padrão de validação JWT
interface JWTPayload {
  sub: string;        // User ID
  email: string;      // User email
  provider: string;   // OAuth provider
  iat: number;        // Issued at
  exp: number;        // Expiration
  iss: string;        // Issuer
  aud: string;        // Audience
}

const validateJWT = (token: string): JWTPayload => {
  try {
    const payload = jwt.verify(token, JWT_SECRET) as JWTPayload;
    
    // Validar claims obrigatórios
    validateRequiredClaims(payload);
    
    // Validar expiração
    if (payload.exp < Date.now() / 1000) {
      throw new TokenExpiredError();
    }
    
    return payload;
  } catch (error) {
    throw new InvalidTokenError();
  }
};
```

#### **Microserviços**
```kotlin
// Padrão de validação de headers
data class AuthContext(
    val userId: UUID,
    val email: String,
    val provider: String,
    val requestId: String,
    val timestamp: Instant
)

fun extractAuthContext(request: HttpServletRequest): AuthContext {
    val userId = UUID.fromString(request.getHeader("X-User-Id"))
    val email = request.getHeader("X-User-Email") ?: ""
    val provider = request.getHeader("X-Provider")
    val requestId = request.getHeader("X-Request-Id")
    val timestamp = Instant.parse(request.getHeader("X-Timestamp"))
    
    return AuthContext(userId, email, provider, requestId, timestamp)
}
```

### **2. Rate Limiting**

#### **BFF (moversemais-store-graphql)**
```typescript
// Rate limiting por usuário
const rateLimiter = new Map<string, RateLimitInfo>();

interface RateLimitInfo {
  count: number;
  windowStart: number;
  blocked: boolean;
}

const checkRateLimit = (userId: string): boolean => {
  const now = Date.now();
  const windowMs = 60 * 1000; // 1 minuto
  const maxRequests = 100; // 100 requests por minuto
  
  const userLimit = rateLimiter.get(userId) || {
    count: 0,
    windowStart: now,
    blocked: false
  };
  
  // Reset window se necessário
  if (now - userLimit.windowStart > windowMs) {
    userLimit.count = 0;
    userLimit.windowStart = now;
    userLimit.blocked = false;
  }
  
  // Verificar limite
  if (userLimit.count >= maxRequests) {
    userLimit.blocked = true;
    return false;
  }
  
  userLimit.count++;
  rateLimiter.set(userId, userLimit);
  return true;
};
```

---

## 🛡️ **Proteção de Dados**

### **1. Criptografia**

#### **Dados Sensíveis**
```kotlin
// Padrão de criptografia para dados sensíveis
@Service
class EncryptionService {
    
    private val key = SecretKeySpec(ENCRYPTION_KEY.toByteArray(), "AES")
    private val cipher = Cipher.getInstance("AES/GCM/NoPadding")
    
    fun encrypt(data: String): String {
        cipher.init(Cipher.ENCRYPT_MODE, key)
        val iv = cipher.iv
        val encryptedData = cipher.doFinal(data.toByteArray())
        return Base64.getEncoder().encodeToString(iv + encryptedData)
    }
    
    fun decrypt(encryptedData: String): String {
        val data = Base64.getDecoder().decode(encryptedData)
        val iv = data.sliceArray(0..15)
        val encrypted = data.sliceArray(16 until data.size)
        
        cipher.init(Cipher.DECRYPT_MODE, key, GCMParameterSpec(128, iv))
        val decrypted = cipher.doFinal(encrypted)
        return String(decrypted)
    }
}
```

#### **Hashing de Senhas (se necessário)**
```kotlin
// Padrão de hashing (para casos especiais)
@Service
class PasswordService {
    
    fun hashPassword(password: String): String {
        val salt = generateSalt()
        val hash = BCrypt.hashpw(password, salt)
        return "$salt:$hash"
    }
    
    fun verifyPassword(password: String, hashedPassword: String): Boolean {
        val parts = hashedPassword.split(":")
        val salt = parts[0]
        val hash = parts[1]
        return BCrypt.checkpw(password, salt + hash)
    }
}
```

### **2. Sanitização de Dados**

#### **Input Validation**
```kotlin
// Padrão de validação de entrada
@Component
class InputValidator {
    
    fun validateEmail(email: String): String {
        val emailRegex = "^[A-Za-z0-9+_.-]+@[A-Za-z0-9.-]+\\.[A-Za-z]{2,}$".toRegex()
        if (!emailRegex.matches(email)) {
            throw ValidationException("Invalid email format")
        }
        return email.lowercase().trim()
    }
    
    fun sanitizeString(input: String): String {
        return input.trim()
            .replace(Regex("[<>\"'&]"), "") // Remove HTML/XML chars
            .replace(Regex("\\s+"), " ") // Normalize whitespace
    }
    
    fun validateUUID(uuid: String): UUID {
        return try {
            UUID.fromString(uuid)
        } catch (e: IllegalArgumentException) {
            throw ValidationException("Invalid UUID format")
        }
    }
}
```

---

## 📊 **Logging e Auditoria**

### **1. Padrão de Logs de Segurança**

#### **Estrutura de Log**
```json
{
  "timestamp": "2025-01-01T12:00:00Z",
  "level": "INFO|WARN|ERROR",
  "service": "moversemais-objective",
  "event": "AUTHENTICATION_SUCCESS|AUTHENTICATION_FAILURE|AUTHORIZATION_DENIED|DATA_ACCESS|DATA_MODIFICATION",
  "userId": "uuid-v4",
  "ip": "192.168.1.1",
  "userAgent": "Mozilla/5.0...",
  "endpoint": "/api/v1/objectives",
  "method": "POST",
  "requestId": "uuid-v4",
  "details": {
    "action": "create_objective",
    "resource": "objective",
    "resourceId": "uuid-v4"
  }
}
```

#### **Implementação em Microserviços**
```kotlin
// Padrão de logging de segurança
@Component
class SecurityLogger {
    
    private val logger = LoggerFactory.getLogger(SecurityLogger::class.java)
    
    fun logAuthenticationSuccess(userId: UUID, ip: String, userAgent: String) {
        logger.info("Authentication successful", mapOf(
            "event" to "AUTHENTICATION_SUCCESS",
            "userId" to userId.toString(),
            "ip" to ip,
            "userAgent" to userAgent
        ))
    }
    
    fun logDataAccess(userId: UUID, resource: String, action: String, requestId: String) {
        logger.info("Data access", mapOf(
            "event" to "DATA_ACCESS",
            "userId" to userId.toString(),
            "resource" to resource,
            "action" to action,
            "requestId" to requestId
        ))
    }
    
    fun logSecurityViolation(userId: UUID?, ip: String, violation: String, details: Map<String, Any>) {
        logger.warn("Security violation", mapOf(
            "event" to "SECURITY_VIOLATION",
            "userId" to userId?.toString(),
            "ip" to ip,
            "violation" to violation,
            "details" to details
        ))
    }
}
```

### **2. Monitoramento de Segurança**

#### **Métricas Obrigatórias**
```kotlin
// Padrão de métricas de segurança
@Component
class SecurityMetrics {
    
    private val authenticationAttempts = Counter.build()
        .name("authentication_attempts_total")
        .help("Total authentication attempts")
        .labelNames("provider", "result")
        .register()
    
    private val authorizationFailures = Counter.build()
        .name("authorization_failures_total")
        .help("Total authorization failures")
        .labelNames("resource", "action")
        .register()
    
    private val rateLimitHits = Counter.build()
        .name("rate_limit_hits_total")
        .help("Total rate limit hits")
        .labelNames("userId")
        .register()
    
    fun recordAuthenticationAttempt(provider: String, success: Boolean) {
        authenticationAttempts.labels(provider, if (success) "success" else "failure").inc()
    }
    
    fun recordAuthorizationFailure(resource: String, action: String) {
        authorizationFailures.labels(resource, action).inc()
    }
    
    fun recordRateLimitHit(userId: String) {
        rateLimitHits.labels(userId).inc()
    }
}
```

---

## 🚨 **Tratamento de Erros de Segurança**

### **1. Exceções de Segurança**

#### **Hierarquia de Exceções**
```kotlin
// Padrão de exceções de segurança
abstract class SecurityException(message: String, cause: Throwable? = null) : Exception(message, cause)

class AuthenticationException(message: String, cause: Throwable? = null) : SecurityException(message, cause)
class AuthorizationException(message: String, cause: Throwable? = null) : SecurityException(message, cause)
class TokenExpiredException(message: String = "Token expired") : SecurityException(message)
class InvalidTokenException(message: String = "Invalid token") : SecurityException(message)
class RateLimitExceededException(message: String = "Rate limit exceeded") : SecurityException(message)
class SecurityViolationException(message: String, cause: Throwable? = null) : SecurityException(message, cause)
```

#### **Tratamento Global**
```kotlin
// Padrão de tratamento global de exceções de segurança
@ControllerAdvice
class SecurityExceptionHandler {
    
    @ExceptionHandler(AuthenticationException::class)
    fun handleAuthenticationException(ex: AuthenticationException): ResponseEntity<ErrorResponse> {
        logger.warn("Authentication failed: ${ex.message}")
        return ResponseEntity.status(HttpStatus.UNAUTHORIZED)
            .body(ErrorResponse("AUTH_001", "Authentication failed"))
    }
    
    @ExceptionHandler(AuthorizationException::class)
    fun handleAuthorizationException(ex: AuthorizationException): ResponseEntity<ErrorResponse> {
        logger.warn("Authorization failed: ${ex.message}")
        return ResponseEntity.status(HttpStatus.FORBIDDEN)
            .body(ErrorResponse("AUTH_002", "Access denied"))
    }
    
    @ExceptionHandler(RateLimitExceededException::class)
    fun handleRateLimitExceeded(ex: RateLimitExceededException): ResponseEntity<ErrorResponse> {
        logger.warn("Rate limit exceeded: ${ex.message}")
        return ResponseEntity.status(HttpStatus.TOO_MANY_REQUESTS)
            .body(ErrorResponse("RATE_001", "Rate limit exceeded"))
    }
}
```

### **2. Códigos de Erro Padronizados**

#### **Códigos de Segurança**
```
AUTH_001: Authentication failed
AUTH_002: Access denied
AUTH_003: Token expired
AUTH_004: Invalid token
AUTH_005: Missing authentication headers
RATE_001: Rate limit exceeded
RATE_002: Too many requests from IP
SEC_001: Security violation detected
SEC_002: Invalid request origin
SEC_003: Request signature invalid
```

---

## 🔧 **Configurações de Segurança**

### **1. Variáveis de Ambiente**

#### **BFF (moversemais-store-graphql)**
```bash
# JWT Configuration
JWT_SECRET=your-jwt-secret-key
JWT_EXPIRATION=3600
JWT_ISSUER=moversemais-auth
JWT_AUDIENCE=moversemais-platform

# Rate Limiting
RATE_LIMIT_WINDOW_MS=60000
RATE_LIMIT_MAX_REQUESTS=100

# CORS
ALLOWED_ORIGINS=http://localhost:5173,https://moversemais.com
```

#### **Microserviços**
```bash
# Security
ALLOWED_BFF_IPS=127.0.0.1,::1,10.0.0.0/8
REQUEST_TIMEOUT_MS=30000
MAX_REQUEST_SIZE=1048576

# Logging
LOG_LEVEL=INFO
SECURITY_LOG_LEVEL=WARN
AUDIT_LOG_ENABLED=true
```

### **2. Configuração de Headers de Segurança**

#### **BFF (moversemais-store-graphql)**
```typescript
// Headers de segurança obrigatórios
const securityHeaders = {
  'Strict-Transport-Security': 'max-age=31536000; includeSubDomains',
  'X-Content-Type-Options': 'nosniff',
  'X-Frame-Options': 'DENY',
  'X-XSS-Protection': '1; mode=block',
  'Referrer-Policy': 'strict-origin-when-cross-origin',
  'Content-Security-Policy': "default-src 'self'; script-src 'self' 'unsafe-inline'"
};
```

#### **Microserviços**
```kotlin
// Configuração de segurança
@Configuration
class SecurityConfig {
    
    @Bean
    fun securityFilterChain(http: HttpSecurity): SecurityFilterChain {
        return http
            .csrf { it.disable() }
            .headers { headers ->
                headers
                    .frameOptions { it.deny() }
                    .contentTypeOptions { it.and() }
                    .httpStrictTransportSecurity { it.disable() }
            }
            .sessionManagement { it.sessionCreationPolicy(SessionCreationPolicy.STATELESS) }
            .build()
    }
}
```

---

## 📋 **Checklist de Implementação por Serviço**

### **Frontend (moversemais-store-front)**
- [ ] Implementar OAuth flow seguro
- [ ] Usar HttpOnly cookies para tokens
- [ ] Implementar logout seguro
- [ ] Adicionar proteção contra XSS
- [ ] Configurar CSP headers
- [ ] Implementar rate limiting no cliente

### **BFF (moversemais-store-graphql)**
- [ ] Implementar middleware de autenticação
- [ ] Adicionar validação JWT
- [ ] Implementar rate limiting por usuário
- [ ] Configurar logs de auditoria
- [ ] Adicionar validação de origem
- [ ] Implementar proxy seguro para microserviços

### **Auth Service (novo)**
- [ ] Implementar integração OAuth
- [ ] Gerar JWT com claims seguros
- [ ] Implementar refresh token
- [ ] Adicionar logs de autenticação
- [ ] Configurar rate limiting
- [ ] Implementar revogação de tokens

### **User Service (moversemais-user)**
- [ ] Implementar dados mínimos (LGPD)
- [ ] Adicionar validação de entrada
- [ ] Implementar logs de auditoria
- [ ] Configurar criptografia de dados sensíveis
- [ ] Adicionar validação de headers de segurança

### **Objective Service (moversemais-objective)**
- [ ] Adicionar interceptors de segurança
- [ ] Implementar validação de headers
- [ ] Configurar logs de acesso
- [ ] Adicionar validação de origem
- [ ] Implementar auditoria de operações

---

## 🚀 **Próximos Passos**

### **Fase 1: Implementação Base**
1. Criar Auth Service
2. Implementar OAuth no Frontend
3. Adicionar middleware de segurança no BFF
4. Configurar logs de auditoria

### **Fase 2: Proteção de APIs**
1. Implementar interceptors nos microserviços
2. Configurar validação de origem
3. Adicionar rate limiting
4. Implementar monitoramento

### **Fase 3: Compliance e Monitoramento**
1. Implementar métricas de segurança
2. Configurar alertas
3. Realizar testes de penetração
4. Documentar procedimentos de segurança

---

**Última Atualização**: Outubro 2025  
**Versão**: 1.0  
**Mantenedor**: Equipe MoverseMais

---

*Este documento deve ser atualizado sempre que houver mudanças nos padrões de segurança.*
