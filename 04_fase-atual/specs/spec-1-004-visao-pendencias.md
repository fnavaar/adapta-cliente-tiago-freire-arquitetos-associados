# SPEC-1-004 — Visão de gestão à vista e fila de pendências

**Fase:** 1
**Status:** bloqueada
**Dono:** coordenador de projetos
**Origem no escopo:** RQ-004, RQ-008, RQ-002, RQ-006; DEC-002, DEC-006, DEC-007; Fase 1 do `02-Escopo-Definitivo.md`
**Degrau da solução:** construção mínima — entrega uma visão filtrável sobre os registros complementares do piloto e uma fila de pendências, sem substituir a visão do DoIt.

## Contexto e decisões fechadas

- **Estado atual:** não existe mapa de processo completo nem visão reproduzível de projeto/etapa/responsável/estado nas fontes; a utilização do DoIt foi observada, mas não há contrato de exportação, API ou permissão de leitura confirmado.
- **Estado desejado:** Tiago/coordenador visualizam, por projeto, etapa, responsável e estado, registros válidos, desatualizados, incompletos, ambíguos e pendentes; cada pendência tem dono e próximo passo; sem dado, zero e não aplicável permanecem distintos.
- **Decisões já fechadas:** a visão é operacional, não financeira; origem e estado de integração ficam visíveis; falha não é escondida; nenhuma notificação ou correção automática é ativada.
- **Bloqueios:** **BLOQUEIO P1-007:** superfície técnica da visão, contrato de atualização e cadência de “desatualizado” dependem de P1-001/P1-005; não há valor de prazo nem ferramenta aprovados nas fontes.

## Resultado observável

Uma visão do piloto com filtros por projeto, etapa, responsável e estado, contendo pelo menos um registro válido e uma pendência acionável. O champion consegue abrir a pendência, identificar causa, dono, próximo passo, origem e estado de integração, e distinguir `sem dado`, `zero` e `não aplicável`.

## Limites e dependências

- **Inclui:** tabela/visão operacional; filtros mínimos; indicadores de completude e atualização conforme cadência aprovada; fila de pendências; causa/dono/próximo passo; origem/estado de integração; controle de acesso; links para auditoria.
- **Fora de escopo:** painel financeiro; margem/custo HH/faturamento/cobrança; relatório de produtividade; integração de escrita; notificação automática; rollout amplo; áudio/timer; correção automática de registros.
- **Entradas e pré-condições:** SPEC-1-001 aceita; SPEC-1-002 e SPEC-1-003 aceitas; registros de fixture; P1-007 aprovado; acesso de consulta do champion/coordenador conforme SPEC-1-002.
- **Saídas/artefatos:** visão demonstrável; fila de pendências; evidência de filtros e papéis; comparação baseline/observado quando o baseline estiver aprovado; veredito humano.
- **Dependências e responsáveis:** coordenador define filtros, dono e próximo passo; Tiago valida visão e alvo; administrador técnico confirma consulta; responsável pelo DoIt fornece referência se autorizada.
- **Atores e permissões mínimas:** champion e coordenador consultam o piloto; participantes veem somente o permitido pela matriz; financeiro permanece restrito; visão não concede permissão de edição.
- **Superfícies/arquivos/configurações afetadas:** visão e consulta na plataforma aprovada; nenhum dashboard, banco ou conector específico é escolhido nesta SPEC.
- **Risco e plano B:** se a fonte não puder ser consultada, demonstrar visão somente com fixture marcado como teste e fila manual de pendências; não preencher ausência com zero nem afirmar cobertura produtiva.
- **Rollback ou reversão:** desativar a visão de teste e preservar export/capturas; não apagar registros de origem nem logs.

## Dados e integrações

| Origem/destino | Fonte de verdade | Campos/contrato | Autenticação/permissão | Timeout/retry/idempotência | Tratamento de erro |
|---|---|---|---|---|---|
| Registros complementares → visão | camada complementar; DoIt somente como referência quando autorizada | projeto, etapa, responsável, status, momento, observação, origem, estado de integração, qualidade, event_id | leitura conforme SPEC-1-002 | consulta repetida não altera dados; cache só com timestamp visível | mostrar indisponibilidade como erro/pendência |
| Visão → fila de pendências | regras de qualidade do contrato | pending_id, motivo, registro_id, dono, próximo passo, estado, created_at, updated_at | somente coordenador/role aprovado encaminha | pending_id único; reabertura preserva histórico | impedir fechamento sem resolução/veredito |
| Visão → evidência de auditoria | trilha da SPEC-1-002 | link/event_id, ação, autor, momento, resultado | leitura restrita | nenhum dado de auditoria é reescrito | ausência de log marca `erro` e bloqueia aceite |

