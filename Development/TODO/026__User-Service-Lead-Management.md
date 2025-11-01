# 🎯 Card 026 - User Service: Sistema de Gestão de Leads

**Agente Responsável:** Alex  
**Microserviço:** moversemais-user  
**Prioridade:** Alta  
**Status:** TODO  
**Estimativa:** 1 dia

---

## 📋 CONTEXTO

### **Situação Atual**
A plataforma MoverseMais está pronta para lançamento oficial do assessment de competências. Usuários completam o assessment de forma anônima e veem seus resultados na tela, mas não temos forma de capturar interesse desses usuários para futuro contato.

### **Problema Identificado**
- Perdemos contato com 100% dos usuários após o assessment
- Não temos lista de leads interessados na plataforma
- Não conseguimos notificar sobre novidades e evolução da plataforma
- Zero capacidade de marketing e aquisição de usuários

### **Solução Proposta**
Criar sistema de registro de leads que permita capturar email e preferências de comunicação de usuários que completaram o assessment, com compliance LGPD desde o início.

### **Onde se Encaixa na Arquitetura**
```
Frontend (Formulário de Lead)
    ↓
BFF GraphQL (Mutation registerLead)
    ↓
User Service - VOCÊ (POST /api/v1/leads) ← ESTE CARD
    ↓
PostgreSQL (tabela leads)
```

### **Impacto se Não For Feito**
- Impossível construir lista de interessados
- Zero capacidade de marketing
- Perda de oportunidade de conversão
- Lançamento sem captura de leads

---

## 🎯 REQUISITOS OBRIGATÓRIOS

### **1. Banco de Dados**

**Nova Tabela `leads` (Flyway Migration):**
- UUID como primary key
- Email armazenado de forma segura (LGPD compliant)
- Hash do email para busca rápida
- Source (origem do lead - ASSESSMENT, LANDING_PAGE, etc.)
- Dois campos de opt-in separados:
  - opt_in_platform_news (notícias da plataforma)
  - opt_in_blog_news (notícias do blog)
- Timestamps de criação e atualização
- Status do lead (NEW, CONTACTED, CONVERTED, UNSUBSCRIBED)
- Índices para performance

**Compliance LGPD Obrigatório:**
- Email DEVE ser criptografado no banco (usar padrão existente do projeto)
- Hash SHA-256 do email para busca sem expor dados
- Logs NÃO devem expor email (apenas hash)

### **2. API REST**

**Endpoint Principal:**
- `POST /api/v1/leads` - Registrar novo lead
  - Recebe: email, optInPlatformNews, optInBlogNews, source
  - Retorna: id do lead, mensagem de sucesso
  - Status HTTP: 201 Created (novo) ou 200 OK (já existe)
  - Comportamento idempotente: se email já existe, retorna sucesso

**Validações Obrigatórias:**
- Email válido (formato correto)
- Source válido (enum de opções permitidas)
- optInPlatformNews e optInBlogNews são booleanos

**Tratamento de Erros:**
- Email inválido: 400 Bad Request com mensagem clara
- Erro de banco: 500 Internal Server Error
- Duplicate email: Retornar sucesso (idempotente - UX)

### **3. Arquitetura**

**Camadas Obrigatórias:**
- Entity JPA (mapeamento da tabela)
- Repository (Spring Data JPA)
- Service (lógica de negócio - validação, criptografia, hash)
- Controller (REST endpoints - apenas delegação)

**Seguir Padrão Existente:**
- Controller → Service → Repository
- Service contém TODA lógica de negócio
- Controller apenas delega e trata HTTP

### **4. Segurança e LGPD**

- Email criptografado com AES (usar EncryptedStringConverter existente)
- Hash SHA-256 do email antes de criptografar
- Logs estruturados SEM expor email
- Documentação Swagger completa

### **5. Documentação**

- Swagger/OpenAPI para todos endpoints
- Comentários em código onde necessário
- Atualizar AGENTS.md do User Service se necessário

---

## ⚠️ RESTRIÇÕES

### **O que NÃO PODE ser alterado:**

1. ❌ **NÃO alterar tabela `users` existente**
2. ❌ **NÃO modificar LGPDController existente**
3. ❌ **NÃO alterar EncryptedStringConverter** (usar como está)
4. ❌ **NÃO alterar SecurityConfig** sem necessidade
5. ❌ **NÃO expor emails em logs** (apenas hash)
6. ❌ **NÃO quebrar padrões arquiteturais** existentes

### **O que DEVE ser preservado:**

1. ✅ **Padrão de criptografia** (AES via EncryptedStringConverter)
2. ✅ **Padrão de arquitetura** (Controller → Service → Repository)
3. ✅ **Padrão de nomenclatura** (PascalCase classes, camelCase métodos)
4. ✅ **Padrão de commits** (feat:, fix:, etc.)
5. ✅ **Padrão de logs** (estruturados com contexto)

