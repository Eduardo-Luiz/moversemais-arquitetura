# 📋 LGPD Compliance - MoverseMais

## 📋 **Visão Geral**

Este documento define as práticas de compliance com a Lei Geral de Proteção de Dados (LGPD) que todos os serviços da plataforma MoverseMais devem seguir, garantindo proteção dos dados pessoais dos usuários.

---

## 🎯 **Princípios da LGPD**

### **1. Finalidade**
- **Dados coletados** apenas para finalidades específicas e legítimas
- **Transparência** sobre o uso dos dados
- **Necessidade** de coleta para o funcionamento da plataforma

### **2. Adequação**
- **Dados adequados** à finalidade declarada
- **Não excessivos** em relação à finalidade
- **Relevantes** para o serviço prestado

### **3. Necessidade**
- **Mínimo necessário** para atingir a finalidade
- **Eliminação** de dados desnecessários
- **Retenção** apenas pelo tempo necessário

### **4. Livre Acesso**
- **Transparência** sobre o tratamento de dados
- **Informações claras** sobre coleta e uso
- **Facilidade** de acesso às informações

### **5. Qualidade dos Dados**
- **Exatidão** e atualização dos dados
- **Integridade** e segurança
- **Precisão** das informações

---

## 📊 **Dados Pessoais na MoverseMais**

### **1. Dados Coletados (Mínimos)**

#### **Dados de Identificação**
```json
{
  "id": "uuid-v4",                    // Identificador único interno
  "email": "user@example.com",        // Email (hash para busca)
  "name": "Nome do Usuário",          // Nome (opcional)
  "provider": "google",               // Provedor OAuth
  "providerId": "google-user-id"      // ID no provedor OAuth
}
```

#### **Dados de Sessão**
```json
{
  "createdAt": "2025-01-01T00:00:00Z",     // Data de criação da conta
  "lastAccessAt": "2025-01-01T00:00:00Z",  // Último acesso
  "status": "active",                      // Status da conta
  "timezone": "America/Sao_Paulo"          // Fuso horário (opcional)
}
```

#### **Dados de Uso (Anonimizados)**
```json
{
  "assessmentCount": 3,              // Número de assessments realizados
  "objectivesCount": 15,             // Número de objetivos criados
  "lastAssessmentDate": "2025-01-01T00:00:00Z", // Data do último assessment
  "preferredLanguage": "pt-BR"       // Idioma preferido
}
```

### **2. Dados NÃO Coletados**

#### **Dados Sensíveis Proibidos**
- ❌ **Dados biométricos** (impressão digital, reconhecimento facial)
- ❌ **Dados de saúde** (condições médicas, medicamentos)
- ❌ **Dados financeiros** (renda, patrimônio, investimentos)
- ❌ **Dados de localização** (GPS, endereço residencial)
- ❌ **Dados de comunicação** (mensagens, chamadas)
- ❌ **Dados de navegação** (histórico de sites, cookies de terceiros)

#### **Dados Desnecessários**
- ❌ **Senhas** (não existem na plataforma)
- ❌ **Documentos pessoais** (CPF, RG, CNH)
- ❌ **Dados de contato** (telefone, endereço)
- ❌ **Dados de relacionamento** (estado civil, filhos)
- ❌ **Dados de preferências** (hobbies, interesses pessoais)

---

## 🔒 **Base Legal para Tratamento**

### **1. Consentimento (Art. 7º, I)**
- **Finalidade**: Fornecimento de serviços da plataforma
- **Escopo**: Dados mínimos para autenticação e funcionalidades
- **Revogação**: Usuário pode revogar a qualquer momento
- **Forma**: Consentimento explícito via OAuth

### **2. Execução de Contrato (Art. 7º, V)**
- **Finalidade**: Prestação de serviços contratados
- **Escopo**: Dados necessários para funcionalidades
- **Necessidade**: Essencial para o funcionamento da plataforma

### **3. Legítimo Interesse (Art. 7º, IX)**
- **Finalidade**: Melhoria dos serviços e segurança
- **Escopo**: Dados anonimizados para analytics
- **Ponderação**: Interesse legítimo vs. privacidade do usuário

---

## 🛡️ **Medidas de Segurança**

### **1. Criptografia de Dados**

#### **Dados em Trânsito**
```typescript
// Padrão obrigatório para comunicação
const secureConfig = {
  protocol: 'HTTPS',
  tlsVersion: 'TLS 1.3',
  cipherSuites: ['TLS_AES_256_GCM_SHA384', 'TLS_CHACHA20_POLY1305_SHA256'],
  hsts: 'max-age=31536000; includeSubDomains'
};
```

