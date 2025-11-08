# 📄 PRD 01 — Criação de Metas e Geração Automática de Plano (Goal Chunking)

## 🧭 Contexto
A funcionalidade **Criação de Metas e Geração Automática de Plano** é o ponto de partida do módulo de Metas da MoverseMais. Ela representa o primeiro contato do usuário com o assistente de IA e define o tom da experiência — simples, guiada e eficaz.

O objetivo é permitir que o usuário descreva sua meta em linguagem natural (ex: *“Quero perder 5kg até dezembro”* ou *“Quero aprender Kotlin em 3 meses”*), e que a IA transforme isso automaticamente em um **plano estruturado**, composto por etapas, ações executáveis e indicadores de progresso (KRs).

---

## 🎯 Objetivo da Funcionalidade
Oferecer uma experiência de criação de metas que:
- **Remova o esforço cognitivo** de planejar manualmente (para quem quiser a ajuda da IA);
- **Permita também a criação manual de planos**, para usuários que já sabem o caminho;
- **Traduza aspirações em planos concretos** com etapas, ações e indicadores (KRs);
- **Inclua contexto opcional** (como pretende alcançar) e **motivo da meta** (por que ela é importante);
- **Estabeleça KRs visíveis, automáticos e ligados às ações**, que servirão como base para o check-in de progresso;
- **Prepare o usuário para o foco diário e o acompanhamento inteligente de resultados.**

---

## 💡 Visão Funcional

### Fluxo do Usuário
1. **Usuário cria uma nova meta**
   - Informa: título e prazo final (que pode ser uma data fixa ou um período relativo: *próximo mês*, *próximo trimestre*, etc.).
   - Opcional: descrição.
   - Exemplo: “Quero correr 10km em 2 meses.”

2. **Usuário adiciona contexto e motivo (opcionais)**
   - Contexto: como pretende atingir a meta.  
     Exemplo: “Pretendo fazer uma dieta e musculação 3x por semana.”
   - Motivo: por que a meta é importante.  
     Exemplo: “Quero melhorar minha saúde e disposição física.”
   - Ambos ajudam a IA a gerar planos e indicadores mais personalizados.

3. **Usuário escolhe o modo de criação do plano**
   - **Automático (IA):** o sistema gera um plano estruturado com etapas, ações e KRs, levando em conta o contexto e motivo.
   - **Manual:** o usuário cria suas próprias etapas, ações e indicadores.

4. **IA gera o plano de execução e KRs associados**
   - O plano contém **etapas**, **ações** e **indicadores (KRs)**, que representam marcos de progresso mensuráveis.
   - Exemplo:
     - Etapa 1: Avaliar condicionamento físico atual
       - Ação: Fazer uma corrida de teste de 2km
       - Ação: Registrar tempo e frequência cardíaca
       - KR: Correr 2km em até 12 minutos
     - Etapa 2: Criar rotina de treinos
       - Ação: Planejar 3 treinos semanais
       - KR: Cumprir 80% dos treinos nas 4 primeiras semanas

5. **Usuário revisa e aprova o plano**
   - Pode editar, excluir ou adicionar novas ações e indicadores.
   - Após aprovação, o plano é ativado e as ações são vinculadas aos KRs.

6. **Integração futura com o check-in**
   - Cada ação gerada é vinculada a um ou mais KRs.  
   - No momento do check-in (PRD 02), o usuário escolherá quais KRs quer registrar progresso, e o sistema atualizará automaticamente o status da meta.

---

## 🧠 Regras e Comportamentos Esperados
- O usuário pode optar entre **criação manual** e **automática**.
- A IA deve sempre propor **ações específicas e mensuráveis**.
- Cada meta deve conter pelo menos **1 etapa, 3 ações e 1 KR**.
- Os **KRs devem ser visíveis e editáveis** antes da ativação da meta.
- As ações devem estar **vinculadas a KRs** para permitir cálculo de progresso posterior.
- O sistema deve permitir **nova geração de plano** caso o usuário queira uma alternativa.

---

## 🤖 Recomendação de Abordagem de IA
- Utilizar LLMs com **saída estruturada** (ex.: JSON) contendo meta, contexto, etapas, ações e KRs.
- Implementar **validação automática** para garantir consistência entre meta, ações e indicadores.
- Incluir camada de **autoavaliação** para detectar redundância e checar se o plano é factível.
- Usar o **contexto e o motivo da meta** para ajustar o plano e indicadores à realidade do usuário.

---

## 📋 Considerações Importantes
- O arquiteto definirá a integração de IA e o modelo de dados das relações Meta ↔ KRs ↔ Ações.
- Indicadores devem ser intuitivos e compreensíveis, não técnicos.
- O foco deve estar em **clareza e mensurabilidade**, mantendo simplicidade de uso.

---

## 🔄 Próximos Passos
1. Implementar o fluxo de criação com os campos opcionais e KRs visíveis.
2. Preparar a estrutura para associar Ações ↔ KRs.
3. Criar o **PRD 02 — Check-in de Progresso**.

---

**MoverseMais — O primeiro passo é sonhar, o segundo é planejar.**

