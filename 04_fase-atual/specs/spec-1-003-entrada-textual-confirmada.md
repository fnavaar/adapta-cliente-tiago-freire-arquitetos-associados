# SPEC-1-003 — Entrada conversacional textual com prévia e confirmação

**Fase:** 1
**Status:** bloqueada
**Dono:** coordenador de projetos
**Origem no escopo:** RQ-002, RQ-006, RQ-007; DEC-001, DEC-003, DEC-006; Fase 1 do `02-Escopo-Definitivo.md`
**Degrau da solução:** construção mínima — implementa somente a jornada textual complementar, com confirmação humana e sem escrita presumida no DoIt.

## Contexto e decisões fechadas

- **Estado atual:** a equipe usa o DoIt, mas a jornada de registro e o atrito do input não estão descritos em fluxo reproduzível; áudio e integração são condicionais e não podem ser usados como base desta SPEC.
- **Estado desejado:** usuário autorizado informa projeto, etapa e evento por texto; o sistema identifica campos ausentes/ambíguos, apresenta prévia, recebe confirmação explícita e registra autoria, momento, origem e estado sem criar registro órfão.
- **Decisões já fechadas:** chat textual é o primeiro caminho; confirmação humana é obrigatória; seleção deve ser inequívoca; erro, duplicidade e conflito não aparecem como sucesso; a camada complementar não substitui o DoIt.
- **Bloqueios:** **BLOQUEIO P1-005:** canal/plataforma de chat e superfície técnica não estão identificados. **BLOQUEIO P1-006:** o formato final da prévia e o vocabulário aprovado de estados ainda não foram validados por Tiago/coordenador. A SPEC define o conteúdo mínimo, mas não escolhe interface, stack ou conector.

## Resultado observável

Em uma conversa textual de teste, o usuário autorizado envia uma atualização, recebe prévia com todos os campos, corrige ou esclarece o que faltar, confirma uma única vez e obtém um registro aceito na camada complementar. A visão posterior consegue localizar o registro e mostrar que a escrita no DoIt continua `pendente` ou fora do recorte.

## Limites e dependências

- **Inclui:** entrada de texto; extração/proposta de projeto, etapa, responsável, status, momento e observação; pedidos de esclarecimento; prévia; confirmar, editar/esclarecer e cancelar; event_id/idempotência; autoria e origem; estados de erro e pendência.
- **Fora de escopo:** áudio/transcrição; timer; escrita automática no DoIt; leitura financeira; notificação a cliente; classificação autônoma sem revisão; alteração de preço, contrato ou proposta.
- **Entradas e pré-condições:** SPEC-1-001 aceita; SPEC-1-002 com papel e sessão aprovados; projeto, etapa e responsáveis de teste; canal P1-005 disponível; contrato de prévia P1-006 aprovado.
- **Saídas/artefatos:** registro operacional confirmado; log de prévia/decisão; pendência quando incompleto; event_id único; evidência de demonstração.
- **Dependências e responsáveis:** coordenador define campos/estados e valida ambiguidade; usuário confirma; administrador técnico disponibiliza canal; responsável pelo DoIt confirma somente a referência de leitura, sem autorizar escrita por inferência.
- **Atores e permissões mínimas:** participante autorizado pode iniciar e confirmar seu registro; coordenador pode revisar/corrigir conforme matriz; consultor/agent não confirma em nome do usuário; leitura e edição seguem SPEC-1-002.
- **Superfícies/arquivos/configurações afetadas:** canal textual e camada complementar aprovados; nenhum caminho de API, banco, webhook, secret ou prompt persistente é criado sem P1-005.
- **Risco e plano B:** se a extração não for inequívoca, retornar perguntas e manter rascunho não produtivo; se o canal cair, registrar pendência manual com dono; nunca assumir valor ou enviar sucesso falso.
- **Rollback ou reversão:** cancelar rascunho antes da confirmação; para correção posterior, criar evento de correção preservando o valor anterior; não apagar auditoria nem alterar DoIt.

## Dados e integrações

