# 🎯 Card 024 - Frontend: Formulário de Lead nos Resultados do Assessment

**Agente Responsável:** Lisa  
**Microserviço:** moversemais-store-front  
**Prioridade:** Alta  
**Status:** TODO  
**Estimativa:** 1 dia

---

## 📋 CONTEXTO

### **Situação Atual**
Assessment está completo e funcional. Usuários veem seus resultados na tela (DiagnosisResultsPage), mas após sair, perdemos 100% do contato. Não temos forma de notificar sobre evoluções da plataforma ou engajar usuários futuramente.

### **Problema Identificado**
- Zero captura de leads após assessment
- Impossível construir lista de interessados
- Sem canal de comunicação com usuários
- Botão "Falar com o Mentor" presente mas não funcional no momento

### **Solução Proposta**
Adicionar formulário opcional de cadastro de lead no final da página de resultados, permitindo que usuários optem por receber novidades. Remover botão "Falar com o Mentor" temporariamente.

### **Onde se Encaixa na Arquitetura**
```
Frontend - VOCÊ (Formulário de Lead) ← ESTE CARD
    ↓
BFF GraphQL (Mutation registerLead)
    ↓
User Service (POST /api/v1/leads)
    ↓
PostgreSQL
```

### **Impacto se Não For Feito**
- Lançamento sem captura de leads
- Impossível notificar usuários sobre evoluções
- Perda de oportunidade de conversão
- Botão não funcional na interface

---

## 🎯 REQUISITOS OBRIGATÓRIOS

### **1. Adicionar Formulário de Lead**

**Localização:**
- Página: `DiagnosisResultsPage.tsx` (ou `ResultsPage.tsx` - onde resultado é exibido)
- Posição: Ao final da página, após todo conteúdo de resultados

**Campos Obrigatórios:**
1. Input Email:
   - Tipo: email
   - Placeholder: "seu@email.com"
   - Validação: formato de email válido
   - Obrigatório para envio

2. Checkbox 1 (pré-selecionado):
   - Label: "Tenho interesse em receber notícias sobre a evolução da plataforma MoverseMais para líderes de tecnologia"
   - Default: checked (true)
   - Pode ser desmarcado

3. Checkbox 2 (pré-selecionado):
   - Label: "Quero receber notícias do blog da MoverseMais"
   - Default: checked (true)
   - Pode ser desmarcado

4. Botão:
   - Texto: "Cadastrar Interesse"
   - Ação: Chamar mutation registerLead
   - Estado de loading enquanto processa

**Design Esperado:**
- Card/seção destacada visualmente
- Call-to-action claro e convidativo
- Mobile responsive
- Não invasivo (após resultados, não bloqueante)

**Título/Descrição da Seção:**
- Título: "💌 Fique por dentro da MoverseMais"
- Descrição curta sobre receber novidades

### **2. Integração GraphQL**

**Mutation:**
```typescript
mutation RegisterLead($email: String!, $optInPlatformNews: Boolean!, $optInBlogNews: Boolean!) {
  registerLead(
    email: $email
    optInPlatformNews: $optInPlatformNews
    optInBlogNews: $optInBlogNews
  ) {
    success
    message
    leadId
  }
}
```

**Chamada:**
- Usar Apollo Client (padrão do projeto)
- Variables: email, optInPlatformNews, optInBlogNews
- Tratamento de loading, error, data

### **3. Feedback Visual**

**Estados:**
1. **Inicial**: Formulário vazio e pronto
2. **Loading**: Botão desabilitado, spinner, "Cadastrando..."
3. **Sucesso**: 
   - Mensagem: "✅ Obrigado! Entraremos em contato."
   - Esconder formulário ou desabilitar campos
   - Opcional: Confetti ou animação sutil
4. **Erro**:
   - Mensagem: "❌ Erro ao cadastrar. Tente novamente."
   - Manter formulário editável
   - Permitir nova tentativa