---

## 📚 DOCUMENTAÇÃO DE REFERÊNCIA

### **Arquivos para Estudar (OBRIGATÓRIO):**

1. **Estrutura Existente:**
   - `src/main/kotlin/com/moversemais/user/entity/User.kt` - Padrão de entidade
   - `src/main/kotlin/com/moversemais/user/repository/UserRepository.kt` - Padrão de repository
   - `src/main/kotlin/com/moversemais/user/service/UserService.kt` - Padrão de service
   - `src/main/kotlin/com/moversemais/user/controller/LGPDController.kt` - Padrão de controller

2. **Segurança e LGPD:**
   - `src/main/kotlin/com/moversemais/user/config/EncryptedStringConverter.kt` - Criptografia AES
   - `src/main/resources/db/migration/V2__Update_users_table_lgpd.sql` - Exemplo de migração LGPD

3. **Configuração:**
   - `src/main/resources/application.yml` - Config Spring Boot
   - `build.gradle.kts` - Dependências disponíveis

4. **Documentação:**
   - `../moversemais-user/AGENTS.md` - Políticas do User Service
   - `../moversemais-arquitetura/AGENTS.md` - Visão geral da plataforma

### **Cards Relacionados:**
- Card 025: BFF Mutation registerLead (depende deste card)
- Card 024: Frontend Formulário de Lead (depende do 025)

---

## 🔧 WORKFLOW

### **1. ESTUDAR (OBRIGATÓRIO - Antes de Codificar)**

```bash
# Estudar estrutura do User Service
cd moversemais-user
tree src/main/kotlin/com/moversemais/user/

# Analisar padrões existentes
cat src/main/kotlin/com/moversemais/user/entity/User.kt
cat src/main/kotlin/com/moversemais/user/service/UserService.kt
cat src/main/kotlin/com/moversemais/user/controller/LGPDController.kt

# Verificar migração LGPD
cat src/main/resources/db/migration/V2__Update_users_table_lgpd.sql

# Verificar criptografia
cat src/main/kotlin/com/moversemais/user/config/EncryptedStringConverter.kt

# Ler AGENTS.md do projeto
cat AGENTS.md
```

**Perguntas para Responder Antes de Implementar:**
- Qual padrão de nomenclatura de pacotes?
- Como funciona a criptografia AES existente?
- Qual formato de DTO usado nos controllers?
- Qual padrão de tratamento de erros?
- Qual padrão de logs?

### **2. CRIAR BRANCH**

```bash
git checkout -b feature/lead-management
```

### **3. IMPLEMENTAR (VOCÊ DECIDE COMO)**

**Ordem Sugerida (mas você decide):**
1. Criar migration Flyway (V3__Create_leads_table.sql ou próximo número)
2. Criar entity Lead
3. Criar repository LeadRepository
4. Criar service LeadService (lógica de negócio)
5. Criar DTOs (request/response)
6. Criar controller LeadController
7. Configurar Swagger

**Você decide:**
- Nomes exatos de classes, métodos, variáveis
- Estrutura de pacotes (seguir padrão existente)
- Validações adicionais
- Mensagens de erro
- Estrutura de logs
- Tratamento de exceções customizado

### **4. TESTAR**

**Testes Manuais Obrigatórios:**

```bash
# 1. Registrar lead novo
curl -X POST http://localhost:8083/api/v1/leads \
  -H "Content-Type: application/json" \
  -d '{
    "email": "teste@moversemais.com",
    "optInPlatformNews": true,
    "optInBlogNews": true,
    "source": "ASSESSMENT"
  }'
# Esperado: 201 Created

# 2. Registrar mesmo email novamente (idempotência)
curl -X POST http://localhost:8083/api/v1/leads \
  -H "Content-Type: application/json" \
  -d '{
    "email": "teste@moversemais.com",
    "optInPlatformNews": false,
    "optInBlogNews": true,
    "source": "ASSESSMENT"
  }'
# Esperado: 200 OK (já existe)

# 3. Email inválido
curl -X POST http://localhost:8083/api/v1/leads \
  -H "Content-Type: application/json" \
  -d '{
    "email": "email-invalido",
    "optInPlatformNews": true,
    "optInBlogNews": true,
    "source": "ASSESSMENT"
  }'
# Esperado: 400 Bad Request

# 4. Verificar Swagger
open http://localhost:8083/swagger-ui/index.html

# 5. Verificar no banco (email deve estar criptografado)
# Conectar ao PostgreSQL e verificar tabela leads
```

**Testes Unitários (Recomendado):**
- LeadServiceTest (validação, hash, criptografia)
- LeadRepositoryTest (persistência)

### **5. DOCUMENTAR DECISÕES**

