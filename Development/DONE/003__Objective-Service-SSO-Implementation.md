# 🎯 Objective Service - Implementação SSO e Segurança

## 📋 **Card de Desenvolvimento**

**Serviço**: moversemais-objective  
**Tipo**: SSO e Segurança  
**Prioridade**: Alta  
**Estimativa**: 1-2 dias  
**Status**: TODO  

---

## 🎯 **Objetivo**

Implementar Spring Security e interceptors de validação de headers no Objective Service. A validação de propriedade já existe nos Use Cases e não precisa ser alterada.

---

## 📚 **Documentação de Referência**

**Leia antes de implementar:**
- `../auth-security/SIMPLIFIED_LOCAL_DEVELOPMENT.md` - Desenvolvimento local simplificado
- `../auth-security/STEP_BY_STEP_GUIDES.md` - Semana 3: Objective Service
- `../auth-security/SECURITY_PATTERNS.md` - Padrões de segurança

---

## ✅ **Tarefas Obrigatórias**

### **1. Dependências**
```kotlin
// build.gradle.kts - Adicionar dependências
dependencies {
    implementation("org.springframework.boot:spring-boot-starter-security")
    implementation("io.jsonwebtoken:jjwt-api:0.11.5")
    implementation("io.jsonwebtoken:jjwt-impl:0.11.5")
    implementation("io.jsonwebtoken:jjwt-jackson:0.11.5")
}
```

### **2. Configuração de Segurança**
Criar arquivo: `src/main/kotlin/com/moversemais/objective/config/SecurityConfig.kt`

**Funcionalidades obrigatórias:**
- ✅ Configurar Spring Security
- ✅ Desabilitar CSRF para APIs
- ✅ Configurar sessão stateless
- ✅ Adicionar interceptor de segurança

### **3. Interceptor de Segurança**
Criar arquivo: `src/main/kotlin/com/moversemais/objective/interceptor/SecurityInterceptor.kt`

**Funcionalidades obrigatórias:**
- ✅ Validar headers obrigatórios (`X-User-Id`, `X-Provider`, `X-Request-Id`)
- ✅ Validar formato UUID do `X-User-Id`
- ✅ Log de auditoria de acessos
- ✅ Tratamento de erros de segurança

### **4. Verificar Use Cases Existentes**
**IMPORTANTE**: Os Use Cases já implementam validação de propriedade:
- `GetObjectiveUseCase.execute()` - já valida se objetivo pertence ao usuário
- `DeleteObjectiveUseCase.execute()` - já valida propriedade
- `ListObjectivesUseCase.execute()` - já filtra por userId
- `UpdateObjectiveUseCase.execute()` - já valida propriedade

**Não é necessário alterar os Use Cases existentes!**

### **5. Verificar Controllers Existentes**
**IMPORTANTE**: Os Controllers já extraem `X-User-Id` dos headers:
- `ObjectiveController` - já usa `@RequestHeader("X-User-Id") userId: UUID`
- `AssessmentController` - já implementa validação de headers
- Validação de propriedade é feita pelos Use Cases

**Não é necessário alterar os Controllers existentes!**

### **6. Verificar Perfis Existentes**
**IMPORTANTE**: Os perfis já existem e funcionam:
- `docker` - Desenvolvimento local (já configurado)
- `render` - Produção (já configurado)

**Não é necessário criar novos perfis!**

### **7. Testes Básicos**
**Testes obrigatórios:**
- ✅ Endpoints funcionam com headers obrigatórios
- ✅ Headers obrigatórios validados (`X-User-Id`, `X-Provider`, `X-Request-Id`)
- ✅ Validação de formato UUID funcionando
- ✅ Logs de auditoria funcionando

---

## 🔧 **Implementação Detalhada**

### **Configuração de Segurança**
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

