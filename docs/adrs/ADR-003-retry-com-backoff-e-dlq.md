# ADR-003: 5 tentativas com backoff exponencial e DLQ em tabela separada, com replay manual

- **Status:** Aceito
- **Decidido em:** reunião técnica registrada em `TRANSCRICAO.md`, [09:17] Larissa; [09:18] Diego; [09:19] Larissa
- **Decisores:** Larissa (Tech Lead), Diego (Eng. Sênior, Plataforma), Bruno (Eng. Pleno, Pedidos), Marcos (PM, aceitou a janela de cerca de 15 horas às 09:17)
- **Relacionado a:** [PRD-DEC-03](../PRD.md#prd-dec-03-decisão-5-tentativas-com-intervalo-crescente-e-dlq-separada), [PRD-FR-08](../PRD.md#prd-fr-08-retentar-com-intervalo-crescente-e-separar-falhas-permanentes), [PRD-FR-09](../PRD.md#prd-fr-09-reprocessar-manualmente-eventos-da-dlq), [ADR-002](ADR-002-worker-separado-com-polling.md), [ADR-006](ADR-006-reuso-dos-padroes-do-projeto.md)

## Status

Aceito. A Larissa fechou as 5 tentativas e a progressão do backoff às 09:17. O Diego propôs a tabela de dead letter separada às 09:18, e a Larissa registrou o replay manual às 09:19.

## Contexto

- `ADR-003-CTX-01` O cliente pode estar offline no momento do envio, e a equipe precisava decidir o que fazer nesse caso.
- `ADR-003-CTX-02` Já houve cliente com duas horas de indisponibilidade em manutenção planejada.
- `ADR-003-CTX-03` Um cliente que não responde em 10 segundos é tratado como falha e vai para retry.
- `ADR-003-CTX-04` O evento que falhou de vez precisa continuar disponível para debug e reprocessamento.

## Decisão

`ADR-003` Vamos retentar com backoff exponencial, em 5 tentativas com intervalos de 1m, 5m, 30m, 2h e 12h. São quase 15 horas entre a primeira falha e a última tentativa. Quando as tentativas se esgotam, o evento é considerado falha permanente e vai para uma tabela de dead letter separada da outbox, com o payload, o motivo e o horário da falha. O reprocessamento é manual, por um endpoint admin, e recoloca o evento na outbox como pendente. A restrição desse endpoint à role ADMIN está no [ADR-006](ADR-006-reuso-dos-padroes-do-projeto.md).

## Alternativas Consideradas

### `ADR-003-ALT-01` Retry indefinido com backoff

O Diego citou às 09:15 que "algumas pessoas defendem" essa opção e a descartou: se o cliente sumiu, o evento fica pendurado para sempre.

### `ADR-003-ALT-02` 3 tentativas

O Bruno propôs às 09:16, como uma opção mais agressiva. O Diego descartou: 3 tentativas caberiam em cerca de 30 minutos, e já houve cliente duas horas fora do ar. A Larissa fechou em 5 às 09:16 e às 09:17.

### `ADR-003-ALT-03` Marcar o evento como "failed" na própria outbox

A Larissa colocou a dúvida às 09:17: tabela separada ou status na própria outbox. O Diego preferiu a tabela separada às 09:18, porque a leitura da outbox principal fica mais limpa.

## Consequências

### Positivas

- `ADR-003-POS-01` A política cobre indisponibilidades longas do cliente, como o caso real de 2 horas.
- `ADR-003-POS-02` Nenhum evento fica pendurado para sempre.
- `ADR-003-POS-03` A outbox fica mais simples de ler, e a DLQ guarda evidência para debug e para o reprocessamento.

### Negativas

- `ADR-003-NEG-01` Um cliente fora do ar por mais de cerca de 15 horas deixa de receber os eventos automaticamente. O Marcos considerou isso aceitável.
- `ADR-003-NEG-02` Tirar um evento da DLQ depende da ação manual de um administrador.
- `ADR-003-NEG-03` Um evento pode levar até cerca de 15 horas para ser entregue ou declarado falho.

### Trade-off

Ganhamos tolerância a indisponibilidades longas sem eventos eternos na fila, e uma trilha clara do que falhou. Em troca, depois de cerca de 15 horas a recuperação passa a ser manual, por um administrador.
