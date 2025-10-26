# 🔐 SSO e Segurança - Arquitetura MoverseMais

## 📋 **Visão Geral**

Este documento define os padrões arquiteturais para implementação de SSO (Single Sign-On) e segurança na plataforma MoverseMais, seguindo boas práticas de segurança e compliance LGPD.

---

## 🎯 **Objetivos de Segurança**

### **1. Autenticação Unificada**
- **SSO Exclusivo**: Google, Facebook e LinkedIn
- **Zero Senhas**: Eliminação completa de senhas locais
- **Experiência Unificada**: Login único para toda a plataforma

### **2. Compliance LGPD**
- **Minimização de Dados**: Apenas dados essenciais armazenados
- **Transparência**: Usuário sabe exatamente quais dados são coletados
- **Controle**: Usuário pode revogar acesso a qualquer momento

### **3. Segurança de APIs**
- **Proteção de Endpoints**: Backend não exposto publicamente
- **Autorização Granular**: Controle de acesso por recurso
- **Auditoria**: Logs de todas as operações sensíveis

---

## 🏗️ **Arquitetura de SSO**

### **Fluxo de Autenticação**
```
┌─────────────┐    ┌─────────────┐    ┌─────────────┐    ┌─────────────┐
│   Frontend  │    │   BFF       │    │   Auth      │    │   Provider  │
│   (React)   │    │   (Next.js) │    │   Service   │    │   (Google)  │
└─────────────┘    └─────────────┘    └─────────────┘    └─────────────┘
       │                   │                   │                   │
       │ 1. Redirect SSO   │                   │                   │
       ├──────────────────►│                   │                   │
       │                   │ 2. Auth URL       │                   │
       │                   ├──────────────────►│                   │
       │                   │                   │ 3. OAuth URL      │
       │                   │                   ├──────────────────►│
       │                   │                   │ 4. Auth Code      │
       │                   │                   ◄──────────────────┤
       │                   │ 5. Exchange Code  │                   │
       │                   ├──────────────────►│                   │
       │                   │ 6. JWT Token      │                   │
       │                   ◄──────────────────┤                   │
       │ 7. JWT + User     │                   │                   │
       ◄──────────────────┤                   │                   │
```

### **Componentes da Arquitetura**

#### **1. Frontend (moversemais-store-front)**
- **Responsabilidade**: Interface de login e gerenciamento de sessão
- **Tecnologia**: React + OAuth2/OpenID Connect
- **Bibliotecas**: `@auth0/nextjs-auth0` ou `next-auth`
- **Armazenamento**: HttpOnly cookies para tokens

#### **2. BFF (moversemais-store-graphql)**
- **Responsabilidade**: Proxy de autenticação e autorização
- **Tecnologia**: Next.js + JWT validation
- **Funções**:
  - Validar tokens JWT
  - Adicionar headers de autorização
  - Rate limiting por usuário
  - Logs de auditoria

#### **3. Auth Service (novo serviço)**
- **Responsabilidade**: Gerenciamento de autenticação centralizado
- **Tecnologia**: Spring Boot + Kotlin
- **Funções**:
  - Integração com providers SSO
  - Geração e validação de JWT
  - Gerenciamento de sessões
  - Refresh tokens

#### **4. User Service (moversemais-user)**
- **Responsabilidade**: Dados mínimos do usuário
- **Tecnologia**: Spring Boot + Kotlin
- **Dados Armazenados**:
  - ID único (UUID)
  - Email (hash para busca)
  - Nome (opcional)
  - Provider (google/facebook/linkedin)
  - Data de criação/último acesso
  - Status (ativo/inativo)

---

## 🔒 **Padrões de Segurança**

### **1. JWT (JSON Web Token)**
```json
{
  "header": {
    "alg": "RS256",
    "typ": "JWT"
  },
  "payload": {
    "sub": "user-uuid",
    "email": "user@example.com",
    "provider": "google",
    "iat": 1640995200,
    "exp": 1641081600,
    "iss": "moversemais-auth",
    "aud": "moversemais-platform"
  }
}
```

### **2. Estrutura de Claims**
- **sub**: User ID (UUID)
- **email**: Email do usuário
- **provider**: google/facebook/linkedin
- **iat**: Issued at timestamp
- **exp**: Expiration timestamp
- **iss**: Issuer (moversemais-auth)
- **aud**: Audience (moversemais-platform)

### **3. Headers de Autorização**
```
Authorization: Bearer <JWT_TOKEN>
X-User-Id: <USER_UUID>
X-Provider: google|facebook|linkedin
```

---

## 🛡️ **Controle de Acesso**

### **1. Estratégia de Proteção de APIs**

