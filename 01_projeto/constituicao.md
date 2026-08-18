# Constituição do projeto — Tiago Freire Arquitetos Associados

> Regras estáveis deste projeto. Valem em todas as fases e só mudam por decisão registrada da
> consultoria (emenda datada na seção final). Mantida pela consultoria — dúvida vira registro
> no `changelog.md`, não edição. (Conceito adaptado do Spec Kit, decisão D18.)

## Papéis

- **Champion:** Tiago Freire — executa as tasks da fase atual quando liberadas e valida com evidência.
- **Consultor Adapta:** Consultoria Adapta — define escopo, SPECs e critérios; fecha as fases. O nome nominal será confirmado no setup.
- **Agente (Claude):** guia a execução dentro destas regras; não legisla sobre escopo.

## Stack e ferramentas permitidas

- DoIt como sistema existente de projetos, etapas, agenda e módulo financeiro; interface conversacional textual como recorte da Fase 1; Ethos como ambiente de configuração do consultor; Google Workspace/Gemini no aculturamento paralelo.
- Dependência ou ferramenta nova só entra por decisão do consultor — registre `DÚVIDA:` antes.

## O que o champion pode e não pode tocar

- **Pode:** trabalhar uma task liberada por vez em `04_fase-atual/`, produzir evidências no recorte aprovado e usar o canal de comunicação e os sistemas somente com papel e acesso confirmados.
- **Não pode:** specs, `fase.md` (além de marcar tasks), `01_projeto/`, escrita não autorizada no DoIt, dados financeiros fora da matriz aprovada, preço, contrato ou mensagem a cliente.

## A SPEC é lei

Toda implementação segue o critério de aceite e o TDD da SPEC — nem menos (critério reprovado),
nem mais (**o aceite é teto**: código além do aceite é superfície não verificada, D17). O que
não está na SPEC da fase não se implementa: vira `DÚVIDA:` para o consultor decidir.

## Linha vermelha (nunca simplificar)

Validação de entrada em fronteira de confiança; tratamento de erro que evita perda de dados;
segurança; acessibilidade; LGPD/dados pessoais. Corte nessas áreas reprova a task — sem exceção
e sem julgamento de mérito (D17).

## Dívida deliberada

Simplificação intencional leva marca no ponto exato da decisão:
`adapta-divida: <teto atual>; <upgrade quando gatilho>`. O consultor acompanha essas marcas na
sincronização — é o combinado do método.

## Emendas

| Data | O que mudou | Decisão/motivo |
|---|---|---|
| | | |
