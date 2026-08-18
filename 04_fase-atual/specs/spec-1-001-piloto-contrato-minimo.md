# SPEC-1-001 — Piloto e contrato mínimo do registro operacional

**Fase:** 1
**Status:** bloqueada
**Dono:** Tiago Freire / coordenador de projetos
**Origem no escopo:** RQ-001, RQ-002, RQ-008; DEC-002, DEC-004, DEC-010; Fase 1 do `02-Escopo-Definitivo.md`
**Degrau da solução:** construção mínima — cria somente o contrato operacional necessário para demonstrar o primeiro registro e a primeira visão, mantendo o DoIt como referência existente.

## Contexto e decisões fechadas

- **Estado atual:** o DoIt é usado pela equipe, mas os mapas de processo não descrevem um fluxo completo reproduzível; não há projeto-piloto, grupo, baseline, alvo, cadência ou matriz de papéis confirmados nos artefatos disponíveis (`04-Mapeamento-Processos/02-Processos_mapeados/`).
- **Estado desejado:** existe uma ficha de piloto e um contrato mínimo de dados que permitem registrar e visualizar uma atualização operacional sem alegar sincronização automática com o DoIt.
- **Decisões já fechadas:** o DoIt não é substituído; chat textual é o primeiro caminho; o registro exige projeto, etapa, responsável, status, momento, observação e confirmação; áudio, timer, finanças, rollout e escrita automática no DoIt estão fora desta fase.
- **Bloqueios:** **BLOQUEIO P1-001:** faltam valores aprovados para projeto-piloto, grupo-piloto, período, cadência, definição de “atualizado”, baseline, alvo, denominador, fonte de medição e dono das pendências. **BLOQUEIO P1-002:** a plataforma/superfície de implementação e a matriz concreta de papéis ainda não foram identificadas.

## Resultado observável

Uma ficha de piloto aprovada e um contrato de registro preenchido com um fixture não produtivo. O champion consegue demonstrar qual projeto, etapa, responsável, status, momento, observação, origem e estado de integração são aceitos como registro mínimo; a demonstração para no bloqueio se qualquer valor operacional obrigatório estiver ausente.

## Limites e dependências

- **Inclui:** ficha de piloto; grupo autorizado; campos mínimos; estados mínimos; definição de registro válido/atualizado; baseline e alvo; dono e cadência de pendências; contrato de origem e estado de integração; fixture controlado.
- **Fora de escopo:** escrita no DoIt; API, scraping ou banco direto; áudio; timer; horas, custo HH, margem, faturamento ou cobrança; rollout amplo; notificações automáticas; alteração de preço, contrato, proposta ou mensagem a cliente.
- **Entradas e pré-condições:** check de escopo aprovado; confirmação humana dos bloqueios P1-001 e P1-002; acesso somente de leitura às referências existentes que forem usadas; fixture identificado como teste.
- **Saídas/artefatos:** ficha de piloto aprovada; contrato de campos e estados; matriz inicial de papéis; registro de fixture; evidência da decisão de seguir, ajustar ou parar.
- **Dependências e responsáveis:** Tiago decide piloto, grupo, alvo e veredito; coordenador define operação das pendências; administrador do DoIt confirma a referência de dados e limites técnicos; consultor registra o contrato sem aprovar gates em nome deles.
- **Atores e permissões mínimas:** champion aprova o piloto e consulta resultado; coordenador opera pendências; participante registra somente dentro do grupo autorizado; administrador técnico fornece limites de acesso; administrativo/financeiro não recebe dados financeiros por padrão. A concessão concreta fica bloqueada até a SPEC-1-002.
- **Superfícies/arquivos/configurações afetadas:** ficha/configuração do piloto e camada complementar aprovada; nenhum caminho técnico de implementação foi inventado. O Ethos deve parar antes de escolher aplicativo, banco, conector ou arquivo de cliente.
- **Risco e plano B:** se não houver acesso ao DoIt ou referência confiável, manter o fixture e marcar a origem como `pendente`; não usar dado inventado como dado produtivo. Se os valores do piloto não forem aprovados, não iniciar registro real.
- **Rollback ou reversão:** arquivar a ficha e excluir/desativar somente o fixture de teste, preservando a evidência da decisão; não remover dados existentes do DoIt.

## Dados e integrações