#### **Nível 1: BFF como Gateway**
- **Todas as requisições** passam pelo BFF
- **Validação JWT** obrigatória
- **Rate limiting** por usuário
- **Logs de auditoria** centralizados

#### **Nível 2: Microserviços Protegidos**
- **Network isolation**: Apenas BFF pode acessar
- **Validação de origem**: Verificar se requisição vem do BFF
- **Headers obrigatórios**: X-User-Id, X-Provider
- **Logs de acesso**: Todas as operações logadas

### **2. Implementação por Serviço**

#### **BFF (moversemais-store-graphql)**
```typescript
// Middleware de autenticação
const authMiddleware = (req: NextRequest) => {
  const token = req.headers.get('authorization')?.replace('Bearer ', '');
  if (!token) throw new UnauthorizedError();
  
  const payload = jwt.verify(token, JWT_SECRET);
  req.user = payload;
};

// Adicionar headers para microserviços
const addAuthHeaders = (user: User) => ({
  'X-User-Id': user.sub,
  'X-Provider': user.provider,
  'X-Request-Id': generateRequestId()
});
```

#### **Microserviços (Objective/User)**
```kotlin
// Interceptor de segurança
@Component
class SecurityInterceptor : HandlerInterceptor {
    override fun preHandle(request: HttpServletRequest, response: HttpServletResponse, handler: Any): Boolean {
        val userId = request.getHeader("X-User-Id")
        val provider = request.getHeader("X-Provider")
        
        if (userId.isNullOrEmpty() || provider.isNullOrEmpty()) {
            throw UnauthorizedException("Missing auth headers")
        }
        
        // Validar se requisição vem do BFF
        validateRequestOrigin(request)
        
        return true
    }
}
```

---

## 📊 **Dados Mínimos (LGPD Compliance)**

### **1. Dados Coletados**
```json
{
  "id": "uuid-v4",
  "email": "user@example.com",
  "name": "Nome do Usuário",
  "provider": "google",
  "providerId": "google-user-id",
  "createdAt": "2025-01-01T00:00:00Z",
  "lastAccessAt": "2025-01-01T00:00:00Z",
  "status": "active"
}
```

### **2. Dados NÃO Coletados**
- ❌ Senhas (não existem)
- ❌ Dados pessoais sensíveis
- ❌ Histórico de navegação
- ❌ Dados de localização
- ❌ Informações de contato adicionais

### **3. Política de Retenção**
- **Dados ativos**: Mantidos enquanto usuário ativo
- **Dados inativos**: Removidos após 2 anos de inatividade
- **Logs de auditoria**: Mantidos por 1 ano
- **Dados deletados**: Remoção física em 30 dias

---

## 🔧 **Implementação por Serviço**

### **1. Frontend (moversemais-store-front)**
```typescript
// Configuração OAuth
const authConfig = {
  providers: [
    GoogleProvider({
      clientId: process.env.GOOGLE_CLIENT_ID,
      clientSecret: process.env.GOOGLE_CLIENT_SECRET,
    }),
    FacebookProvider({
      clientId: process.env.FACEBOOK_CLIENT_ID,
      clientSecret: process.env.FACEBOOK_CLIENT_SECRET,
    }),
    LinkedInProvider({
      clientId: process.env.LINKEDIN_CLIENT_ID,
      clientSecret: process.env.LINKEDIN_CLIENT_SECRET,
    })
  ],
  callbacks: {
    async jwt({ token, account, profile }) {
      if (account && profile) {
        // Enviar dados para Auth Service
        const userData = await createOrUpdateUser(profile, account);
        token.userId = userData.id;
        token.provider = account.provider;
      }
      return token;
    }
  }
};
```

### **2. BFF (moversemais-store-graphql)**
```typescript
// Middleware de autenticação
export const authMiddleware = async (req: NextRequest) => {
  const token = extractToken(req);
  const user = await validateToken(token);
  
  // Adicionar headers para microserviços
  req.headers.set('X-User-Id', user.sub);
  req.headers.set('X-Provider', user.provider);
  req.headers.set('X-Request-Id', generateRequestId());
  
  return user;
};

// Rate limiting por usuário
const rateLimiter = new Map<string, { count: number; resetTime: number }>();
```

### **3. Auth Service (novo)**
```kotlin
@Service
class AuthService {
    fun createOrUpdateUser(profile: OAuthProfile, account: OAuthAccount): User {
        val user = userRepository.findByProviderAndProviderId(account.provider, account.providerId)
            ?: User(
                id = UUID.randomUUID(),
                email = profile.email,
                name = profile.name,
                provider = account.provider,
                providerId = account.providerId
            )
        
        user.lastAccessAt = Instant.now()
        return userRepository.save(user)
    }
    
    fun generateJWT(user: User): String {
        return Jwts.builder()
            .setSubject(user.id.toString())
            .claim("email", user.email)
            .claim("provider", user.provider)
            .setIssuedAt(Date())
            .setExpiration(Date(System.currentTimeMillis() + 3600000)) // 1 hora
            .signWith(SignatureAlgorithm.RS256, privateKey)
            .compact()
    }
}
```

