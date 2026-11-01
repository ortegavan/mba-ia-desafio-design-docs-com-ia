# ADR-005: Entrega at-least-once com `X-Event-Id` para deduplicação no cliente

- **Status:** Aceito
- **Decidido em:** reunião técnica registrada em `TRANSCRICAO.md`, [09:26] Larissa
- **Decisores:** Diego (Eng. Sênior, Plataforma), Larissa (Tech Lead), Sofia (Eng. Segurança), Marcos (PM), Bruno (Eng. Pleno, Pedidos)
- **Relacionado a:** [PRD-DEC-05](../PRD.md#prd-dec-05-decisão-entrega-pelo-menos-uma-vez-com-identificador-de-evento), [PRD-FR-07](../PRD.md#prd-fr-07-enviar-o-evento-assinado-ao-cliente), [ADR-001](ADR-001-outbox-no-mysql.md), [ADR-003](ADR-003-retry-com-backoff-e-dlq.md)

## Status

Aceito. A Larissa fechou às 09:26: "At-least-once com X-Event-Id pra dedup do lado do cliente. Decisão."

## Contexto

- `ADR-005-CTX-01` Com outbox, worker e retry, a plataforma garante que o evento chega, mas o mesmo evento pode chegar ao cliente duas vezes.
- `ADR-005-CTX-02` O cliente precisa de uma forma de reconhecer que um evento é duplicado.

**Referências ao código existente**

- `ADR-005-COD-01` `prisma/schema.prisma`: todos os ids do projeto são UUID (`@default(uuid())`), então identificar cada evento por um UUID segue o padrão existente.

## Decisão

`ADR-005` Vamos garantir entrega at-least-once. Cada evento recebe um UUID gerado no momento em que entra na outbox, único por evento, e enviado no header `X-Event-Id`. Se o cliente receber o mesmo evento duas vezes, ele faz a deduplicação pelo `event_id` do lado dele.

## Alternativas Consideradas

### `ADR-005-ALT-01` Garantia exactly-once

O Diego descartou às 09:25: garantir exactly-once exigiria coordenação dos dois lados e ficaria muito mais complexo. Ele argumentou que at-least-once com `event_id` resolve 99% dos casos.

## Consequências

### Positivas

- `ADR-005-POS-01` Segue o padrão de mercado. O Diego citou Stripe e GitHub como exemplos.
- `ADR-005-POS-02` Não há coordenação entre a plataforma e o cliente para garantir entrega única, e a solução fica bem mais simples.

### Negativas

- `ADR-005-NEG-01` O cliente pode receber o mesmo evento duas vezes e precisa estar preparado para isso.
- `ADR-005-NEG-02` A responsabilidade pela deduplicação vai para o cliente, como a Sofia apontou.
- `ADR-005-NEG-03` A garantia precisa ser documentada com destaque no portal de desenvolvedor.

### Trade-off

Ganhamos uma entrega simples e confiável, no padrão de mercado. Em troca, o trabalho de deduplicar fica com o cliente, e a documentação para ele vira obrigatória.