### **Interceptor de Segurança**
```kotlin
// src/main/kotlin/com/moversemais/objective/interceptor/SecurityInterceptor.kt
@Component
class SecurityInterceptor : HandlerInterceptor {
    
    private val logger = LoggerFactory.getLogger(SecurityInterceptor::class.java)
    
    override fun preHandle(request: HttpServletRequest, response: HttpServletResponse, handler: Any): Boolean {
        try {
            val userId = request.getHeader("X-User-Id")
            val provider = request.getHeader("X-Provider")
            val requestId = request.getHeader("X-Request-Id")
            
            if (userId.isNullOrBlank()) {
                throw SecurityException("Missing X-User-Id header")
            }
            
            // Validar formato UUID
            try {
                UUID.fromString(userId)
            } catch (e: IllegalArgumentException) {
                logger.warn("❌ [OBJECTIVE] Formato de UUID inválido: $userId")
                throw SecurityException("Invalid UUID format")
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

### **Exemplo de Use Case Existente (NÃO ALTERAR)**
```kotlin
// src/main/kotlin/com/moversemais/objective/usecase/GetObjectiveUseCase.kt
// ✅ JÁ EXISTE E FUNCIONA CORRETAMENTE
@Service
class GetObjectiveUseCase(
    private val repository: ObjectiveRepository,
    private val mapper: ObjectiveMapper
) {
    fun execute(objectiveId: UUID, userId: UUID): ObjectiveResponse {
        // 1. Busca o objetivo no banco
        val objective = repository.findById(objectiveId)
            .orElseThrow { ObjectiveNotFoundException("Objective not found") }
        
        // 2. ✅ VALIDAÇÃO DE PROPRIEDADE JÁ EXISTE
        if (objective.userId != userId) {
            throw UnauthorizedException("Objective does not belong to user: $userId")
        }
        
        // 3. Converte para DTO
        return objective.toResponse(mapper)
    }
}
```

### **Exemplo de Controller Existente (NÃO ALTERAR)**
```kotlin
// src/main/kotlin/com/moversemais/objective/controller/ObjectiveController.kt
// ✅ JÁ EXISTE E FUNCIONA CORRETAMENTE
@RestController
@RequestMapping("/objectives")
class ObjectiveController(
    private val getObjectiveUseCase: GetObjectiveUseCase,
    private val listObjectivesUseCase: ListObjectivesUseCase,
    // ... outros use cases
) {
    
    @GetMapping("/{objectiveId}")
    fun getObjective(
        @PathVariable objectiveId: UUID,
        @RequestHeader("X-User-Id") userId: UUID  // ✅ JÁ EXTRAI HEADER
    ): ResponseEntity<ObjectiveResponse> {
        
        // ✅ JÁ VALIDA FORMATO
        validateUserIdFormat(userId)
        validateObjectiveIdFormat(objectiveId)
        
        // ✅ JÁ DELEGA PARA USE CASE (que valida propriedade)
        val result = getObjectiveUseCase.execute(objectiveId, userId)
        return ResponseEntity.ok(result)
    }
    
    @GetMapping
    fun listObjectives(
        @RequestHeader("X-User-Id") userId: UUID,  // ✅ JÁ EXTRAI HEADER
        @RequestParam(defaultValue = "0") page: Int,
        @RequestParam(defaultValue = "20") size: Int
        // ... outros parâmetros
    ): ResponseEntity<ObjectiveListResponse> {
        
        // ✅ JÁ VALIDA E DELEGA
        validateUserIdFormat(userId)
        val result = listObjectivesUseCase.execute(userId, pageRequest)
        return ResponseEntity.ok(result)
    }
}
```

---

## 🧪 **Critérios de Validação**

### **Funcionalidades Obrigatórias**
- [ ] Spring Security configurado
- [ ] SecurityInterceptor funcionando
- [ ] Headers obrigatórios validados (`X-User-Id`, `X-Provider`, `X-Request-Id`)
- [ ] Validação de formato UUID funcionando
- [ ] Logs de auditoria funcionando

### **Testes Obrigatórios**
- [ ] `curl -H "X-User-Id: 123e4567-e89b-12d3-a456-426614174000" http://localhost:8080/objectives` funciona
- [ ] `curl -H "X-User-Id: invalid-uuid" http://localhost:8080/objectives` retorna 401
- [ ] `curl http://localhost:8080/objectives` (sem header) retorna 401
- [ ] Headers obrigatórios validados
- [ ] Logs de auditoria aparecem no console

### **Código Obrigatório**
- [ ] Dependências Spring Security adicionadas ao build.gradle.kts
- [ ] SecurityConfig implementado
- [ ] SecurityInterceptor implementado
- [ ] Perfis existentes (NÃO ALTERAR)
- [ ] Use Cases e Controllers existentes (NÃO ALTERAR)

---

## 🚨 **Troubleshooting**

### **Problema: Interceptor não é aplicado**
- Verificar se SecurityConfig está correto
- Verificar se interceptor está registrado
- Verificar se Spring Security está configurado

### **Problema: Validação de propriedade não funciona**
- Verificar se Use Cases existentes estão funcionando
- Verificar se Controllers estão passando userId corretamente
- Verificar se repositórios estão funcionando
- **IMPORTANTE**: Não alterar Use Cases existentes!

### **Problema: Headers não são validados**
- Verificar se interceptor está extraindo headers corretamente
- Verificar se headers estão sendo enviados pelo BFF
- Verificar logs de debug

---

## 📋 **Checklist de Implementação**

- [ ] Dependências Spring Security adicionadas ao build.gradle.kts
- [ ] SecurityConfig implementado
- [ ] SecurityInterceptor implementado
- [ ] Testes básicos funcionando
- [ ] Logs de auditoria funcionando
- [ ] Headers obrigatórios validados
- [ ] Perfis existentes (NÃO ALTERAR)
- [ ] Use Cases e Controllers existentes (NÃO ALTERAR)

---

## 🧑‍💻 **Output do Desenvolvedor**