| Origem/destino | Fonte de verdade | Campos/contrato | Autenticação/permissão | Timeout/retry/idempotência | Tratamento de erro |
|---|---|---|---|---|---|
| DoIt → camada complementar, somente leitura se autorizado | DoIt para projetos, etapas e responsáveis já existentes | identificador, nome, responsável, estado conhecido; ausência deve ser explícita | **BLOQUEIO P1-003:** acesso, método e administrador ainda não confirmados | não há chamada de escrita nesta fase; consulta indisponível vira `pendente` | criar pendência com causa, dono e próximo passo; nunca afirmar sincronização |
| Ficha de piloto → camada complementar | ficha aprovada por Tiago/coordenador | projeto-piloto, grupo, período, cadência, baseline, alvo, denominador, fonte e dono | somente aprovadores nomeados no gate P1-001 | uma versão ativa por piloto; nova versão exige histórico | impedir ativação se campo obrigatório estiver vazio |

| Regra de negócio | Condição | Ação/resultado | Exceção | Fonte |
|---|---|---|---|---|
| RN-101 | registro mínimo | exigir projeto, etapa, responsável, status, momento e observação | campo não disponível gera pendência, não registro válido | `02-Escopo-Definitivo.md`, seção 6 |
| RN-102 | valor de qualidade | distinguir `sem dado`, `zero` e `não aplicável` | valor desconhecido permanece `sem dado` | `02-Escopo-Definitivo.md`, seção 7 |
| RN-103 | integração não comprovada | separar `registro_status` de `estado_integracao`; o primeiro pode ser `aceito` após confirmação, o segundo permanece `pendente` quando o DoIt não foi escrito | erro de consulta não vira sucesso | DEC-001, DEC-005 |
| RN-104 | dado financeiro | impedir campos de custo, margem, faturamento, cobrança e contrato no contrato da fase 1 | pedido financeiro vira rejeitado/fora do recorte com auditoria | DEC-007, seção 9 |

## Fluxo e regras

1. Tiago e o coordenador fornecem os valores de P1-001 e aprovam a ficha do piloto.
2. O administrador técnico confirma a referência de leitura e registra P1-003; se não confirmar, a origem fica `pendente`.
3. O responsável cria o contrato mínimo com os campos e estados de RN-101 a RN-104.
4. O responsável registra o fixture `FIX-1-001` e verifica que a ficha e o contrato permitem a demonstração da SPEC-1-003 e SPEC-1-004.
5. O responsável para e solicita validação se projeto, grupo, cadência, baseline, alvo, papel ou superfície estiverem ausentes.

| Cenário | Dado/condição | Resultado esperado | Caminho de erro/recuperação |
|---|---|---|---|
| Principal | ficha completa + `FIX-1-001` | contrato ativo e fixture pronto para entrada textual | registrar evidência e solicitar aceite |
| Limite | observação vazia ou projeto não selecionado | contrato rejeita o fixture como registro válido | manter pendência com dono e próximo passo |
| Falha | referência DoIt indisponível | contrato permanece utilizável para o complemento, com `estado_integracao=pendente` | não reprocessar automaticamente; registrar causa |
| Permissão | pessoa fora do grupo tenta ativar piloto | ação negada e auditada | coordenador revisa acesso; não ampliar permissão por inferência |

## Instruções de execução para o Ethos

1. **Ler antes de alterar:** `03-Projeto/02-Escopo-Definitivo.md` §§ 4–7, 9–12; `03-Projeto/02-Plano_de_acao/matriz-de-rastreabilidade.md`; esta SPEC; SPEC-1-002; SPEC-1-003; SPEC-1-004.
2. **Alterar somente:** ficha/contrato do piloto na superfície aprovada e fixture de teste.
3. **Não alterar:** DoIt, dados financeiros, credenciais, permissões fora da matriz aprovada ou qualquer integração não documentada.
4. **Executar nesta ordem:** fechar P1-001 → fechar P1-002/P1-003 → criar ficha → criar contrato → registrar fixture → demonstrar o caminho.
5. **Parar e pedir validação quando:** qualquer bloqueio P1-001, P1-002 ou P1-003 estiver aberto; houver necessidade de escolher plataforma, banco, conector, regra de cálculo ou permissão.
6. **Estado válido ao parar:** nenhuma escrita no DoIt; ficha e contrato permanecem inativos até validação; fixture permanece marcado como teste.

## Checklist de execução

