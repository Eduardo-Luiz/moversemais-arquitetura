# 📄 PRD 02 — Check-in de Progresso (Atualização de KRs)

## 🧭 Contexto
O **Check-in de Progresso** é a funcionalidade que conecta planejamento à execução. Ela permite que o usuário **registre seu avanço real** nas metas, atualizando os **KRs (Indicadores de Resultado)** que medem o progresso de forma simples, prática e significativa.

O foco é tornar o check-in **rápido, inteligente e orientado por resultados**, substituindo a ideia tradicional de registrar tarefas por um sistema mais focado em conquistas.

---

## 🎯 Objetivo da Funcionalidade
- Permitir que o usuário **selecione quais KRs deseja atualizar** em cada check-in.
- Registrar o **progresso quantitativo ou qualitativo** de forma intuitiva.
- Atualizar automaticamente o progresso geral da meta com base nas mudanças nos KRs.
- Fornecer feedback visual e motivacional sobre a evolução.
- Criar base para insights e reajustes automáticos do plano pela IA.

---

## 💡 Visão Funcional

### Fluxo do Usuário
1. **Acesso ao módulo de check-in**
   - O usuário entra na tela de check-in diário ou semanal.
   - A plataforma exibe todas as metas ativas e seus respectivos KRs.

2. **Seleção dos KRs**
   - O usuário escolhe **quais KRs deseja registrar progresso** naquele momento.  
     Exemplo: marcar “Peso corporal” e “Treinos semanais”.

3. **Registro do progresso**
   - Para cada KR selecionado, o sistema oferece o tipo de input mais adequado:
     - Numérico (ex: peso atual, quantidade, percentual)
     - Binário (sim/não, concluído/parcial)
     - Descritivo (comentário livre opcional)
   - O usuário preenche e confirma.

4. **Atualização automática dos KRs e da meta**
   - A IA calcula o novo percentual de conclusão de cada KR.
   - O progresso global da meta é atualizado automaticamente com base na média ponderada dos KRs.
   - As ações relacionadas aos KRs são marcadas como avançadas.

5. **Feedback inteligente (opcional)**
   - A IA fornece uma devolutiva breve:
     - Ex: “Excelente avanço! Você atingiu 75% do seu objetivo de treinos semanais.”
     - Ou: “Você está um pouco abaixo no ritmo de leitura. Que tal revisar o plano?”

---

## 🧠 Regras e Comportamentos Esperados
- O usuário deve poder escolher um ou mais KRs por check-in.
- O sistema deve oferecer o tipo de input correto conforme o tipo de KR (numérico, binário ou descritivo).
- O progresso global da meta deve refletir automaticamente os avanços nos KRs.
- A IA deve poder gerar **feedback e recomendações** (reajustes de ações, novas prioridades etc.).
- O sistema deve manter histórico de check-ins e evolução de cada KR.

---

## ⚙️ Lógica de Cálculo de Progresso (Alta Nível)
1. Cada KR tem um **peso proporcional** dentro da meta (gerado automaticamente pela IA).
2. O progresso de cada KR é calculado com base no input do usuário.
3. O **progresso total da meta** é uma média ponderada dos KRs.
4. Ações vinculadas aos KRs são atualizadas de acordo com o avanço registrado.

---

## 🔍 Tipos de KR e Input Esperado
| Tipo de KR | Exemplo | Tipo de Input | Interpretação |
|-------------|----------|----------------|----------------|
| Numérico | Peso, Horas estudadas | Valor atual | Calcula % vs meta final |
| Percentual | Conclusão de projeto | Percentual | Atualiza diretamente |
| Binário | “Treino semanal cumprido” | Sim / Não | Incrementa ou mantém progresso |
| Frequência | “Dias com leitura” | Quantidade semanal | IA interpreta e ajusta meta |

---

## 🧩 Integração com Outras Funcionalidades
- O módulo de check-in **consome os KRs criados no PRD 01**.
- Cada ação gerada no plano está vinculada a um ou mais KRs — quando um KR avança, as ações correspondentes também progridem.
- O histórico de check-ins alimenta o **dashboard de progresso e a análise de desempenho pessoal**.

---

## 📋 Considerações Importantes
- O arquiteto definirá o modelo de dados para histórico de check-ins e cálculo de progresso.
- O sistema deve permitir inputs flexíveis (número, texto, escolha rápida).
- A IA poderá sugerir quais KRs atualizar com base na frequência anterior e no impacto.
- A experiência deve ser fluida e rápida: o check-in deve levar **menos de 1 minuto**.

---

## 🔄 Próximos Passos
1. Desenhar o wireflow da jornada de check-in por KRs.
2. Definir o modelo de dados e cálculos de progresso junto ao arquiteto.
3. Implementar o primeiro protótipo funcional (check-in manual com feedback visual).
4. Conectar a IA para geração de recomendações automáticas pós-check-in.

---

**MoverseMais — Check-ins que transformam ações em conquistas.**