**Validações Frontend:**
- Email vazio: "Por favor, insira seu email"
- Email inválido: "Email inválido"
- Mostrar validações em tempo real (onBlur ou onChange)

### **4. Remover Botão "Falar com o Mentor"**

**Localização:**
- `DiagnosisResultsPage.tsx`
- `ResultsPage.tsx` (se existir)
- Qualquer outra página de resultado

**Ação:**
- Remover completamente o botão
- Remover imports relacionados (se houver)
- Remover handlers de navegação relacionados

### **5. Política de Privacidade**

**Link Pequeno Abaixo do Formulário:**
- Texto: "Política de Privacidade | Você pode descadastrar a qualquer momento"
- Link para Política de Privacidade (se existir, senão deixar como texto)

---

## ⚠️ RESTRIÇÕES

### **O que NÃO PODE ser alterado:**

1. ❌ **NÃO alterar conteúdo de resultados** (competências, diagnóstico, key results)
2. ❌ **NÃO alterar fluxo de assessment** (navegação, steps)
3. ❌ **NÃO bloquear visualização** de resultado (lead é opcional)
4. ❌ **NÃO quebrar responsividade** existente
5. ❌ **NÃO alterar mutations** de assessment ou objectives

### **O que DEVE ser preservado:**

1. ✅ **Padrão de componentes** React existentes
2. ✅ **Padrão Tailwind CSS** para styling
3. ✅ **Padrão Apollo Client** para GraphQL
4. ✅ **Mobile-first** responsivo
5. ✅ **Feedback de loading** em botões (padrão existente)

---

## 📚 DOCUMENTAÇÃO DE REFERÊNCIA

### **Arquivos para Estudar (OBRIGATÓRIO):**

1. **Páginas de Resultado (CRÍTICO - ESTUDAR PRIMEIRO):**
   - `src/components/assessment/DiagnosisResultsPage.tsx` - **Página ativa (V2)**
   - Linhas 177-230: Seção "CTA para Mentoria" - **DEVE SER SUBSTITUÍDA**
   - Essa seção completa (todo o bloco) será removida e substituída pelo formulário de lead

2. **Botão "Falar com Mentor" (REMOVER):**
   - `DiagnosisResultsPage.tsx` linhas 210-223
   - Botão WhatsApp com handler onClick
   - **REMOVER TODO ESSE BLOCO** (linhas 177-230)

3. **Padrão de Mutations GraphQL:**
   - `src/graphql/queries.ts` - **TODAS as mutations estão aqui**
   - Exemplos: `SUBMIT_ASSESSMENT_V2` (linha 11), `CREATE_OBJECTIVE` (linha 203)
   - Padrão: `export const NOME = gql\`mutation...\``
   - **ADICIONAR** `REGISTER_LEAD` seguindo esse padrão

4. **Padrão de useMutation:**
   - `src/components/assessment/ResponsiveAssessment.tsx` (linhas 1-50)
   - Veja como importa: `import { useMutation } from '@apollo/client'`
   - Veja como usa: `const [submitAssessment] = useMutation(SUBMIT_ASSESSMENT_V2)`
   - **SEGUIR ESSE PADRÃO**

5. **Styling e Responsividade:**
   - `DiagnosisResultsPage.tsx` - Classes Tailwind usadas
   - Padrão: `bg-gradient-to-br`, `rounded-2xl`, `p-6 sm:p-10`
   - Mobile-first: `text-base sm:text-lg`, `grid-cols-1 md:grid-cols-3`
   - **SEGUIR ESSE PADRÃO DE CLASSES**

6. **Documentação:**
   - `../moversemais-store-front/AGENTS.md` - Políticas do Frontend
   - `../moversemais-arquitetura/AGENTS.md` - Visão geral

### **Cards Relacionados:**
- Card 025: BFF Mutation registerLead (pré-requisito - DEVE estar pronto)
- Card 026: User Service Lead Management (pré-requisito indireto)

---

## 🔧 WORKFLOW

### **1. ESTUDAR (OBRIGATÓRIO - Antes de Codificar)**

