# SPEC-1-002 — Papéis, sessão e trilha de auditoria do piloto

**Fase:** 1
**Status:** bloqueada
**Dono:** administrador/responsável técnico do DoIt + coordenador de projetos
**Origem no escopo:** RQ-006, RQ-002; DEC-006; Fase 1 do `02-Escopo-Definitivo.md`
**Degrau da solução:** recurso nativo da plataforma aprovada, com construção mínima somente se a plataforma não oferecer sessão, controle por papel e auditoria; o Ethos não escolhe a plataforma por conta própria.

## Contexto e decisões fechadas

- **Estado atual:** o escopo exige papéis, permissões/RLS, sessão e auditoria, mas a matriz concreta de papéis, o ambiente de execução, os acessos do DoIt e a exposição autorizada de dados não estão documentados nas fontes.
- **Estado desejado:** cada ação do piloto é permitida ou negada por papel e sessão; criação, confirmação, correção, consulta, encaminhamento e falha deixam autoria, momento, origem e resultado auditáveis.
- **Decisões já fechadas:** dados financeiros ficam restritos e fora da visão da fase 1; nenhum agente altera preço, contrato, proposta ou mensagem a cliente; confirmação humana é obrigatória antes de criar/atualizar registro.
- **Bloqueios:** **BLOQUEIO P1-004:** faltam nomes dos papéis reais, permissões por ação, responsável pela administração, mecanismo de sessão/RLS, retenção da auditoria e plataforma onde a regra será aplicada.

## Resultado observável

Uma matriz de autorização do piloto e uma trilha de auditoria demonstrável: um usuário autorizado consegue iniciar e confirmar um registro; um usuário não autorizado é impedido; ambas as ações, inclusive a negativa, deixam evento auditável sem expor dados financeiros.

## Limites e dependências

- **Inclui:** papéis do piloto; sessão; autorização por ação; proteção de dados financeiros; auditoria de entrada, prévia, confirmação, edição, cancelamento, rejeição, erro e tentativa negada.
- **Fora de escopo:** login novo sem decisão de plataforma; SSO; provisionamento em massa; integração de identidade não autorizada; acesso a horas/custos; auditoria de dados financeiros; autonomia de agente; comunicação externa.
- **Entradas e pré-condições:** SPEC-1-001 aceita; matriz P1-004 aprovada; contas de teste com papéis distintos; política de retenção e administrador nomeados.
- **Saídas/artefatos:** matriz permitido/negado; configuração de sessão; eventos de auditoria; relatório de teste de acesso; decisão de seguir, ajustar ou parar.
- **Dependências e responsáveis:** administrador técnico implementa; coordenador valida operação; Tiago aprova o recorte de usuários; administrativo/financeiro valida o bloqueio financeiro; consultor registra evidências.
- **Atores e permissões mínimas:** os nomes e concessões são P1-004. O contrato mínimo exige pelo menos uma conta autorizada a registrar, uma conta autorizada a consultar, uma conta negada e uma conta de teste sem exposição financeira.
- **Superfícies/arquivos/configurações afetadas:** controle de acesso, sessão e auditoria na plataforma aprovada; nenhum arquivo, conector ou serviço é escolhido nesta SPEC.
- **Risco e plano B:** se a plataforma não suportar RLS/auditoria mínima, manter o piloto em dry-run com exportação controlada e registrar a limitação; não substituir o controle por planilha não protegida ou promessa verbal.
- **Rollback ou reversão:** desativar contas de teste e remover somente regras/configurações criadas para o piloto, preservando logs e evidência; não revogar acessos existentes fora do recorte sem autorização.

## Dados e integrações