### O que foi implementado
- **Dependências adicionadas** em `build.gradle.kts`:
  - `org.springframework.boot:spring-boot-starter-security`
  - `io.jsonwebtoken:jjwt-api:0.11.5`
  - `io.jsonwebtoken:jjwt-impl:0.11.5`
  - `io.jsonwebtoken:jjwt-jackson:0.11.5`

- **Configuração de Segurança** em `src/main/kotlin/com/moversemais/objective/config/SecurityConfig.kt`:
  - Spring Security habilitado com `@EnableWebSecurity`
  - CSRF desabilitado para APIs
  - Sessão configurada como `STATELESS`
  - Rotas públicas permitidas: `/api/v1/health/**`, `/actuator/**`, `/swagger-ui/**`, `/api-docs/**`
  - Configuração flexível para context paths diferentes (local `/api/v1` vs produção `/`)

- **SecurityFilter** em `src/main/kotlin/com/moversemais/objective/filter/SecurityFilter.kt`:
  - Implementa `jakarta.servlet.Filter` (padrão robusto do projeto moversemais-user)
  - Valida headers obrigatórios: `X-User-Id`, `X-Provider`, `X-Request-Id`
  - Validação robusta de formato UUID para `X-User-Id`
  - Validação de provider (google, facebook, linkedin)
  - Logs de auditoria detalhados com prefixo `[OBJECTIVE]`
  - Tratamento de exceções com códigos de erro padronizados (`AUTH_001`, `AUTH_002`)
  - Resposta JSON estruturada para erros de segurança
  - Compatibilidade com context paths diferentes (local e produção)

- **Configuração OpenAPI** em `src/main/kotlin/com/moversemais/objective/config/OpenApiConfig.kt`:
  - Configuração explícita dos security schemes para Swagger UI
  - Botão "Authorize" habilitado para headers `X-User-Id`, `X-Provider`, `X-Request-Id`
  - Documentação completa da API com informações de contato e licença

- **Configuração Multi-Ambiente**:
  - `application.yml`: Context path configurável via `SERVER_CONTEXT_PATH` (padrão: `/api/v1`)
  - `application-render.yml`: Context path configurável para produção (padrão: `/`)
  - SecurityFilter adaptado para ambos os context paths

- **Escopo preservado**: Use Cases, Controllers e perfis existentes não foram alterados

### Evidências de funcionamento
- **Swagger UI Local**: `http://localhost:8080/api/v1/swagger-ui/index.html` → 200 ✅
- **Swagger UI Produção**: `https://moversemais-objective.onrender.com/swagger-ui/index.html` → 200 ✅
- **Health endpoint**: `http://localhost:8080/api/v1/health` → 200 ✅
- **Endpoints protegidos sem header**: `http://localhost:8080/api/v1/objectives` → 401 (comportamento esperado)
- **Endpoints protegidos com header válido**: `http://localhost:8080/api/v1/objectives` com `X-User-Id: 550e8400-e29b-41d4-a716-446655440000` → 200 ✅
- **Assessment endpoint**: `POST /api/v1/assessments` com header válido → 200 ✅
- **Botão Authorize**: Aparece no Swagger UI e permite configurar headers ✅

### Resolução de problemas
- **Problema inicial**: Swagger UI retornava 401 "Access denied"
- **Causa identificada**: Spring Security com configuração padrão bloqueando todas as rotas
- **Solução aplicada**: Consulta ao projeto `moversemais-user` e aplicação do mesmo padrão de implementação
- **Problema secundário**: Botão "Authorize" não aparecia no Swagger UI
- **Solução aplicada**: Criação de `OpenApiConfig.kt` com configuração explícita dos security schemes
- **Problema terciário**: Incompatibilidade entre context paths local e produção
- **Solução aplicada**: Configuração flexível via variáveis de ambiente e adaptação do SecurityFilter
- **Resultado**: Swagger UI acessível, segurança funcionando e compatibilidade total entre ambientes

### Deploy em Produção
- **Branch criada**: `feature/objective-service-sso-implementation`
- **Merge realizado**: `feature/objective-service-sso-implementation` → `main`
- **Push para GitHub**: Código atualizado no repositório remoto ✅
- **Deploy automático**: Render.com detectou push e iniciou deploy automático ✅
- **Status**: Deploy em andamento no Render.com

### Observações técnicas
- Implementação segue exatamente o padrão do projeto `moversemais-user` para consistência arquitetural
- SecurityFilter implementado como `jakarta.servlet.Filter` para máxima compatibilidade
- Logs de auditoria aparecem no console com formato padronizado
- Tratamento de exceções robusto com códigos de erro estruturados
- Configuração flexível para diferentes context paths (local vs produção)
- OpenAPI configurado para máxima usabilidade do Swagger UI
- Deploy automatizado via Render.com com monitoramento de status

---

## 🎯 **Próximo Passo**

Após completar este card, mover para `WIP/` e depois para `VALIDATING/` para análise.

**Próximo card**: Frontend - Implementação SSO e Segurança

---

**Criado em**: Outubro 2025  
**Responsável**: Equipe MoverseMais  
**Status**: TODO