```bash
cd moversemais-store-front

# CRÍTICO: Analisar DiagnosisResultsPage.tsx
cat src/components/assessment/DiagnosisResultsPage.tsx

# Localizar seção "CTA para Mentoria" (linhas 177-230)
sed -n '177,230p' src/components/assessment/DiagnosisResultsPage.tsx

# Estudar padrão de mutations
cat src/graphql/queries.ts | grep -A 20 "mutation"

# Estudar padrão de useMutation
cat src/components/assessment/ResponsiveAssessment.tsx | head -50

# Verificar estrutura de componentes
ls -la src/components/assessment/

# Ler AGENTS.md
cat AGENTS.md
```

**Perguntas para Responder Antes de Implementar:**
- ✅ **RESPOSTA:** Página ativa é `DiagnosisResultsPage.tsx` (V2)
- ✅ **RESPOSTA:** Botão "Falar com Mentor" está em linhas 177-230
- ✅ **RESPOSTA:** Mutations em `src/graphql/queries.ts`
- ✅ **RESPOSTA:** Padrão: `export const NOME = gql\`mutation...\``
- ❓ Como validar email no frontend? (useState ou useForm?)
- ❓ Onde colocar mutation (queries.ts - confirmar)
- ❓ Feedback: inline message ou toast?

**O que SUBSTITUIR:**
- **TODO O BLOCO** das linhas 177-230 (seção "CTA para Mentoria")
- Substituir por formulário de lead com mesma estrutura visual

### **2. CRIAR BRANCH**

```bash
git checkout -b feature/frontend-lead-form
```

### **3. IMPLEMENTAR (VOCÊ DECIDE COMO)**

**Ordem Sugerida (mas você decide):**
1. Localizar página de resultados correta
2. Remover botão "Falar com o Mentor"
3. Criar componente de formulário de lead (ou inline na página)
4. Adicionar mutation GraphQL
5. Implementar validações
6. Implementar feedback visual (loading, sucesso, erro)
7. Testar responsividade mobile

**Você decide:**
- Criar componente separado ou implementar inline
- Estrutura exata do JSX
- Classes Tailwind específicas
- Validação: useForm ou estado manual
- Feedback: toast, inline message, modal
- Animações e transições

### **4. TESTAR**

**Testes Manuais Obrigatórios:**

**Desktop:**
- [ ] Formulário aparece no final dos resultados
- [ ] Email válido: sucesso
- [ ] Email inválido: erro de validação
- [ ] Checkboxes pré-selecionados
- [ ] Desmarcar checkboxes funciona
- [ ] Botão "Cadastrar Interesse" funciona
- [ ] Loading state durante envio
- [ ] Feedback de sucesso aparece
- [ ] Botão "Falar com o Mentor" removido

**Mobile (iPhone/Android):**
- [ ] Layout responsivo
- [ ] Formulário legível
- [ ] Inputs funcionam no mobile
- [ ] Checkboxes clicáveis
- [ ] Botão acessível

**Cenários de Erro:**
- [ ] BFF indisponível: mensagem amigável
- [ ] Email vazio: validação
- [ ] Email duplicado: sucesso (idempotente)

### **5. DOCUMENTAR DECISÕES**

Ao final do card, documente:
- Estrutura de componente escolhida
- Validações implementadas
- Feedback visual implementado
- Design escolhido
- Dificuldades encontradas

### **6. COMMIT E PUSH**

```bash
git add .
git commit -m "feat(frontend): adiciona formulário de lead nos resultados

- Formulário de cadastro no final do assessment
- Campos: email + 2 checkboxes (pré-selecionados)
- Mutation registerLead integrada
- Validação de email
- Feedback visual (loading, sucesso, erro)
- Remove botão 'Falar com o Mentor'
- Mobile responsive"

git push origin feature/frontend-lead-form
```

### **7. MOVER PARA VALIDAÇÃO**

```bash
mv Development/TODO/024__Frontend-Lead-Form-Results.md \
   Development/VALIDATING/024__Frontend-Lead-Form-Results.md
```

