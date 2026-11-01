# ADR-001: Outbox no MySQL existente, gravada na mesma transação da mudança de status

- **Status:** Aceito
- **Decidido em:** reunião técnica registrada em `TRANSCRICAO.md`, [09:08] Larissa; [09:40] Bruno
- **Decisores:** Larissa (Tech Lead), Diego (Eng. Sênior, Plataforma), Bruno (Eng. Pleno, Pedidos)
- **Relacionado a:** [PRD-DEC-01](../PRD.md#prd-dec-01-decisão-outbox-no-mysql-existente), [PRD-FR-05](../PRD.md#prd-fr-05-filtrar-eventos-por-status), [PRD-FR-06](../PRD.md#prd-fr-06-registrar-o-evento-junto-com-a-mudança-de-status), [ADR-002](ADR-002-worker-separado-com-polling.md), [ADR-007](ADR-007-snapshot-do-payload-na-insercao.md)

## Status

Aceito. A Larissa fechou a outbox no MySQL às 09:08. Às 09:40, o Bruno fixou que a inserção acontece dentro da transação do `changeStatus`, com rollback se ela falhar, e o Diego confirmou às 09:41.

## Contexto

- `ADR-001-CTX-01` Toda mudança de status precisa gerar um evento para o cliente. Não pode existir status mudado sem evento.
- `ADR-001-CTX-02` A transação de mudança de status já é pesada: atualiza o pedido, grava o histórico e mexe no estoque. Uma chamada HTTP dentro dela faria um cliente lento travar a mudança de status de outros pedidos.
- `ADR-001-CTX-03` Se o cliente estiver fora do ar, desfazer a mudança de status não é uma opção.
- `ADR-001-CTX-04` O time é pequeno, e subir infraestrutura nova só para isso foi considerado excesso.

**Referências ao código existente**

- `ADR-001-COD-01` `src/modules/orders/order.service.ts`: o método `changeStatus` roda dentro de `this.prisma.$transaction`. Nessa transação ele atualiza o pedido, cria o registro em `orderStatusHistory` e debita ou repõe o estoque. É nessa mesma transação que o evento passa a ser gravado.
- `ADR-001-COD-02` `prisma/schema.prisma`: o datasource da aplicação já é MySQL, então a outbox fica no banco que já existe.

## Decisão

`ADR-001` Vamos usar o padrão outbox no MySQL que já existe. Dentro da mesma transação que muda o status do pedido, o sistema grava o evento numa tabela de outbox. Um worker separado lê essa tabela e faz as chamadas HTTP (ver [ADR-002](ADR-002-worker-separado-com-polling.md)). Se a inserção na outbox falhar, a transação inteira faz rollback e o status não muda.

## Alternativas Consideradas

### `ADR-001-ALT-01` Disparo síncrono dentro do `changeStatus`

A Larissa colocou a dúvida às 09:03: disparar a chamada no service de orders, no momento da mudança, ou usar fila/outbox. O Bruno descartou às 09:04, porque um cliente lento travaria a mudança de status de outros pedidos e um cliente fora do ar obrigaria a desfazer uma mudança válida. O Diego reforçou às 09:06: "Síncrono está fora de questão".

### `ADR-001-ALT-02` Fila em infraestrutura nova (Redis Streams ou similar)

A Larissa levantou às 09:07, já apontando que seria preciso subir mais infraestrutura. O Diego concordou e descartou: para um time pequeno, subir um Redis Cluster seria overengineering, e a outbox no MySQL existente resolve.

## Consequências

### Positivas

- `ADR-001-POS-01` Não há inconsistência possível entre pedido e evento. Se a transação commitou, o evento está registrado. Se fez rollback, o evento some junto.
- `ADR-001-POS-02` Nenhuma infraestrutura nova para subir e operar.
- `ADR-001-POS-03` O filtro de status pode ser aplicado na inserção: se nenhum webhook do cliente quer aquele status, nada é gravado, e isso economiza linhas na tabela. O Diego perguntou se o filtro seria no envio, e o grupo ficou com a inserção.
- `ADR-001-POS-04` A integração com o `OrderService` é uma função que recebe o `tx` da transação atual, sem injetar um repository inteiro.

### Negativas

- `ADR-001-NEG-01` A tabela da outbox cresce com o tempo. O arquivamento das linhas já entregues ficou fora do escopo desta feature.
- `ADR-001-NEG-02` Uma falha ao inserir o evento passa a desfazer a mudança de status. O fluxo de pedidos ganha um novo motivo de rollback.
- `ADR-001-NEG-03` O `changeStatus` passa a depender do módulo de webhooks, mesmo que só por uma função.

### Trade-off

Ganhamos consistência total entre a mudança de status e o evento, sem infraestrutura nova. Em troca, a tabela cresce, a transação de pedidos ganha um novo ponto de falha e o `changeStatus` fica acoplado ao módulo de webhooks.