### **4. User Service (moversemais-user)**
```kotlin
// Entidade mínima
@Entity
data class User(
    @Id val id: UUID,
    val email: String,
    val name: String?,
    val provider: String,
    val providerId: String,
    val createdAt: Instant = Instant.now(),
    val lastAccessAt: Instant = Instant.now(),
    val status: UserStatus = UserStatus.ACTIVE
)

// Endpoints protegidos
@RestController
@RequestMapping("/api/v1/users")
class UserController {
    
    @GetMapping("/me")
    fun getCurrentUser(@RequestHeader("X-User-Id") userId: String): User {
        return userService.findById(UUID.fromString(userId))
    }
    
    @DeleteMapping("/me")
    fun deleteCurrentUser(@RequestHeader("X-User-Id") userId: String) {
        userService.deleteUser(UUID.fromString(userId))
    }
}
```

---

## 🚨 **Monitoramento e Auditoria**

### **1. Logs Obrigatórios**
- **Login/Logout**: Timestamp, IP, provider, user ID
- **Acesso a APIs**: Endpoint, método, user ID, timestamp
- **Operações sensíveis**: Criação/edição/deleção de dados
- **Erros de autenticação**: Tentativas de acesso não autorizado

### **2. Métricas de Segurança**
- **Taxa de login**: Sucessos vs falhas
- **Tempo de sessão**: Duração média das sessões
- **Acessos por usuário**: Frequência de uso
- **Erros de autorização**: Tentativas de acesso negado

### **3. Alertas**
- **Múltiplas falhas de login**: > 5 tentativas em 5 minutos
- **Acesso de IP suspeito**: IP não reconhecido
- **Sessões longas**: > 24 horas ativas
- **Alta frequência de requests**: Possível ataque DDoS

---

## 📋 **Checklist de Implementação**

### **Fase 1: Infraestrutura Base**
- [ ] Criar Auth Service
- [ ] Configurar OAuth providers (Google, Facebook, LinkedIn)
- [ ] Implementar JWT generation/validation
- [ ] Configurar network isolation

### **Fase 2: Frontend**
- [ ] Implementar OAuth flow no React
- [ ] Configurar token storage (HttpOnly cookies)
- [ ] Implementar logout e refresh token
- [ ] Adicionar proteção de rotas

### **Fase 3: BFF**
- [ ] Implementar middleware de autenticação
- [ ] Adicionar rate limiting
- [ ] Configurar logs de auditoria
- [ ] Implementar proxy para microserviços

### **Fase 4: Microserviços**
- [ ] Adicionar interceptors de segurança
- [ ] Implementar validação de headers
- [ ] Configurar logs de acesso
- [ ] Atualizar User Service com dados mínimos

### **Fase 5: Monitoramento**
- [ ] Configurar logs centralizados
- [ ] Implementar métricas de segurança
- [ ] Configurar alertas
- [ ] Testes de penetração

---

## 🔄 **Migração de Dados**

### **1. Usuários Existentes**
- **Estratégia**: Migração gradual via SSO
- **Processo**: Usuário faz login com SSO, sistema associa conta existente
- **Fallback**: Manter dados antigos por 90 dias

### **2. Sessões Ativas**
- **Invalidar**: Todas as sessões atuais
- **Redirecionar**: Para novo fluxo de login
- **Notificar**: Usuários sobre mudança

---

## 📚 **Referências e Padrões**

### **OAuth 2.0 / OpenID Connect**
- [RFC 6749 - OAuth 2.0](https://tools.ietf.org/html/rfc6749)
- [OpenID Connect Core](https://openid.net/specs/openid-connect-core-1_0.html)

### **JWT**
- [RFC 7519 - JSON Web Token](https://tools.ietf.org/html/rfc7519)
- [JWT Best Practices](https://tools.ietf.org/html/draft-ietf-oauth-jwt-bcp)

### **LGPD**
- [Lei Geral de Proteção de Dados](https://www.gov.br/cidadania/pt-br/acesso-a-informacao/lgpd)
- [Guia de Implementação LGPD](https://www.gov.br/cidadania/pt-br/acesso-a-informacao/lgpd/guia-de-implementacao)

---

**Última Atualização**: Outubro 2025  
**Versão**: 1.0  
**Mantenedor**: Equipe MoverseMais

---

*Este documento deve ser atualizado sempre que houver mudanças na arquitetura de segurança.*