- [ ] Projeto, grupo, período e cadência do piloto estão registrados com aprovador.
- [ ] Baseline, alvo, denominador, fonte e definição de atualização estão registrados.
- [ ] Dono e próximo passo de cada pendência estão definidos.
- [ ] Campos, estados e separação de origem/integração foram conferidos.
- [ ] Matriz inicial de papéis foi confirmada pelo responsável.
- [ ] `FIX-1-001` foi exercitado sem dados produtivos.
- [ ] Evidência do veredito humano foi anexada.

## Critérios de aceite

- [ ] **CA-1-001:** ficha de piloto aprovada contém projeto, grupo, período, cadência, baseline, alvo, denominador, fonte, dono e veredito.
- [ ] **CA-1-002:** contrato rejeita atualização sem projeto, etapa, responsável, status, momento ou observação.
- [ ] **CA-1-003:** contrato representa `sem dado`, `zero` e `não aplicável` como estados distintos.
- [ ] **CA-1-004:** nenhum campo financeiro ou escrita no DoIt é ativado pela SPEC.
- [ ] **CA-1-005:** `FIX-1-001` está identificável como teste e possui evidência de origem e estado de integração.

## TDD da SPEC

| Etapa | Prova | Comando/ação | Resultado esperado | Evidência |
|---|---|---|---|---|
| RED | ativação com ficha incompleta | tentar ativar sem alvo, denominador ou dono de pendência | falha explícita com `BLOQUEIO P1-001`; nada produtivo é criado | registro da tentativa e captura da mensagem |
| GREEN | ativação com ficha completa | preencher a ficha aprovada e registrar `FIX-1-001` | contrato ativo, fixture aceito como teste e campos de origem/integração preenchidos | ficha, fixture e aceite do coordenador |
| REFACTOR/REGRESSÃO | distinção de qualidade e fora de recorte | enviar `sem dado`, `zero`, `não aplicável` e pedido financeiro | estados permanecem distintos; pedido financeiro é rejeitado/fora do recorte | tabela de resultados e auditoria |

**Dados/fixtures:** `FIX-1-001`: projeto fictício `Projeto Casa Azul`, etapa `Estudo preliminar`, responsável `Ana Lima`, status `em andamento`, momento `2026-08-18T09:00:00-03:00`, observação `Revisar planta até 2026-08-22`; marcar como não produtivo. Usar também projeto inexistente e observação vazia.
**Caminhos de erro obrigatórios:** ficha incompleta, projeto/etapa/responsável inexistente, papel negado, referência DoIt indisponível e pedido de dado financeiro.
**Evidência exigida:** ficha aprovada, contrato versionado, resultados do fixture, registro de bloqueios e veredito humano.

## Handoff e operação

- **Como demonstrar:** abrir a ficha, mostrar o contrato, registrar `FIX-1-001` e apontar a origem/estado sem alegar sincronização.
- **Como operar depois:** coordenador mantém ficha, grupo e pendências; Tiago valida mudanças de baseline/alvo.
- **Como monitorar:** revisão da ficha antes de cada ciclo e contagem de fixtures/pendências abertas.
- **Pendência conhecida:** P1-001, P1-002 e P1-003 bloqueiam a execução produtiva.

## Tasks vinculadas

| ID | Task | Dono | SPEC | Critério | Recorte da prova | Evidência esperada | Pré-condições | Status |
|---|---|---|---|---|---|---|---|---|
| F1-T001 | Registrar ficha do piloto e contrato mínimo | Coordenador de projetos | SPEC-1-001 | CA-1-001, CA-1-005 | preencher ficha, contrato e `FIX-1-001` | ficha, contrato, fixture e veredito | P1-001/P1-002/P1-003 fechados | bloqueada — exceção humana |
| F1-T002 | Exercitar campos obrigatórios e estados de qualidade | Coordenador de projetos | SPEC-1-001 | CA-1-002 a CA-1-004 | testar ausência, `sem dado`, `zero`, `não aplicável` e financeiro | tabela, auditoria e rejeições | F1-T001 concluída; contrato ativo | bloqueada — exceção humana |
| F1-T003 | Registrar rollback e handoff do piloto | Coordenador de projetos | SPEC-1-001 | CA-1-005 | arquivar fixture, preservar evidência e demonstrar roteiro | rollback, roteiro e aceite | F1-T001 concluída; fixture não produtivo | bloqueada — exceção humana |

## Emendas

<!-- Append-only (D19): mudanças aprovadas depois da geração. A história não é reescrita. -->

| Data | Origem do sinal | Micro-spec/task | Motivo |
|---|---|---|---|
| | | | |
