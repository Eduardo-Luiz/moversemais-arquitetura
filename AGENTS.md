# 🤖 AGENTS.md - MoverseMais Platform Architecture

## 📋 **Visão Geral da Plataforma**

**Nome**: MoverseMais  
**Tipo**: Plataforma de desenvolvimento contínuo para líderes de tecnologia  
**Arquitetura**: Microserviços com BFF GraphQL  
**Status**: Em desenvolvimento ativo  
**Público-alvo**: Líderes técnicos (tech leads, engineering managers, squad leads)

---

## 📋 **Políticas de Trabalho**

### **🔍 Regra Fundamental #1: Estudar Antes de Propor**

**IMPORTANTE**: Antes de propor qualquer mudança em uma aplicação existente, o arquiteto DEVE:

1. **Ler e entender a estrutura atual** da aplicação
2. **Analisar arquivos de configuração** (build.gradle.kts, application.yml, etc.)
3. **Estudar a arquitetura implementada** (Clean Architecture, padrões, etc.)
4. **Verificar funcionalidades existentes** (Use Cases, Controllers, Services)
5. **Identificar o que já funciona** e não precisa ser alterado
6. **Propor apenas o que realmente é necessário**

### **❌ Problemas Evitados:**
- Propor criação de classes que já existem
- Sugerir alterações em código que já funciona
- Ignorar padrões arquiteturais já estabelecidos
- Criar configurações desnecessárias
- Duplicar funcionalidades existentes

### **✅ Benefícios:**
- Propostas mais precisas e focadas
- Respeito à arquitetura existente
- Menor risco de quebrar funcionalidades
- Implementação mais eficiente
- Melhor aproveitamento do código existente

### **📝 Exemplo de Boa Prática:**
```
ANTES: "Criar ObjectiveSecurityService para validação de propriedade"
DEPOIS: "Use Cases já validam propriedade, apenas adicionar Spring Security"
```

---

### **🎯 Regra Fundamental #2: Empoderar a IA Desenvolvedora como Especialista**

**PRINCÍPIO ARQUITETURAL**: O Arquiteto define **O QUE** precisa ser feito, a IA Desenvolvedora decide **COMO** será implementado.

#### **🏗️ Papel do Arquiteto:**

**O Arquiteto DEVE:**
- ✅ Definir **requisitos funcionais** claros e objetivos
- ✅ Estabelecer **restrições arquiteturais** (o que não pode ser alterado)
- ✅ Especificar **critérios de validação** (como saber se está correto)
- ✅ Fornecer **contexto de negócio** (por que estamos fazendo isso)
- ✅ Indicar **padrões e políticas** do projeto
- ✅ Exigir **robustez e qualidade** nas soluções
- ✅ **Criar cards AUTOSSUFICIENTES** - Toda informação necessária dentro do card

**O Arquiteto NÃO DEVE:**
- ❌ Prescrever implementação detalhada (nomes de classes, métodos, estruturas)
- ❌ Escrever código ou pseudocódigo
- ❌ Definir arquitetura interna (DTOs, Services, camadas)
- ❌ Limitar a criatividade técnica
- ❌ Subestimar a expertise da IA desenvolvedora
- ❌ **Criar cards incompletos que exijam instruções adicionais**

#### **💻 Papel da IA Desenvolvedora:**

**A IA Desenvolvedora DEVE:**
- ✅ **Tornar-se especialista** no contexto da aplicação
- ✅ **Estudar profundamente** o código existente antes de propor mudanças
- ✅ **Propor a melhor arquitetura** técnica possível
- ✅ **Tomar decisões técnicas** fundamentadas
- ✅ **Seguir padrões** do projeto existente
- ✅ **Pensar em edge cases** e robustez
- ✅ **Documentar decisões** arquiteturais tomadas
- ✅ **Testar exaustivamente** a solução

**A IA Desenvolvedora TEM AUTONOMIA para:**
- ✅ Escolher nomes de classes, métodos e variáveis
- ✅ Definir estrutura de DTOs e modelos
- ✅ Organizar pacotes e camadas
- ✅ Adicionar validações e tratamentos de erro
- ✅ Melhorar código existente (sem quebrar)
- ✅ Propor otimizações e refatorações
- ✅ Sugerir melhorias arquiteturais

