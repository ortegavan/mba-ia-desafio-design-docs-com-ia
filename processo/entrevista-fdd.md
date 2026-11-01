# Entrevista técnica simulada: FDD do Sistema de Webhooks

Log da entrevista conduzida com o roteiro de `entrevista-fdd.md`. O entrevistador faz uma pergunta por vez. Os participantes respondem só com o que disseram ou ouviram na reunião (`TRANSCRICAO.md`), e os fatos do sistema vêm do código.

A usuária faz o papel da sessão de revisão técnica marcada em [09:50] Larissa. Ela respondeu "pode começar e segue direto, decide você". Por isso, em todas as etapas a escolha registrada é a **opção recomendada**. Antes de escrever o FDD, a lista consolidada de hipóteses foi apresentada à usuária.

Janela de presença:
- Diego: entrou às 09:05.
- Marcos e Sofia: saíram às 09:50.
- Larissa e Bruno: ficaram até as 09:53.

O Marcos só responde nos assuntos que tratou (contratos que pediu, payload, headers, histórico e prazo).

---

## Etapa 1. Contexto e motivação técnica

**Entrevistador:** Qual problema técnico a feature resolve no sistema atual?

**Larissa (Tech Lead):** O cliente precisa saber da mudança de status sem ficar consultando a API. A primeira coisa que discuti foi se o disparo seria síncrono no service de orders ou por fila/outbox. · Fontes: [09:03] Larissa
**Bruno (Eng. Pleno, Pedidos):** A transação de mudança de status já é pesada: atualiza orders, insere no histórico e mexe no estoque. Uma chamada HTTP no meio travaria a mudança de status de outros pedidos. · Fontes: [09:04] Bruno
**Diego (Eng. Sênior, Plataforma):** Por isso o padrão é outbox: o evento é gravado na mesma transação e um worker separado faz as chamadas HTTP. · Fontes: [09:06] Diego
**Sofia (Eng. Segurança):** Do meu lado, é outbound webhook: só sai da plataforma para o cliente. · Fontes: [09:03] Sofia

**Entrevistador:** Como a feature se encaixa no que já existe?

**Larissa (Tech Lead):** O worker é um ponto de entrada novo, no molde do `src/server.ts`, com `src/worker.ts` e um script "npm run worker". O módulo segue o padrão dos outros. · Fontes: [09:11] Larissa; [09:30] Larissa
**Bruno (Eng. Pleno, Pedidos):** O módulo fica em `src/modules/webhooks`, com controller, service, repository, routes e schemas. A integração crítica é no `changeStatus`. · Fontes: [09:27] Bruno; [09:40] Bruno
**Diego (Eng. Sênior, Plataforma):** Mesmo banco e mesma stack, mas não o mesmo processo. · Fontes: [09:11] Diego
**Sofia (Eng. Segurança):** Não tratei disso na reunião.

**Entrevistador:** Quem são os atores?

**Larissa (Tech Lead):** Há o administrador que reprocessa a DLQ e os usuários autenticados que mexem na configuração. · Fontes: [09:36] Larissa
**Bruno (Eng. Pleno, Pedidos):** O JWT é do usuário operador, não do cliente. · Fontes: [09:32] Bruno
**Diego (Eng. Sênior, Plataforma):** O worker e o endpoint do cliente, que recebe as chamadas. · Fontes: [09:06] Diego
**Sofia (Eng. Segurança):** O replay é do ADMIN. Para o resto do CRUD, por enquanto, vale qualquer role autenticada. · Fontes: [09:36] Sofia; [09:37] Sofia
**Marcos (PM):** O cliente cadastra pela nossa API, com usuários que representam o cliente. · Fontes: [09:32] Marcos

**Resumo da etapa.**
- O FDD detalha a implementação da [RFC](../docs/RFC.md) e dos ADRs 001 a 007.
- Atores:
  - usuário autenticado que representa o cliente;
  - administrador (ADMIN);
  - worker;
  - endpoint https do cliente;
  - `OrderService.changeStatus`, que produz o evento.
- Restrições: mesmo banco e mesma stack, processo separado, nenhuma infraestrutura nova.

**Divergências.** O Marcos propôs o `customer_id` vindo do JWT ([09:31] Marcos). O Bruno apontou que o token é do operador, e a Larissa decidiu que o `customer_id` vai no body ou no path ([09:32] Larissa).

**Lacunas da etapa.** Nenhuma nova. As hipóteses de implementação ficam reunidas em "Suposições e restrições".

**Escolha da usuária:** resumo confirmado ("segue direto").

---

## Etapa 2. Objetivos técnicos

**Entrevistador:** Quais resultados técnicos mensuráveis são esperados?

**Larissa (Tech Lead):** Worker em polling de 2 segundos. A latência mínima é de 2 segundos no pior caso, e aceitamos. · Fontes: [09:10] Larissa
**Bruno (Eng. Pleno, Pedidos):** Não pode existir status mudado sem evento. · Fontes: [09:40] Bruno
**Diego (Eng. Sênior, Plataforma):** Polling de 2 segundos atende o "abaixo de 10 segundos" com folga. · Fontes: [09:09] Diego
**Sofia (Eng. Segurança):** Timeout de 10 segundos na chamada, que achei bom. · Fontes: [09:42] Sofia
**Marcos (PM):** Para os clientes, abaixo de 10 segundos já é tempo real. · Fontes: [09:02] Marcos

**Entrevistador:** Que garantias determinísticas precisam existir?

**Larissa (Tech Lead):** Ordem só por order_id e enquanto houver um único worker. At-least-once, com `X-Event-Id`. · Fontes: [09:13] Larissa; [09:26] Larissa
**Bruno (Eng. Pleno, Pedidos):** Se a inserção na outbox falhar, rollback. · Fontes: [09:40] Bruno
**Diego (Eng. Sênior, Plataforma):** Se a transação commitou, o evento existe; se fez rollback, some junto. São 5 tentativas e depois DLQ. · Fontes: [09:06] Diego; [09:15] Diego
**Sofia (Eng. Segurança):** Todo envio é assinado com HMAC-SHA256 sobre o corpo, com secret por endpoint. · Fontes: [09:22] Sofia

**Resumo da etapa.** Objetivos com medida:
1. Primeira tentativa em menos de 10 s, com polling de 2 s (`PRD-OBJ-01`).
2. Zero mudanças de status sem evento (`PRD-OBJ-02`).
3. Atomicidade entre status e evento.
4. Política de retry com backoff e DLQ (`PRD-OBJ-03`).
5. 100% dos envios assinados.
6. Ordem por (webhook, pedido) com um único worker.
7. Timeout de 10 s por chamada.

**Divergências.** Nenhuma.

**Lacunas da etapa.** A contagem de tentativas é tratada na etapa 6 (`FDD-HIP-01`).

**Escolha da usuária:** resumo confirmado.

---

## Etapa 3. Escopo e exclusões

**Entrevistador:** O que está incluído nesta entrega?

**Larissa (Tech Lead):** Outbox, worker, retry com DLQ, HMAC com rotação, at-least-once e CRUD autenticado. O replay exige ADMIN. · Fontes: [09:48] Larissa
**Bruno (Eng. Pleno, Pedidos):** POST, PATCH, DELETE e GET por customer, com filtro de status por endpoint. · Fontes: [09:33] Bruno
**Diego (Eng. Sênior, Plataforma):** O endpoint admin de replay recoloca o evento na outbox como pendente. · Fontes: [09:18] Diego
**Sofia (Eng. Segurança):** A rotação de secret pela API, com convivência de 24 horas. · Fontes: [09:21] Sofia
**Marcos (PM):** O histórico de entregas por webhook. · Fontes: [09:34] Marcos

**Entrevistador:** O que está explicitamente fora?

**Larissa (Tech Lead):** E-mail fica para a próxima fase. Rate limiting fica em observação. Dashboard é projeto do frontend. · Fontes: [09:37] Larissa; [09:39] Larissa; [09:40] Larissa
**Bruno (Eng. Pleno, Pedidos):** Não tratei disso na reunião.
**Diego (Eng. Sênior, Plataforma):** Arquivamento das linhas entregues fica fora do escopo. Exactly-once foi descartado. · Fontes: [09:08] Diego; [09:25] Diego
**Sofia (Eng. Segurança):** Webhook de entrada fica fora. Endurecer as roles do CRUD fica para mais adiante. · Fontes: [09:03] Sofia; [09:37] Sofia