#### **Dados em Repouso**
```kotlin
// Padrão de criptografia para dados sensíveis
@Entity
data class User(
    @Id val id: UUID,
    
    // Email criptografado para busca
    @Column(name = "email_hash")
    val emailHash: String, // SHA-256 hash do email
    
    // Dados não sensíveis em texto claro
    val name: String?,
    val provider: String,
    val providerId: String,
    
    // Dados sensíveis criptografados
    @Convert(converter = EncryptedStringConverter::class)
    val email: String, // Email original criptografado
    
    val createdAt: Instant,
    val lastAccessAt: Instant,
    val status: UserStatus
)
```

### **2. Controle de Acesso**

#### **Princípio do Menor Privilégio**
```kotlin
// Padrão de controle de acesso
enum class DataAccessLevel {
    PUBLIC,      // Dados públicos (não sensíveis)
    INTERNAL,    // Dados internos (equipe técnica)
    SENSITIVE,   // Dados sensíveis (apenas sistema)
    RESTRICTED   // Dados restritos (apenas usuário)
}

// Aplicação do controle de acesso
fun getUserData(userId: UUID, requesterId: UUID, level: DataAccessLevel): UserData {
    return when (level) {
        DataAccessLevel.PUBLIC -> getPublicUserData(userId)
        DataAccessLevel.INTERNAL -> getInternalUserData(userId)
        DataAccessLevel.SENSITIVE -> {
            if (userId == requesterId) getSensitiveUserData(userId)
            else throw AccessDeniedException("Cannot access sensitive data of other users")
        }
        DataAccessLevel.RESTRICTED -> {
            if (userId == requesterId) getRestrictedUserData(userId)
            else throw AccessDeniedException("Cannot access restricted data of other users")
        }
    }
}
```

### **3. Logs de Auditoria**

#### **Estrutura de Log LGPD**
```json
{
  "timestamp": "2025-01-01T12:00:00Z",
  "event": "DATA_ACCESS|DATA_MODIFICATION|DATA_DELETION|CONSENT_GIVEN|CONSENT_REVOKED",
  "userId": "uuid-v4",
  "dataSubject": "uuid-v4",
  "dataType": "personal_data|sensitive_data|anonymized_data",
  "purpose": "authentication|service_provision|analytics|security",
  "legalBasis": "consent|contract|legitimate_interest",
  "action": "read|create|update|delete|export|anonymize",
  "ip": "192.168.1.1",
  "userAgent": "Mozilla/5.0...",
  "retentionPeriod": "2_years|1_year|6_months",
  "details": {
    "fieldsAccessed": ["email", "name"],
    "reason": "user_profile_update",
    "consentVersion": "1.0"
  }
}
```

---

## 📋 **Direitos dos Titulares**

### **1. Confirmação e Acesso (Art. 9º)**
```kotlin
// Endpoint para confirmação de tratamento
@GetMapping("/api/v1/users/me/data-confirmation")
fun confirmDataTreatment(@RequestHeader("X-User-Id") userId: String): DataTreatmentConfirmation {
    val user = userService.findById(UUID.fromString(userId))
    return DataTreatmentConfirmation(
        isDataProcessed = true,
        purposes = listOf("authentication", "service_provision", "analytics"),
        legalBasis = listOf("consent", "contract", "legitimate_interest"),
        dataTypes = listOf("identification", "contact", "usage"),
        retentionPeriod = "2_years"
    )
}

// Endpoint para acesso aos dados
@GetMapping("/api/v1/users/me/data")
fun getUserData(@RequestHeader("X-User-Id") userId: String): UserDataExport {
    val user = userService.findById(UUID.fromString(userId))
    return UserDataExport(
        personalData = user.toPersonalData(),
        processedData = user.toProcessedData(),
        consentHistory = consentService.getConsentHistory(userId),
        dataSources = listOf("oauth_google", "oauth_facebook", "oauth_linkedin")
    )
}
```

### **2. Correção (Art. 10º)**
```kotlin
// Endpoint para correção de dados
@PutMapping("/api/v1/users/me")
fun updateUserData(
    @RequestHeader("X-User-Id") userId: String,
    @RequestBody @Valid request: UpdateUserDataRequest
): UserData {
    val user = userService.findById(UUID.fromString(userId))
    
    // Validar se dados podem ser alterados
    validateDataModification(user, request)
    
    // Atualizar dados
    val updatedUser = user.copy(
        name = request.name,
        email = request.email,
        updatedAt = Instant.now()
    )
    
    // Log de auditoria
    auditService.logDataModification(userId, "user_data", "update", request)
    
    return userService.save(updatedUser)
}
```