#### **📐 Formato de Instrução Ideal:**

**❌ ERRADO (Muito Prescritivo):**
```markdown
Crie a classe UserController em src/controller com os métodos:
- syncUser(request: SyncUserRequest): UserResponse
- getUser(id: UUID): UserResponse

Crie o DTO SyncUserRequest com campos:
- email: String
- name: String
...
```

**✅ CORRETO (Requisitos e Autonomia):**
```markdown
## Requisitos:
- Endpoint para sincronizar usuário (criar ou atualizar)
- Endpoint para buscar usuário por ID
- Validar headers de segurança
- Logs de auditoria

## Restrições:
- NÃO alterar entidade User existente
- NÃO modificar LGPDController

## Expectativas:
- Você é o especialista em Spring Boot/Kotlin
- Estude o código existente e siga os padrões
- Proponha a melhor arquitetura possível
- Documente suas decisões técnicas
```

#### **🎯 Benefícios desta Abordagem:**

1. **Soluções Superiores**: IA desenvolvedora conhece as ferramentas melhor que o arquiteto
2. **Profundidade**: Força estudo real do código existente
3. **Ownership**: Desenvolvedor se sente responsável pela solução
4. **Escalabilidade**: Funciona para qualquer tecnologia/contexto
5. **Aprendizado**: Ambas as IAs evoluem (arquiteto e desenvolvedor)
6. **Flexibilidade**: Permite inovação e criatividade técnica

#### **📝 Exemplo Real de Aplicação:**

**Card de Desenvolvimento - Estrutura Ideal:**

```markdown
# 🎯 [Nome do Card]

## 📋 CONTEXTO
[Por que precisamos disso? Qual problema resolve?]

## 🎯 REQUISITOS OBRIGATÓRIOS
[O que PRECISA ter - funcionalidades, endpoints, comportamentos]

## ⚠️ RESTRIÇÕES
[O que NÃO PODE ser alterado - código existente, padrões, políticas]

## ✅ CRITÉRIOS DE VALIDAÇÃO
[Como saber se está correto - testes, comportamentos esperados]

## 🔧 WORKFLOW
1. Estude a aplicação (OBRIGATÓRIO)
2. Crie sua branch
3. Desenvolva a solução (VOCÊ DECIDE COMO)
4. Teste exaustivamente
5. Documente suas decisões
6. Mova para validação

## 🎯 EXPECTATIVAS
- Você é o especialista técnico
- Proponha a melhor solução possível
- Siga padrões do projeto
- Pense em robustez e edge cases
- Documente decisões arquiteturais
```

#### **💡 Filosofia:**

> "Contrate pessoas inteligentes e diga a elas o que fazer, é insultar sua inteligência. Contrate pessoas inteligentes para que elas possam dizer o que fazer."
> - Steve Jobs (adaptado para IAs)

**Confiamos na expertise técnica da IA Desenvolvedora. O Arquiteto guia a VISÃO, a IA Desenvolvedora executa a EXCELÊNCIA TÉCNICA.**

---

### **📋 Regra Fundamental #3: Cards Autossuficientes**

**PRINCÍPIO**: Todo card de desenvolvimento DEVE conter 100% da informação necessária para a IA desenvolvedora trabalhar de forma autônoma, SEM necessidade de instruções adicionais do arquiteto.

#### **🎯 Card Autossuficiente Contém:**

**✅ Contexto Completo:**
- Por que esse card existe?
- Qual problema resolve?
- Onde se encaixa na arquitetura geral?
- Qual o impacto se não for feito?

**✅ Requisitos Funcionais Claros:**
- O que DEVE ser implementado (lista objetiva)
- Comportamentos esperados
- Endpoints/APIs a criar
- Validações obrigatórias
- Logs e auditoria necessários

**✅ Restrições Explícitas:**
- O que NÃO PODE ser alterado
- Código existente a preservar
- Padrões a seguir
- Limitações técnicas

**✅ Documentação de Referência:**
- Cards relacionados (pré-requisitos, dependências)
- Arquivos existentes para estudar
- Documentação externa (se aplicável)
- Exemplos de implementações similares