Ao final do card, documente:
- Que nomes escolheu e por quê
- Que validações adicionou além das obrigatórias
- Que estrutura de pacotes seguiu
- Dificuldades encontradas
- Melhorias implementadas

### **6. COMMIT E PUSH**

```bash
git add .
git commit -m "feat(user-service): adiciona sistema de gestão de leads

- Cria tabela leads com compliance LGPD
- Implementa endpoint POST /api/v1/leads
- Email criptografado com AES
- Hash SHA-256 para busca
- Comportamento idempotente
- Documentação Swagger completa"

git push origin feature/lead-management
```

### **7. MOVER PARA VALIDAÇÃO**

```bash
# Mover card
mv Development/TODO/026__User-Service-Lead-Management.md \
   Development/VALIDATING/026__User-Service-Lead-Management.md
```

---

## ✅ CRITÉRIOS DE VALIDAÇÃO

### **Funcionalidades:**
- [ ] Tabela `leads` criada no PostgreSQL
- [ ] Endpoint `POST /api/v1/leads` funcionando
- [ ] Email criptografado no banco (não legível)
- [ ] Hash do email sendo usado para busca
- [ ] Duplicate email retorna 200 OK (idempotente)
- [ ] Email inválido retorna 400 Bad Request
- [ ] opt_in_platform_news e opt_in_blog_news salvos corretamente
- [ ] Source sendo salvo corretamente
- [ ] Status default é "NEW"

### **Segurança e LGPD:**
- [ ] Email criptografado no banco
- [ ] Hash SHA-256 funcionando
- [ ] Logs NÃO expõem email (apenas hash)
- [ ] EncryptedStringConverter sendo usado

### **Qualidade de Código:**
- [ ] Arquitetura em camadas (Controller → Service → Repository)
- [ ] Service contém lógica de negócio
- [ ] Controller apenas delega
- [ ] Swagger documentado
- [ ] Código segue padrões do projeto
- [ ] Commits seguem convenção (feat:, fix:)

### **Testes:**
- [ ] Endpoint testado manualmente (curl)
- [ ] Swagger acessível e funcional
- [ ] Banco de dados verificado (email criptografado)
- [ ] Idempotência testada (mesmo email 2x)
- [ ] Validação de email testada

---

## 🚨 TROUBLESHOOTING

### **Problema: Flyway não executa migration**
**Solução:**
- Verificar numeração da migration (V3, V4, etc.)
- Verificar se banco está limpo ou resetar: `docker-compose down -v`
- Verificar sintaxe SQL

### **Problema: Email não está sendo criptografado**
**Solução:**
- Verificar se `@Convert(converter = EncryptedStringConverter::class)` está na entidade
- Verificar se EncryptedStringConverter existe e está configurado
- Verificar logs de inicialização do Spring

### **Problema: Duplicate key error**
**Solução:**
- Verificar se `UNIQUE` constraint no `email_hash`
- Tratar exceção de duplicate e retornar 200 OK (idempotente)

### **Problema: Hash não funciona**
**Solução:**
- Implementar função de hash SHA-256 no Service
- Usar `MessageDigest.getInstance("SHA-256")`

### **Problema: Swagger não aparece**
**Solução:**
- Verificar dependência `springdoc-openapi-starter-webmvc-ui` no build.gradle.kts
- Verificar se controller tem `@RestController` e `@RequestMapping`
- Acessar: http://localhost:8083/swagger-ui/index.html

---

## 🎯 EXPECTATIVAS

### **Você é o Especialista Técnico**

**Alex, você é o guardião do User Service.** Eu confio que você:

1. ✅ **Conhece Spring Boot + Kotlin** melhor que eu
2. ✅ **Conhece Spring Data JPA** melhor que eu
3. ✅ **Conhece padrões de segurança** melhor que eu
4. ✅ **Sabe como estruturar código** melhor que eu

**Por isso:**
- Estude o código existente profundamente
- Siga os padrões já estabelecidos
- Tome decisões técnicas fundamentadas
- Adicione validações que julgar necessárias
- Proponha melhorias se identificar oportunidades
- Documente suas escolhas

**Eu defini O QUE precisa ser feito. Você decide COMO fazer.**

**Se algo no card não está claro sobre O QUE fazer, é falha minha. Me avise.**

**Se você precisar decidir COMO fazer, é sua responsabilidade. Decida com confiança.**

---

## 📊 OUTPUT ESPERADO

Ao finalizar, documente aqui:

### **Decisões Técnicas Tomadas:**
(Você preenche)

### **Estrutura Criada:**
(Liste arquivos criados)

### **Testes Realizados:**
(Liste cenários testados)

### **Dificuldades Encontradas:**
(Se houver)

### **Melhorias Implementadas:**
(Além do requisitado)

---

**Data de Criação:** 01/11/2025  
**Criado por:** Arquiobaldo (Arquiteto MoverseMais)  
**Versão:** 1.0