**Resumo da etapa.** O escopo incluído segue `PRD-ESC-01` a `PRD-ESC-09`. As exclusões apontam para `PRD-FORA-01` a `PRD-FORA-13`, sem repetir a justificativa.

**Divergências.** Nenhuma.

**Lacunas da etapa.**
1. **A criação do pedido gera evento?** O `create` grava PENDING sem passar pelo `changeStatus`.
   - **A. Não gera evento** (recomendada). A reunião só fala de mudança de status ([09:40] Bruno), e PENDING não é destino de nenhuma transição (`src/modules/orders/order.status.ts`).
   - **B. Gera evento na criação.** Contra: cria requisito de produto.
   - **Escolha:** A (`FDD-HIP-18`).

**Escolha da usuária:** resumo confirmado, lacuna 1 com a opção recomendada.

---

## Etapa 4. Fluxos detalhados

**Entrevistador:** Como é o fluxo de criação do evento na outbox?

**Larissa (Tech Lead):** O payload é renderizado na inserção, como snapshot. · Fontes: [09:52] Larissa
**Bruno (Eng. Pleno, Pedidos):** Proponho uma função `publishWebhookEvent(tx, order, fromStatus, toStatus)` que recebe o tx da transação atual. Ela filtra na inserção: se nenhum webhook do customer quer o status, nem insere. · Fontes: [09:41] Bruno; [09:34] Bruno
**Diego (Eng. Sênior, Plataforma):** É função pura recebendo o tx. O UUID do evento é gerado quando ele entra na outbox. · Fontes: [09:41] Diego; [09:25] Diego
**Sofia (Eng. Segurança):** Não tratei disso na reunião.

**Entrevistador:** Como o worker processa os eventos?

**Larissa (Tech Lead):** É polling de 2 segundos, num processo separado. · Fontes: [09:10] Larissa
**Bruno (Eng. Pleno, Pedidos):** O worker usa um PrismaClient próprio, com a mesma `DATABASE_URL`. · Fontes: [09:30] Bruno
**Diego (Eng. Sênior, Plataforma):** Há índice em status (pendente, processando, falhou, entregue) e em `created_at`. O worker lê os pendentes mais antigos em batch pequeno, processa e marca como entregue. Com um único worker, ele processa em ordem de `created_at`. · Fontes: [09:08] Diego; [09:09] Diego; [09:12] Diego
**Sofia (Eng. Segurança):** O envio é assinado com HMAC-SHA256 sobre o corpo. · Fontes: [09:22] Sofia

**Entrevistador:** Como funcionam o retry e a DLQ?

**Larissa (Tech Lead):** São 5 tentativas com backoff 1m/5m/30m/2h/12h. Depois, DLQ em tabela separada. · Fontes: [09:17] Larissa
**Bruno (Eng. Pleno, Pedidos):** Não tratei disso na reunião além das perguntas que fiz.
**Diego (Eng. Sênior, Plataforma):** A tabela de dead letter guarda payload, motivo e timestamp. O replay manual recoloca o evento na outbox como pendente. · Fontes: [09:18] Diego
**Sofia (Eng. Segurança):** O replay tem que logar quem fez. · Fontes: [09:36] Sofia

**Entrevistador:** E os fluxos de configuração?

**Larissa (Tech Lead):** O `customer_id` vai no body ou no path, e não vem do JWT. · Fontes: [09:32] Larissa
**Bruno (Eng. Pleno, Pedidos):** A configuração guarda url, secret, `customer_id` e estado ativo. · Fontes: [09:21] Bruno
**Diego (Eng. Sênior, Plataforma):** Não tratei disso na reunião.
**Sofia (Eng. Segurança):** A secret é rotacionável. A antiga vale 24 horas em paralelo e depois morre. · Fontes: [09:21] Sofia
**Marcos (PM):** A secret é gerada por nós e devolvida na criação, junto com a lista de status. O histórico mostra os últimos 100 envios, com sucesso ou falha, payload, response e tempo. · Fontes: [09:31] Marcos; [09:34] Marcos

**Resumo da etapa.** Seis fluxos:
1. Criação do evento na transação.
2. Ciclo do worker.
3. Retry.
4. DLQ e replay.
5. Cadastro e edição.
6. Rotação da secret.

Diagramas previstos: um `sequenceDiagram` do caminho principal e um `stateDiagram-v2` da outbox.

**Divergências.** Nenhuma. A fala do Bruno em [09:11] ("mesmo Prisma client") foi corrigida por ele em [09:30]: a instância é separada.

**Lacunas da etapa.**
1. **Uma linha de outbox por evento ou por webhook?**
   - **A. Uma linha por webhook interessado, com `event_id` próprio, gerado na inserção de cada linha** (recomendada). O filtro é por webhook ([09:34] Bruno), o retry é por endpoint e o UUID nasce "quando o evento entra na outbox" ([09:25] Diego). O `X-Webhook-Id` diferencia os cadastros ([09:44] Sofia).
   - **B. Uma linha por webhook, com `event_id` compartilhado.** Contra: o `event_id` deixaria de ser único por linha, e o cliente que deduplica globalmente descartaria a entrega do segundo endpoint.
   - **C. Uma linha por mudança, com fan-out no worker.** Contra: contradiz o filtro na inserção e o retry por endpoint.
   - **Escolha:** A (`FDD-HIP-03`).
2. **Agendamento do backoff.**
   - **A. Colunas `attempts` e `nextAttemptAt`, com o status FAILED significando "última tentativa falhou, aguardando nova tentativa"** (recomendada). Usa os quatro estados citados em [09:08] Diego. O FAILED não é a DLQ, que continua em tabela separada (ADR-003).
   - **B. Recalcular o próximo horário a partir do histórico de entregas.** Contra: consulta mais cara a cada ciclo.
   - **Escolha:** A (`FDD-HIP-08`).
3. **A ordem por pedido durante os retries.** Com backoff, um evento posterior do mesmo pedido poderia sair antes de um anterior em espera.
   - **A. O worker não envia um evento de um par (webhook, pedido) enquanto houver evento anterior não entregue do mesmo par** (recomendada). Preserva a ordem por pedido decidida em [09:12] Diego e [09:13] Larissa.
   - **B. Sem bloqueio, com a ordem valendo só quando não há retry.** Contra: enfraquece uma garantia decidida.
   - **Escolha:** A (`FDD-HIP-09`).
4. **Armazenamento do histórico de entregas.** O verificador da RFC apontou essa lacuna (`processo/revisao-rfc.md`).
   - **A. Tabela de entregas com uma linha por tentativa HTTP: payload enviado, status code, corpo da resposta até 64KB com indicador de corte e duração** (recomendada). Cobre o que foi pedido em [09:34] Marcos.
   - **B. Só status code e duração, sem corpo.** Contra: não atende "payload, response".
   - **Escolha:** A (`FDD-HIP-11`).
5. **Tamanho do lote e paralelismo.**
   - **A. Lote de 10, envio em paralelo dentro do lote (no máximo um evento por par webhook e pedido), e o próximo ciclo 2 s depois do fim do anterior** (recomendada). "Batch pequeno" vem de [09:08] Diego. O paralelismo evita que um cliente lento atrase os outros ([09:04] Bruno).
   - **B. Lote de 10, sequencial.** Contra: dez clientes lentos somariam 100 s por ciclo e estourariam a meta de 10 s.
   - **Escolha:** A (`FDD-HIP-21`).
6. **Remoção de pedido e FKs.** O `OrderService.delete` apaga pedidos em PENDING ou CANCELLED.
   - **A. A outbox, a DLQ e as entregas guardam `orderId` sem FK para `orders`, porque o snapshot é autossuficiente. O webhook tem FK para `customers`, e as tabelas de eventos têm FK com cascade para o webhook** (recomendada). Não muda o comportamento atual do delete de pedidos.
   - **B. FK para `orders` com cascade.** Contra: apaga eventos já registrados.
   - **C. FK para `orders` com restrict.** Contra: muda o comportamento existente do DELETE de pedidos.
   - **Escolha:** A (`FDD-HIP-17`).
