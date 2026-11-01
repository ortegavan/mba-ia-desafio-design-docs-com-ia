# ADR-006: Módulo de webhooks seguindo os padrões existentes do projeto

- **Status:** Aceito
- **Decidido em:** reunião técnica registrada em `TRANSCRICAO.md`, [09:30] Larissa; [09:36] Larissa
- **Decisores:** Larissa (Tech Lead), Bruno (Eng. Pleno, Pedidos), Diego (Eng. Sênior, Plataforma). A Sofia (Eng. Segurança) participou da exigência de role ADMIN no replay, às 09:36. A escolha de UUID foi feita às 09:51, só por Larissa, Bruno e Diego.
- **Relacionado a:** [PRD-DEC-06](../PRD.md#prd-dec-06-decisão-reuso-máximo-dos-padrões-do-projeto), [PRD-DEC-07](../PRD.md#prd-dec-07-decisão-reprocessamento-só-para-administrador-gerenciamento-de-webhooks-para-qualquer-usuário-autenticado), [ADR-001](ADR-001-outbox-no-mysql.md), [ADR-002](ADR-002-worker-separado-com-polling.md), [ADR-003](ADR-003-retry-com-backoff-e-dlq.md)

## Status

Aceito. A Larissa fechou o "reuso máximo do que já existe" às 09:30 e a role ADMIN com `requireRole` no replay às 09:36. O padrão de UUID foi confirmado às 09:51.

## Contexto

- `ADR-006-CTX-01` A codebase tem um padrão claro: cada domínio é um módulo em `src/modules` com controller, service, repository, routes e schemas.
- `ADR-006-CTX-02` O time é pequeno e o prazo é de três sprints.
- `ADR-006-CTX-03` O JWT atual é do usuário operador do sistema, e não do cliente.

**Referências ao código existente**

- `ADR-006-COD-01` `src/modules/orders/order.routes.ts`: exemplo do padrão de módulo. As rotas usam `authenticate` e o middleware `validate` com schemas Zod.
- `ADR-006-COD-02` `src/shared/errors/app-error.ts`: `AppError` carrega `statusCode`, `errorCode` e `details`.
- `ADR-006-COD-03` `src/shared/errors/http-errors.ts`: classes específicas como `InvalidStatusTransitionError` (`INVALID_STATUS_TRANSITION`) e `InsufficientStockError` (`INSUFFICIENT_STOCK`) mostram o padrão de códigos de erro.
- `ADR-006-COD-04` `src/middlewares/error.middleware.ts`: o error middleware centralizado já trata `AppError`, `ZodError` e erros conhecidos do Prisma.
- `ADR-006-COD-05` `src/middlewares/auth.middleware.ts`: `authenticate` e `requireRole` já existem. O `JwtPayload` traz só `sub`, `email` e `role`.
- `ADR-006-COD-06` `src/shared/logger/index.ts`: o logger Pino do projeto, com redação de campos sensíveis.
- `ADR-006-COD-07` `prisma/schema.prisma`: os ids são UUID (`@default(uuid())`), e o enum `UserRole` tem só `ADMIN` e `OPERATOR`.

## Decisão

`ADR-006` Vamos construir os webhooks como um módulo igual aos outros, num novo módulo de webhooks em `src/modules`, reaproveitando o que já existe:

- `AppError` e classes derivadas, com códigos de erro prefixados por `WEBHOOK_` (por exemplo, `WEBHOOK_NOT_FOUND` e `WEBHOOK_INVALID_URL`);
- o error middleware centralizado, sem alteração;
- o logger Pino, sem biblioteca nova;
- o padrão de schemas Zod;
- `authenticate` e `requireRole`: o CRUD de configuração aceita qualquer role autenticada, e o replay da DLQ exige `ADMIN`;
- ids em UUID, como o resto do projeto.

## Alternativas Consideradas

### `ADR-006-ALT-01` Tirar o `customer_id` do JWT

O Marcos propôs às 09:31 que o `customer_id` viesse implícito do JWT. O Bruno apontou às 09:32 que o JWT atual é do usuário operador, e não do cliente. A Larissa decidiu às 09:32 que o endpoint é autenticado normalmente e que o `customer_id` vai no body ou no path. Isso mantém o `authenticate` como está, sem mudar o conteúdo do token.

### `ADR-006-ALT-02` Id autoincremental na outbox

O Diego perguntou às 09:51 se a outbox teria id autoincremental ou UUID. A Larissa escolheu UUID para seguir o padrão do resto do projeto.

## Consequências

### Positivas

- `ADR-006-POS-01` O error middleware pega os erros do módulo novo sem precisar de mudança.
- `ADR-006-POS-02` Nenhuma dependência nova de log: o Pino já está no projeto inteiro.
- `ADR-006-POS-03` O replay fica protegido pelo `requireRole` que já existe, sem código novo de autorização.
- `ADR-006-POS-04` Os ids da outbox seguem o mesmo padrão UUID das outras tabelas.

### Negativas

- `ADR-006-NEG-01` Como só existem as roles `ADMIN` e `OPERATOR`, o CRUD de configuração fica aberto a qualquer usuário autenticado. A Sofia disse que dá para endurecer mais adiante.
- `ADR-006-NEG-02` O `customer_id` precisa ser informado explicitamente no body ou no path de cada chamada.
- `ADR-006-NEG-03` (consequência inferida) Como o token não liga o usuário a um cliente (`JwtPayload` só tem `sub`, `email` e `role`), a API não tem como restringir de qual cliente o usuário gerencia os webhooks.

### Trade-off

Ganhamos velocidade e consistência: o módulo nasce com os mesmos erros, logs, validação e autorização do resto do sistema. Em troca, herdamos os limites do modelo de acesso atual, que não tem papel de cliente e não liga o usuário a um cliente.