**✅ Workflow Completo:**
- Passo a passo do que fazer
- Como estudar o código existente
- Como criar branch
- Como testar
- Como documentar
- Como mover para validação

**✅ Critérios de Validação:**
- Checklist de funcionalidades
- Cenários de teste obrigatórios
- Métricas de qualidade
- Como saber se está pronto

**✅ Troubleshooting:**
- Problemas comuns e soluções
- Onde buscar ajuda
- Logs e debugging

#### **❌ Card NÃO Deve Exigir:**

- ❌ **Perguntar ao arquiteto** "qual instrução devo passar?"
- ❌ **Buscar informações externas** não referenciadas
- ❌ **Adivinhar requisitos** não especificados
- ❌ **Esperar aprovação** para decisões técnicas (dentro das restrições)
- ❌ **Solicitar clarificações** sobre o que fazer (como fazer é livre)

#### **✅ Card Pode Assumir:**

- ✅ IA desenvolvedora **lê o card completamente**
- ✅ IA desenvolvedora **estuda código referenciado**
- ✅ IA desenvolvedora **toma decisões técnicas** (dentro das restrições)
- ✅ IA desenvolvedora **documenta escolhas** no output
- ✅ IA desenvolvedora **testa exaustivamente**

#### **📝 Template de Card Autossuficiente:**

```markdown
# 🎯 [Nome do Card]

## 📋 CONTEXTO
[Por que? Qual problema? Onde se encaixa? Impacto?]
- Situação atual
- Problema identificado
- Solução proposta
- Impacto no sistema

## 🎯 REQUISITOS OBRIGATÓRIOS
[O que DEVE ter - lista objetiva e completa]
1. Funcionalidade X
2. Endpoint Y
3. Validação Z
4. Logs de auditoria

## ⚠️ RESTRIÇÕES
[O que NÃO PODE ser alterado]
- Código existente A (não alterar)
- Padrão B (seguir)
- Configuração C (preservar)

## 📚 DOCUMENTAÇÃO DE REFERÊNCIA
[Onde buscar informação]
- Card XXX (pré-requisito)
- Arquivo YYY (estudar)
- Link ZZZ (documentação)

## 🔧 WORKFLOW
1. Estudar aplicação (arquivos A, B, C)
2. Criar branch
3. Implementar (VOCÊ DECIDE COMO)
4. Testar (cenários X, Y, Z)
5. Documentar decisões
6. Mover para validação

## ✅ CRITÉRIOS DE VALIDAÇÃO
[Como saber se está correto]
- [ ] Funcionalidade A funcionando
- [ ] Teste B passando
- [ ] Performance C adequada

## 🚨 TROUBLESHOOTING
[Problemas comuns e soluções]

## 🎯 EXPECTATIVAS
- Você é o especialista técnico
- Estude código existente
- Proponha melhor solução
- Documente decisões
- Teste exaustivamente
```

#### **🎓 Benefícios de Cards Autossuficientes:**

1. **Autonomia Total** - IA desenvolvedora trabalha independente
2. **Sem Fricção** - Não precisa voltar ao arquiteto para perguntas
3. **Paralelização** - Múltiplos cards podem ser desenvolvidos simultaneamente
4. **Rastreabilidade** - Toda informação está documentada no card
5. **Onboarding** - Novos desenvolvedores entendem o contexto completo
6. **Histórico** - Cards são documentação viva do projeto

#### **📊 Como Validar se Card é Autossuficiente:**

**Pergunte-se:**
1. ✅ A IA consegue começar imediatamente após ler o card?
2. ✅ Todas as informações necessárias estão no card ou referenciadas?
3. ✅ Está claro O QUE fazer (mas livre COMO fazer)?
4. ✅ Restrições e validações estão explícitas?
5. ✅ Existe exemplo ou referência similar?

**Se responder NÃO para qualquer pergunta: Card NÃO está autossuficiente!**

#### **💡 Exemplo Real:**

**❌ Card Incompleto (Exige Instrução):**
```markdown
# Card 006 - Sync User
Criar endpoint de sincronização de usuários.
```
→ IA precisa perguntar: "Qual instrução devo passar?"