7. **Remoção de webhook** (`PRD-FR-03`).
   - **A. Hard delete com cascade para os eventos pendentes, a DLQ e o histórico daquele webhook** (recomendada). É o padrão de cascade do schema (`OrderItem`, `OrderStatusHistory`), e a auditoria de replay fica nos logs ([09:36] Sofia).
   - **B. Soft delete.** Contra: estado novo, e o GET e a listagem precisariam filtrar.
   - **Escolha:** A (`FDD-HIP-17`).
8. **Webhook inativo.**
   - **A. O PATCH aceita `active`. Um webhook inativo não recebe eventos novos (o filtro de inserção só considera os ativos), e os eventos já gravados seguem** (recomendada). O snapshot já foi registrado como válido ([09:52] Larissa).
   - **B. O worker descarta os eventos de webhook inativo.** Contra: exige um estado novo.
   - **Escolha:** A (`FDD-HIP-19`).

**Escolha da usuária:** resumo confirmado, lacunas 1 a 8 com a opção recomendada.

---

## Etapa 5. Contratos públicos

**Entrevistador:** Quais endpoints a feature expõe?

**Larissa (Tech Lead):** O CRUD é autenticado normal, e o replay exige ADMIN. · Fontes: [09:48] Larissa
**Bruno (Eng. Pleno, Pedidos):** POST, PATCH, DELETE e GET para listar os webhooks de um customer. · Fontes: [09:33] Bruno
**Diego (Eng. Sênior, Plataforma):** `POST /admin/webhooks/dead-letter/:id/replay`. · Fontes: [09:35] Diego
**Sofia (Eng. Segurança):** Um endpoint para o cliente pedir uma secret nova pela API. · Fontes: [09:21] Sofia
**Marcos (PM):** Cadastro por POST, com url e lista de status; a secret volta na criação. Histórico em `GET /webhooks/:id/deliveries`. · Fontes: [09:31] Marcos; [09:34] Marcos

**Entrevistador:** Como é o request de saída para o cliente?

**Larissa (Tech Lead):** Não tratei disso na reunião.
**Bruno (Eng. Pleno, Pedidos):** O payload é enxuto. · Fontes: [09:44] Bruno
**Diego (Eng. Sênior, Plataforma):** JSON com `event_id`, `event_type` (`order.status_changed`), timestamp ISO 8601, `order_id`, `order_number`, `from_status`, `to_status`, `customer_id` e campos básicos como `total_cents`, sem items. Headers: `X-Event-Id`, `X-Signature`, `X-Timestamp` e `Content-Type: application/json`. O timeout é de 10 s. · Fontes: [09:43] Diego; [09:44] Diego; [09:42] Diego
**Sofia (Eng. Segurança):** Mais o `X-Webhook-Id`. URL só https, com limite de 64KB e erro acima disso. · Fontes: [09:44] Sofia; [09:23] Sofia
**Marcos (PM):** Fui eu que perguntei o formato do payload e os headers. · Fontes: [09:43] Marcos; [09:44] Marcos

**Entrevistador:** E a função interna que o `changeStatus` chama?

**Larissa (Tech Lead):** Não tratei disso na reunião.
**Bruno (Eng. Pleno, Pedidos):** `publishWebhookEvent(tx, order, fromStatus, toStatus)`, que aceita o tx client da transação atual. · Fontes: [09:41] Bruno
**Diego (Eng. Sênior, Plataforma):** Função pura recebendo o tx. · Fontes: [09:41] Diego
**Sofia (Eng. Segurança):** Não tratei disso na reunião.

**Resumo da etapa.**
- Sete endpoints sob `/api/v1` (`src/app.ts`), com corpo de erro no formato do `src/middlewares/error.middleware.ts`.
- A função `publishWebhookEvent`.
- O request de saída, com cinco headers e o payload decidido.

**Divergências.** Nenhuma.

**Lacunas da etapa.**
1. **Onde vai o `customer_id`?**
   - **A. No path: `/api/v1/customers/:customerId/webhooks` para criar e listar; `/api/v1/webhooks/:id` para os demais** (recomendada). A listagem "dos webhooks de um customer" ([09:33] Bruno) não tem body, e o `GET /webhooks/:id/deliveries` ([09:34] Marcos) já usa o id do webhook.
   - **B. No body na criação.** Contra: a listagem precisaria de query, que não é nenhuma das opções de [09:32] Larissa.
   - **Escolha:** A (`FDD-HIP-04`).
2. **Prefixo das rotas.** Segue o código: tudo sob `/api/v1` (`src/app.ts`). É padrão do código, e não hipótese.
3. **Rota de rotação da secret.**
   - **A. `POST /api/v1/webhooks/:id/secret/rotate`, com 200 e a secret nova** (recomendada). É uma ação, não uma edição parcial.
   - **B. `PATCH /api/v1/webhooks/:id` com uma flag.** Contra: mistura edição e geração de credencial.
   - **Escolha:** A (`FDD-HIP-05`).
4. **Formato do `X-Signature` durante a rotação.**
   - **A. Um header com a lista `sha256=<hex>`; nas 24 h, `sha256=<hex_nova>,sha256=<hex_antiga>`. O cliente aceita se qualquer uma bater** (recomendada). É a única forma de as duas secrets valerem em paralelo ([09:21] Sofia) quando quem assina é a plataforma.
   - **B. Um header extra com a assinatura antiga.** Contra: dois headers para o cliente tratar, e um header que a reunião não listou.
   - **C. Assinar só com a nova.** Contra: o cliente que não migrou rejeita, e a convivência deixa de existir.
   - **Escolha:** A (`FDD-HIP-02`).
5. **Formato do `X-Timestamp`.**
   - **A. ISO 8601 em UTC, fora da assinatura** (recomendada). Segue o formato de horário do payload ([09:43] Diego). A assinatura é "sobre o corpo do request" ([09:22] Sofia).
   - **B. Unix epoch em segundos.** Contra: formato diferente do resto.
   - **Escolha:** A (`FDD-HIP-20`).
6. **Paginação.**
   - **A. A listagem de webhooks usa `paginated()` de `src/shared/http/response.ts`, com `page` e `pageSize`. O histórico devolve sempre as 100 entregas mais recentes, sem paginação** (recomendada).
   - **B. Ambos paginados.** Contra: o pedido foi "os últimos 100".
   - **Escolha:** A (`FDD-HIP-23`).
7. **Exposição da secret.**
   - **A. A secret só aparece na resposta da criação e da rotação, nunca no GET, na listagem ou no PATCH** (recomendada). Vai para a revisão da Sofia ([09:46] Sofia).
   - **B. O GET mostra a secret mascarada.** Contra: expõe parte da credencial sem necessidade.
   - **Escolha:** A (`FDD-HIP-14`).
8. **Campos do payload.**
   - **A. Só os campos citados: "campos básicos tipo `total_cents`" vira apenas `total_cents`; `timestamp` é o momento da mudança de status** (recomendada). Nenhum campo é inventado.
   - **B. Acrescentar `subtotal_cents` e `discount_cents`.** Contra: amplia um contrato que não foi decidido.
   - **Escolha:** A (`FDD-HIP-25`).
9. **Retorno e ponto de chamada da função interna.**
   - **A. Retorna `Promise<number>` com a quantidade de eventos gravados. É chamada depois do `orderStatusHistory.create` e antes da releitura do pedido. O filtro é feito em memória sobre os webhooks ativos do cliente** (recomendada).
   - **B. Retorna `void`.** Contra: os testes não teriam como verificar quantos eventos foram gravados.
   - **Escolha:** A (`FDD-HIP-26`).

**Escolha da usuária:** resumo confirmado, lacunas 1 e 3 a 9 com a opção recomendada.

---

## Etapa 6. Erros, exceções e fallback

**Entrevistador:** Quais erros a feature prevê e como cada um é tratado?