| Origem/destino | Fonte de verdade | Campos/contrato | Autenticação/permissão | Timeout/retry/idempotência | Tratamento de erro |
|---|---|---|---|---|---|
| Conta/sessão → camada complementar | diretório/plataforma de identidade aprovada | user_id, papel, status de sessão, início/fim, motivo de negativa | **BLOQUEIO P1-004:** mecanismo e administrador não confirmados | reautenticação conforme política aprovada; nenhuma criação automática de conta | negar por padrão e registrar causa sem revelar dados sensíveis |
| Ação → trilha de auditoria | camada complementar; DoIt não é escrito | event_id, ação, actor_id, papel, timestamp, origem, registro_id, resultado, motivo, before/after quando permitido | acesso de leitura restrito a aprovadores | event_id único; duplicidade de evento deve ser detectável | falha de auditoria bloqueia confirmação produtiva e cria incidente/pendência |

| Regra de negócio | Condição | Ação/resultado | Exceção | Fonte |
|---|---|---|---|---|
| RN-201 | sessão inválida ou expirada | negar leitura/escrita | não usar sessão anônima para confirmação | DEC-006 |
| RN-202 | papel sem permissão para ação | negar antes da alteração e auditar negativa | não oferecer atalho por prompt ou link | RQ-006 |
| RN-203 | dado financeiro solicitado na fase 1 | ocultar/não coletar e registrar fora do recorte | administrativo-financeiro só participa da validação de acesso | DEC-007 |
| RN-204 | auditoria indisponível | não confirmar registro produtivo | permitir somente fixture isolado, identificado como teste | risco de segurança do escopo |
| RN-205 | correção autorizada | preservar valor anterior, autor, momento, origem e justificativa | justificativa obrigatória quando definida por P1-004 | seção 7 do escopo |

## Fluxo e regras

1. Administrador técnico aplica somente a matriz P1-004 na plataforma aprovada.
2. Criar contas de teste: autorizado a registrar, autorizado a consultar, negado e sem acesso financeiro.
3. Iniciar sessão e executar uma entrada do fixture por cada papel.
4. Verificar que a prévia é visível antes da confirmação e que a auditoria registra a ação.
5. Tentar ação proibida, sessão expirada e indisponibilidade da auditoria.
6. Parar antes de qualquer uso produtivo se uma negativa não for auditada ou houver exposição financeira.

| Cenário | Dado/condição | Resultado esperado | Caminho de erro/recuperação |
|---|---|---|---|
| Principal | papel autorizado + sessão válida | ação permitida; evento de auditoria completo | anexar evidência ao piloto |
| Limite | papel de consulta tenta confirmar ou editar | ação negada sem alteração | coordenador revisa matriz; não ampliar permissão |
| Falha | sessão expirada durante confirmação | confirmação não ocorre; usuário precisa autenticar novamente | evento de falha sem duplicar registro |
| Privacidade | usuário sem acesso financeiro procura campo financeiro | campo não aparece e tentativa é negada/auditada | incidente encaminhado ao responsável por dados |
| Recuperação | auditoria volta após indisponibilidade | somente nova tentativa controlada é permitida; tentativa anterior segue pendente | não reconstruir sucesso por ausência de erro |

## Instruções de execução para o Ethos

1. **Ler antes de alterar:** SPEC-1-001; `02-Escopo-Definitivo.md` §§ 4, 6, 7, 9 e 12; política de segurança aprovada; esta SPEC.
2. **Alterar somente:** contas/configurações de teste e matriz de autorização da superfície aprovada.
3. **Não alterar:** acessos do DoIt fora do piloto, dados financeiros, credenciais em arquivos, integrações externas ou regras não aprovadas.
4. **Executar nesta ordem:** fechar P1-004 → criar contas de teste → aplicar matriz → executar cenários → revisar auditoria → solicitar aceite.
5. **Parar e pedir validação quando:** faltar papel, responsável, mecanismo de sessão, retenção, acesso técnico ou regra de dado financeiro.
6. **Estado válido ao parar:** nenhuma alteração produtiva sem evento de auditoria; acessos de teste identificados e reversíveis.

## Checklist de execução

