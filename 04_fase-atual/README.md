# 04_fase-atual — a unidade em execução

O projeto avança **uma fase por vez**. Aqui vive a fase corrente:

- `fase.md` — o contrato do ciclo: objetivo da fase, o que será demonstrado na reunião, o
  critério de pronto, a **tabela de tasks** (cada uma com dono e critério próprio) e a seção
  **"Fora desta fase"** — o que não construir agora, mesmo que pareça útil (vira DÚVIDA, não
  implementação).
- `specs/` — o detalhe de cada entrega da fase: o que construir/fazer, como saber que está
  certo, limites e o TDD da SPEC. É o material de apoio das tasks.

Como trabalhar: `/adapta-cliente:trabalhar` → executar → se travar, `/adapta-cliente:destravar-task` → `/adapta-cliente:finalizar-task`.
Quando a fase fecha na reunião de ciclo, ela é arquivada em `05_entregas/` e a próxima chega
aqui.

> Specs e `fase.md` são mantidos pela consultoria — dúvidas viram registro no `changelog.md`,
> não edição.