**Larissa (Tech Lead):** O prefixo é `WEBHOOK_` em tudo do módulo. O limite é de 64KB, com erro se ultrapassar. · Fontes: [09:29] Larissa; [09:24] Larissa
**Bruno (Eng. Pleno, Pedidos):** Sigo o padrão de `AppError`, com códigos como `WEBHOOK_NOT_FOUND`, `WEBHOOK_INVALID_URL`, `WEBHOOK_SECRET_REQUIRED`. O error middleware já trata `AppError`, Zod e Prisma. · Fontes: [09:28] Bruno; [09:29] Bruno
**Diego (Eng. Sênior, Plataforma):** Cliente que não responde em 10 s é tratado como falha e marcado para retry. · Fontes: [09:42] Diego
**Sofia (Eng. Segurança):** URL http é recusada com erro de validação. Payload grande não é enviado: prefiro errar a truncar. Replay só para ADMIN. · Fontes: [09:23] Sofia; [09:36] Sofia

**Entrevistador:** Quais estratégias de resiliência entram?

**Larissa (Tech Lead):** Retry com backoff exponencial, total de 5 tentativas, depois DLQ. · Fontes: [09:48] Larissa
**Bruno (Eng. Pleno, Pedidos):** Não tratei disso na reunião.
**Diego (Eng. Sênior, Plataforma):** O retry indefinido deixa evento pendurado para sempre. São 1m, 5m, 30m, 2h e 12h, quase 15 horas entre a primeira falha e a última tentativa. · Fontes: [09:15] Diego; [09:17] Diego
**Sofia (Eng. Segurança):** Timeout de 10 s, que achei bom. · Fontes: [09:42] Sofia

**Entrevistador:** Circuit breaker foi discutido?

**Larissa (Tech Lead):** Não tratei disso na reunião.
**Bruno (Eng. Pleno, Pedidos):** Não tratei disso na reunião.
**Diego (Eng. Sênior, Plataforma):** Não tratei disso na reunião.
**Sofia (Eng. Segurança):** Não tratei disso na reunião.

**Resumo da etapa.**
- A matriz combina códigos `WEBHOOK_*` novos com códigos já existentes (`VALIDATION_ERROR`, `UNAUTHORIZED`, `FORBIDDEN`, `NOT_FOUND`).
- Resiliência: timeout de 10 s, backoff fixo, DLQ e replay. Circuit breaker não foi discutido.
- Invariantes: atomicidade, ordem por par e nenhuma tentativa além da política.

**Divergências.** Nenhuma.

**Lacunas da etapa.**
1. **Contagem de tentativas.** Os cinco intervalos somam 14h36m. Isso só fecha com "quase 15 horas entre primeira falha e última tentativa" ([09:17] Diego) se houver o envio inicial mais 5 retentativas.
   - **A. Até 6 chamadas HTTP: a inicial e mais 5 retentativas, nos intervalos 1m, 5m, 30m, 2h e 12h contados de cada falha** (recomendada).
   - **B. 5 chamadas no total, usando só os quatro primeiros intervalos.** Contra: daria cerca de 2h36m e contradiz as 15 horas.
   - **Escolha:** A (`FDD-HIP-01`). O `PRD-FR-08` diz "5 tentativas no total, cobrindo cerca de 15 horas", o que é inconsistente. Vai como inconsistência no relatório, sem alterar o PRD.
2. **Evento acima de 64KB.** A verificação não pode desfazer a mudança de status.
   - **A. O evento é gravado normalmente. O worker mede o corpo antes de enviar e, acima de 64KB, não envia: move direto para a DLQ com o motivo `WEBHOOK_PAYLOAD_TOO_LARGE`, sem retry, e loga erro** (recomendada). Cumpre "não envia" e "erra" ([09:23] Sofia; [09:24] Larissa) e mantém a função de publicação simples.
   - **B. Mandar direto para a DLQ na inserção, dentro da transação.** Contra: põe lógica de entrega dentro da transação de pedidos.
   - **C. Lançar erro na inserção.** Contra: faz rollback da mudança de status, o que é proibido.
   - **Escolha:** A (`FDD-HIP-06`).
3. **Quais respostas contam como falha** (`RFC-LAC-02`).
   - **A. Sucesso é só 2xx. Qualquer outra resposta (3xx, 4xx, 5xx), erro de rede ou timeout é falha com retry. Redirects não são seguidos** (recomendada). Não cria política nova além de "falha vai para retry" ([09:42] Diego).
   - **B. 4xx, exceto 408 e 429, vai direto para a DLQ.** Contra: é uma regra de negócio nova.
   - **Escolha:** A (`FDD-HIP-07`).
4. **Eventos presos em PROCESSING e shutdown.**
   - **A. Ao iniciar, o worker devolve todas as linhas em PROCESSING para PENDING, sem contar tentativa, o que é seguro com um único worker. No SIGINT ou SIGTERM, ele para de buscar, espera o lote em andamento (cada chamada tem no máximo 10 s) e faz `prisma.$disconnect`, no molde do `src/server.ts`** (recomendada). Pode gerar reenvio, coberto pelo at-least-once ([09:24] Diego).
   - **B. Lease com expiração por linha.** Contra: só faz diferença com vários workers, o que é problema do futuro ([09:13] Diego).
   - **Escolha:** A (`FDD-HIP-10`).
5. **`WEBHOOK_SECRET_REQUIRED`.**
   - **A. Não usar. A secret é gerada pela plataforma ([09:31] Marcos). Um body com campo `secret` é recusado pelo schema `.strict()`, com `VALIDATION_ERROR`** (recomendada).
   - **B. Usar quando o cliente tentar mandar uma secret.** Contra: o código sugeriria que a secret é obrigatória, o contrário do decidido.
   - **Escolha:** A (`FDD-HIP-12`).
6. **Classes de erro com código `WEBHOOK_*`.** O `NotFoundError` sempre usa `NOT_FOUND`.
   - **A. Classes novas em `src/shared/errors/http-errors.ts`, no molde do `InvalidStatusTransitionError`: `WebhookNotFoundError` estende `AppError` (404); `WebhookInvalidUrlError` estende `BadRequestError` (400); `WebhookDeadLetterNotFoundError` estende `AppError` (404); `WebhookDeadLetterAlreadyReplayedError` estende `ConflictError` (409). A https é validada por um schema Zod de URL executado no service, que converte a falha em `WebhookInvalidUrlError`** (recomendada). Mantém a validação "no schema Zod" ([09:23] Sofia) e devolve o código `WEBHOOK_INVALID_URL` previsto no `PRD-FR-01`.
   - **B. Validar só com `validate()`.** Contra: o middleware sempre devolve `VALIDATION_ERROR`, e o código previsto se perde.
   - **Escolha:** A (`FDD-HIP-13`).
7. **Replay.**
   - **A. Cria uma linha nova na outbox com o mesmo `event_id` e o mesmo payload, com `attempts` zerado. A linha da DLQ fica marcada com `replayedAt` e `replayedById`. Um segundo replay devolve 409 `WEBHOOK_DEAD_LETTER_ALREADY_REPLAYED`. A resposta é 200** (recomendada). Mantém a evidência ([09:18] Diego) e a auditoria ([09:36] Sofia), e o cliente deduplica pelo mesmo id.
   - **B. Apagar a linha da DLQ no replay.** Contra: perde a evidência.
   - **Escolha:** A (`FDD-HIP-24`).

**Escolha da usuária:** resumo confirmado, lacunas 1 a 7 com a opção recomendada.

---

## Etapa 7. Observabilidade

**Entrevistador:** Que logs, métricas e tracing a feature precisa?

**Larissa (Tech Lead):** Reuso máximo do que já existe, Pino incluído. · Fontes: [09:30] Larissa
**Bruno (Eng. Pleno, Pedidos):** O logger Pino já está no projeto inteiro. Não vamos botar nada novo. · Fontes: [09:29] Bruno
**Diego (Eng. Sênior, Plataforma):** A DLQ serve de evidência para debug. Um cliente já vazou a secret no log da aplicação dele. · Fontes: [09:18] Diego; [09:22] Diego
**Sofia (Eng. Segurança):** O replay tem que ser logado com quem fez, para auditoria. · Fontes: [09:36] Sofia

**Entrevistador:** Métricas e tracing foram definidos?

**Larissa (Tech Lead):** Não tratei disso na reunião.
**Bruno (Eng. Pleno, Pedidos):** Não tratei disso na reunião.
**Diego (Eng. Sênior, Plataforma):** Não tratei disso na reunião.
**Sofia (Eng. Segurança):** Não tratei disso na reunião.