**✅ Card Autossuficiente:**
```markdown
# Card 006 - Sync User

## CONTEXTO
Auth Service precisa criar/atualizar usuários após OAuth.
Atualmente não há integração entre Auth e User.

## REQUISITOS
- Endpoint POST /sync (criar ou atualizar)
- Validar headers X-User-Id, X-Provider
- Logs de auditoria

## RESTRIÇÕES
- NÃO alterar User entity
- NÃO alterar LGPD endpoints

## WORKFLOW
1. Estudar UserService.kt, LGPDController.kt
2. Criar branch feature/user-sync
3. Implementar (você decide arquitetura)
4. Testar cenários: criar, atualizar, validações
5. Documentar output
6. Mover para validação

## VALIDAÇÃO
- [ ] Criar usuário funciona
- [ ] Atualizar usuário funciona
- [ ] Headers validados
```
→ IA começa imediatamente, sem perguntas!

#### **🎯 Compromisso do Arquiteto:**

> **"Todo card que eu criar será autossuficiente. Se a IA desenvolvedora precisar me perguntar algo, é falha minha, não dela."**

**Quando criar um card:**
1. Escrever todo o contexto
2. Listar todos os requisitos
3. Especificar todas as restrições
4. Referenciar toda documentação necessária
5. Definir workflow completo
6. Especificar critérios de validação
7. **Perguntar: "A IA consegue começar apenas com isso?"**
8. Se não, **completar o card**

---

## 🏗️ **Arquitetura Geral do Sistema**

### **Visão de Alto Nível**
```
┌─────────────────┐    ┌─────────────────┐    ┌─────────────────┐
│   Frontend      │    │   BFF GraphQL   │    │   Microserviços │
│   (React/Vite)  │◄──►│   (Next.js)     │◄──►│   (Spring Boot) │
│   Porta: 5173   │    │   Porta: 3001   │    │   Portas: 8080+ │
└─────────────────┘    └─────────────────┘    └─────────────────┘
```

### **Stack Tecnológica Consolidada**

#### **Frontend (moversemais-store-front)**
- **Framework**: React 18 + TypeScript + Vite
- **Styling**: Tailwind CSS + CSS customizado
- **Roteamento**: React Router DOM
- **Estado**: React Hooks (useState, useEffect)
- **Autenticação**: localStorage (temporário, será migrado para JWT)

#### **BFF - Backend for Frontend (moversemais-store-graphql)**
- **Framework**: Next.js 15.4.4 + TypeScript
- **API**: GraphQL (Apollo Server)
- **Comunicação**: HTTP REST com microserviços
- **Porta**: 3001

#### **Microserviços Backend**
- **Framework**: Spring Boot 3.x + Kotlin
- **Banco de Dados**: PostgreSQL 15
- **ORM**: Spring Data JPA + Hibernate
- **Migração**: Flyway
- **Containerização**: Docker + Docker Compose

---

## 🎯 **Serviços da Plataforma**

### **1. moversemais-store-frontend**
**Função**: Interface do usuário da plataforma  
**Tecnologia**: React + TypeScript + Vite  
**Porta**: 5173 (desenvolvimento)

#### **Funcionalidades Implementadas**
- ✅ **Menu Público Responsivo**: Navegação unificada para páginas públicas
- ✅ **Assessment Completo**: Sistema de avaliação de competências
  - Fluxo: Welcome → Contexto Profissional → Mapeamento de Competências → Resultados
  - Validação: Pergunta "pressões atuais" limitada a 2 opções
  - UX: Feedback visual de loading em botões críticos
- ✅ **Timeline de Objetivos**: Visualização com barras coloridas por status
- ✅ **Responsividade**: Mobile-first com breakpoints específicos
- ✅ **Integração BFF**: Comunicação exclusiva via GraphQL BFF

#### **Decisões Arquiteturais**
- **Menu Unificado**: PublicMenu.tsx em todas as páginas públicas
- **BFF Exclusivo**: Sem fallbacks locais, sempre consultar BFF
- **Gerenciamento de Estado**: localStorage + sincronização com BFF
- **Scroll Management**: Automático para topo em transições

### **2. moversemais-store-graphql**
**Função**: BFF (Backend for Frontend) - Camada intermediária  
**Tecnologia**: Next.js + GraphQL + TypeScript  
**Porta**: 3001