| Origem/destino | Fonte de verdade | Campos/contrato | Autenticação/permissão | Timeout/retry/idempotência | Tratamento de erro |
|---|---|---|---|---|---|
| Mensagem textual → prévia | mensagem confirmada pelo usuário; projeto/etapa/responsável vêm da fonte operacional autorizada | texto, actor_id, projeto, etapa, responsável, status, momento, observação, campos ausentes/ambíguos | sessão e papel da SPEC-1-002 | cada tentativa recebe request_id; timeout não confirma | manter `pendente`, pedir esclarecimento ou cancelar |
| Prévia confirmada → registro complementar | camada complementar | event_id, valores confirmados, autoria, timestamp, origem, `registro_status=aceito`, `estado_integracao=pendente` quando não houver escrita DoIt | somente papel autorizado | confirmação repetida com mesmo request_id não cria segundo registro | mostrar resultado idempotente e auditar repetição |
| Registro complementar → DoIt | DoIt permanece referência existente; escrita fora desta fase | nenhuma escrita autorizada por esta SPEC | sem credencial de escrita | não tentar retry de escrita | estado fica `pendente` com dono/próximo passo |

| Regra de negócio | Condição | Ação/resultado | Exceção | Fonte |
|---|---|---|---|---|
| RN-301 | projeto, etapa ou responsável não são inequívocos | pedir esclarecimento e não permitir confirmação | se não houver opção válida, criar pendência | RQ-002, seção 5 |
| RN-302 | prévia sem todos os campos mínimos | bloquear confirmação | usuário pode editar ou cancelar | seção 6, Fase 1 |
| RN-303 | usuário confirma prévia válida | criar um registro com event_id único | retry idêntico retorna o mesmo resultado | seção 7 |
| RN-304 | timeout, conflito ou erro de fonte | estado `pendente` ou `erro`, com causa/dono/próximo passo | nunca usar mensagem de sucesso | RQ-007 |
| RN-305 | usuário cancela ou rejeita | não criar registro aceito; preservar evento de auditoria | rascunho pode expirar conforme política aprovada | DEC-003 |

## Fluxo e regras

1. Usuário autorizado envia texto com projeto, etapa e evento.
2. Sistema consulta somente fontes autorizadas e apresenta os campos identificados.
3. Se houver campo ausente/ambíguo, sistema pergunta e mantém rascunho não confirmado.
4. Sistema apresenta prévia contendo projeto, etapa, responsável, status, momento, observação, origem, estado de integração e campos ainda pendentes.
5. Usuário escolhe `confirmar`, `editar/esclarecer` ou `cancelar`.
6. Após `confirmar`, sistema cria o registro complementar uma vez, registra auditoria e retorna event_id/estado.
7. Falha, duplicidade ou DoIt não autorizado mantém o evento pendente e encaminhável.

| Cenário | Dado/condição | Resultado esperado | Caminho de erro/recuperação |
|---|---|---|---|
| Principal | texto completo + usuário autorizado | prévia correta; uma confirmação; registro aceito na camada complementar | anexar event_id e evidência |
| Limite | texto sem etapa ou com duas etapas possíveis | pergunta de esclarecimento; nenhuma escrita | após resposta, regenerar prévia; após abandono, pendência |
| Falha | fonte operacional indisponível | mensagem de pendência sem sucesso falso | registrar causa e dono; não confirmar |
| Duplicidade | mesma confirmação reenviada | mesmo event_id/resultado, sem segundo registro | auditar retry |
| Cancelamento | usuário rejeita prévia | nenhum registro aceito | preservar rejeição para auditoria |
| Segurança | usuário sem papel de registro | ação negada antes da prévia sensível | auditar negativa; não revelar dados |

## Instruções de execução para o Ethos

1. **Ler antes de alterar:** SPEC-1-001 e SPEC-1-002; `02-Escopo-Definitivo.md` §§ 5–7 e 9; contrato de campos; esta SPEC.
2. **Alterar somente:** fluxo textual e camada complementar da superfície aprovada.
3. **Não alterar:** DoIt, dados financeiros, áudio, timer, credenciais, prompts/skills fora do recorte ou permissões.
4. **Executar nesta ordem:** fechar P1-005/P1-006 → validar sessão → criar rascunho → exercitar esclarecimento → prévia → confirmação → retry/falha → evidência.
5. **Parar e pedir validação quando:** faltar canal, fonte de referência, campo, regra de estado, papel ou formato aprovado da prévia; ou quando a implementação exigir escrita no DoIt.
6. **Estado válido ao parar:** rascunhos não confirmados não aparecem como aceitos; nenhum retry gera duplicata; DoIt permanece sem escrita.

## Checklist de execução