**Resumo da etapa.**
- Logs Pino em `snake_case`, no estilo de `server_started` e `http_request`.
- Métricas calculadas a partir das tabelas novas e dos logs, sem biblioteca nova.
- Tracing por correlação de ids.
- Nenhum log com secret ou assinatura.

**Divergências.** Nenhuma.

**Lacunas da etapa.**
1. **Métricas sem biblioteca** (`package.json` não tem nenhuma).
   - **A. Métricas derivadas do banco (outbox, DLQ, entregas) e um log periódico `webhook_worker_stats` a cada 60 s com contagens e idade do pendente mais antigo. O tempo até a primeira tentativa é medido entre a criação da linha e a entrega `attempt = 1` (`PRD-OBJ-01`)** (recomendada).
   - **B. Adotar uma biblioteca de métricas.** Contra: dependência nova não decidida ([09:29] Bruno).
   - **Escolha:** A (`FDD-HIP-15`).
2. **Tracing.**
   - **A. Correlação por `event_id`, `webhook_id` e `order_id` em todos os logs da feature. O replay e o CRUD também levam o `requestId` do `request-logger.middleware.ts`. Não há ferramenta de tracing distribuído, que não foi definida na reunião** (recomendada).
   - **B. Propagar o `requestId` da mudança de status até o worker.** Contra: muda a assinatura decidida da função interna.
   - **Escolha:** A (`FDD-HIP-15`).
3. **Secret em log e em repouso.** É hipótese para a revisão da Sofia ([09:46] Sofia), e não decisão.
   - **A. Acrescentar `*.secret`, `*.previousSecret`, `*.secretCiphertext` e `*.previousSecretCiphertext` ao `redactPaths`. Nunca logar `X-Signature` nem corpo de request com secret. Guardar a secret cifrada com AES-256-GCM (`crypto` nativo) com chave em variável de ambiente nova, porque o HMAC exige secret recuperável e não pode ser hash** (recomendada).
   - **B. Guardar em texto puro e só redigir os logs.** Contra: um vazamento do banco expõe todas as secrets.
   - **Escolha:** A (`FDD-HIP-14`), marcada como proposta para a revisão de segurança.

**Escolha da usuária:** resumo confirmado, lacunas 1 a 3 com a opção recomendada.

---

## Etapa 8. Dependências e compatibilidade

**Entrevistador:** Que versões e componentes a feature usa?

**Larissa (Tech Lead):** Mesmo projeto, com um script novo de worker. · Fontes: [09:11] Larissa
**Bruno (Eng. Pleno, Pedidos):** Mesmo banco e mesma `DATABASE_URL`, com instância nova de PrismaClient. · Fontes: [09:30] Bruno
**Diego (Eng. Sênior, Plataforma):** Outbox no MySQL existente, sem Redis. · Fontes: [09:07] Diego
**Sofia (Eng. Segurança):** HMAC-SHA256 é padrão de mercado, e todo cliente tem biblioteca para isso. · Fontes: [09:20] Sofia

**Entrevistador:** O que não pode mudar para quem já usa a API?

**Larissa (Tech Lead):** Não tratei disso na reunião.
**Bruno (Eng. Pleno, Pedidos):** A mudança é dentro do `changeStatus`. O error middleware pega os erros novos sem precisar mudar. · Fontes: [09:40] Bruno; [09:29] Bruno
**Diego (Eng. Sênior, Plataforma):** Não tratei disso na reunião.
**Sofia (Eng. Segurança):** Não tratei disso na reunião.

**Resumo da etapa.**
- As versões vêm do `package.json` (Node 20 ou mais, Express 4.21.1, Prisma 5.22.0, Zod 3.23.8, Pino 9.5.0, uuid 11.0.3, Vitest 2.1.4, Supertest 7.0.0) e o banco vem do `prisma/schema.prisma` (MySQL).
- Nenhuma dependência nova.
- O contrato de `PATCH /api/v1/orders/:id/status` não muda.

**Divergências.** Nenhuma.

**Lacunas da etapa.**
1. **Cliente HTTP.**
   - **A. `fetch` nativo do Node 20 com `AbortSignal.timeout(10000)` e `redirect: 'manual'`. HMAC e geração de secret com o módulo `crypto` nativo** (recomendada).
   - **B. Uma biblioteca HTTP.** Contra: dependência nova sem decisão.
   - **Escolha:** A (`FDD-HIP-22`).
2. **Parâmetros configuráveis.**
   - **A. As constantes ficam no módulo (2 s, 10 s, lote de 10, 64KB, backoff). A única variável de ambiente nova é a chave de cifra da `FDD-HIP-14`** (recomendada). Os números são decididos e não precisam variar por ambiente.
   - **B. Tudo em `src/config/env.ts`.** Contra: permite mudar números decididos sem revisão.
   - **Escolha:** A (`FDD-HIP-21`).

**Escolha da usuária:** resumo confirmado, lacunas 1 e 2 com a opção recomendada.

---

## Etapa 9. Critérios de aceite técnicos

**Entrevistador:** Que critérios objetivos definem a feature pronta?

**Larissa (Tech Lead):** Integração no `order.service` e testes ponta a ponta. Revisão da Sofia no fim. · Fontes: [09:46] Larissa; [09:47] Larissa
**Bruno (Eng. Pleno, Pedidos):** Não pode ter caso de status mudar e evento não sair. · Fontes: [09:40] Bruno
**Diego (Eng. Sênior, Plataforma):** Timeout de 10 s conta como falha e vai para retry. · Fontes: [09:42] Diego
**Sofia (Eng. Segurança):** Pelo menos dois dias úteis de revisão de segurança antes do deploy, com foco em HMAC e geração de secret. · Fontes: [09:46] Sofia

**Resumo da etapa.** Os critérios cobrem quatro frentes:
- **Funcional:** CRUD, filtro, snapshot, histórico e replay.
- **Resiliência:** timeout, contagem de tentativas, DLQ e recuperação de PROCESSING.
- **Segurança:** HMAC, dupla assinatura, https, ADMIN e secret fora dos logs.
- **Observabilidade:** logs com ids e métricas calculáveis.

A suíte existente de pedidos continua passando.

**Divergências.** Nenhuma.

**Lacunas da etapa.**
1. **Testes do worker.**
   - **A. O worker expõe uma função de ciclo único (`processBatch`) chamada direto nos testes, com o endpoint do cliente simulado por um servidor HTTP local. O `tests/setup.ts` precisa apagar as tabelas novas antes de `customers`** (recomendada). O FDD só registra isso; o arquivo não é alterado agora.
   - **B. Subir o processo do worker nos testes.** Contra: testes lentos e instáveis.
   - **Escolha:** A (`FDD-HIP-16`).

**Escolha da usuária:** resumo confirmado, lacuna 1 com a opção recomendada.

---

## Etapa 10. Riscos e mitigação

**Entrevistador:** Quais riscos técnicos de implementação existem?

**Larissa (Tech Lead):** A ordem global não é garantida. Fica documentada como limitação conhecida. · Fontes: [09:13] Larissa
**Bruno (Eng. Pleno, Pedidos):** Se acumular evento, o worker fica lento? · Fontes: [09:07] Bruno
**Diego (Eng. Sênior, Plataforma):** Índice em status e em `created_at`, leitura só dos pendentes. Escalar exige particionar ou lock pessimista, e isso é do futuro. · Fontes: [09:08] Diego; [09:13] Diego
**Sofia (Eng. Segurança):** Quero olhar HMAC e geração de secret com calma. · Fontes: [09:46] Sofia

**Resumo da etapa.** Riscos técnicos:
1. Rollback de pedidos por falha da outbox (`RFC-RISCO-02`).
2. Ciclo lento estourar os 10 s.
3. Crescimento das tabelas, sem arquivamento.
4. Reenvio após queda do worker.
5. Exposição da secret.
6. Bloqueio de ordem atrasando os eventos de um pedido.
7. A limpeza dos testes quebrar.

Cada risco cita o ID do PRD ou da RFC quando existe.

**Divergências.** Nenhuma.