### **3. Anonimização e Exclusão (Art. 11º e 12º)**
```kotlin
// Endpoint para anonimização
@PostMapping("/api/v1/users/me/anonymize")
fun anonymizeUserData(@RequestHeader("X-User-Id") userId: String): AnonymizationResult {
    val user = userService.findById(UUID.fromString(userId))
    
    // Anonimizar dados pessoais
    val anonymizedUser = user.copy(
        email = "anonymized_${user.id}@deleted.local",
        name = "Usuário Anonimizado",
        status = UserStatus.ANONYMIZED,
        anonymizedAt = Instant.now()
    )
    
    // Manter dados anonimizados para analytics
    val anonymizedData = AnonymizedUserData(
        id = user.id,
        createdAt = user.createdAt,
        lastAccessAt = user.lastAccessAt,
        assessmentCount = user.assessmentCount,
        objectivesCount = user.objectivesCount
    )
    
    // Salvar dados anonimizados
    anonymizedDataService.save(anonymizedData)
    
    // Remover dados pessoais
    userService.delete(user)
    
    // Log de auditoria
    auditService.logDataAnonymization(userId, "user_data", "anonymize")
    
    return AnonymizationResult(
        success = true,
        anonymizedDataId = anonymizedData.id,
        message = "Dados anonimizados com sucesso"
    )
}

// Endpoint para exclusão completa
@DeleteMapping("/api/v1/users/me")
fun deleteUserData(@RequestHeader("X-User-Id") userId: String): DeletionResult {
    val user = userService.findById(UUID.fromString(userId))
    
    // Excluir todos os dados do usuário
    userService.deleteAllUserData(userId)
    objectiveService.deleteAllUserData(userId)
    assessmentService.deleteAllUserData(userId)
    
    // Log de auditoria
    auditService.logDataDeletion(userId, "all_user_data", "delete")
    
    return DeletionResult(
        success = true,
        message = "Todos os dados foram excluídos com sucesso"
    )
}
```

### **4. Portabilidade (Art. 13º)**
```kotlin
// Endpoint para exportação de dados
@GetMapping("/api/v1/users/me/export")
fun exportUserData(@RequestHeader("X-User-Id") userId: String): UserDataExport {
    val user = userService.findById(UUID.fromString(userId))
    val objectives = objectiveService.findByUserId(userId)
    val assessments = assessmentService.findByUserId(userId)
    
    return UserDataExport(
        personalData = user.toPersonalData(),
        objectives = objectives.map { it.toExportData() },
        assessments = assessments.map { it.toExportData() },
        exportDate = Instant.now(),
        format = "json",
        version = "1.0"
    )
}
```

---

## ⏰ **Política de Retenção de Dados**

### **1. Períodos de Retenção**

#### **Dados Pessoais**
- **Dados ativos**: Enquanto usuário ativo (último acesso < 2 anos)
- **Dados inativos**: 2 anos após último acesso
- **Dados de sessão**: 30 dias
- **Logs de auditoria**: 1 ano

#### **Dados Anonimizados**
- **Dados de analytics**: 5 anos
- **Dados de pesquisa**: 10 anos
- **Dados de segurança**: 3 anos

### **2. Processo de Retenção**

#### **Rotina de Limpeza**
```kotlin
// Rotina de limpeza automática
@Component
class DataRetentionService {
    
    @Scheduled(cron = "0 0 2 * * ?") // Diariamente às 2h
    fun cleanupExpiredData() {
        val cutoffDate = Instant.now().minus(2, ChronoUnit.YEARS)
        
        // Identificar usuários inativos
        val inactiveUsers = userService.findInactiveUsers(cutoffDate)
        
        inactiveUsers.forEach { user ->
            // Anonimizar dados pessoais
            anonymizeUserData(user.id)
            
            // Manter dados anonimizados para analytics
            preserveAnonymizedData(user)
            
            // Log de auditoria
            auditService.logDataRetention(user.id, "anonymize_inactive_user")
        }
    }
    
    @Scheduled(cron = "0 0 3 * * ?") // Diariamente às 3h
    fun cleanupExpiredLogs() {
        val cutoffDate = Instant.now().minus(1, ChronoUnit.YEARS)
        
        // Limpar logs de auditoria antigos
        auditService.deleteExpiredLogs(cutoffDate)
        
        // Limpar logs de sessão antigos
        sessionService.deleteExpiredSessions(cutoffDate)
    }
}
```

---

## 🔍 **Monitoramento e Conformidade**

### **1. Métricas de Compliance**

