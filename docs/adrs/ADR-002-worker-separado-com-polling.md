# ADR-002: Worker em processo separado da API, com polling a cada 2 segundos

- **Status:** Aceito
- **Decidido em:** reunião técnica registrada em `TRANSCRICAO.md`, [09:10] Larissa; [09:11] Diego
- **Decisores:** Larissa (Tech Lead), Diego (Eng. Sênior, Plataforma), Bruno (Eng. Pleno, Pedidos), Marcos (PM, aceitou os 2 segundos às 09:10)
- **Relacionado a:** [PRD-DEC-02](../PRD.md#prd-dec-02-decisão-processo-de-envio-separado-com-polling-a-cada-2-segundos), [PRD-FR-07](../PRD.md#prd-fr-07-enviar-o-evento-assinado-ao-cliente), [ADR-001](ADR-001-outbox-no-mysql.md), [ADR-003](ADR-003-retry-com-backoff-e-dlq.md)

## Status

Aceito. A Larissa registrou o polling de 2 segundos como decisão às 09:10. O Diego exigiu processo separado às 09:11, e a Larissa anotou às 09:12.

## Contexto

- `ADR-002-CTX-01` Para os clientes, qualquer coisa abaixo de 10 segundos já é tempo real.
- `ADR-002-CTX-02` Com a outbox ([ADR-001](ADR-001-outbox-no-mysql.md)), algum processo precisa ler os eventos pendentes e disparar as chamadas HTTP.
- `ADR-002-CTX-03` O MySQL não tem listener nativo como o NOTIFY/LISTEN do Postgres. Uma trigger só executa SQL e não notifica processo externo.
- `ADR-002-CTX-04` Se o worker rodar dentro da API, um reinício da API derruba o envio junto.

**Referências ao código existente**

- `ADR-002-COD-01` `src/server.ts`: hoje é o único ponto de entrada da aplicação. O worker vira uma entry-point nova no mesmo molde, como a Larissa sugeriu às 09:11.
- `ADR-002-COD-02` `src/config/database.ts`: `createPrismaClient()` cria uma instância nova de `PrismaClient`. O worker usa a dele, no mesmo banco e com a mesma `DATABASE_URL`. O Bruno corrigiu às 09:30 a fala das 09:11, em que tinha dito "o mesmo Prisma client".
- `ADR-002-COD-03` `package.json`: os scripts atuais (`dev` e `start`) só sobem a API. O worker vai precisar de um script próprio.

## Decisão

`ADR-002` Vamos rodar o worker como um processo Node separado da API, no mesmo projeto, na mesma stack e no mesmo banco. Ele faz polling em loop: a cada 2 segundos, busca os eventos pendentes mais antigos em lote pequeno, processa e marca o resultado. Nesta fase roda um único worker.

## Alternativas Consideradas

### `ADR-002-ALT-01` Trigger no banco para acordar o worker

O Bruno sugeriu às 09:09, para o envio ser mais reativo. O Diego descartou na mesma hora: a trigger do MySQL só executa SQL. Para avisar o worker, seria preciso improvisar, por exemplo escrevendo em arquivo ou chamando um endpoint. Além disso, o polling de 2 segundos já atende o requisito de menos de 10 segundos.

### `ADR-002-ALT-02` Worker dentro do processo da API

O Diego descartou às 09:11: se a API reinicia, o worker morre junto. O Diego e o Bruno concordaram com "mesmo banco, mesma stack, só não pode ser o mesmo processo".

## Consequências

### Positivas

- `ADR-002-POS-01` O polling de 2 segundos atende o "abaixo de 10 segundos" com folga, e o Marcos confirmou que serve.
- `ADR-002-POS-02` Um reinício da API não interrompe o envio.
- `ADR-002-POS-03` Com um único worker, os eventos são processados na ordem em que entraram na outbox, e o cliente recebe os eventos de cada pedido em ordem.
- `ADR-002-POS-04` A solução usa a mesma stack e o mesmo banco, sem infraestrutura nova.

### Negativas

- `ADR-002-NEG-01` Um evento pode esperar até 2 segundos para ser pego, no pior caso. A equipe aceitou.
- `ADR-002-NEG-02` Não há garantia de ordem global. A ordem só é garantida por pedido e enquanto houver um único worker. Isso ficou documentado como limitação conhecida.
- `ADR-002-NEG-03` Escalar para vários workers em paralelo quebra a garantia de ordem. Será preciso particionar por pedido ou usar lock pessimista, o que ficou como problema do futuro.
- `ADR-002-NEG-04` Passa a existir mais um processo para subir, com script próprio e instância própria de `PrismaClient`.

### Trade-off

Ganhamos simplicidade (sem infraestrutura nova), isolamento em relação aos reinícios da API e ordem por pedido. Em troca, aceitamos até 2 segundos de espera, um segundo processo para operar e um teto de escala enquanto houver um único worker.
