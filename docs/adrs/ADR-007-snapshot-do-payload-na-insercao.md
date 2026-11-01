# ADR-007: Payload renderizado e gravado na inserção do evento

- **Status:** Aceito
- **Decidido em:** reunião técnica registrada em `TRANSCRICAO.md`, [09:52] Larissa; [09:52] Bruno
- **Decisores:** Larissa (Tech Lead), Diego (Eng. Sênior, Plataforma), Bruno (Eng. Pleno, Pedidos). Marcos e Sofia já tinham saído da call às 09:50.
- **Relacionado a:** [PRD-FR-06](../PRD.md#prd-fr-06-registrar-o-evento-junto-com-a-mudança-de-status), [ADR-001](ADR-001-outbox-no-mysql.md), [ADR-003](ADR-003-retry-com-backoff-e-dlq.md)

## Status

Aceito. A Larissa propôs às 09:52, o Diego concordou e o Bruno fechou: "Beleza, snapshot. Decidido."

## Contexto

- `ADR-007-CTX-01` Pode passar tempo entre a mudança de status e o envio: até 2 segundos no caso normal e cerca de 15 horas quando há retries.
- `ADR-007-CTX-02` Nesse intervalo, o pedido pode mudar de novo.
- `ADR-007-CTX-03` O payload combinado é enxuto e não leva os itens do pedido.

**Referências ao código existente**

- `ADR-007-COD-01` `src/modules/orders/order.service.ts`: dentro da transação do `changeStatus`, o pedido é relido no fim, com itens, histórico e cliente. Ou seja, os dados do pedido já estão disponíveis no momento em que o evento é inserido.

## Decisão

`ADR-007` Vamos renderizar o payload do evento no momento da inserção na outbox e gravá-lo pronto. O worker envia esse conteúdo como está, sem consultar o pedido de novo.

## Alternativas Consideradas

### `ADR-007-ALT-01` Guardar só o `order_id` e renderizar na hora do envio

O Bruno levantou a dúvida às 09:51. A Larissa descartou às 09:52: se o pedido mudar depois, o evento deixaria de refletir o estado de quando o status mudou, e isso gera "caso esquisito".

## Consequências

### Positivas

- `ADR-007-POS-01` O evento reflete o estado do pedido no momento da mudança, mesmo que o pedido mude antes do envio.
- `ADR-007-POS-02` Como a DLQ guarda o payload, um reprocessamento envia exatamente o mesmo conteúdo que falhou.

### Negativas

- `ADR-007-NEG-01` (consequência inferida) Cada linha da outbox carrega o payload completo, e não só o id do pedido. Por isso a tabela cresce mais rápido do que cresceria com a alternativa.

### Trade-off

Ganhamos eventos fiéis ao momento da mudança e reenvios idênticos. Em troca, a outbox fica maior, porque guarda o conteúdo inteiro de cada evento.