**Lacunas da etapa.** Nenhuma nova. A secret em repouso e em log foi tratada na etapa 7.

**Escolha da usuária:** resumo confirmado.

---

## Etapa 11. Integração com o sistema existente

**Entrevistador:** Como o módulo se integra com cada arquivo existente?

**Larissa (Tech Lead):** `src/worker.ts` no molde do `src/server.ts`, com o script "npm run worker". O replay reaproveita o `requireRole`. · Fontes: [09:11] Larissa; [09:36] Larissa
**Bruno (Eng. Pleno, Pedidos):** O `changeStatus` chama a função dentro da transação. A pasta é `src/modules/webhooks`, com a lógica do worker em `webhook.worker.ts` ou `webhook.processor.ts`. O error middleware não muda. · Fontes: [09:41] Bruno; [09:28] Bruno; [09:29] Bruno
**Diego (Eng. Sênior, Plataforma):** Perguntei se o worker abria o mesmo PrismaClient ou um separado; ficou separado, porque é outro processo. · Fontes: [09:29] Diego; [09:30] Bruno
**Sofia (Eng. Segurança):** Não tratei disso na reunião.

**Resumo da etapa.** A tabela de integração cobre doze arquivos existentes:
- `order.service.ts`, `app.ts`, `routes/index.ts`, `server.ts`, `database.ts`, `http-errors.ts`;
- `auth.middleware.ts`, `error.middleware.ts`, `logger/index.ts`, `schema.prisma`, `package.json`, `tests/setup.ts`.

Os arquivos novos ficam no módulo `src/modules/webhooks/` e em `src/worker.ts`.

**Divergências.** O Bruno ofereceu dois nomes para o processador ([09:28] Bruno).

**Lacunas da etapa.**
1. **Nome do arquivo do worker no módulo.**
   - **A. `webhook.worker.ts`** (recomendada). É o primeiro nome citado em [09:28] Bruno.
   - **B. `webhook.processor.ts`.**
   - **Escolha:** A (registrada junto com a `FDD-HIP-21`).

**Escolha da usuária:** resumo confirmado, lacuna 1 com a opção recomendada.

---

## Hipóteses consolidadas (apresentadas antes de escrever o FDD)

| ID | Hipótese adotada | Âncora |
| --- | --- | --- |
| FDD-HIP-01 | Até 6 chamadas: a inicial e mais 5 retentativas (1m, 5m, 30m, 2h, 12h), em 14h36m | [09:17] Diego |
| FDD-HIP-02 | `X-Signature` com lista `sha256=<hex>`: a atual mais cada secret anterior dentro das suas 24 h *(revisada na verificação, rodada 3)* | [09:21] Sofia |
| FDD-HIP-03 | Uma linha de outbox por webhook interessado, cada uma com `event_id` próprio | [09:25] Diego |
| FDD-HIP-04 | `customer_id` no path (`/customers/:customerId/webhooks`) | [09:32] Larissa |
| FDD-HIP-05 | Rotação em `POST /api/v1/webhooks/:id/secret/rotate`; rotações seguidas aceitas, cada secret substituída com as suas 24 h *(revisada na verificação, rodada 3)* | [09:21] Sofia |
| FDD-HIP-06 | Evento acima de 64KB: o worker não envia e move para a DLQ (`WEBHOOK_PAYLOAD_TOO_LARGE`) | [09:24] Larissa |
| FDD-HIP-07 | Sucesso só com 2xx; o resto, rede e timeout são falha com retry; sem seguir redirect | [09:42] Diego |
| FDD-HIP-08 | `attempts` e `nextAttemptAt`; FAILED quer dizer aguardando nova tentativa | [09:08] Diego |
| FDD-HIP-09 | Bloqueio por par (webhook, pedido) na consulta de seleção; a espera por bloqueio é exceção declarada ao `PRD-CA-03` *(revisada na verificação, rodadas 2 e 3)* | [09:12] Diego |
| FDD-HIP-10 | Tentativa contada antes do envio; PROCESSING volta para FAILED, ou vai para a DLQ com 6 tentativas; shutdown gracioso *(revisada na verificação, rodada 2; substitui "PENDING sem contar tentativa")* | `src/server.ts` |
| FDD-HIP-11 | Tabela de entregas por tentativa, com corpo da resposta até 64KB | [09:34] Marcos |
| FDD-HIP-12 | `WEBHOOK_SECRET_REQUIRED` não é usado; `secret` no body é recusado pelo schema | [09:31] Marcos |
| FDD-HIP-13 | Classes `Webhook*Error` em `http-errors.ts`; https validada por schema Zod no service | `src/shared/errors/http-errors.ts` |
| FDD-HIP-14 | Secret com `randomBytes(32)`, cifrada AES-256-GCM, redigida nos logs, exposta só na criação e na rotação (para a Sofia) | [09:22] Diego; [09:46] Sofia |
| FDD-HIP-15 | Métricas pelo banco e por `webhook_worker_stats`; tracing por correlação de ids | [09:29] Bruno |
| FDD-HIP-16 | `processBatch` testável e limpeza das tabelas novas no `tests/setup.ts` | `tests/setup.ts` |
| FDD-HIP-17 | Sem FK para `orders`; cascade de customer para webhook e de webhook para eventos; hard delete | `prisma/schema.prisma` |
| FDD-HIP-18 | Criação de pedido não gera evento | [09:40] Bruno |
| FDD-HIP-19 | `active` no PATCH; inativo não recebe eventos novos; os já gravados seguem | [09:21] Bruno |
| FDD-HIP-20 | `X-Timestamp` em ISO 8601 UTC, fora da assinatura | [09:44] Diego |
| FDD-HIP-21 | Polling em intervalo fixo de 2 s, com até 10 envios simultâneos e seleção serializada; constantes no módulo; `webhook.worker.ts` *(revisada na verificação, rodadas 2 e 3; substitui "2 s depois do fim do lote")* | [09:08] Diego |
| FDD-HIP-22 | `fetch` nativo e `crypto` nativo, sem dependência nova | `package.json` |
| FDD-HIP-23 | Listagem paginada com `paginated()`; histórico fixo nas 100 mais recentes | `src/shared/http/response.ts` |
| FDD-HIP-24 | Replay cria linha nova com o mesmo `event_id`, marca a DLQ, segundo replay dá 409, resposta 200 | [09:18] Diego |
| FDD-HIP-25 | Payload só com os campos citados; `timestamp` é o momento da mudança | [09:43] Diego |
| FDD-HIP-26 | `publishWebhookEvent` retorna a quantidade gravada; chamada após o histórico; filtro em memória | [09:41] Bruno |

---

## Verificação independente

O verificador é um subagente do Claude Code criado só com os caminhos dos arquivos e o prompt adversarial da tarefa, sem resumo do FDD.

### Rodada 1: NEEDS_FIX