- [ ] Canal textual e superfície técnica foram confirmados.
- [ ] Conteúdo e ações da prévia foram aprovados.
- [ ] Campos ausentes, ambíguos e inválidos geram esclarecimento/pendência.
- [ ] Confirmação exige sessão e papel válidos.
- [ ] Registro confirmado contém event_id, origem, autoria e estado de integração.
- [ ] Retry, timeout, cancelamento e fonte indisponível foram exercitados.
- [ ] Nenhuma escrita no DoIt ocorreu.

## Critérios de aceite

- [ ] **CA-1-011:** entrada completa produz prévia com todos os campos mínimos antes de qualquer confirmação.
- [ ] **CA-1-012:** entrada ambígua ou incompleta não pode ser confirmada e gera pergunta/pendência com dono.
- [ ] **CA-1-013:** confirmação válida cria exatamente um registro complementar com event_id, autoria, momento, origem e estado de integração.
- [ ] **CA-1-014:** reenvio do mesmo request_id não cria duplicata.
- [ ] **CA-1-015:** timeout, cancelamento, erro de fonte e papel negado não aparecem como sucesso.
- [ ] **CA-1-016:** a demonstração comprova que o DoIt não recebeu escrita nesta fase.

## TDD da SPEC

| Etapa | Prova | Comando/ação | Resultado esperado | Evidência |
|---|---|---|---|---|
| RED | confirmação sem etapa | enviar `Projeto Casa Azul — revisar` e tentar confirmar | sistema pede esclarecimento e bloqueia confirmação | conversa/captura + estado do rascunho |
| GREEN | registro completo | enviar fixture completo, responder esclarecimentos, confirmar uma vez | prévia aprovada, um event_id, registro aceito e estado DoIt pendente | conversa, registro, auditoria |
| REFACTOR/REGRESSÃO | retry + timeout + cancelamento | reenviar request_id; interromper fonte; rejeitar prévia | resultado idempotente; erro recuperável; nenhum falso sucesso/duplicata | relatório de cenários |

**Dados/fixtures:** `FIX-1-001` da SPEC-1-001; entrada ambígua `Projeto Casa Azul — atualizar`; entrada inválida `Projeto Inexistente`; reenviar o mesmo request_id; usar contas da SPEC-1-002.
**Caminhos de erro obrigatórios:** campo ausente, múltiplas opções, entidade inexistente, sessão expirada, papel negado, timeout, fonte indisponível, confirmação duplicada e cancelamento.
**Evidência exigida:** conversa completa, prévia, decisão, event_id, registro, auditoria, estados de erro e confirmação de ausência de escrita no DoIt.

## Handoff e operação

- **Como demonstrar:** enviar o fixture, resolver uma ambiguidade, confirmar, repetir a confirmação e mostrar o estado pendente do DoIt.
- **Como operar depois:** equipe registra por texto; coordenador acompanha pendências e corrige conforme matriz; Tiago decide expansão.
- **Como monitorar:** taxa de esclarecimentos, rejeições, duplicidades, erros e registros sem dono.
- **Pendência conhecida:** P1-005 e P1-006 bloqueiam implementação; áudio e escrita DoIt permanecem fora.

## Tasks vinculadas

| ID | Task | Dono | SPEC | Critério | Recorte da prova | Evidência esperada | Pré-condições | Status |
|---|---|---|---|---|---|---|---|---|
| F1-T007 | Fechar canal textual e contrato da prévia | Coordenador de projetos | SPEC-1-003 | CA-1-011, CA-1-012 | enviar entrada completa, ambígua e incompleta | conversa, prévia e rascunhos | P1-005/P1-006 fechados; canal aprovado | bloqueada — exceção humana |
| F1-T008 | Demonstrar confirmação única e idempotência | Coordenador de projetos | SPEC-1-003 | CA-1-013, CA-1-014 | confirmar fixture e reenviar request_id | event_id, registro único e retry | F1-T007 concluída; sessão válida | bloqueada — exceção humana |
| F1-T009 | Exercitar erros, cancelamento e ausência de escrita no DoIt | Coordenador de projetos | SPEC-1-003 | CA-1-015, CA-1-016 | testar timeout, cancelamento, papel negado e consulta DoIt | estados de erro e prova de não escrita | F1-T007 concluída; acesso somente leitura | bloqueada — exceção humana |

## Emendas

<!-- Append-only (D19): mudanças aprovadas depois da geração. A história não é reescrita. -->

| Data | Origem do sinal | Micro-spec/task | Motivo |
|---|---|---|---|
| | | | |