- [ ] P1-004 foi preenchido e aprovado por Tiago, coordenador e responsável técnico.
- [ ] Contas de teste cobrem permitido, consulta, negado e sem acesso financeiro.
- [ ] Sessão expirada e reautenticação foram testadas.
- [ ] Confirmação só ocorre após prévia e dentro de sessão válida.
- [ ] Negativas, correções, cancelamentos e falhas deixam auditoria.
- [ ] Nenhum dado financeiro aparece no piloto.
- [ ] Logs e configuração podem ser revertidos sem apagar evidência.

## Critérios de aceite

- [ ] **CA-1-006:** matriz permite/nega as ações de criar, consultar, confirmar, corrigir, cancelar e encaminhar por papel.
- [ ] **CA-1-007:** sessão inválida ou expirada impede confirmação e não cria duplicata.
- [ ] **CA-1-008:** tentativa permitida e tentativa negada registram actor, papel, momento, ação, resultado e motivo.
- [ ] **CA-1-009:** usuário sem permissão não visualiza nem registra dado financeiro.
- [ ] **CA-1-010:** indisponibilidade da auditoria impede confirmação produtiva e deixa pendência recuperável.

## TDD da SPEC

| Etapa | Prova | Comando/ação | Resultado esperado | Evidência |
|---|---|---|---|---|
| RED | papel negado tenta confirmar | executar confirmação com conta negada | ação falha antes da alteração e gera evento de negativa | captura + log com event_id |
| GREEN | papel autorizado confirma fixture | executar entrada e confirmação com sessão válida | uma alteração aceita, uma auditoria completa e nenhum dado financeiro | matriz, captura, log e fixture |
| REFACTOR/REGRESSÃO | sessão expira e auditoria cai | repetir confirmação após expiração e durante indisponibilidade da auditoria | ambas falham com segurança; retomada não duplica; dados permanecem restritos | relatório de cenários |

**Dados/fixtures:** contas `USER-AUTORIZADO`, `USER-CONSULTA`, `USER-NEGADO` e `USER-SEM-FINANCEIRO`; usar `FIX-1-001`; todos devem ser contas de teste descartáveis.
**Caminhos de erro obrigatórios:** papel ausente, sessão expirada, tentativa de alteração não autorizada, auditoria indisponível, consulta financeira e reenvio após timeout.
**Evidência exigida:** matriz assinada/aprovada, configuração, eventos de auditoria, capturas de permitido/negado e veredito humano.

## Handoff e operação

- **Como demonstrar:** executar o mesmo fixture com usuário autorizado e negado, mostrando prévia, decisão e trilha de auditoria.
- **Como operar depois:** administrador técnico mantém acessos; coordenador revisa exceções; Tiago aprova alteração de escopo.
- **Como monitorar:** auditoria de negativas, sessões expiradas, alterações e tentativas financeiras.
- **Pendência conhecida:** P1-004 bloqueia a execução; nenhum nome de papel ou retenção deve ser inferido.

## Tasks vinculadas

| ID | Task | Dono | SPEC | Critério | Recorte da prova | Evidência esperada | Pré-condições | Status |
|---|---|---|---|---|---|---|---|---|
| F1-T004 | Aplicar matriz de papéis, sessão e contas de teste | Administrador técnico | SPEC-1-002 | CA-1-006, CA-1-007 | criar contas e testar sessão expirada | matriz, configuração e capturas | P1-004 fechado; mecanismo aprovado | bloqueada — exceção humana |
| F1-T005 | Validar auditoria, negativas e privacidade financeira | Administrador técnico | SPEC-1-002 | CA-1-008, CA-1-009 | executar permitido, negado e tentativa financeira | eventos completos e capturas | F1-T004 concluída; retenção aprovada | bloqueada — exceção humana |
| F1-T006 | Exercitar indisponibilidade da auditoria e recuperação segura | Administrador técnico | SPEC-1-002 | CA-1-010 | interromper auditoria e repetir após recuperação | log, pendência e ausência de duplicata | F1-T004 concluída; falha controlada | bloqueada — exceção humana |

## Emendas

<!-- Append-only (D19): mudanças aprovadas depois da geração. A história não é reescrita. -->

| Data | Origem do sinal | Micro-spec/task | Motivo |
|---|---|---|---|
| | | | |