| # | Apontamento | Decisão | O que mudou |
| --- | --- | --- | --- |
| B1 | As 6 chamadas contradizem PRD, RFC e ADR-003 ("5 tentativas") | Aplicado em parte | A leitura foi mantida, porque é a recomendada pela usuária no prompt e a única que fecha com [09:17] Diego. O FDD agora declara o conflito na `FDD-HIP-01` e no `FDD-FLUXO-03`, com pedido de confirmação da Larissa e o ajuste necessário se a decisão for 5 no total. Vai no relatório final como inconsistência |
| B2 | O `LIMIT 10` antes do filtro por par pode encher o lote de eventos bloqueados e parar os outros clientes | Aplicado | O bloqueio por par entrou na própria consulta (`NOT EXISTS`, antes do `LIMIT`), cobrindo linhas `FAILED` com retry futuro |
| B3a | `publishWebhookEvent` retorna `number`, mas o exemplo era `{ "inserted": 1 }` | Aplicado | O exemplo virou `1` |
| B3b | Contratos com `:id` não listavam 400 para id que não é UUID | Aplicado | `400 VALIDATION_ERROR` em `CONTRATO-03`, `05`, `06` e `07` |
| I1 | Colunas, tipos, remoção da outbox ao mover para a DLQ, P2003 e P2025, limiares de alerta, janelas por hora, regra de versionamento, `dev:worker` e nomes de arquivo sem ID de hipótese | Aplicado | Cada detalhe foi ligado à hipótese que o originou (ver "Detalhamentos" abaixo). A regra de versionamento foi retirada e virou "não definida na reunião". O tratamento de P2003 e P2025 foi corrigido |
| I1b | Índice composto atribuído a [09:08] Diego | Aplicado | Os índices simples continuam citados; a composição ficou marcada como `FDD-HIP-08` |
| I2 | `FDD-RISCO-07` diz que a limpeza quebra, mas o cascade evita isso | Aplicado | O risco foi reescrito: a limpeza passa a depender do cascade, e a mitigação é a limpeza explícita |
| I3 | `X-Timestamp` fora da assinatura não detecta replay attack | Aplicado | O FDD registra a limitação. A detecção usa o `event_id` e o `timestamp` do corpo assinado, como ponto para a Sofia |
| I4 | A métrica da primeira tentativa incluía o timeout e era ambígua depois de um replay | Aplicado | `webhook_deliveries` ganhou `startedAt` e `outboxId`, e o `FDD-OBJ-01` e o `FDD-OBS-03` passaram a usar os dois |
| I5 | O replay quebra a ordem prometida | Aplicado | Ressalva em `FDD-FLUXO-04`, `FDD-RES-11` e `FDD-CA-12` |
| T3 | A validação da URL para `WEBHOOK_INVALID_URL` estava indefinida | Aplicado | O schema de rota usa `min(1).max(2048)`; o `webhookUrlSchema` roda no service |
| T4 e T5 | Chave de cifra (obrigatoriedade, API, `.env.example`, testes) e `env` no worker | Aplicado | Detalhado em `FDD-HIP-14`, `FDD-INT-05` e `FDD-INT-13` |
| T6 | Logs do worker e da API com o mesmo `service` | Aplicado | `logger.child({ component: 'webhook-worker' })` (`FDD-HIP-15`) |
| T7 | Replay com webhook inativo | Aplicado | O replay envia mesmo assim (`FDD-HIP-24`) |
| T8 | Relógio controlado sem definição | Aplicado | `processBatch(now)` com horário em JS (`FDD-HIP-16`) |
| T10 | Origem do `timestamp` do payload | Aplicado | `new Date()` na publicação (`FDD-HIP-25`) |
| M1 | `.strict()` só no POST | Aplicado | POST e PATCH usam `.strict()` (`FDD-HIP-12`) |
| M2 | Erros conhecidos do Prisma não viram 500 | Aplicado | Nota em `FDD-ERRO-09` |
| M3 | Títulos com `###` no documento e nas seções | Recusado | É o formato exato do esqueleto do curso, e o prompt manda segui-lo |
| M4 | As escolhas foram delegadas pela usuária | Recusado | A usuária respondeu "segue direto, decide você". A regra do prompt prevê esse caso: adotar as recomendadas e mostrar a lista antes de escrever, o que foi feito |
| M5 | Requisições `{}` nos GET, DELETE, rotate e replay | Aplicado | Os exemplos mostram método, path e header de autorização |

Total: 21 apontamentos. 18 aplicados, 1 aplicado em parte e 2 recusados.

### Detalhamentos incorporados às hipóteses já escolhidas

Nenhum desses detalhes cria escolha nova. Cada um detalha a opção já adotada na etapa indicada:
- `FDD-HIP-08` (etapa 4, lacuna 2): as colunas `lockedAt`, `lastError` e `deliveredAt` e os índices compostos.
- `FDD-HIP-11` (etapa 4, lacuna 4): as colunas `outboxId`, `startedAt` e `MediumText`.
- `FDD-HIP-12` (etapa 6, lacuna 5): `.strict()` também no PATCH.
- `FDD-HIP-14` (etapa 7, lacuna 3): o nome e a obrigatoriedade da chave, e o arquivo `webhook.signature.ts`.
- `FDD-HIP-15` (etapa 7, lacunas 1 e 2): os limiares de alerta, as janelas de uma hora e o `logger.child`.
- `FDD-HIP-16` (etapa 9): `processBatch(now)`.
- `FDD-HIP-17` (etapa 4, lacunas 6 e 7): os nomes e tipos de coluna e o descarte por P2003 e P2025.
- `FDD-HIP-20` (etapa 5, lacuna 5): a limitação contra replay.
- `FDD-HIP-21` (etapas 4 e 8): o script `dev:worker` e a pausa de 2 s após o lote. A pausa foi substituída na rodada 2 pelo intervalo fixo.
- `FDD-HIP-24` (etapa 6, lacuna 7): a remoção da outbox ao mover para a DLQ e o replay com webhook inativo.
- `FDD-HIP-25` (etapa 5, lacuna 8): a origem do `timestamp`.
- `FDD-HIP-26` (etapa 5, lacuna 9): o arquivo `webhook.publisher.ts`.

### Rodada 2: NEEDS_FIX

| # | Apontamento | Decisão | O que mudou |
| --- | --- | --- | --- |
| B1 | As 6 chamadas continuam contradizendo PRD, RFC e ADR-003, e o texto fazia o desenvolvedor esperar uma confirmação | Recusado em parte | A leitura de 6 chamadas foi pedida no prompt da tarefa ("Minha leitura recomendada é essa ... Não altere o PRD ... reporte a inconsistência"), e PRD, RFC e ADRs não podem ser alterados neste trabalho. O texto do `FDD-FLUXO-03` passou a ser afirmativo ("Este FDD adota 6 chamadas"), sem condicionar a implementação. A divergência vai para o relatório final |
| B2 | Esperar 2 s depois do fim do lote deixa um cliente lento atrasar todos (10 s + 2 s) | Aplicado | O polling virou intervalo fixo de 2 s, que não espera os envios, com até 10 envios simultâneos (`FDD-HIP-21`, `FDD-FLUXO-02`, `FDD-RES-07`, `FDD-RISCO-02`) |
| B3 | O bloqueio por par contamina a meta de 10 s e o alerta | Aplicado | Os eventos bloqueados saem da medida e do alerta (`FDD-HIP-09`, `FDD-OBJ-01`, `FDD-OBS-02`, `FDD-OBS-03`). O `FDD-RISCO-06` declara a espera visível para o cliente |
| B4 | A recuperação sem contar tentativa permite mais de 6 chamadas, e o histórico e a outbox não ficavam numa transação | Aplicado | A tentativa é contada antes do envio. A recuperação volta para `FAILED` ou para a DLQ. O histórico e a atualização da outbox ficam numa transação (`FDD-HIP-10`, `FDD-RES-05`, `FDD-RES-09`, `FDD-CA-09`, `FDD-CA-11`, diagrama de estados) |
| B5 | O p95 afrouxa a meta "abaixo de 10 s" | Aplicado | `FDD-OBJ-01` agora exige todo evento não bloqueado abaixo de 10 s, e a `FDD-OBS-03` usa o máximo |
| B6 | A segunda rotação encurtava as 24 h | Aplicado | Uma rotação durante a convivência é recusada com 409 `WEBHOOK_SECRET_ROTATION_IN_PROGRESS` (`FDD-HIP-05`, `FDD-FLUXO-06`, `FDD-ERRO-15`, `FDD-CONTRATO-05`, `FDD-INT-06`) |
| D1 | Códigos internos e tabela de logs sem marca de hipótese | Aplicado | Os códigos estão listados na `FDD-HIP-13`, e a tabela de logs ficou marcada como `FDD-HIP-15` |
| D2 | PATCH vazio e `events` vazio sem hipótese | Aplicado | `FDD-HIP-12` e `FDD-FLUXO-05` |
| D3 | `FDD-FORA-10` sem marca | Aplicado | Marcada como `FDD-HIP-24` |
| D4 | Corpos de resposta de rotação, replay e histórico sem marca | Aplicado | `FDD-HIP-05`, `FDD-HIP-24` e `FDD-HIP-23` |
| D5 | A frase da L12 dizia que todas as hipóteses foram escolhidas na sessão | Aplicado | A frase agora explica a delegação e aponta os detalhamentos da verificação |
| D6 | A rodada 1 do log dizia "recomendada pela usuária" | Esclarecido | A leitura é a "leitura recomendada" escrita pela usuária no prompt da tarefa (ambiguidade 1), e não uma escolha feita durante a entrevista |
| D7 | `FDD-RISCO-03` atribuía índices compostos a Diego | Aplicado | Corrigido |
| D8 | `FDD-RES-10` atribuía o `eventId` no replay a [09:25] Diego | Aplicado | A parte do replay passou a citar `FDD-HIP-24` |
| C1 | O `X-Timestamp` esvazia o `PRD-RNF-11` | Recusado | A assinatura só do corpo foi decidida ([09:22] Sofia). O FDD registra a limitação e a alternativa assinada (o `timestamp` do corpo) para a revisão da Sofia, sem mudar decisão |
| C2 | O hard delete apaga a evidência da DLQ | Aplicado em parte | A escolha foi mantida (etapa 4, lacuna 7), e o efeito ficou explícito no `FDD-FLUXO-05` e no `FDD-DEP-11` |
| C3 | O DELETE de cliente passa a apagar em cascata | Aplicado | Explicitado em `FDD-DEP-11` |
| X1 | P2025 viraria `NOT_FOUND` | Aplicado | O service busca por id antes de alterar e lança `WebhookNotFoundError` (`FDD-FLUXO-05`) |
| X2 | `now()` contra o relógio injetado | Aplicado | Os fluxos usam `now` calculado em JS |
| X3 | Ordem das chaves no JSON do MySQL | Aplicado | Nota no `FDD-FLUXO-02`: a ordem dos exemplos é ilustrativa |
| X4 | Ordem entre `requireRole` e `validate` | Aplicado | `requireRole` antes, o que dá 403 (`FDD-INT-07`) |
| T2 | Codificação do AES-GCM | Aplicado | Formato `iv:authTag:ciphertext` em base64. A rotação da chave fica registrada como não definida (`FDD-HIP-14`) |
| T3 | `NOT EXISTS` não cabe no `findMany` | Aplicado | SQL do `$queryRaw` com desempate por `id` |
| T5 e T6 | Conteúdo do `lastError` e do `PAYLOAD_TOO_LARGE` | Aplicado | Formato `"<CÓDIGO>: <detalhe>"` e `"body com N bytes"` |
| T7 | O timeout cobre a leitura do corpo? | Aplicado | Sim, o `AbortSignal` cobre a resposta inteira |
| T8 | `webhook_event_enqueued` dentro da transação | Aplicado | O evento de log foi removido; a correlação usa o `orderId` |
| T9 | Classificar timeout, rede e TLS | Aplicado | `TimeoutError` vira timeout, os demais erros do `fetch` viram rede, e não 2xx vira HTTP |