---

## ✅ CRITÉRIOS DE VALIDAÇÃO

### **Funcionalidades:**
- [ ] Formulário aparece na página de resultados
- [ ] Input email funciona
- [ ] Checkbox 1 pré-selecionado
- [ ] Checkbox 2 pré-selecionado
- [ ] Checkboxes podem ser desmarcados
- [ ] Botão "Cadastrar Interesse" funciona
- [ ] Validação de email funciona
- [ ] Mutation registerLead é chamada
- [ ] Email + optIns enviados corretamente

### **Feedback Visual:**
- [ ] Loading state no botão
- [ ] Mensagem de sucesso clara
- [ ] Mensagem de erro clara
- [ ] Validações em tempo real
- [ ] UX não invasiva

### **Remoções:**
- [ ] Botão "Falar com o Mentor" removido
- [ ] Imports relacionados removidos
- [ ] Sem código órfão

### **Responsividade:**
- [ ] Desktop: layout OK
- [ ] Tablet: layout OK
- [ ] Mobile: layout OK
- [ ] Inputs acessíveis em todos tamanhos

### **Qualidade:**
- [ ] TypeScript sem erros
- [ ] Build passa (npm run build)
- [ ] Código segue padrões do projeto
- [ ] Commits seguem convenção

---

## 🚨 TROUBLESHOOTING

### **Problema: Mutation não encontrada**
**Solução:**
- Verificar se BFF está rodando (porta 3001)
- Verificar se Card 025 foi implementado
- Verificar nome exato da mutation no schema GraphQL

### **Problema: Checkboxes não pré-selecionam**
**Solução:**
- Estado inicial: `useState(true)` para ambos
- Verificar atributo `checked` ou `defaultChecked`

### **Problema: Validação de email não funciona**
**Solução:**
- Regex de email: `/^[^\s@]+@[^\s@]+\.[^\s@]+$/`
- Ou usar biblioteca de validação (yup, zod)

### **Problema: Loading não aparece**
**Solução:**
- Usar `loading` do `useMutation` hook
- Condicional: `loading ? "Cadastrando..." : "Cadastrar Interesse"`

### **Problema: Mobile não responsivo**
**Solução:**
- Usar classes Tailwind: `w-full`, `px-4`, `sm:`, `md:`
- Testar com DevTools mobile emulation

---

## 🎯 EXPECTATIVAS

### **Você é a Especialista Frontend + UX**

**Lisa, você domina React, TypeScript e UX.** Eu confio que você:

1. ✅ **Conhece React + TypeScript** melhor que eu
2. ✅ **Conhece UX/UI** melhor que eu
3. ✅ **Sabe Tailwind CSS** melhor que eu
4. ✅ **Entende responsividade** melhor que eu

**Por isso:**
- Estude o código existente profundamente
- Siga os padrões já estabelecidos
- Tome decisões de UX fundamentadas
- Crie interface intuitiva e bonita
- Pense em mobile desde o início

**Eu defini O QUE precisa ser feito. Você decide COMO fazer.**

**Dupla responsabilidade Frontend:**
- Excelência Técnica (código limpo, TypeScript correto)
- Excelência UX/UI (bonito, intuitivo, acessível)

---

## 🎨 DESIGN - SEGUIR PADRÃO EXISTENTE

### **SUBSTITUIR Bloco Existente (linhas 177-230):**

**De (REMOVER):**
```tsx
{/* CTA para Mentoria */}
<div className="bg-gradient-to-br from-green-50 to-emerald-50 rounded-2xl p-6 sm:p-10 ...">
  <h3>Pronto para acelerar seu desenvolvimento?</h3>
  ...
  <button>Falar com Mentor no WhatsApp</button>
</div>
```