#### **Funcionalidades Implementadas**
- ✅ **Sistema de Objetivos**: CRUD completo com filtros avançados
- ✅ **Sistema de Assessment**: Sessões de avaliação com 33 competências
- ✅ **Funcionalidade de IA**: Sugestão de Key Results via ChatGPT
- ✅ **Tratamento de Erros**: Mapeamento de códigos backend → frontend
- ✅ **CORS Configurado**: Para desenvolvimento local

#### **Padrões Estabelecidos**
- **Resolvers**: Lógica de resolução GraphQL
- **Services**: Lógica de negócio
- **Types**: Interfaces TypeScript
- **Error Handling**: processBackendError() centralizado

### **3. moversemais-objective**
**Função**: Gestão de objetivos e sistema de assessment  
**Tecnologia**: Spring Boot 3.2.1 + Kotlin + PostgreSQL  
**Porta**: 8080

#### **Funcionalidades Implementadas**
- ✅ **Gestão de Objetivos**: CRUD com tipos diversos (BUSINESS_GOAL, PERSONAL_ACHIEVEMENT, etc.)
- ✅ **Sistema de Assessment**: 12 competências com sistema de desempate
- ✅ **Módulo de IA**: Integração ChatGPT para sugestão de Key Results
- ✅ **Sistema de Desempate**: Strategy Pattern para resolver empates
- ✅ **Clean Architecture**: Controller → UseCase → Repository

#### **Arquitetura Avançada**
- **Strategy Pattern**: Para desempate de competências
- **Clean Architecture**: Separação clara de responsabilidades
- **Exception Handling**: Hierarquia de exceções customizadas
- **Migrações Flyway**: V022 (competência "Domínio de Contexto")

### **4. moversemais-user**
**Função**: Gerenciamento de usuários  
**Tecnologia**: Spring Boot 3.5.6 + Kotlin + PostgreSQL  
**Porta**: 8083

#### **Funcionalidades Implementadas**
- ✅ **Estrutura Base**: Configuração Spring Boot completa
- ✅ **Health Checks**: Endpoints de monitoramento
- ✅ **Documentação**: Swagger/OpenAPI 3.0
- ✅ **Migração Inicial**: Tabela users criada
- ✅ **Docker**: Containerização configurada

#### **Próximos Passos**
- Implementar entidades JPA para User
- Criar repositórios Spring Data JPA
- Implementar endpoints CRUD para usuários
- Adicionar validações com Bean Validation
- Implementar autenticação e autorização

---

## 🔄 **Fluxo de Dados da Plataforma**

### **1. Assessment de Competências**
```
Frontend → BFF GraphQL → Objective Service → PostgreSQL
    ↓
ChatGPT API (Key Results)
    ↓
Frontend (Resultados)
```

### **2. Gestão de Objetivos**
```
Frontend → BFF GraphQL → Objective Service → PostgreSQL
    ↓
Frontend (Timeline/CRUD)
```

### **3. Autenticação (Futuro)**
```
Frontend → BFF GraphQL → User Service → PostgreSQL
    ↓
JWT Token → Frontend
```

---

## 🛠️ **Configurações e Deploy**

### **Ambientes de Desenvolvimento**
- **Frontend**: `npm run dev` (porta 5173)
- **BFF**: `npm run dev` (porta 3001)
- **Objective Service**: `./gradlew bootRun` (porta 8080)
- **User Service**: `./gradlew bootRun` (porta 8083)

### **Banco de Dados**
- **PostgreSQL**: Porta 5433 (desenvolvimento)
- **Database**: moversemais_objective, moversemais_user
- **Migrações**: Flyway (V022 para objective, V1 para user)

### **Docker**
- **Desenvolvimento**: `docker-compose up`
- **Produção**: Build automático no Render.com

---

## 📊 **Padrões e Convenções**

### **Nomenclatura**
- **Classes**: PascalCase (ex: `ObjectiveController`)
- **Métodos**: camelCase (ex: `createObjective`)
- **Constantes**: UPPER_SNAKE_CASE (ex: `MAX_DURATION_DAYS`)
- **Arquivos**: snake_case (ex: `objective_controller.kt`)

