# Architectural Decision Records

Este diretório guarda os ADRs (Architectural Decision Records) do Sistema de Webhooks de Notificação de Pedidos. Cada decisão arquitetural fica em um arquivo próprio, no formato MADR, com as seções Status, Contexto, Decisão, Alternativas Consideradas e Consequências.

## Convenção de nomes

Os arquivos seguem o padrão `ADR-NNN-titulo-em-kebab-case.md`, com numeração sequencial de três dígitos (por exemplo, `ADR-001-outbox-no-mysql.md`). O número nunca é reaproveitado. Se uma decisão mudar, um novo ADR é criado para substituir o anterior.

## Índice

| Nº | ADR | Status | Resumo |
| --- | --- | --- | --- |
| 001 | [Outbox no MySQL existente](ADR-001-outbox-no-mysql.md) | Aceito | O evento é gravado numa outbox no MySQL, na mesma transação da mudança de status. |
| 002 | [Worker separado com polling](ADR-002-worker-separado-com-polling.md) | Aceito | Um worker em processo separado da API lê a outbox a cada 2 segundos. |
| 003 | [Retry com backoff e DLQ](ADR-003-retry-com-backoff-e-dlq.md) | Aceito | São 5 tentativas (1m/5m/30m/2h/12h) e depois uma tabela de dead letter, com replay manual. |
| 004 | [HMAC-SHA256 com secret por endpoint](ADR-004-hmac-sha256-com-secret-por-endpoint.md) | Aceito | O corpo é assinado com HMAC-SHA256, com secret única por endpoint e rotação com 24 h de convivência. |
| 005 | [At-least-once com X-Event-Id](ADR-005-entrega-at-least-once-com-x-event-id.md) | Aceito | A entrega é pelo menos uma vez, com UUID por evento para deduplicação no cliente. |
| 006 | [Reuso dos padrões do projeto](ADR-006-reuso-dos-padroes-do-projeto.md) | Aceito | O módulo segue o padrão existente: AppError, WEBHOOK_, error middleware, Pino, Zod, requireRole e UUID. |
| 007 | [Snapshot do payload na inserção](ADR-007-snapshot-do-payload-na-insercao.md) | Aceito | O payload é renderizado e gravado no momento em que o evento entra na outbox. |