Total: 27 apontamentos. 22 aplicados, 2 aplicados em parte ou esclarecidos, 1 esclarecido e 2 recusados.

**Detalhamentos desta rodada**, feitos pelo entrevistador sob a delegação "decide você", dentro das opções já escolhidas:
- `FDD-HIP-05`: o 409 na rotação durante a convivência e o corpo da resposta.
- `FDD-HIP-09`: os eventos bloqueados fora da meta.
- `FDD-HIP-10`: a tentativa contada antes do envio.
- `FDD-HIP-12`: body não vazio.
- `FDD-HIP-13`: a lista de códigos.
- `FDD-HIP-14`: a codificação do texto cifrado.
- `FDD-HIP-21`: o intervalo fixo com vagas.
- `FDD-HIP-23`: o invólucro do histórico.
- `FDD-HIP-24`: o corpo do replay e a origem do id.


### Rodada 3: NEEDS_FIX (última rodada permitida)

| # | Apontamento | Decisão | O que mudou |
| --- | --- | --- | --- |
| B1 | 6 chamadas contra "total 5 tentativas" ([09:48] Larissa); a L50 ainda pedia confirmação | Aplicado em parte | A frase de confirmação saiu da `FDD-HIP-01`. A hipótese passou a citar [09:48] Larissa e explica a leitura (os mesmos 5 intervalos). A contagem continua 6, como pede o prompt da tarefa, e a divergência vai para o relatório final |
| B2 | Hipóteses mudadas depois da escolha; o log e a tabela não batiam | Aplicado | A tabela consolidada foi atualizada, com a marca "revisada na verificação" nas `FDD-HIP-02`, `05`, `09`, `10` e `21`. Essas mudanças vão para a usuária no relatório final, como escolhas do entrevistador sob a delegação "decide você" |
| B3 | O 409 na rotação cria regra de produto e conflita com o `PRD-PUB-08` | Aplicado | O 409 saiu. Rotações seguidas são aceitas, e cada secret substituída fica numa lista `previousSecrets` com as suas 24 h. O `FDD-ERRO-15` e a quinta classe de erro foram removidos |
| B4 | A exceção à meta de 10 s enfraquece o `PRD-CA-03` | Aplicado | O `FDD-OBJ-01` declara a exceção como divergência com o `PRD-CA-03`, aceita pela ordem por pedido. Vai para o relatório final |
| I1 | Probabilidades de risco inventadas; `FDD-RISCO-04` contradizia o PRD | Aplicado | Viraram "não definida na reunião", menos a `FDD-RISCO-05`, que herda "média" do `PRD-RISCO-02` |
| I2 | Teste do worker com https | Aplicado | Função `send` injetável com stub nos testes (`FDD-HIP-16`) |
| I3 | O intervalo fixo pode selecionar a mesma linha duas vezes | Aplicado | A seleção e a marcação foram serializadas por flag (`FDD-FLUXO-02`, passo 2) |
| I4 | A chave obrigatória quebra o boot e os testes | Aplicado | `FDD-DEP-13` registra a mudança de deploy e de testes |
| I5 | Log desatualizado | Aplicado | Tabela consolidada e lista de detalhamentos atualizadas |
| M1 | `createdAt` atribuído a [09:09] Diego | Aplicado | Passou a citar [09:12] Diego |
| M2 | A falta de notificação vinha do README | Aplicado | Agora cita `src/routes/index.ts` |
| M3 | Campos do histórico e `error` versus `lastError` | Aplicado | Campos listados no `FDD-CONTRATO-06`, `outboxId` e `startedAt` no exemplo, e `error` guarda só o código |
| M4 | 400 de `customerId` e a redação da URL | Aplicado | `FDD-CONTRATO-01`, `02` e `03` |
| M5 | `FDD-OBS-09` e a recusa de `PENDING` sem marca | Aplicado | `FDD-HIP-15` e `FDD-HIP-12` |
| M6 | `*.secret` não cobre a raiz | Aplicado | `secret` acrescentado na raiz do `redactPaths` |

Total: 15 apontamentos. 14 aplicados e 1 aplicado em parte. Pelo limite de três rodadas, o verificador não foi rodado de novo.

**Veredito final:** NEEDS_FIX na rodada 3. Os apontamentos procedentes foram aplicados. Ficam duas divergências deliberadas com os documentos anteriores, reportadas à usuária:
- a contagem de 6 chamadas contra "5 tentativas";
- a exceção à meta de 10 s para eventos bloqueados por ordem.

## Validação final

As 11 validações rodaram no terminal, sem scripts no repositório. Uma falhou na primeira execução:
- **Validação 3:** o FDD citava a pasta `prisma/migrations` como caminho. O texto passou a apontar o script existente `npm run db:migrate`.

Os arquivos novos propostos (`src/worker.ts`, `src/modules/webhooks/*`, `tests/webhooks.test.ts`) aparecem só como "novo" ou "propostos" e nunca como Fonte `CODIGO`.

Resultado das 11 validações:
1. Todas as citações existem na transcrição.
2. Nenhuma citação está fora do formato.
3. Todos os caminhos existentes conferem, e as linhas `CODIGO` do tracker apontam para arquivos reais.
4. A seção 11 cita 14 arquivos existentes.
5. Há 7 endpoints completos, com requisição, resposta e status codes.
6. Todos os códigos novos da matriz começam com `WEBHOOK_`.
7. Os 168 IDs `FDD-*` batem entre o FDD e o tracker, sem duplicados na tabela.
8. Os links para o PRD, a RFC e os ADRs estão válidos.
9. Os 18 blocos JSON são válidos.
10. O tracker tem 84,3% das linhas com Fonte `TRANSCRICAO`.
11. Não há travessão longo no FDD nem neste log.
