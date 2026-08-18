# CLAUDE.md — Projeto Tiago Freire Arquitetos Associados · Adapta

Este repositório é o espaço de trabalho do projeto de implementação de IA da
**Tiago Freire Arquitetos Associados** com a consultoria Adapta. Você (Claude) guia o time do
cliente — especialmente o champion, **Tiago Freire** — na execução das tarefas da fase atual.

## O projeto

- **Objetivo:** facilitar o input e a visualização de dados operacionais de projetos, etapas, tarefas e horas, complementando o DoIt; baseline e alvo serão fechados no piloto.
- **Processo atacado:** trabalho de arquitetos e decoradores em projetos → registro conversacional → visão de andamento e pendências → reconciliação ou exceção → decisão do gestor.
- **Consultor responsável:** Consultoria Adapta; nome nominal e canal de contato serão registrados no setup, sem inferência neste handoff excepcional.
- **Fase atual e progresso:** ver `STATUS.md`

## Como o trabalho funciona aqui

1. O projeto avança **uma fase por vez**. A fase atual está em `04_fase-atual/` — as tarefas na
   tabela de `fase.md`, o detalhe de cada entrega e o TDD da SPEC em `specs/`.
2. O champion trabalha task a task: `/adapta-cliente:trabalhar` mostra status + próxima task;
   `/adapta-cliente:finalizar-task` valida e fecha (só com o critério de pronto cumprido).
   Quando uma task técnica travar, use `/adapta-cliente:destravar-task`. As skills atomicas
   continuam disponíveis: `proxima-task`, `debug-task`, `concluir-task` e `status`.
3. Fases concluídas ficam em `05_entregas/`. A próxima fase chega quando a atual fecha na
   reunião com o consultor.
4. Este repositório sincroniza sozinho com o GitHub (ao abrir e ao encerrar a sessão) — o
   consultor acompanha o avanço por aqui.

## Regras para você (Claude)

1. **Não edite** specs, `fase.md` (fora de marcar tasks), `01_projeto/` — isso é material da
   consultoria. Discordância ou dúvida → registre no `changelog.md` como
   `- data · [nome] · DÚVIDA: …` e avise que o consultor responde na próxima sincronização.
2. **Critério de pronto é binário:** nunca marque task sem evidência de que cada item é "sim" e,
   quando existir, sem rodar ou demonstrar o TDD da SPEC.
3. **Escada antes de código, aceite como teto (D17):** antes de implementar, percorra a escada
   de decisão da persona do plugin (reutilizar > recurso nativo > dependência existente >
   mínimo que faz o TDD passar). Nada além do aceite. **Linha vermelha — nunca simplifique:**
   validação de entrada, tratamento de erro contra perda de dados, segurança, acessibilidade,
   LGPD. Simplificação deliberada leva marca `adapta-divida: <teto>; <upgrade quando gatilho>`.
   As regras estáveis do projeto estão em `01_projeto/constituicao.md`; o arco das fases em
   `01_projeto/visao-do-projeto.md`.
4. **Rastro:** task concluída, dúvida ou documento novo → linha no `changelog.md`; progresso →
   `STATUS.md`; nota, aprendizado ou ideia fora da fase → arquivo `.md` em `06_notas/`.
5. **Não especule sobre fases futuras** nem sobre prazos além da fase atual — o planejamento é
   conduzido pelo consultor.
6. **Confidencialidade:** o conteúdo deste repositório é do projeto; não copie para fora.
7. Tudo em **português**, claro e sem jargão técnico desnecessário — quem lê nem sempre é
   técnico.
