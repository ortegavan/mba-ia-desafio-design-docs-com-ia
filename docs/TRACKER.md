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
| PRD-ARQ-03 | docs/PRD.md | Arquitetura | Padrão de módulos existente, tendo o módulo de pedidos como referência | CODIGO | src/modules/orders/order.routes.ts |
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
| ADR-001-CTX-01 | docs/adrs/ADR-001-outbox-no-mysql.md | Contexto | Não pode existir status mudado sem evento | TRANSCRICAO | [09:40] Bruno |
| ADR-001-CTX-02 | docs/adrs/ADR-001-outbox-no-mysql.md | Contexto | Transação de status já é pesada; HTTP dentro travaria outros pedidos | TRANSCRICAO | [09:04] Bruno |
| ADR-001-CTX-03 | docs/adrs/ADR-001-outbox-no-mysql.md | Contexto | Cliente fora do ar não pode desfazer a mudança de status | TRANSCRICAO | [09:04] Bruno |
| ADR-001-CTX-04 | docs/adrs/ADR-001-outbox-no-mysql.md | Contexto | Time pequeno; infraestrutura nova seria excesso | TRANSCRICAO | [09:07] Diego |
| ADR-001-COD-01 | docs/adrs/ADR-001-outbox-no-mysql.md | Referência ao Código | changeStatus roda em this.prisma.$transaction com pedido, histórico e estoque | CODIGO | src/modules/orders/order.service.ts |
| ADR-001-COD-02 | docs/adrs/ADR-001-outbox-no-mysql.md | Referência ao Código | Datasource da aplicação já é MySQL | CODIGO | prisma/schema.prisma |
| ADR-001 | docs/adrs/ADR-001-outbox-no-mysql.md | Decisão | Outbox no MySQL existente, gravada na transação do changeStatus, com rollback se falhar | TRANSCRICAO | [09:08] Larissa; [09:40] Bruno |
| ADR-001-ALT-01 | docs/adrs/ADR-001-outbox-no-mysql.md | Alternativa Descartada | Disparo síncrono no changeStatus | TRANSCRICAO | [09:04] Bruno; [09:03] Larissa |
| ADR-001-ALT-02 | docs/adrs/ADR-001-outbox-no-mysql.md | Alternativa Descartada | Redis Streams ou fila em infraestrutura nova | TRANSCRICAO | [09:07] Diego; [09:07] Larissa |
| ADR-001-POS-01 | docs/adrs/ADR-001-outbox-no-mysql.md | Consequência Positiva | Sem inconsistência possível entre pedido e evento | TRANSCRICAO | [09:06] Diego |
| ADR-001-POS-02 | docs/adrs/ADR-001-outbox-no-mysql.md | Consequência Positiva | Nenhuma infraestrutura nova | TRANSCRICAO | [09:07] Diego |
| ADR-001-POS-03 | docs/adrs/ADR-001-outbox-no-mysql.md | Consequência Positiva | Filtro de status na inserção economiza linhas | TRANSCRICAO | [09:34] Bruno; [09:34] Diego |
| ADR-001-POS-04 | docs/adrs/ADR-001-outbox-no-mysql.md | Consequência Positiva | Integração por função que recebe o tx, sem injetar repository | TRANSCRICAO | [09:41] Diego; [09:41] Bruno |
| ADR-001-NEG-01 | docs/adrs/ADR-001-outbox-no-mysql.md | Consequência Negativa | Outbox cresce; arquivamento fora do escopo | TRANSCRICAO | [09:08] Diego |
| ADR-001-NEG-02 | docs/adrs/ADR-001-outbox-no-mysql.md | Consequência Negativa | Falha ao inserir o evento desfaz a mudança de status | TRANSCRICAO | [09:40] Bruno |
| ADR-001-NEG-03 | docs/adrs/ADR-001-outbox-no-mysql.md | Consequência Negativa | changeStatus passa a depender do módulo de webhooks | TRANSCRICAO | [09:41] Bruno |
| ADR-002-CTX-01 | docs/adrs/ADR-002-worker-separado-com-polling.md | Contexto | Abaixo de 10 s já é tempo real para os clientes | TRANSCRICAO | [09:02] Marcos |
| ADR-002-CTX-02 | docs/adrs/ADR-002-worker-separado-com-polling.md | Contexto | Algum processo precisa ler a outbox e disparar o HTTP | TRANSCRICAO | [09:06] Diego |
| ADR-002-CTX-03 | docs/adrs/ADR-002-worker-separado-com-polling.md | Contexto | MySQL não tem LISTEN/NOTIFY; trigger não notifica processo externo | TRANSCRICAO | [09:09] Diego |
| ADR-002-CTX-04 | docs/adrs/ADR-002-worker-separado-com-polling.md | Contexto | Worker dentro da API morre quando a API reinicia | TRANSCRICAO | [09:11] Diego |
| ADR-002-COD-01 | docs/adrs/ADR-002-worker-separado-com-polling.md | Referência ao Código | src/server.ts é hoje o único ponto de entrada | CODIGO | src/server.ts |
| ADR-002-COD-02 | docs/adrs/ADR-002-worker-separado-com-polling.md | Referência ao Código | createPrismaClient cria instância nova; worker usa a sua | CODIGO | src/config/database.ts |
| ADR-002-COD-03 | docs/adrs/ADR-002-worker-separado-com-polling.md | Referência ao Código | Scripts atuais só sobem a API | CODIGO | package.json |
| ADR-002 | docs/adrs/ADR-002-worker-separado-com-polling.md | Decisão | Worker em processo separado, polling de 2 s, único worker | TRANSCRICAO | [09:10] Larissa; [09:11] Diego |
| ADR-002-ALT-01 | docs/adrs/ADR-002-worker-separado-com-polling.md | Alternativa Descartada | Trigger no banco para acordar o worker | TRANSCRICAO | [09:09] Diego; [09:09] Bruno |
| ADR-002-ALT-02 | docs/adrs/ADR-002-worker-separado-com-polling.md | Alternativa Descartada | Worker dentro do processo da API | TRANSCRICAO | [09:11] Diego |
| ADR-002-POS-01 | docs/adrs/ADR-002-worker-separado-com-polling.md | Consequência Positiva | Polling de 2 s atende o abaixo de 10 s | TRANSCRICAO | [09:09] Diego; [09:10] Marcos |
| ADR-002-POS-02 | docs/adrs/ADR-002-worker-separado-com-polling.md | Consequência Positiva | Reinício da API não interrompe o envio | TRANSCRICAO | [09:11] Diego |
| ADR-002-POS-03 | docs/adrs/ADR-002-worker-separado-com-polling.md | Consequência Positiva | Único worker processa em ordem de criação; ordem por pedido | TRANSCRICAO | [09:12] Diego |
| ADR-002-POS-04 | docs/adrs/ADR-002-worker-separado-com-polling.md | Consequência Positiva | Mesma stack e mesmo banco | TRANSCRICAO | [09:11] Diego |
| ADR-002-NEG-01 | docs/adrs/ADR-002-worker-separado-com-polling.md | Consequência Negativa | Até 2 s de espera no pior caso, aceito | TRANSCRICAO | [09:10] Larissa |
| ADR-002-NEG-02 | docs/adrs/ADR-002-worker-separado-com-polling.md | Consequência Negativa | Sem ordem global; limitação conhecida | TRANSCRICAO | [09:13] Larissa |
| ADR-002-NEG-03 | docs/adrs/ADR-002-worker-separado-com-polling.md | Consequência Negativa | Escalar exige particionar por pedido ou lock pessimista | TRANSCRICAO | [09:13] Diego; [09:12] Diego |
| ADR-002-NEG-04 | docs/adrs/ADR-002-worker-separado-com-polling.md | Consequência Negativa | Mais um processo, com script e PrismaClient próprios | TRANSCRICAO | [09:30] Bruno; [09:11] Larissa |
| ADR-003-CTX-01 | docs/adrs/ADR-003-retry-com-backoff-e-dlq.md | Contexto | Cliente pode estar offline no envio | TRANSCRICAO | [09:14] Larissa |
| ADR-003-CTX-02 | docs/adrs/ADR-003-retry-com-backoff-e-dlq.md | Contexto | Já houve cliente 2 h fora em manutenção planejada | TRANSCRICAO | [09:16] Diego |
| ADR-003-CTX-03 | docs/adrs/ADR-003-retry-com-backoff-e-dlq.md | Contexto | Sem resposta em 10 s conta como falha e vai para retry | TRANSCRICAO | [09:42] Diego |
| ADR-003-CTX-04 | docs/adrs/ADR-003-retry-com-backoff-e-dlq.md | Contexto | Evento falho precisa ficar disponível para debug e reprocessamento | TRANSCRICAO | [09:18] Diego |
| ADR-003 | docs/adrs/ADR-003-retry-com-backoff-e-dlq.md | Decisão | 5 tentativas 1m/5m/30m/2h/12h, dead letter separada e replay manual | TRANSCRICAO | [09:17] Larissa; [09:18] Diego |
| ADR-003-ALT-01 | docs/adrs/ADR-003-retry-com-backoff-e-dlq.md | Alternativa Descartada | Retry indefinido com backoff | TRANSCRICAO | [09:15] Diego |
| ADR-003-ALT-02 | docs/adrs/ADR-003-retry-com-backoff-e-dlq.md | Alternativa Descartada | 3 tentativas | TRANSCRICAO | [09:16] Diego; [09:16] Bruno |
| ADR-003-ALT-03 | docs/adrs/ADR-003-retry-com-backoff-e-dlq.md | Alternativa Descartada | Marcar failed na própria outbox | TRANSCRICAO | [09:18] Diego; [09:17] Larissa |
| ADR-003-POS-01 | docs/adrs/ADR-003-retry-com-backoff-e-dlq.md | Consequência Positiva | Cobre indisponibilidades longas como a de 2 h | TRANSCRICAO | [09:16] Diego |
| ADR-003-POS-02 | docs/adrs/ADR-003-retry-com-backoff-e-dlq.md | Consequência Positiva | Nenhum evento pendurado para sempre | TRANSCRICAO | [09:15] Diego |
| ADR-003-POS-03 | docs/adrs/ADR-003-retry-com-backoff-e-dlq.md | Consequência Positiva | Outbox mais limpa e DLQ como evidência | TRANSCRICAO | [09:18] Diego |
| ADR-003-NEG-01 | docs/adrs/ADR-003-retry-com-backoff-e-dlq.md | Consequência Negativa | Cliente fora por mais de cerca de 15 h deixa de receber automaticamente | TRANSCRICAO | [09:17] Marcos |
| ADR-003-NEG-02 | docs/adrs/ADR-003-retry-com-backoff-e-dlq.md | Consequência Negativa | Sair da DLQ depende de ação manual de admin | TRANSCRICAO | [09:18] Diego |
| ADR-003-NEG-03 | docs/adrs/ADR-003-retry-com-backoff-e-dlq.md | Consequência Negativa | Evento pode levar cerca de 15 h até entrega ou falha | TRANSCRICAO | [09:17] Diego |
| ADR-004-CTX-01 | docs/adrs/ADR-004-hmac-sha256-com-secret-por-endpoint.md | Contexto | Dados de pedidos saem da infra; cliente valida origem e integridade | TRANSCRICAO | [09:19] Sofia |
| ADR-004-CTX-02 | docs/adrs/ADR-004-hmac-sha256-com-secret-por-endpoint.md | Contexto | Cliente já vazou secret em log | TRANSCRICAO | [09:22] Diego |
| ADR-004-CTX-03 | docs/adrs/ADR-004-hmac-sha256-com-secret-por-endpoint.md | Contexto | https obrigatório é validação, não decisão arquitetural | TRANSCRICAO | [09:23] Sofia |
| ADR-004 | docs/adrs/ADR-004-hmac-sha256-com-secret-por-endpoint.md | Decisão | HMAC-SHA256 do corpo em X-Signature, secret por endpoint, rotação com 24 h | TRANSCRICAO | [09:22] Sofia; [09:20] Sofia |
| ADR-004-ALT-01 | docs/adrs/ADR-004-hmac-sha256-com-secret-por-endpoint.md | Alternativa Descartada | Secret global da plataforma | TRANSCRICAO | [09:21] Sofia |
| ADR-004-POS-01 | docs/adrs/ADR-004-hmac-sha256-com-secret-por-endpoint.md | Consequência Positiva | Cliente valida origem e integridade | TRANSCRICAO | [09:19] Sofia |
| ADR-004-POS-02 | docs/adrs/ADR-004-hmac-sha256-com-secret-por-endpoint.md | Consequência Positiva | Padrão de mercado com bibliotecas disponíveis | TRANSCRICAO | [09:20] Sofia |
| ADR-004-POS-03 | docs/adrs/ADR-004-hmac-sha256-com-secret-por-endpoint.md | Consequência Positiva | Vazamento atinge só um endpoint | TRANSCRICAO | [09:21] Sofia |
| ADR-004-POS-04 | docs/adrs/ADR-004-hmac-sha256-com-secret-por-endpoint.md | Consequência Positiva | 24 h de convivência para o cliente migrar | TRANSCRICAO | [09:21] Sofia |
| ADR-004-NEG-01 | docs/adrs/ADR-004-hmac-sha256-com-secret-por-endpoint.md | Consequência Negativa | Cliente implementa a verificação | TRANSCRICAO | [09:20] Sofia |
| ADR-004-NEG-02 | docs/adrs/ADR-004-hmac-sha256-com-secret-por-endpoint.md | Consequência Negativa | Revisão de segurança de 2 dias úteis antes do deploy | TRANSCRICAO | [09:46] Sofia |
| ADR-004-NEG-03 | docs/adrs/ADR-004-hmac-sha256-com-secret-por-endpoint.md | Consequência Negativa | Só 24 h para migrar após rotacionar | TRANSCRICAO | [09:21] Sofia |
| ADR-005-CTX-01 | docs/adrs/ADR-005-entrega-at-least-once-com-x-event-id.md | Contexto | Mesmo evento pode chegar duas vezes | TRANSCRICAO | [09:24] Diego |
| ADR-005-CTX-02 | docs/adrs/ADR-005-entrega-at-least-once-com-x-event-id.md | Contexto | Cliente precisa reconhecer duplicados | TRANSCRICAO | [09:25] Bruno |
| ADR-005-COD-01 | docs/adrs/ADR-005-entrega-at-least-once-com-x-event-id.md | Referência ao Código | Ids do projeto são UUID | CODIGO | prisma/schema.prisma |
| ADR-005 | docs/adrs/ADR-005-entrega-at-least-once-com-x-event-id.md | Decisão | At-least-once com UUID por evento no X-Event-Id | TRANSCRICAO | [09:26] Larissa; [09:25] Diego |
| ADR-005-ALT-01 | docs/adrs/ADR-005-entrega-at-least-once-com-x-event-id.md | Alternativa Descartada | Exactly-once | TRANSCRICAO | [09:25] Diego |
| ADR-005-POS-01 | docs/adrs/ADR-005-entrega-at-least-once-com-x-event-id.md | Consequência Positiva | Padrão de mercado (Stripe, GitHub) | TRANSCRICAO | [09:25] Diego |
| ADR-005-POS-02 | docs/adrs/ADR-005-entrega-at-least-once-com-x-event-id.md | Consequência Positiva | Sem coordenação entre os dois lados | TRANSCRICAO | [09:25] Diego |
| ADR-005-NEG-01 | docs/adrs/ADR-005-entrega-at-least-once-com-x-event-id.md | Consequência Negativa | Cliente pode receber duplicado e precisa estar preparado | TRANSCRICAO | [09:24] Diego |
| ADR-005-NEG-02 | docs/adrs/ADR-005-entrega-at-least-once-com-x-event-id.md | Consequência Negativa | Responsabilidade da dedup vai para o cliente | TRANSCRICAO | [09:25] Sofia |
| ADR-005-NEG-03 | docs/adrs/ADR-005-entrega-at-least-once-com-x-event-id.md | Consequência Negativa | Documentação destacada no portal obrigatória | TRANSCRICAO | [09:26] Marcos |
| ADR-006-CTX-01 | docs/adrs/ADR-006-reuso-dos-padroes-do-projeto.md | Contexto | Padrão de módulo: controller, service, repository, routes, schemas | TRANSCRICAO | [09:27] Bruno |
| ADR-006-CTX-02 | docs/adrs/ADR-006-reuso-dos-padroes-do-projeto.md | Contexto | Time pequeno e prazo de três sprints | TRANSCRICAO | [09:47] Larissa; [09:07] Diego |
| ADR-006-CTX-03 | docs/adrs/ADR-006-reuso-dos-padroes-do-projeto.md | Contexto | JWT atual é do usuário operador, não do cliente | TRANSCRICAO | [09:32] Bruno |
| ADR-006-COD-01 | docs/adrs/ADR-006-reuso-dos-padroes-do-projeto.md | Referência ao Código | Rotas do módulo de pedidos com authenticate e validate Zod | CODIGO | src/modules/orders/order.routes.ts |
| ADR-006-COD-02 | docs/adrs/ADR-006-reuso-dos-padroes-do-projeto.md | Referência ao Código | AppError com statusCode, errorCode e details | CODIGO | src/shared/errors/app-error.ts |
| ADR-006-COD-03 | docs/adrs/ADR-006-reuso-dos-padroes-do-projeto.md | Referência ao Código | InvalidStatusTransitionError e InsufficientStockError com códigos | CODIGO | src/shared/errors/http-errors.ts |
| ADR-006-COD-04 | docs/adrs/ADR-006-reuso-dos-padroes-do-projeto.md | Referência ao Código | Error middleware trata AppError, ZodError e Prisma | CODIGO | src/middlewares/error.middleware.ts |
| ADR-006-COD-05 | docs/adrs/ADR-006-reuso-dos-padroes-do-projeto.md | Referência ao Código | authenticate, requireRole e JwtPayload com sub, email e role | CODIGO | src/middlewares/auth.middleware.ts |
| ADR-006-COD-06 | docs/adrs/ADR-006-reuso-dos-padroes-do-projeto.md | Referência ao Código | Logger Pino com redação de campos sensíveis | CODIGO | src/shared/logger/index.ts |
| ADR-006-COD-07 | docs/adrs/ADR-006-reuso-dos-padroes-do-projeto.md | Referência ao Código | Ids UUID e UserRole com ADMIN e OPERATOR | CODIGO | prisma/schema.prisma |
| ADR-006 | docs/adrs/ADR-006-reuso-dos-padroes-do-projeto.md | Decisão | Módulo de webhooks no padrão existente, com reuso de erros, logs, Zod, requireRole e UUID | TRANSCRICAO | [09:30] Larissa; [09:36] Larissa |
| ADR-006-ALT-01 | docs/adrs/ADR-006-reuso-dos-padroes-do-projeto.md | Alternativa Descartada | customer_id implícito do JWT | TRANSCRICAO | [09:32] Larissa; [09:31] Marcos |
| ADR-006-ALT-02 | docs/adrs/ADR-006-reuso-dos-padroes-do-projeto.md | Alternativa Descartada | Id autoincremental na outbox | TRANSCRICAO | [09:51] Larissa; [09:51] Diego |
| ADR-006-POS-01 | docs/adrs/ADR-006-reuso-dos-padroes-do-projeto.md | Consequência Positiva | Error middleware pega os erros sem mudança | TRANSCRICAO | [09:29] Bruno |
| ADR-006-POS-02 | docs/adrs/ADR-006-reuso-dos-padroes-do-projeto.md | Consequência Positiva | Nenhuma dependência nova de log | TRANSCRICAO | [09:29] Bruno |
| ADR-006-POS-03 | docs/adrs/ADR-006-reuso-dos-padroes-do-projeto.md | Consequência Positiva | Replay protegido pelo requireRole existente | TRANSCRICAO | [09:36] Larissa |
| ADR-006-POS-04 | docs/adrs/ADR-006-reuso-dos-padroes-do-projeto.md | Consequência Positiva | Outbox com UUID, como as outras tabelas | TRANSCRICAO | [09:51] Larissa |
| ADR-006-NEG-01 | docs/adrs/ADR-006-reuso-dos-padroes-do-projeto.md | Consequência Negativa | CRUD aberto a qualquer autenticado; endurecer depois | TRANSCRICAO | [09:37] Sofia |
| ADR-006-NEG-02 | docs/adrs/ADR-006-reuso-dos-padroes-do-projeto.md | Consequência Negativa | customer_id informado explicitamente no body ou path | TRANSCRICAO | [09:32] Larissa |
| ADR-006-NEG-03 | docs/adrs/ADR-006-reuso-dos-padroes-do-projeto.md | Consequência inferida | Token não liga usuário a cliente; API não restringe o cliente gerenciado | TRANSCRICAO | [09:32] Larissa |
| ADR-007-CTX-01 | docs/adrs/ADR-007-snapshot-do-payload-na-insercao.md | Contexto | Tempo entre mudança e envio: até 2 s, ou cerca de 15 h com retries | TRANSCRICAO | [09:10] Larissa; [09:17] Diego |
| ADR-007-CTX-02 | docs/adrs/ADR-007-snapshot-do-payload-na-insercao.md | Contexto | O pedido pode mudar nesse intervalo | TRANSCRICAO | [09:52] Larissa |
| ADR-007-CTX-03 | docs/adrs/ADR-007-snapshot-do-payload-na-insercao.md | Contexto | Payload enxuto, sem itens | TRANSCRICAO | [09:43] Diego |
| ADR-007-COD-01 | docs/adrs/ADR-007-snapshot-do-payload-na-insercao.md | Referência ao Código | changeStatus relê o pedido completo dentro da transação | CODIGO | src/modules/orders/order.service.ts |
| ADR-007 | docs/adrs/ADR-007-snapshot-do-payload-na-insercao.md | Decisão | Payload renderizado e gravado na inserção na outbox | TRANSCRICAO | [09:52] Larissa; [09:52] Bruno |
| ADR-007-ALT-01 | docs/adrs/ADR-007-snapshot-do-payload-na-insercao.md | Alternativa Descartada | Guardar só order_id e renderizar no envio | TRANSCRICAO | [09:52] Larissa; [09:51] Bruno |
| ADR-007-POS-01 | docs/adrs/ADR-007-snapshot-do-payload-na-insercao.md | Consequência Positiva | Evento reflete o pedido no momento da mudança | TRANSCRICAO | [09:52] Larissa |
| ADR-007-POS-02 | docs/adrs/ADR-007-snapshot-do-payload-na-insercao.md | Consequência Positiva | Replay da DLQ reenvia o mesmo conteúdo | TRANSCRICAO | [09:18] Diego |
| ADR-007-NEG-01 | docs/adrs/ADR-007-snapshot-do-payload-na-insercao.md | Consequência inferida | Cada linha guarda o payload completo; outbox cresce mais | TRANSCRICAO | [09:52] Larissa |
| RFC-CTX-01 | docs/RFC.md | Contexto | Aplicação sem notificação externa; clientes consultam a API de tempos em tempos | TRANSCRICAO | [09:00] Marcos |
| RFC-CTX-02 | docs/RFC.md | Contexto | changeStatus já atualiza pedido, histórico e estoque numa transação | CODIGO | src/modules/orders/order.service.ts |
| RFC-CTX-03 | docs/RFC.md | Contexto | Mudança de status não pode esperar o cliente nem ser desfeita por ele | TRANSCRICAO | [09:04] Bruno |
| RFC-CTX-04 | docs/RFC.md | Contexto | Time pequeno | TRANSCRICAO | [09:07] Diego |
| RFC-CTX-05 | docs/RFC.md | Contexto | Infraestrutura nova seria overengineering; MySQL existente resolve | TRANSCRICAO | [09:07] Diego |
| RFC-PROP-01 | docs/RFC.md | Proposta | Captura: evento gravado na transação, com rollback, filtro na inserção e snapshot | TRANSCRICAO | [09:40] Bruno; [09:34] Bruno |
| RFC-PROP-02 | docs/RFC.md | Proposta | Entrega: worker separado em polling de 2 s, HMAC, identificador, 5 tentativas e dead letter | TRANSCRICAO | [09:11] Diego; [09:09] Diego |
| RFC-PROP-03 | docs/RFC.md | Proposta | Gestão: módulo no padrão do projeto, CRUD autenticado, rotação, histórico e replay ADMIN | TRANSCRICAO | [09:30] Larissa; [09:36] Larissa |
| RFC-PROP-04 | docs/RFC.md | Garantia | Atomicidade entre mudança de status e evento | TRANSCRICAO | [09:06] Diego |
| RFC-PROP-05 | docs/RFC.md | Garantia | Evento reflete o pedido no momento da mudança | TRANSCRICAO | [09:52] Larissa |
| RFC-PROP-06 | docs/RFC.md | Garantia | At-least-once, sem exactly-once; cliente deduplica pelo identificador | TRANSCRICAO | [09:25] Diego; [09:24] Diego |
| RFC-PROP-07 | docs/RFC.md | Limitação | Sem ordem global; só por pedido e com um único worker | TRANSCRICAO | [09:13] Larissa |
| RFC-PROP-08 | docs/RFC.md | Limitação | Evento pode esperar até 2 s para ser pego | TRANSCRICAO | [09:10] Larissa |
| RFC-PROP-09 | docs/RFC.md | Limitação | Após cerca de 15 h de falhas, só replay manual | TRANSCRICAO | [09:17] Diego; [09:18] Diego |
| RFC-ALT-01 | docs/RFC.md | Alternativa Descartada | Disparo síncrono na mudança de status | TRANSCRICAO | [09:04] Bruno; [09:03] Larissa |
| RFC-ALT-02 | docs/RFC.md | Alternativa Descartada | Fila em infraestrutura nova (Redis Streams) | TRANSCRICAO | [09:07] Diego; [09:07] Larissa |
| RFC-ALT-03 | docs/RFC.md | Alternativa Descartada | Trigger no banco para acordar o worker | TRANSCRICAO | [09:09] Diego; [09:09] Bruno |
| RFC-ALT-04 | docs/RFC.md | Alternativa Descartada | Worker dentro do processo da API | TRANSCRICAO | [09:11] Diego |
| RFC-ALT-05 | docs/RFC.md | Alternativa Descartada | Entrega exactly-once | TRANSCRICAO | [09:25] Diego |
| RFC-ALT-06 | docs/RFC.md | Alternativa Descartada | Montar o payload só no envio | TRANSCRICAO | [09:52] Larissa; [09:51] Bruno |
| RFC-QA-01 | docs/RFC.md | Questão em Aberto | Rate limiting de saída: observar e decidir depois | TRANSCRICAO | [09:39] Larissa; [09:38] Diego |
| RFC-QA-02 | docs/RFC.md | Questão em Aberto | Escalar para vários workers: particionar por pedido ou lock pessimista | TRANSCRICAO | [09:13] Diego; [09:13] Bruno |
| RFC-QA-03 | docs/RFC.md | Questão em Aberto | Aviso por e-mail adiado até medir o impacto | TRANSCRICAO | [09:37] Larissa; [09:37] Marcos |
| RFC-QA-04 | docs/RFC.md | Questão em Aberto | Arquivamento dos eventos entregues fora do escopo | TRANSCRICAO | [09:08] Diego |
| RFC-QA-05 | docs/RFC.md | Questão em Aberto | Restringir quem gerencia webhooks fica para depois | TRANSCRICAO | [09:37] Sofia; [09:36] Marcos |
| RFC-LAC-01 | docs/RFC.md | Lacuna | Destino do evento acima de 64KB não definido (PRD-FR-07) | TRANSCRICAO | [09:24] Larissa |
| RFC-LAC-02 | docs/RFC.md | Lacuna | Respostas que contam como falha além do timeout não definidas (PRD-FR-08) | TRANSCRICAO | [09:42] Diego |
| RFC-IMP-01 | docs/RFC.md | Impacto | changeStatus ganha a gravação do evento na transação existente | CODIGO | src/modules/orders/order.service.ts |
| RFC-IMP-02 | docs/RFC.md | Impacto | server.ts segue como entrada da API; worker vira nova entrada | CODIGO | src/server.ts |
| RFC-IMP-03 | docs/RFC.md | Impacto | Worker cria o próprio Prisma Client no mesmo banco | CODIGO | src/config/database.ts |
| RFC-IMP-04 | docs/RFC.md | Impacto | AppError e error middleware reaproveitados sem mudança | CODIGO | src/shared/errors/app-error.ts |
| RFC-IMP-05 | docs/RFC.md | Impacto | authenticate e requireRole sem mudança; token sem cliente | CODIGO | src/middlewares/auth.middleware.ts |
| RFC-IMP-06 | docs/RFC.md | Impacto | Logger Pino existente atende módulo e worker | CODIGO | src/shared/logger/index.ts |
| RFC-IMP-07 | docs/RFC.md | Impacto | Schema ganha modelos de outbox, dead letter e configuração | CODIGO | prisma/schema.prisma |
| RFC-IMP-08 | docs/RFC.md | Impacto | Rotas do módulo novo entram no buildApiRouter | CODIGO | src/routes/index.ts |
| RFC-IMP-09 | docs/RFC.md | Impacto | Máquina de estados do pedido não muda | CODIGO | src/modules/orders/order.status.ts |
| RFC-RISCO-01 | docs/RFC.md | Risco | Status muda sem evento; mitigado pela mesma transação e testes ponta a ponta | TRANSCRICAO | [09:40] Bruno; [09:46] Larissa |
| RFC-RISCO-02 | docs/RFC.md | Risco | Falha na outbox desfaz mudança válida; mitigação não definida | TRANSCRICAO | [09:40] Bruno |
| RFC-RISCO-03 | docs/RFC.md | Risco | Perda de ordem ao escalar; worker único com limitação documentada | TRANSCRICAO | [09:13] Larissa; [09:12] Diego |
| RFC-RISCO-04 | docs/RFC.md | Risco | Outbox grande deixa o worker lento; leitura só dos pendentes em lote pequeno | TRANSCRICAO | [09:07] Bruno; [09:08] Diego |
| RFC-RISCO-05 | docs/RFC.md | Risco | Cliente lento ou fora do ar; timeout de 10 s, retry e dead letter | TRANSCRICAO | [09:42] Diego; [09:17] Larissa |
| RFC-RISCO-06 | docs/RFC.md | Risco | Evento duplicado; identificador por evento e destaque no portal | TRANSCRICAO | [09:25] Diego; [09:26] Marcos |
| RFC-RISCO-07 | docs/RFC.md | Risco | Vazamento de secret; secret por endpoint, rotação e revisão de segurança | TRANSCRICAO | [09:21] Sofia; [09:46] Sofia |
| RFC-DEC-01 | docs/RFC.md | Decisão | Link para ADR-001: outbox no MySQL na transação da mudança | TRANSCRICAO | [09:08] Larissa; [09:40] Bruno |
| RFC-DEC-02 | docs/RFC.md | Decisão | Link para ADR-002: worker separado com polling de 2 s | TRANSCRICAO | [09:10] Larissa; [09:11] Diego |
| RFC-DEC-03 | docs/RFC.md | Decisão | Link para ADR-003: 5 tentativas, dead letter e replay manual | TRANSCRICAO | [09:17] Larissa; [09:18] Diego |
| RFC-DEC-04 | docs/RFC.md | Decisão | Link para ADR-004: HMAC-SHA256, secret por endpoint, rotação de 24 h | TRANSCRICAO | [09:22] Sofia |
| RFC-DEC-05 | docs/RFC.md | Decisão | Link para ADR-005: at-least-once com identificador por evento | TRANSCRICAO | [09:26] Larissa |
| RFC-DEC-06 | docs/RFC.md | Decisão | Link para ADR-006: módulo no padrão do projeto | TRANSCRICAO | [09:30] Larissa; [09:36] Larissa |
| RFC-DEC-07 | docs/RFC.md | Decisão | Link para ADR-007: payload montado na inserção | TRANSCRICAO | [09:52] Larissa; [09:52] Bruno |
