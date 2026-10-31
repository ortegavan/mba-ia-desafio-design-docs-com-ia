# Tracker de rastreabilidade

Esta tabela liga cada item dos design docs à origem dele: uma fala da reunião (`TRANSCRICAO.md`) ou um arquivo do código existente. Os IDs são os mesmos que aparecem nos documentos. Os próximos documentos (ADRs, RFC e FDD) vão acrescentar linhas a esta mesma tabela.

| ID | Documento | Tipo | Conteúdo (resumo) | Fonte | Localização |
| --- | --- | --- | --- | --- | --- |
| PRD-CTX-01 | docs/PRD.md | Contexto | Notificar clientes B2B em menos de 10 s quando o status do pedido muda, no lugar do polling | TRANSCRICAO | [09:00] Marcos; [09:02] Marcos |
| PRD-CTX-02 | docs/PRD.md | Contexto | Webhook só de saída, da plataforma para o cliente | TRANSCRICAO | [09:02] Marcos; [09:03] Sofia |
| PRD-CTX-03 | docs/PRD.md | Contexto | Implantada no sistema existente como novo módulo no padrão atual | TRANSCRICAO | [09:30] Larissa; [09:27] Bruno |
| PRD-CTX-04 | docs/PRD.md | Contexto | Mesmo banco MySQL e mesma stack, com envio em processo separado | TRANSCRICAO | [09:11] Diego |
| PRD-CTX-05 | docs/PRD.md | Contexto | changeStatus atualiza pedido, histórico e estoque numa única transação | CODIGO | src/modules/orders/order.service.ts |
| PRD-CTX-06 | docs/PRD.md | Contexto | Máquina de estados do pedido (PENDING, PAID, PROCESSING, SHIPPED, DELIVERED, CANCELLED) | CODIGO | src/modules/orders/order.status.ts |
| PRD-PROB-01 | docs/PRD.md | Problema | Polling no GET /orders deixa a integração lenta e cara; três clientes pediram notificação | TRANSCRICAO | [09:00] Marcos |
| PRD-PROB-02 | docs/PRD.md | Problema | Risco de a Atlas migrar para o concorrente se não houver entrega no prazo | TRANSCRICAO | [09:00] Marcos; [09:45] Marcos |
| PRD-PROB-03 | docs/PRD.md | Problema | Cliente precisa validar origem e integridade de dados enviados para fora da infra | TRANSCRICAO | [09:19] Sofia |
| PRD-PUB-01 | docs/PRD.md | Público-alvo | Clientes B2B: Atlas Comercial, MaxDistribuição e Nova Cargo | TRANSCRICAO | [09:00] Marcos |
| PRD-PUB-02 | docs/PRD.md | Público-alvo | Usuários que representam o cliente, autenticados com JWT da plataforma | TRANSCRICAO | [09:32] Marcos |
| PRD-PUB-03 | docs/PRD.md | Público-alvo | Desenvolvedores dos clientes, via portal de desenvolvedor | TRANSCRICAO | [09:40] Marcos |
| PRD-PUB-04 | docs/PRD.md | Público-alvo | Administradores da plataforma, que reprocessam falhas permanentes | TRANSCRICAO | [09:36] Sofia; [09:18] Diego |
| PRD-PUB-05 | docs/PRD.md | Público-alvo | Cenário: cadastrar webhook escolhendo os status (ex.: SHIPPED e DELIVERED) | TRANSCRICAO | [09:33] Marcos |
| PRD-PUB-06 | docs/PRD.md | Público-alvo | Cenário: receber a notificação e conferir a assinatura | TRANSCRICAO | [09:20] Sofia |
| PRD-PUB-07 | docs/PRD.md | Público-alvo | Cenário: consultar o histórico de entregas | TRANSCRICAO | [09:34] Marcos |
| PRD-PUB-08 | docs/PRD.md | Público-alvo | Cenário: pedir uma nova secret pela API | TRANSCRICAO | [09:21] Sofia |
| PRD-PUB-09 | docs/PRD.md | Público-alvo | Cenário: administrador reprocessa evento da DLQ | TRANSCRICAO | [09:18] Diego |
| PRD-PUB-10 | docs/PRD.md | Público-alvo | Cenário: cliente com vários webhooks identifica o cadastro de cada envio | TRANSCRICAO | [09:44] Sofia |
| PRD-OBJ-01 | docs/PRD.md | Objetivo/Métrica | Primeira tentativa de envio em menos de 10 s, com polling de 2 s | TRANSCRICAO | [09:02] Marcos; [09:10] Larissa |
| PRD-OBJ-02 | docs/PRD.md | Objetivo/Métrica | Zero mudanças de status sem evento registrado | TRANSCRICAO | [09:40] Bruno |
| PRD-OBJ-03 | docs/PRD.md | Objetivo/Métrica | 5 tentativas cobrindo cerca de 15 h antes da falha permanente | TRANSCRICAO | [09:17] Larissa; [09:17] Diego |
| PRD-OBJ-04 | docs/PRD.md | Objetivo/Métrica | Entrega em 3 sprints com revisão de segurança (prazo da Atlas: fim de novembro) | TRANSCRICAO | [09:47] Larissa; [09:45] Marcos |
| PRD-ESC-01 | docs/PRD.md | Escopo | CRUD de webhooks pelo cliente | TRANSCRICAO | [09:33] Bruno |
| PRD-ESC-02 | docs/PRD.md | Escopo | Escolha de status por webhook | TRANSCRICAO | [09:33] Marcos |
| PRD-ESC-03 | docs/PRD.md | Escopo | Registro do evento na mesma operação da mudança de status | TRANSCRICAO | [09:40] Bruno |
| PRD-ESC-04 | docs/PRD.md | Escopo | Envio assíncrono por processo separado | TRANSCRICAO | [09:48] Larissa |
| PRD-ESC-05 | docs/PRD.md | Escopo | Retry com intervalo crescente e DLQ | TRANSCRICAO | [09:17] Larissa |
| PRD-ESC-06 | docs/PRD.md | Escopo | Reprocessamento manual só por administrador | TRANSCRICAO | [09:36] Larissa |
| PRD-ESC-07 | docs/PRD.md | Escopo | Assinatura com secret por webhook e rotação com transição | TRANSCRICAO | [09:22] Sofia |
| PRD-ESC-08 | docs/PRD.md | Escopo | Histórico de entregas por webhook | TRANSCRICAO | [09:34] Marcos |
| PRD-ESC-09 | docs/PRD.md | Escopo | Documentação no portal destacando at-least-once | TRANSCRICAO | [09:26] Marcos; [09:40] Marcos |
| PRD-FORA-01 | docs/PRD.md | Fora de Escopo | E-mail de alerta de falha adiado para a próxima fase | TRANSCRICAO | [09:37] Larissa; [09:38] Marcos |
| PRD-FORA-02 | docs/PRD.md | Fora de Escopo | Painel visual fica com o time de frontend, como projeto separado | TRANSCRICAO | [09:40] Larissa |
| PRD-FORA-03 | docs/PRD.md | Fora de Escopo | Rate limiting de saída: observar e decidir depois | TRANSCRICAO | [09:39] Larissa; [09:39] Diego |
| PRD-FORA-04 | docs/PRD.md | Fora de Escopo | Arquivamento de eventos entregues fora desta feature | TRANSCRICAO | [09:08] Diego |
| PRD-FORA-05 | docs/PRD.md | Fora de Escopo | Webhooks de entrada descartados | TRANSCRICAO | [09:02] Marcos; [09:03] Sofia |
| PRD-FORA-06 | docs/PRD.md | Fora de Escopo | Ordem global e vários workers em paralelo ficam como limitação conhecida | TRANSCRICAO | [09:13] Larissa; [09:13] Diego |
| PRD-FORA-07 | docs/PRD.md | Fora de Escopo | Exactly-once descartado | TRANSCRICAO | [09:26] Larissa; [09:25] Diego |
| PRD-FORA-08 | docs/PRD.md | Fora de Escopo | Envio síncrono na mudança de status descartado | TRANSCRICAO | [09:06] Diego; [09:04] Bruno |
| PRD-FORA-09 | docs/PRD.md | Fora de Escopo | Redis Streams ou Redis Cluster descartados | TRANSCRICAO | [09:08] Larissa; [09:07] Diego |
| PRD-FORA-10 | docs/PRD.md | Fora de Escopo | Trigger no banco descartada, porque o MySQL não notifica processo externo | TRANSCRICAO | [09:09] Diego |
| PRD-FORA-11 | docs/PRD.md | Fora de Escopo | Retry indefinido descartado | TRANSCRICAO | [09:15] Diego |
| PRD-FORA-12 | docs/PRD.md | Fora de Escopo | Truncar evento grande descartado; acima do limite dá erro | TRANSCRICAO | [09:24] Larissa; [09:23] Sofia |
| PRD-FORA-13 | docs/PRD.md | Fora de Escopo | Restrição do CRUD a papéis específicos fica para mais adiante | TRANSCRICAO | [09:37] Sofia |
| PRD-FR-01 | docs/PRD.md | Requisito Funcional | Cadastrar webhook com URL https, status e secret gerada na criação | TRANSCRICAO | [09:31] Marcos; [09:32] Larissa |
| PRD-FR-02 | docs/PRD.md | Requisito Funcional | Editar webhook | TRANSCRICAO | [09:33] Bruno |
| PRD-FR-03 | docs/PRD.md | Requisito Funcional | Remover webhook | TRANSCRICAO | [09:33] Bruno |
| PRD-FR-04 | docs/PRD.md | Requisito Funcional | Listar webhooks de um cliente | TRANSCRICAO | [09:33] Bruno |
| PRD-FR-05 | docs/PRD.md | Requisito Funcional | Filtrar eventos por status no registro; sem interessado, nada é registrado | TRANSCRICAO | [09:34] Bruno; [09:33] Marcos |
| PRD-FR-06 | docs/PRD.md | Requisito Funcional | Registrar o evento na transação da mudança de status, com foto do pedido | TRANSCRICAO | [09:40] Bruno; [09:52] Larissa |
| PRD-FR-07 | docs/PRD.md | Requisito Funcional | Enviar o evento assinado com HMAC pelo processo de polling | TRANSCRICAO | [09:22] Sofia; [09:09] Diego |
| PRD-FR-08 | docs/PRD.md | Requisito Funcional | 5 tentativas (1m/5m/30m/2h/12h) e depois DLQ com conteúdo, motivo e horário | TRANSCRICAO | [09:17] Larissa; [09:18] Diego |
| PRD-FR-09 | docs/PRD.md | Requisito Funcional | Reprocessamento manual da DLQ por ADMIN, devolvendo o evento como pendente | TRANSCRICAO | [09:36] Larissa; [09:18] Diego |
| PRD-FR-10 | docs/PRD.md | Requisito Funcional | Histórico das últimas 100 entregas com resultado, conteúdo, resposta e tempo | TRANSCRICAO | [09:34] Marcos |
| PRD-FR-11 | docs/PRD.md | Requisito Funcional | Rotacionar secret pela API com 24 h de convivência | TRANSCRICAO | [09:21] Sofia |
| PRD-RNF-01 | docs/PRD.md | Requisito Não Funcional | Eventos pendentes lidos a cada 2 s | TRANSCRICAO | [09:10] Larissa |
| PRD-RNF-02 | docs/PRD.md | Requisito Não Funcional | Mudança de status não espera chamada ao cliente | TRANSCRICAO | [09:04] Bruno |
| PRD-RNF-03 | docs/PRD.md | Requisito Não Funcional | Timeout de 10 s por chamada | TRANSCRICAO | [09:42] Diego |
| PRD-RNF-04 | docs/PRD.md | Requisito Não Funcional | Conteúdo enxuto, sem itens do pedido | TRANSCRICAO | [09:43] Diego |
| PRD-RNF-05 | docs/PRD.md | Requisito Não Funcional | Processo de envio separado da API, para resistir a reinícios | TRANSCRICAO | [09:11] Diego |
| PRD-RNF-06 | docs/PRD.md | Requisito Não Funcional | Assinatura HMAC-SHA256 sobre o corpo | TRANSCRICAO | [09:22] Sofia |
| PRD-RNF-07 | docs/PRD.md | Requisito Não Funcional | Secret única por webhook, gerada pela plataforma | TRANSCRICAO | [09:21] Sofia; [09:31] Marcos |
| PRD-RNF-08 | docs/PRD.md | Requisito Não Funcional | Rotação de secret com 24 h de convivência | TRANSCRICAO | [09:22] Sofia; [09:21] Sofia |
| PRD-RNF-09 | docs/PRD.md | Restrição | Só endereços https | TRANSCRICAO | [09:23] Sofia |
| PRD-RNF-10 | docs/PRD.md | Restrição | Limite de 64KB por evento, com erro acima disso | TRANSCRICAO | [09:24] Larissa |
| PRD-RNF-11 | docs/PRD.md | Requisito Não Funcional | Horário do envio informado, para detecção de replay attack | TRANSCRICAO | [09:44] Diego |
| PRD-RNF-12 | docs/PRD.md | Requisito Não Funcional | Reprocessamento exige ADMIN e registra quem fez | TRANSCRICAO | [09:36] Sofia |
| PRD-RNF-13 | docs/PRD.md | Requisito Não Funcional | CRUD liberado para qualquer usuário autenticado nesta fase | TRANSCRICAO | [09:37] Sofia |
| PRD-RNF-14 | docs/PRD.md | Requisito Não Funcional | Autenticação JWT e checagem de papel (requireRole) reaproveitadas | CODIGO | src/middlewares/auth.middleware.ts |
| PRD-RNF-15 | docs/PRD.md | Requisito Não Funcional | Papéis existentes: ADMIN e OPERATOR | CODIGO | prisma/schema.prisma |
| PRD-RNF-16 | docs/PRD.md | Requisito Não Funcional | Logs estruturados com o logger Pino existente | CODIGO | src/shared/logger/index.ts |
| PRD-RNF-17 | docs/PRD.md | Requisito Não Funcional | Erros tratados pelo error middleware centralizado | CODIGO | src/middlewares/error.middleware.ts |
| PRD-RNF-18 | docs/PRD.md | Requisito Não Funcional | DLQ guarda conteúdo, motivo e horário como evidência | TRANSCRICAO | [09:18] Diego |
| PRD-RNF-19 | docs/PRD.md | Requisito Não Funcional | Entrega at-least-once com identificador único por evento | TRANSCRICAO | [09:26] Larissa |
| PRD-RNF-20 | docs/PRD.md | Restrição | Ordem garantida só por pedido e com um único processo de envio | TRANSCRICAO | [09:13] Larissa |
| PRD-RNF-21 | docs/PRD.md | Requisito Não Funcional | Evento reflete o pedido no momento da mudança (snapshot) | TRANSCRICAO | [09:52] Larissa |
| PRD-RNF-22 | docs/PRD.md | Requisito Não Funcional | Registro do evento atômico com a mudança de status | TRANSCRICAO | [09:40] Bruno |
| PRD-RNF-23 | docs/PRD.md | Requisito Não Funcional | Processo de envio como novo ponto de entrada no mesmo projeto | TRANSCRICAO | [09:11] Larissa |
| PRD-RNF-24 | docs/PRD.md | Requisito Não Funcional | Conteúdo em JSON, com horários ISO 8601 | TRANSCRICAO | [09:43] Diego |
| PRD-RNF-25 | docs/PRD.md | Requisito Não Funcional | HMAC-SHA256 como padrão de mercado, com bibliotecas disponíveis | TRANSCRICAO | [09:20] Sofia |
| PRD-RNF-26 | docs/PRD.md | Requisito Não Funcional | Validação de entrada com schemas Zod, padrão do projeto | CODIGO | src/modules/orders/order.schemas.ts |
| PRD-RNF-27 | docs/PRD.md | Requisito Não Funcional | Erros reaproveitam a classe base AppError | CODIGO | src/shared/errors/app-error.ts |
| PRD-RNF-28 | docs/PRD.md | Requisito Não Funcional | Códigos de erro com prefixo WEBHOOK_ | TRANSCRICAO | [09:29] Larissa |
| PRD-ARQ-01 | docs/PRD.md | Arquitetura | Envio assíncrono com padrão outbox no banco existente | TRANSCRICAO | [09:08] Larissa; [09:06] Diego |
| PRD-ARQ-02 | docs/PRD.md | Arquitetura | Módulo de webhooks no mesmo padrão dos outros módulos | TRANSCRICAO | [09:27] Bruno |
| PRD-ARQ-03 | docs/PRD.md | Arquitetura | Padrão de módulos existente, tendo o módulo de pedidos como referência | CODIGO | src/modules/orders |
| PRD-ARQ-04 | docs/PRD.md | Arquitetura | Outbox de eventos pendentes gravada junto com a mudança de status | TRANSCRICAO | [09:06] Diego |
| PRD-ARQ-05 | docs/PRD.md | Arquitetura | Processo de envio separado da API, em polling | TRANSCRICAO | [09:11] Diego; [09:09] Diego |
| PRD-ARQ-06 | docs/PRD.md | Arquitetura | DLQ separada dos eventos pendentes | TRANSCRICAO | [09:18] Diego |
| PRD-ARQ-07 | docs/PRD.md | Arquitetura | Cadastro com endereço, secret, cliente e indicação de ativo | TRANSCRICAO | [09:21] Sofia; [09:21] Bruno |
| PRD-ARQ-08 | docs/PRD.md | Arquitetura | Integração: changeStatus passa a registrar o evento na sua transação | CODIGO | src/modules/orders/order.service.ts |
| PRD-ARQ-09 | docs/PRD.md | Arquitetura | Integração: endpoints https dos clientes fora da infraestrutura | TRANSCRICAO | [09:23] Sofia; [09:19] Sofia |
| PRD-ARQ-10 | docs/PRD.md | Arquitetura | Integração: cliente usa o GET /orders/:id existente para os detalhes | TRANSCRICAO | [09:43] Diego |
| PRD-DEC-01 | docs/PRD.md | Decisão | Outbox no MySQL existente; trade-off: tabela cresce, arquivamento fora | TRANSCRICAO | [09:08] Larissa; [09:07] Diego |
| PRD-DEC-02 | docs/PRD.md | Decisão | Worker separado em polling de 2 s; trade-off: espera de até 2 s e ordem só por pedido | TRANSCRICAO | [09:10] Larissa; [09:11] Diego |
| PRD-DEC-03 | docs/PRD.md | Decisão | 5 tentativas com backoff e DLQ separada; trade-off: depois disso só replay manual | TRANSCRICAO | [09:17] Larissa; [09:18] Diego |
| PRD-DEC-04 | docs/PRD.md | Decisão | HMAC-SHA256 com secret por webhook e rotação de 24 h; trade-off: cliente implementa verificação | TRANSCRICAO | [09:22] Sofia |
| PRD-DEC-05 | docs/PRD.md | Decisão | At-least-once com identificador; trade-off: cliente deduplica | TRANSCRICAO | [09:26] Larissa; [09:25] Diego |
| PRD-DEC-06 | docs/PRD.md | Decisão | Reuso máximo dos padrões do projeto | TRANSCRICAO | [09:30] Larissa |
| PRD-DEC-07 | docs/PRD.md | Decisão | Replay só ADMIN e CRUD para qualquer autenticado; trade-off: CRUD menos restrito | TRANSCRICAO | [09:36] Larissa; [09:37] Sofia |
| PRD-DEP-01 | docs/PRD.md | Dependência | Revisão de segurança de pelo menos 2 dias úteis antes do deploy | TRANSCRICAO | [09:46] Sofia |
| PRD-DEP-02 | docs/PRD.md | Dependência | Documentação no portal de desenvolvedor | TRANSCRICAO | [09:26] Marcos; [09:40] Marcos |
| PRD-DEP-03 | docs/PRD.md | Dependência | Confirmação do prazo com a Atlas | TRANSCRICAO | [09:47] Marcos |
| PRD-DEP-04 | docs/PRD.md | Dependência | Revisão do documento de design antes de codar | TRANSCRICAO | [09:50] Larissa |
| PRD-DEP-05 | docs/PRD.md | Dependência | Extensão da transação de mudança de status | TRANSCRICAO | [09:40] Bruno |
| PRD-DEP-06 | docs/PRD.md | Dependência | Processo de envio no mesmo banco com conexão própria | TRANSCRICAO | [09:30] Bruno; [09:11] Diego |
| PRD-DEP-07 | docs/PRD.md | Dependência | Clientes com https, verificação HMAC, dedup e migração de secret | TRANSCRICAO | [09:25] Diego; [09:23] Sofia |
| PRD-RISCO-01 | docs/PRD.md | Risco | Cliente fora do ar por muito tempo (já aconteceu, 2 h) | TRANSCRICAO | [09:16] Diego |
| PRD-RISCO-02 | docs/PRD.md | Risco | Vazamento de secret do cliente (já aconteceu uma vez) | TRANSCRICAO | [09:22] Diego; [09:21] Sofia |
| PRD-RISCO-03 | docs/PRD.md | Risco | Status muda e evento não é registrado | TRANSCRICAO | [09:40] Bruno; [09:41] Diego |
| PRD-RISCO-04 | docs/PRD.md | Risco | Cliente recebe o mesmo evento mais de uma vez | TRANSCRICAO | [09:24] Diego |
| PRD-RISCO-05 | docs/PRD.md | Risco | Muitas chamadas seguidas para o mesmo cliente (50 em um minuto) | TRANSCRICAO | [09:38] Diego; [09:39] Larissa |
| PRD-RISCO-06 | docs/PRD.md | Risco | Perda de ordem ao escalar para vários workers | TRANSCRICAO | [09:13] Larissa; [09:12] Diego |
| PRD-RISCO-07 | docs/PRD.md | Risco | Perder a Atlas por atraso na entrega | TRANSCRICAO | [09:00] Marcos |
| PRD-CA-01 | docs/PRD.md | Critério de Aceitação | Evento na mesma transação; falha no registro desfaz a mudança | TRANSCRICAO | [09:40] Bruno |
| PRD-CA-02 | docs/PRD.md | Critério de Aceitação | Status sem webhook interessado não gera evento | TRANSCRICAO | [09:34] Bruno |
| PRD-CA-03 | docs/PRD.md | Critério de Aceitação | Processo separado, polling de 2 s, primeira tentativa em menos de 10 s | TRANSCRICAO | [09:10] Larissa; [09:02] Marcos |
| PRD-CA-04 | docs/PRD.md | Critério de Aceitação | Envio assinado com HMAC-SHA256, com identificadores e horário | TRANSCRICAO | [09:22] Sofia; [09:44] Diego |
| PRD-CA-05 | docs/PRD.md | Critério de Aceitação | Endereço http recusado com erro de validação | TRANSCRICAO | [09:23] Sofia |
| PRD-CA-06 | docs/PRD.md | Critério de Aceitação | Evento acima de 64KB não é enviado e gera erro | TRANSCRICAO | [09:24] Larissa |
| PRD-CA-07 | docs/PRD.md | Critério de Aceitação | Sem resposta em 10 s conta como falha e vai para retry | TRANSCRICAO | [09:42] Diego |
| PRD-CA-08 | docs/PRD.md | Critério de Aceitação | Após 5 tentativas (1m/5m/30m/2h/12h), DLQ com conteúdo, motivo e horário | TRANSCRICAO | [09:17] Larissa; [09:18] Diego |
| PRD-CA-09 | docs/PRD.md | Critério de Aceitação | Replay só para ADMIN, auditado, devolve o evento como pendente | TRANSCRICAO | [09:36] Sofia; [09:18] Diego |
| PRD-CA-10 | docs/PRD.md | Critério de Aceitação | Secret antiga válida por 24 h após a rotação | TRANSCRICAO | [09:21] Sofia |
| PRD-CA-11 | docs/PRD.md | Critério de Aceitação | Secret gerada pela plataforma, devolvida na criação, única por webhook | TRANSCRICAO | [09:31] Marcos; [09:21] Sofia |
| PRD-CA-12 | docs/PRD.md | Critério de Aceitação | Histórico com as últimas 100 entregas e seus dados | TRANSCRICAO | [09:34] Marcos |
| PRD-CA-13 | docs/PRD.md | Critério de Aceitação | Prefixo WEBHOOK_ e WEBHOOK_NOT_FOUND para webhook inexistente | TRANSCRICAO | [09:29] Larissa; [09:28] Bruno |
| PRD-CA-14 | docs/PRD.md | Critério de Aceitação | Revisão de segurança concluída antes do deploy | TRANSCRICAO | [09:46] Sofia |
| PRD-CA-15 | docs/PRD.md | Critério de Aceitação | Portal destaca at-least-once e dedup pelo identificador | TRANSCRICAO | [09:26] Marcos |
| PRD-TEST-01 | docs/PRD.md | Teste | Testes ponta a ponta da integração com a mudança de status | TRANSCRICAO | [09:46] Larissa |
| PRD-TEST-02 | docs/PRD.md | Teste | Revisão de código de segurança (HMAC e geração de secret) | TRANSCRICAO | [09:46] Sofia |
| PRD-TEST-03 | docs/PRD.md | Teste | Suíte de integração existente de pedidos continua passando | CODIGO | tests/orders.test.ts |
| PRD-TEST-04 | docs/PRD.md | Teste | Revisão do desenho por Larissa, Bruno e Diego antes de codar | TRANSCRICAO | [09:50] Larissa |
| PRD-TEST-05 | docs/PRD.md | Teste | Revisão de segurança antes do deploy, incluída nas três sprints | TRANSCRICAO | [09:47] Larissa; [09:46] Sofia |