#### **Indicadores de Conformidade**
```kotlin
// Métricas de compliance LGPD
@Component
class LGPDMetrics {
    
    private val dataAccessRequests = Counter.build()
        .name("lgpd_data_access_requests_total")
        .help("Total data access requests")
        .labelNames("type", "status")
        .register()
    
    private val dataModifications = Counter.build()
        .name("lgpd_data_modifications_total")
        .help("Total data modifications")
        .labelNames("type", "action")
        .register()
    
    private val dataDeletions = Counter.build()
        .name("lgpd_data_deletions_total")
        .help("Total data deletions")
        .labelNames("type", "method")
        .register()
    
    fun recordDataAccess(type: String, status: String) {
        dataAccessRequests.labels(type, status).inc()
    }
    
    fun recordDataModification(type: String, action: String) {
        dataModifications.labels(type, action).inc()
    }
    
    fun recordDataDeletion(type: String, method: String) {
        dataDeletions.labels(type, method).inc()
    }
}
```

### **2. Alertas de Compliance**

#### **Alertas Obrigatórios**
```kotlin
// Sistema de alertas de compliance
@Component
class ComplianceAlerts {
    
    fun checkDataRetentionCompliance() {
        val expiredData = dataRetentionService.findExpiredData()
        if (expiredData.isNotEmpty()) {
            alertService.sendAlert(
                type = "DATA_RETENTION_VIOLATION",
                message = "Found ${expiredData.size} expired data records",
                severity = "HIGH",
                action = "IMMEDIATE_CLEANUP_REQUIRED"
            )
        }
    }
    
    fun checkConsentCompliance() {
        val usersWithoutConsent = userService.findUsersWithoutValidConsent()
        if (usersWithoutConsent.isNotEmpty()) {
            alertService.sendAlert(
                type = "CONSENT_VIOLATION",
                message = "Found ${usersWithoutConsent.size} users without valid consent",
                severity = "CRITICAL",
                action = "IMMEDIATE_ACTION_REQUIRED"
            )
        }
    }
}
```

---

## 📋 **Checklist de Implementação LGPD**

### **Fase 1: Estrutura Base**
- [ ] Implementar coleta mínima de dados
- [ ] Configurar criptografia de dados sensíveis
- [ ] Implementar logs de auditoria
- [ ] Configurar política de retenção

### **Fase 2: Direitos dos Titulares**
- [ ] Implementar endpoints de acesso aos dados
- [ ] Implementar correção de dados
- [ ] Implementar anonimização
- [ ] Implementar exclusão de dados
- [ ] Implementar portabilidade

### **Fase 3: Monitoramento**
- [ ] Implementar métricas de compliance
- [ ] Configurar alertas de violação
- [ ] Implementar rotinas de limpeza
- [ ] Configurar relatórios de conformidade

### **Fase 4: Documentação**
- [ ] Criar política de privacidade
- [ ] Documentar base legal
- [ ] Criar termos de uso
- [ ] Implementar consentimento explícito

---

## 📚 **Documentação Obrigatória**

### **1. Política de Privacidade**
- Finalidades do tratamento
- Base legal para cada finalidade
- Dados coletados e fontes
- Compartilhamento de dados
- Direitos dos titulares
- Contato do controlador

### **2. Termos de Uso**
- Condições de uso da plataforma
- Responsabilidades do usuário
- Limitações de responsabilidade
- Modificações dos termos

### **3. Consentimento**
- Consentimento explícito e específico
- Informações claras sobre tratamento
- Facilidade de revogação
- Registro do consentimento

---

## 🚨 **Plano de Resposta a Incidentes**

### **1. Classificação de Incidentes**
- **Baixo**: Acesso não autorizado a dados não sensíveis
- **Médio**: Acesso não autorizado a dados pessoais
- **Alto**: Acesso não autorizado a dados sensíveis
- **Crítico**: Vazamento em massa de dados pessoais

### **2. Procedimentos de Resposta**
1. **Identificação**: Detectar e classificar o incidente
2. **Contenção**: Isolar e conter o incidente
3. **Investigação**: Analisar causa e impacto
4. **Notificação**: Informar autoridades e titulares
5. **Remediação**: Corrigir vulnerabilidades
6. **Documentação**: Registrar lições aprendidas

---

## 📞 **Contato e Responsabilidades**

### **1. Encarregado de Dados (DPO)**
- **Nome**: [A definir]
- **Email**: dpo@moversemais.com
- **Telefone**: [A definir]
- **Responsabilidades**: Supervisão do cumprimento da LGPD

### **2. Controlador de Dados**
- **Empresa**: MoverseMais
- **CNPJ**: [A definir]
- **Endereço**: [A definir]
- **Responsabilidades**: Decisões sobre tratamento de dados

### **3. Operador de Dados**
- **Empresa**: [Provedor de hospedagem]
- **Responsabilidades**: Execução do tratamento de dados

---

**Última Atualização**: Outubro 2025  
**Versão**: 1.0  
**Mantenedor**: Equipe MoverseMais  
**Aprovação**: [A definir]

---

*Este documento deve ser atualizado sempre que houver mudanças na legislação ou nas práticas da empresa.*