### **Estrutura de Commits**
```
feat: nova funcionalidade
fix: correção de bug
docs: documentação
style: formatação
refactor: refatoração
test: testes
chore: tarefas de manutenção
```

### **Tratamento de Erros**
- **Códigos Padronizados**: OBJ_001, ASS_001, AI_001, etc.
- **HTTP Status**: Mapeamento correto de status
- **Mensagens**: Formato consistente de erro

---

## 🚀 **Funcionalidades Principais da Plataforma**

### **1. Sistema de Assessment**
- **12 Competências**: Incluindo "Domínio de Contexto"
- **Sistema de Desempate**: Strategy Pattern configurável
- **Múltiplos Assessments**: Por usuário sem restrições
- **IA Integration**: Sugestão de Key Results via ChatGPT

### **2. Gestão de Objetivos**
- **Tipos Diversos**: BUSINESS_GOAL, PERSONAL_ACHIEVEMENT, HEALTH, FINANCIAL_GAIN
- **Timeline Visual**: Barras coloridas por status de progresso
- **Filtros Avançados**: Status, tipo, prioridade, datas
- **Paginação**: Para grandes volumes de dados

### **3. Interface Responsiva**
- **Mobile-First**: Breakpoints específicos (sm, md, lg, xl)
- **Menu Unificado**: Navegação consistente
- **Feedback Visual**: Loading states em operações críticas
- **Scroll Management**: Automático para melhor UX

---

## 🎯 **Próximos Desenvolvimentos**

### **Funcionalidades Planejadas**
- [ ] **Sistema de Autenticação**: JWT com User Service
- [ ] **Dashboard Completo**: Métricas e relatórios
- [ ] **Sistema de Notificações**: Push notifications
- [ ] **Integração com Calendário**: Sincronização de deadlines
- [ ] **Relatórios Avançados**: Análise de tendências
- [ ] **Recomendações Personalizadas**: Baseadas em histórico

### **Melhorias Técnicas**
- [ ] **Cache Redis**: Para performance
- [ ] **Rate Limiting**: Proteção contra abuso
- [ ] **Logs Estruturados**: ELK Stack
- [ ] **Métricas de Performance**: Prometheus + Grafana
- [ ] **CI/CD Pipeline**: GitHub Actions
- [ ] **Testes Automatizados**: Unit + Integration + E2E

---

## 🔧 **Comandos Úteis**

### **Desenvolvimento Local**
```bash
# Frontend
cd moversemais-store-front
npm run dev

# BFF
cd moversemais-store-graphql
npm run dev

# Objective Service
cd moversemais-objective
./gradlew bootRun

# User Service
cd moversemais-user
./gradlew bootRun
```

### **Docker**
```bash
# Todos os serviços
docker-compose up

# Apenas banco
docker-compose -f docker-compose.db.yml up -d
```

### **Testes**
```bash
# Frontend
npm run test:all

# Backend
./gradlew test
```

---

## 📚 **Documentação dos Serviços**

### **Arquivos AGENTS.md Individuais**
- **Frontend**: `/moversemais-store-front/AGENTS.md`
- **BFF**: `/moversemais-store-graphql/AGENTS.md`
- **Objective**: `/moversemais-objective/AGENTS.md`
- **User**: `/moversemais-user/AGENTS.md`

### **Documentação Técnica**
- **API Docs**: Swagger/OpenAPI 3.0 em cada serviço
- **GraphQL Playground**: http://localhost:3001/api/graphql
- **Health Checks**: Endpoints de monitoramento em cada serviço

---

## 🏷️ **Tags da Plataforma**

`#microservices` `#graphql` `#react` `#springboot` `#kotlin` `#typescript` `#postgresql` `#docker` `#assessment` `#objectives` `#ai` `#leadership` `#bff`

---

## 📞 **Contato e Suporte**

**Desenvolvedor Principal**: Eduardo Souza  
**Projeto**: MoverseMais Platform  
**Última Atualização**: Outubro 2025  
**Versão**: 1.0.0

---

**💡 Dica**: Este arquivo consolida a visão arquitetural de toda a plataforma MoverseMais. Para detalhes específicos de cada serviço, consulte os arquivos AGENTS.md individuais.

---

*Este arquivo deve ser atualizado sempre que houver mudanças significativas na arquitetura da plataforma.*