| Regra de negócio | Condição | Ação/resultado | Exceção | Fonte |
|---|---|---|---|---|
| RN-401 | filtro por projeto/etapa/responsável/estado | retornar somente itens permitidos ao papel | filtro vazio mostra ausência, não todos os dados | RQ-004 |
| RN-402 | campo mínimo ausente | indicar `incompleto` e criar/associar pendência | não converter em zero | seção 6 e 7 |
| RN-403 | sem evento no período aprovado | indicar `desatualizado` conforme cadência P1-007 | sem cadência aprovada permanece `não calculado`/pendente | RQ-008 |
| RN-404 | origem ou integração não conhecida | exibir estado e causa; encaminhar pendência | não mostrar como reconciliado | DEC-001, DEC-005 |
| RN-405 | dado financeiro | não coletar ou exibir | tentativa fica negada/auditada | DEC-007 |
| RN-406 | pendência sem dono ou próximo passo | impedir encerramento | coordenador deve encaminhar | seção 5 |

## Fluxo e regras

1. Usuário autorizado abre a visão do piloto.
2. Sistema aplica o filtro de permissão e mostra projeto, etapa, responsável, estado, última atualização, qualidade, origem e integração.
3. Usuário filtra por projeto, etapa, responsável e estado; a contagem e a lista respeitam os mesmos filtros.
4. Usuário abre um item incompleto, desatualizado, ambíguo, com erro ou pendente.
5. Sistema mostra causa, event_id/registro, dono, próximo passo, estado e link de auditoria.
6. Coordenador encaminha ou corrige conforme permissão; a visão não altera o registro sem o fluxo da SPEC-1-003.
7. Champion compara a demonstração com baseline/alvo somente quando P1-001 estiver preenchido e registra veredito.

| Cenário | Dado/condição | Resultado esperado | Caminho de erro/recuperação |
|---|---|---|---|
| Principal | fixture aceito + pendência com dono | visão mostra registro e fila acionável | capturar filtros, detalhes e veredito |
| Limite | `sem dado`, `zero` e `não aplicável` | três estados distintos e sem soma enganosa | coordenador revisa item sem normalizar |
| Falha | consulta indisponível | visão informa erro/última atualização e não inventa cobertura | criar pendência técnica |
| Permissão | participante consulta projeto fora do grupo | item não aparece; tentativa é auditada se aplicável | não contornar filtro |
| Operação | pendência sem dono | item destacado e não pode ser encerrado | encaminhar ao coordenador |
| Regressão | registro corrigido pela entrada textual | visão atualiza preservando histórico/auditoria | comparar before/after |

## Instruções de execução para o Ethos

1. **Ler antes de alterar:** SPEC-1-001, SPEC-1-002 e SPEC-1-003; `02-Escopo-Definitivo.md` §§ 5–7, 9 e 10; esta SPEC.
2. **Alterar somente:** consulta, visão e fila de pendências da superfície aprovada.
3. **Não alterar:** fonte original do DoIt, dados financeiros, permissões, registro sem confirmação, integrações externas ou notificações.
4. **Executar nesta ordem:** fechar P1-007 → carregar fixture → aplicar controle de acesso → validar filtros → validar qualidade → abrir/encaminhar pendência → executar regressão.
5. **Parar e pedir validação quando:** faltar cadência, baseline, alvo, denominador, plataforma, permissão de consulta ou regra de desatualização.
6. **Estado válido ao parar:** visão de teste identificada; ausência de dado visível; nenhuma pendência encerrada sem dono/próximo passo/veredito.

## Checklist de execução

- [ ] Visão e cadência de atualização foram aprovadas.
- [ ] Filtros mínimos funcionam por projeto, etapa, responsável e estado.
- [ ] Controle de acesso foi exercitado com papéis distintos.
- [ ] Registro válido e pendência aparecem na mesma demonstração.
- [ ] `sem dado`, `zero` e `não aplicável` permanecem distintos.
- [ ] Origem, estado de integração, event_id e auditoria são visíveis.
- [ ] Falha de consulta e pendência sem dono foram testadas.
- [ ] Champion/coordenador registraram veredito.