**Para (CRIAR):**
```tsx
{/* Formulário de Lead - Captura de Interesse */}
<div className="bg-gradient-to-br from-indigo-50 to-purple-50 rounded-2xl p-6 sm:p-10 shadow-xl border-2 border-indigo-200 text-center animate-slide-up">
  <h3 className="text-2xl sm:text-3xl font-bold text-gray-800 mb-4">
    💌 Fique por dentro da MoverseMais
  </h3>
  <p className="text-base sm:text-lg text-gray-600 max-w-3xl mx-auto mb-6">
    Deixe seu email para receber novidades sobre a evolução da plataforma para líderes de tecnologia.
  </p>
  
  <form onSubmit={handleSubmit} className="max-w-2xl mx-auto space-y-4">
    {/* Input Email */}
    <input
      type="email"
      placeholder="seu@email.com"
      className="w-full px-4 py-3 border-2 border-indigo-200 rounded-lg focus:ring-2 focus:ring-indigo-500 focus:border-indigo-500 text-lg"
      value={email}
      onChange={(e) => setEmail(e.target.value)}
      required
    />
    
    {/* Checkbox 1 - Platform News */}
    <label className="flex items-start gap-3 text-left bg-white/70 rounded-lg p-4 border border-indigo-100 cursor-pointer hover:bg-white transition-colors">
      <input 
        type="checkbox" 
        checked={optInPlatformNews} 
        onChange={(e) => setOptInPlatformNews(e.target.checked)}
        className="mt-1 w-5 h-5 text-indigo-600"
      />
      <span className="text-sm sm:text-base text-gray-700">
        Tenho interesse em receber notícias sobre a evolução da plataforma MoverseMais para líderes de tecnologia
      </span>
    </label>
    
    {/* Checkbox 2 - Blog News */}
    <label className="flex items-start gap-3 text-left bg-white/70 rounded-lg p-4 border border-indigo-100 cursor-pointer hover:bg-white transition-colors">
      <input 
        type="checkbox" 
        checked={optInBlogNews} 
        onChange={(e) => setOptInBlogNews(e.target.checked)}
        className="mt-1 w-5 h-5 text-indigo-600"
      />
      <span className="text-sm sm:text-base text-gray-700">
        Quero receber notícias do blog da MoverseMais
      </span>
    </label>
    
    {/* Botão Submit */}
    <button
      type="submit"
      disabled={loading}
      className="w-full px-8 py-4 bg-gradient-to-r from-indigo-500 to-purple-600 text-white rounded-full font-bold text-lg hover:shadow-2xl hover:shadow-indigo-500/30 hover:-translate-y-1 transition-all duration-300 disabled:opacity-50 disabled:cursor-not-allowed"
    >
      {loading ? 'Cadastrando...' : 'Cadastrar Interesse'}
    </button>
    
    {/* Feedback Messages */}
    {successMessage && (
      <p className="text-green-600 font-medium">✅ {successMessage}</p>
    )}
    {errorMessage && (
      <p className="text-red-600 font-medium">❌ {errorMessage}</p>
    )}
  </form>
  
  <p className="text-xs text-gray-500 mt-6">
    Política de Privacidade | Você pode descadastrar a qualquer momento
  </p>
</div>
```

**Notas:**
- Classes Tailwind **IGUAIS** ao padrão existente (linhas 177-230)
- Estrutura visual **CONSISTENTE** com seção de mentoria
- Apenas trocar cores: verde → indigo/purple (tema MoverseMais)
- Animação `animate-slide-up` já existe no projeto

---

## 📊 OUTPUT ESPERADO

Ao finalizar, documente aqui:

### **Decisões Técnicas Tomadas:**
(Você preenche)

### **Decisões de UX/Design:**
(Você preenche)

### **Estrutura Criada:**
(Arquivos criados/modificados)

### **Testes Realizados:**
(Cenários testados + screenshots)

### **Dificuldades Encontradas:**
(Se houver)

### **Melhorias Implementadas:**
(Além do requisitado)

---

**Data de Criação:** 01/11/2025  
**Criado por:** Arquiobaldo (Arquiteto MoverseMais)  
**Versão:** 1.0