## Critérios de aceite

- [ ] **CA-1-017:** visão filtrável mostra projeto, etapa, responsável, status e última atualização somente para o papel permitido.
- [ ] **CA-1-018:** visão distingue registro incompleto, desatualizado, ambíguo, erro e pendente sem ocultar falha.
- [ ] **CA-1-019:** fila mostra causa, registro/event_id, origem, estado de integração, dono e próximo passo.
- [ ] **CA-1-020:** `sem dado`, `zero` e `não aplicável` aparecem como estados diferentes.
- [ ] **CA-1-021:** usuário sem permissão não visualiza itens fora do recorte nem dados financeiros.
- [ ] **CA-1-022:** falha de consulta não gera cobertura falsa e produz pendência técnica recuperável.
- [ ] **CA-1-023:** uma demonstração com fixture, pendência, filtro e auditoria é aceita por Tiago/coordenador.

## TDD da SPEC

| Etapa | Prova | Comando/ação | Resultado esperado | Evidência |
|---|---|---|---|---|
| RED | visão recebe item incompleto e pendência sem dono | carregar fixture sem responsável e abrir a fila | item não é apresentado como válido; pendência destaca falta de dono | captura da visão e fila |
| GREEN | visão com registro aceito + pendência acionável | filtrar por projeto/etapa/responsável/estado e abrir detalhes | resultados corretos, estados distintos, dono/próximo passo/auditoria visíveis | roteiro, capturas e veredito |
| REFACTOR/REGRESSÃO | fonte indisponível + papel restrito + correção textual | interromper consulta, acessar fora do grupo e corrigir fixture pela SPEC-1-003 | erro/negação seguros; visão atualiza sem perder histórico | relatório integrado |

**Dados/fixtures:** `FIX-1-001` aceito; `FIX-1-002` sem responsável; `FIX-1-003` com `sem dado`; `FIX-1-004` com valor `zero`; `FIX-1-005` com `não aplicável`; pendência sem dono; contas da SPEC-1-002.
**Caminhos de erro obrigatórios:** filtro sem resultado, falta de permissão, fonte indisponível, estado de integração pendente, registro incompleto, pendência sem dono e dado financeiro.
**Evidência exigida:** capturas dos filtros e detalhes, auditoria, fila, resultado de consulta indisponível, matriz de permissão e aceite humano.

## Handoff e operação

- **Como demonstrar:** abrir a visão com registro e pendência, filtrar por responsável, abrir causa/dono/próximo passo e mostrar origem/auditoria.
- **Como operar depois:** coordenador revisa a fila na cadência aprovada; Tiago dá o veredito do piloto; participantes não encerram exceções sem regra.
- **Como monitorar:** quantidade de registros válidos, incompletos, desatualizados, pendências sem dono, erros e duplicidades.
- **Pendência conhecida:** P1-007 bloqueia a implementação; sem baseline/cadência não há aceite quantitativo.

## Tasks vinculadas

| ID | Task | Dono | SPEC | Critério | Recorte da prova | Evidência esperada | Pré-condições | Status |
|---|---|---|---|---|---|---|---|---|
| F1-T010 | Configurar visão operacional e filtros mínimos | Coordenador de projetos | SPEC-1-004 | CA-1-017, CA-1-018 | carregar fixtures e filtrar por projeto, etapa, responsável e estado | capturas, contagens e estados | P1-007 fechado; superfície aprovada | bloqueada — exceção humana |
| F1-T011 | Validar fila de pendências, origem, integração e permissões | Coordenador de projetos | SPEC-1-004 | CA-1-019 a CA-1-021 | abrir pendência, mostrar dono/próximo passo e testar papel restrito | fila, detalhes, event_id e auditoria | F1-T010 concluída; matriz disponível | bloqueada — exceção humana |
| F1-T012 | Exercitar falha de consulta e registrar aceite do champion | Tiago Freire | SPEC-1-004 | CA-1-022, CA-1-023 | interromper consulta e executar roteiro do champion | relatório, pendência e veredito | F1-T010/F1-T011 concluídas; baseline aprovado | bloqueada — exceção humana |

## Emendas

<!-- Append-only (D19): mudanças aprovadas depois da geração. A história não é reescrita. -->

| Data | Origem do sinal | Micro-spec/task | Motivo |
|---|---|---|---|
| | | | |
