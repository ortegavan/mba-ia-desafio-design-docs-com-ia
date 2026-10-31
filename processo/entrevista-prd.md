# Log da entrevista simulada para o PRD

Feature: Sistema de Webhooks de Notificação de Pedidos.
Roteiro: `entrevista.txt` (com as adaptações do README do desafio). Respostas dos participantes baseadas apenas em `TRANSCRICAO.md` e no código do repositório.

Participantes e janela de presença na call:

- Larissa (Tech Lead, conduz): 09:00 a 09:53
- Marcos (PM): 09:00 a 09:50
- Bruno (Eng. Pleno, Pedidos): 09:00 a 09:53
- Diego (Eng. Sênior, Plataforma): 09:05 a 09:53
- Sofia (Eng. Segurança): 09:00 a 09:50

---

## Etapa 1: Contexto e visão geral

**Entrevistador:** Qual é o produto ou sistema em que essa feature entra?

**Marcos (PM):** Entra na nossa plataforma, na parte de pedidos. Os clientes já usam a nossa API e hoje consultam o GET /orders de tempos em tempos para saber se o status mudou. A feature avisa esses clientes quando o status de um pedido muda. · Fontes: [09:00] Marcos

**Larissa (Tech Lead):** No serviço de orders. A dúvida que levantei no começo foi se o disparo seria síncrono, dentro desse serviço, ou por fila/outbox. Também sugeri que o worker seja uma nova entry-point do mesmo projeto, como já existe o src/server.ts. · Fontes: [09:03] Larissa; [09:11] Larissa

**Bruno (Eng. Pleno, Pedidos):** No módulo de pedidos, mais precisamente na mudança de status. Hoje o changeStatus já faz numa transação o update do pedido, o insert no histórico de status e a mudança de estoque. O evento de webhook entra nessa mesma transação. · Fontes: [09:04] Bruno; [09:40] Bruno; código `src/modules/orders/order.service.ts`

**Diego (Eng. Sênior, Plataforma):** Na plataforma que já existe: mesmo banco MySQL e mesma stack. A diferença é que o envio roda num processo separado da API. · Fontes: [09:07] Diego; [09:11] Diego

**Sofia (Eng. Segurança):** No nosso sistema, como webhook outbound: só sai de nós para o cliente. O cliente não manda nada para nós. · Fontes: [09:02] Marcos; [09:03] Sofia

**Entrevistador:** Essa feature pertence a um sistema que já existe ou faz parte de um sistema novo?

**Marcos (PM):** Sistema que já existe. O cliente cadastra o webhook direto na nossa API, autenticado com o JWT do nosso sistema. · Fontes: [09:32] Marcos

**Larissa (Tech Lead):** Sistema existente. A decisão foi reaproveitar ao máximo o que já temos, e webhook vira um módulo igual aos outros. · Fontes: [09:30] Larissa

**Bruno (Eng. Pleno, Pedidos):** Existente. Vamos seguir o padrão de módulos que já existe em src/modules, com uma pasta nova para webhooks. · Fontes: [09:27] Bruno

**Diego (Eng. Sênior, Plataforma):** Existente. A outbox fica no MySQL que já usamos, sem subir infraestrutura nova. Mesmo banco e mesma stack, só que em outro processo. · Fontes: [09:07] Diego; [09:11] Diego

**Sofia (Eng. Segurança):** Não tratei de onde a feature vai rodar. Só lembrei que a validação da URL https é feita no schema Zod, que já é como o projeto valida. · Fontes: [09:23] Sofia

**Entrevistador:** Quem é o público-alvo?

**Marcos (PM):** Clientes B2B que integram com a nossa API. O pedido formal veio de três deles: Atlas Comercial, MaxDistribuição e Nova Cargo. Eles têm usuários no nosso sistema que os representam. Os desenvolvedores desses clientes vão ler o portal de desenvolvedor, onde vou documentar a integração. · Fontes: [09:00] Marcos; [09:32] Marcos; [09:40] Marcos

**Larissa (Tech Lead):** Os clientes B2B que o Marcos citou. Há também o administrador da plataforma, que é quem pode fazer o replay da DLQ. · Fontes: [09:00] Marcos; [09:36] Larissa

**Bruno (Eng. Pleno, Pedidos):** Os clientes B2B. Lembrei que o JWT atual é de usuário operador, não de cliente, e o Marcos explicou que existem usuários que representam o cliente. · Fontes: [09:32] Bruno; [09:32] Marcos

**Diego (Eng. Sênior, Plataforma):** Cheguei depois da abertura, então só conheço a Atlas pelo nome, que o Marcos citou na parte de prazo. Para mim o público são os clientes que recebem os eventos e fazem a dedup do lado deles, além do admin que faz o replay. · Fontes: [09:45] Marcos; [09:25] Diego; [09:18] Diego

**Sofia (Eng. Segurança):** Clientes com endpoints fora da nossa infraestrutura, que precisam conferir que a requisição saiu de nós. Para o replay, o público é só a role ADMIN. · Fontes: [09:19] Sofia; [09:36] Sofia

**Entrevistador:** Em duas ou três frases, qual é o objetivo de negócio desta feature?

**Marcos (PM):** Avisar os clientes em tempo real quando o status de um pedido muda, para eles pararem de fazer polling, que deixa a integração lenta e cara. Para eles, tempo real é qualquer coisa abaixo de 10 segundos. A Atlas disse que pode migrar para o concorrente se não entregarmos até o fim do trimestre, e depois falou em fim de novembro. · Fontes: [09:00] Marcos; [09:02] Marcos; [09:45] Marcos

**Larissa (Tech Lead):** Atender o pedido dos clientes que o Marcos trouxe dentro do prazo. Estimei três sprints, já incluindo a revisão da Sofia. · Fontes: [09:00] Marcos; [09:47] Larissa

**Bruno (Eng. Pleno, Pedidos):** Notificar a mudança de status. Perguntei se era tempo real de verdade e o Marcos respondeu que abaixo de 10 segundos atende. · Fontes: [09:01] Bruno; [09:02] Marcos

**Diego (Eng. Sênior, Plataforma):** Entregar a notificação dentro do requisito de "abaixo de 10 segundos". O polling de 2 segundos atende, e o Marcos confirmou. · Fontes: [09:09] Diego; [09:10] Marcos

**Sofia (Eng. Segurança):** O objetivo comercial foi o Marcos quem trouxe: notificar em tempo real e tirar o cliente do polling. A minha parte é garantir que o cliente consiga confirmar que a requisição veio de nós e não foi alterada no caminho. · Fontes: [09:00] Marcos; [09:19] Sofia

**Entrevistador:** Quais são os cenários de uso principais?

**Marcos (PM):** O cliente cadastra o webhook e escolhe os status que quer receber, por exemplo só SHIPPED e DELIVERED. Ele também precisa consultar o histórico de entregas: sucesso ou falha, o que foi enviado, a resposta e o tempo de resposta. · Fontes: [09:31] Marcos; [09:33] Marcos; [09:34] Marcos

**Larissa (Tech Lead):** Além do cadastro e do histórico, o admin faz o replay manual de eventos que foram para a DLQ. · Fontes: [09:35] Larissa; [09:36] Larissa

**Bruno (Eng. Pleno, Pedidos):** Gerenciar o webhook: criar, editar, remover e listar os webhooks de um customer, escolhendo os eventos por endpoint. · Fontes: [09:33] Bruno

**Diego (Eng. Sênior, Plataforma):** Quando o cliente fica fora do ar, tentamos de novo e, se esgotar as tentativas, o evento vai para a DLQ, de onde o admin pode reprocessar. Cliente com vários cadastros precisa saber de qual cadastro veio cada envio. · Fontes: [09:15] Diego; [09:18] Diego; [09:44] Sofia; [09:45] Diego

**Sofia (Eng. Segurança):** O cliente confere a assinatura de cada envio e pode pedir uma secret nova pela API quando quiser rotacionar. A antiga continua valendo por 24 horas. · Fontes: [09:20] Sofia; [09:21] Sofia

### Resumo da etapa 1

- A feature entra no sistema existente de gestão de pedidos, no mesmo banco MySQL e na mesma stack. O envio roda num processo separado da API e o código segue o padrão de módulos que já existe.
- O público são os clientes B2B que integram via API (Atlas Comercial, MaxDistribuição e Nova Cargo pediram formalmente) e os desenvolvedores desses clientes. O admin da plataforma também é público, para o replay da DLQ.
- O objetivo de negócio é notificar a mudança de status em menos de 10 segundos, acabar com o polling no GET /orders e reduzir o risco de a Atlas ir para o concorrente.
- Os cenários principais são: cadastrar e gerenciar webhooks com filtro de status, receber e validar o envio assinado, consultar o histórico de entregas, rotacionar a secret e fazer o replay da DLQ pelo admin.

### Divergências e como a reunião resolveu

- **Prazo da Atlas.** Marcos falou em "fim do trimestre" às 09:00 e em "fim de novembro" às 09:45. As duas falas não se contradizem. Vou usar a versão mais específica (fim de novembro), junto com a estimativa de três sprints que a Larissa deu às 09:47.
- **Quem cadastra.** Bruno notou às 09:32 que o JWT é de operador. Marcos explicou que existem usuários que representam o cliente, e a Larissa fechou que o customer_id não vem do JWT. Detalhado na etapa 5.
- **Janela do Diego.** Ele não ouviu a abertura, então só conhece a Atlas pelo nome. Mesmo assim, às 09:09 ele cita o requisito de "abaixo de 10 segundos", que o Marcos tinha dado às 09:02, antes de ele entrar. Tratado como algo que ele ficou sabendo depois, usando a fala dele às 09:09 como fonte própria.

### Confirmação do usuário

> Correto

---

## Etapa 2: Problema e oportunidade

**Entrevistador:** O que está acontecendo hoje que torna essa feature necessária?

**Marcos (PM):** Os clientes não têm como saber quando o status de um pedido muda. Eles ficam consultando o GET /orders de tempos em tempos, e isso deixa a integração lenta e cara para eles. Por isso Atlas Comercial, MaxDistribuição e Nova Cargo fizeram um pedido formal na semana passada. · Fontes: [09:00] Marcos

**Larissa (Tech Lead):** O problema de negócio é o que o Marcos trouxe na abertura: os clientes fazem polling e pediram notificação. Do meu lado, não tratei de outro problema além desse. · Fontes: [09:00] Marcos

**Bruno (Eng. Pleno, Pedidos):** O problema dos clientes eu ouvi do Marcos. Do lado técnico, a restrição é que a transação de mudança de status já é pesada. Uma chamada HTTP dentro dela faria um cliente lento travar a mudança de status de outros pedidos. Isso limita como resolver o problema, não é o problema em si. · Fontes: [09:00] Marcos; [09:04] Bruno

**Diego (Eng. Sênior, Plataforma):** Não tratei disso na reunião. Cheguei às 09:05, depois que o Marcos já tinha explicado o contexto.

**Sofia (Eng. Segurança):** A necessidade comercial veio do Marcos. O problema que eu trouxe é que vamos mandar dados de pedidos para endpoints fora da nossa infraestrutura. O cliente precisa conseguir confirmar que a requisição veio de nós e que ninguém alterou o conteúdo no caminho. · Fontes: [09:00] Marcos; [09:19] Sofia

**Entrevistador:** Existe um exemplo real recente, com números aproximados, que mostre o impacto?

**Marcos (PM):** O exemplo concreto é que três clientes B2B pediram formalmente, e a Atlas avisou que pode migrar para o concorrente se não entregarmos até o fim do trimestre. Não trouxe números de custo nem de volume de chamadas no polling. · Fontes: [09:00] Marcos

**Larissa (Tech Lead):** Não tratei disso na reunião.

**Bruno (Eng. Pleno, Pedidos):** Não tratei disso na reunião.

**Diego (Eng. Sênior, Plataforma):** Trouxe dois exemplos que já aconteceram. Um cliente nosso ficou duas horas fora do ar numa manutenção planejada, por isso três tentativas seriam pouco. E um cliente já vazou a secret no log da própria aplicação. · Fontes: [09:16] Diego; [09:22] Diego

**Sofia (Eng. Segurança):** Usei o vazamento que o Diego contou como motivo para a secret ser única por endpoint e rotacionável. Com uma secret global, se uma vazar, vaza tudo. · Fontes: [09:21] Sofia; [09:22] Diego

**Entrevistador:** O que já foi tentado e não funcionou?

**Marcos (PM):** Do nosso lado, nada foi tentado. O que existe hoje é o contorno dos próprios clientes, que fazem polling no GET /orders, e é isso que está lento e caro. · Fontes: [09:00] Marcos

**Larissa (Tech Lead):** Não tratei disso na reunião.

**Bruno (Eng. Pleno, Pedidos):** Não tratei disso na reunião.

**Diego (Eng. Sênior, Plataforma):** Não tratei disso na reunião.

**Sofia (Eng. Segurança):** Não tratei disso na reunião.

**Entrevistador:** Entre os problemas levantados, qual tem a maior prioridade?

**Marcos (PM):** O risco comercial: a Atlas pode migrar para o concorrente se não houver entrega no prazo. O prazo que ela pediu é fim de novembro, e vou confirmar com eles. · Fontes: [09:00] Marcos; [09:45] Marcos; [09:47] Marcos

**Larissa (Tech Lead):** Tratei a entrega dentro do prazo como o ponto principal. Estimei três sprints, com a revisão da Sofia no fim. · Fontes: [09:46] Larissa; [09:47] Larissa

**Bruno (Eng. Pleno, Pedidos):** Não tratei disso na reunião.

**Diego (Eng. Sênior, Plataforma):** Não tratei disso na reunião.

**Sofia (Eng. Segurança):** Não classifiquei a prioridade dos problemas. Só pedi pelo menos dois dias úteis para revisar o código de segurança antes do deploy, porque quero olhar o HMAC e a geração de secret com calma. · Fontes: [09:46] Sofia

### Resumo da etapa 2

- **Problema 1:** os clientes B2B fazem polling no GET /orders para descobrir mudanças de status, e a integração fica lenta e cara. Três clientes pediram notificação formalmente.
- **Problema 2:** há risco de perder cliente, porque a Atlas disse que pode ir para o concorrente se não houver entrega no prazo. Esse é o ponto que Marcos e Larissa trataram como principal.
- **Problema 3:** enviar dados de pedidos para fora da nossa infraestrutura exige que o cliente consiga confirmar a origem e a integridade de cada envio.
- **Fatos de apoio:** um cliente já ficou duas horas fora do ar em manutenção planejada e outro já vazou a secret em log.
- **Não definido na reunião:** números de custo ou de volume do polling, e tentativas anteriores de resolver.

### Divergências e como a reunião resolveu

- Não houve divergência nesta etapa.
- Diego não ouviu a abertura, por isso não fala sobre o problema de negócio.
- A restrição técnica que o Bruno apontou (a transação já é pesada) vai para as etapas 7 e 8, como justificativa de arquitetura, e não entra como problema de produto.
- Prioridade proposta: alta para o risco comercial e para o polling, com base na fala do Marcos às 09:00. Para o problema de segurança, alta também, com base na decisão fechada da Sofia às 09:22 e no pedido de revisão dela às 09:46.

### Confirmação do usuário

> Correto (prioridade alta mantida também para o problema de segurança)

---

## Etapa 3: Objetivos e métricas de sucesso

**Entrevistador:** Que resultado mensurável vocês querem alcançar com essa feature?

**Marcos (PM):** Que o cliente seja avisado da mudança de status sem precisar ficar atualizando por conta própria, e em menos de 10 segundos. Para eles, isso já é tempo real. · Fontes: [09:02] Marcos

**Larissa (Tech Lead):** Registrei como decisão o worker em polling a cada 2 segundos. Isso define quanto tempo um evento espera, no pior caso, até ser pego, e aceitamos esse custo. Também quero entregar dentro de três sprints. · Fontes: [09:10] Larissa; [09:47] Larissa

**Bruno (Eng. Pleno, Pedidos):** Nenhuma mudança de status pode acontecer sem que o evento seja registrado. Se a inserção do evento falhar, a mudança de status volta atrás. · Fontes: [09:40] Bruno

**Diego (Eng. Sênior, Plataforma):** Atender o "abaixo de 10 segundos" com folga, usando o polling de 2 segundos. E não perder evento quando o cliente fica fora do ar por um tempo, porque as tentativas cobrem uma janela longa. · Fontes: [09:09] Diego; [09:15] Diego; [09:16] Diego

**Sofia (Eng. Segurança):** Não tratei disso na reunião.

**Entrevistador:** Qual métrica representa cada um desses resultados?

**Marcos (PM):** O tempo entre a mudança de status e o cliente receber o aviso. · Fontes: [09:02] Marcos

**Larissa (Tech Lead):** O intervalo do polling, que é o tempo máximo que o evento fica esperando antes de o worker pegá-lo. Para o prazo, o número de sprints. · Fontes: [09:10] Larissa; [09:47] Larissa

**Bruno (Eng. Pleno, Pedidos):** A quantidade de mudanças de status que acontecem sem evento registrado na outbox. · Fontes: [09:40] Bruno; [09:41] Diego

**Diego (Eng. Sênior, Plataforma):** Para o cliente fora do ar, o número de tentativas e a janela de tempo coberta até o evento ir para a DLQ. · Fontes: [09:15] Diego; [09:17] Diego

**Sofia (Eng. Segurança):** Não tratei disso na reunião.

**Entrevistador:** Qual é a meta para cada métrica?

**Marcos (PM):** Menos de 10 segundos para o aviso. O polling de 2 segundos serve para mim. Sobre a janela de tentativas: se um cliente ficar 15 horas fora, o problema já é dele, então considero aceitável. · Fontes: [09:02] Marcos; [09:10] Marcos; [09:17] Marcos

**Larissa (Tech Lead):** Polling a cada 2 segundos. São 5 tentativas com backoff de 1m, 5m, 30m, 2h e 12h. A entrega em três sprints já inclui a revisão da Sofia. · Fontes: [09:10] Larissa; [09:17] Larissa; [09:47] Larissa

**Bruno (Eng. Pleno, Pedidos):** Zero. Não pode existir caso de status mudar e evento não sair. · Fontes: [09:40] Bruno

**Diego (Eng. Sênior, Plataforma):** 5 tentativas, com quase 15 horas entre a primeira falha e a última tentativa. Três tentativas seria pouco, porque já tivemos cliente fora do ar por duas horas. · Fontes: [09:16] Diego; [09:17] Diego

**Sofia (Eng. Segurança):** Não tratei disso na reunião.

### Resumo da etapa 3

| Objetivo | Métrica | Meta |
| --- | --- | --- |
| Notificar o cliente em tempo real | Tempo entre a mudança de status e a primeira tentativa de envio | Menos de 10 s, com polling a cada 2 s |
| Não perder eventos de mudança de status | Mudanças de status sem evento registrado | 0 |
| Absorver indisponibilidade do cliente | Tentativas e janela coberta antes da DLQ | 5 tentativas, cerca de 15 h |
| Entregar no prazo pedido pela Atlas | Sprints até a entrega, com revisão de segurança | 3 sprints (fim de novembro) |

Não definido na reunião: uma métrica para a redução do polling no GET /orders e uma métrica de retenção dos clientes.

### Divergências e como a reunião resolveu

- **Número de tentativas.** Bruno propôs 3 às 09:16 e Diego defendeu 5. A Larissa fechou em 5 às 09:17, e o Bruno aceitou o resumo às 09:49.
- **Interpretação (1).** A Larissa disse às 09:10 que "a latência mínima vai ser 2 segundos no pior caso". Entendido como a espera máxima até o worker pegar o evento, e não como latência total de entrega.
- **Interpretação (2).** Na meta de menos de 10 s, foi considerada a primeira tentativa de envio. Os retries ficam de fora, porque com o cliente fora do ar nenhum prazo é possível.
- **Ponto para decidir.** O prazo de três sprints é um compromisso de entrega, não uma métrica de produto.

### Confirmação do usuário

> Correto (prazo de três sprints mantido como objetivo)

---

## Etapa 4: Escopo

**Entrevistador:** O que precisa obrigatoriamente estar pronto nesta entrega?

**Marcos (PM):** Do lado do cliente: cadastrar o webhook, com a secret gerada por nós; escolher os status que quer receber; editar, remover e listar os webhooks; ver o histórico de entregas. Do meu lado, documento a integração no portal de desenvolvedor, deixando bem claro que o mesmo evento pode chegar duas vezes. · Fontes: [09:31] Marcos; [09:33] Marcos; [09:34] Marcos; [09:26] Marcos; [09:40] Marcos

**Larissa (Tech Lead):** O que listei no resumo final: outbox no MySQL, na mesma transação da mudança de status; worker separado, em polling de 2 segundos; 5 tentativas com backoff e depois DLQ; HMAC com secret por endpoint e rotação com 24 horas de transição; at-least-once com identificador de evento; reuso dos padrões do projeto; CRUD autenticado e replay da DLQ só para ADMIN. · Fontes: [09:48] Larissa

**Bruno (Eng. Pleno, Pedidos):** O CRUD de configuração e o filtro de status. O filtro é aplicado no momento em que o evento é registrado, ou seja, sem nenhum webhook interessado, nada é registrado. Também a integração com a mudança de status do pedido, na mesma transação. Depois que o Marcos e a Sofia saíram, fechamos que o evento guarda uma foto do pedido no momento da mudança. · Fontes: [09:33] Bruno; [09:34] Bruno; [09:40] Bruno; [09:52] Bruno

**Diego (Eng. Sênior, Plataforma):** Outbox; worker em processo separado; retry com DLQ; replay manual pelo admin; identificador único por evento para dedup; timeout de 10 segundos por chamada; payload enxuto, sem os itens do pedido. · Fontes: [09:06] Diego; [09:11] Diego; [09:15] Diego; [09:18] Diego; [09:25] Diego; [09:42] Diego; [09:43] Diego

**Sofia (Eng. Segurança):** Assinatura HMAC-SHA256 sobre o corpo, com secret única por endpoint; rotação da secret com 24 horas de convivência entre a antiga e a nova; URL só https; limite de 64KB, com erro quando passar; replay só para ADMIN, com registro de quem fez. · Fontes: [09:22] Sofia; [09:23] Sofia; [09:24] Larissa; [09:36] Sofia

**Entrevistador:** O que está explicitamente fora de escopo nesta entrega?

**Marcos (PM):** O e-mail avisando o cliente quando o webhook dele falha ficou como "futuro". O painel visual para o cliente também ficou de fora. Os clientes não enviam webhooks para nós, só recebem. Eles nunca pediram ordem global de eventos. · Fontes: [09:37] Larissa; [09:38] Marcos; [09:40] Larissa; [09:02] Marcos; [09:14] Marcos

**Larissa (Tech Lead):** O e-mail fica para a próxima fase, depois de medirmos o impacto. O rate limiting de envio fica como "observar e decidir depois". O painel visual é projeto separado, do time de frontend. A ordem global de eventos não é garantida, e isso foi registrado como limitação conhecida. · Fontes: [09:37] Larissa; [09:39] Larissa; [09:40] Larissa; [09:13] Larissa

**Bruno (Eng. Pleno, Pedidos):** O disparo síncrono dentro da transação foi descartado. Sugeri trigger no banco para o worker reagir mais rápido, mas o Diego explicou que o MySQL não notifica processo externo, então ficou o polling. A minha proposta de 3 tentativas perdeu para 5. · Fontes: [09:04] Bruno; [09:09] Diego; [09:17] Larissa

**Diego (Eng. Sênior, Plataforma):** Ficam de fora: disparo síncrono; Redis Streams ou Redis Cluster, que seria overengineering para um time pequeno; retry indefinido; exactly-once. O arquivamento dos eventos entregues, depois de uns 30 dias, está fora do escopo desta feature. Rodar vários workers em paralelo é problema do futuro. Rate limiting de envio: sugeri observar e implementar se virar problema. · Fontes: [09:06] Diego; [09:07] Diego; [09:15] Diego; [09:25] Diego; [09:08] Diego; [09:13] Diego; [09:39] Diego

**Sofia (Eng. Segurança):** Webhook de entrada, do cliente para nós, não faz parte. Truncar payload grande também não: quando passar do limite, a decisão é dar erro. Restringir o CRUD a roles específicas fica para mais adiante. Por enquanto, qualquer role autenticada pode usar. · Fontes: [09:03] Sofia; [09:23] Sofia; [09:24] Larissa; [09:37] Sofia

### Resumo da etapa 4

Incluso:
- Registro do evento junto com a mudança de status, de forma atômica e com filtro por status.
- Envio assíncrono por worker separado em polling de 2 s, com timeout de 10 s.
- 5 tentativas com backoff, DLQ e replay manual pelo ADMIN com auditoria.
- Assinatura HMAC-SHA256 com secret por endpoint e rotação com 24 h de transição.
- Somente https e limite de 64KB.
- Entrega at-least-once com identificador de evento.
- CRUD de configuração, histórico de entregas e documentação no portal.

Fora de escopo, por tipo:
- Adiado: e-mail de alerta de falha (próxima fase); restrição de roles no CRUD (mais adiante).
- Em aberto: rate limiting de saída ("observar e decidir depois").
- Fora desta feature: painel visual (time de frontend); arquivamento de eventos antigos.
- Descartado: disparo síncrono, Redis Streams, trigger no banco, retry indefinido, exactly-once, truncamento de payload e webhook de entrada.
- Limitação conhecida: sem ordem global e sem vários workers em paralelo.

### Divergências e como a reunião resolveu

- **Trigger no banco.** Bruno sugeriu às 09:09, Diego explicou que não funciona com o MySQL, e ficou o polling. Decisão registrada por Larissa às 09:10.
- **3 ou 5 tentativas.** Ficou em 5, decidido às 09:17.
- **Truncar ou dar erro.** Sofia preferiu erro, Diego sugeriu 64KB, e Larissa fechou às 09:24.
- **E-mail de alerta.** Marcos propôs às 09:37, Larissa tirou do escopo, e Marcos anotou como futuro às 09:38.
- **Foto do pedido no evento.** Decidida às 09:52, depois que Marcos e Sofia saíram; só os três que ficaram respondem por ela. No PRD fica em linguagem de produto: o evento reflete o pedido no momento da mudança.

### Confirmação do usuário

> Correto

---

## Etapa 5: Requisitos funcionais

**Entrevistador:** Quais funcionalidades a feature precisa oferecer?

**Marcos (PM):** O cliente precisa: cadastrar o webhook; escolher os status que quer receber; editar, remover e listar os cadastros; ver o histórico de entregas. · Fontes: [09:31] Marcos; [09:33] Marcos; [09:34] Marcos

**Larissa (Tech Lead):** Além do que o Marcos listou: o evento registrado junto com a mudança de status; envio com retry e DLQ; replay da DLQ pelo admin; assinatura com rotação de secret. · Fontes: [09:48] Larissa

**Bruno (Eng. Pleno, Pedidos):** CRUD completo (POST, PATCH, DELETE, GET), filtro por status e o registro do evento dentro do changeStatus. · Fontes: [09:33] Bruno; [09:40] Bruno

**Diego (Eng. Sênior, Plataforma):** Envio pelo worker, retry com backoff, DLQ e replay manual. · Fontes: [09:09] Diego; [09:15] Diego; [09:18] Diego

**Sofia (Eng. Segurança):** Assinatura em cada envio e rotação de secret pela API. · Fontes: [09:20] Sofia; [09:21] Sofia

Lista consolidada:

| ID | Requisito |
| --- | --- |
| FR-01 | Cadastrar webhook |
| FR-02 | Editar webhook |
| FR-03 | Remover webhook |
| FR-04 | Listar webhooks de um cliente |
| FR-05 | Filtrar eventos por status |
| FR-06 | Registrar o evento junto com a mudança de status |
| FR-07 | Enviar o evento assinado ao cliente |
| FR-08 | Retentar com backoff e mover para DLQ |
| FR-09 | Replay manual da DLQ pelo admin |
| FR-10 | Consultar histórico de entregas |
| FR-11 | Rotacionar a secret |

### FR-01 Cadastrar webhook

**Entrevistador:** Qual é o fluxo principal do cadastro?

**Marcos (PM):** O cliente chama um POST na nossa API, autenticado, informando a URL e a lista de status que quer receber. Nós geramos a secret e devolvemos na resposta da criação. · Fontes: [09:31] Marcos; [09:32] Marcos

**Larissa (Tech Lead):** É um endpoint autenticado normal. Corrigi o Marcos: o customer_id vai no body ou no path, não vem do JWT. · Fontes: [09:32] Larissa

**Bruno (Eng. Pleno, Pedidos):** O cadastro guarda URL, secret, customer_id e se está ativo. · Fontes: [09:21] Bruno; [09:21] Sofia

**Diego (Eng. Sênior, Plataforma):** Não tratei disso na reunião.

**Sofia (Eng. Segurança):** Cada cadastro tem uma secret única, nunca uma secret global da plataforma. · Fontes: [09:21] Sofia

**Entrevistador:** Em que condições o cadastro é bloqueado ou retorna erro?

**Marcos (PM):** Não tratei disso na reunião.

**Larissa (Tech Lead):** Sem autenticação não passa, porque é endpoint autenticado. Qualquer role autenticada pode cadastrar. · Fontes: [09:32] Larissa; [09:37] Sofia; código `src/middlewares/auth.middleware.ts`

**Bruno (Eng. Pleno, Pedidos):** Os erros do módulo seguem o padrão AppError com prefixo WEBHOOK_. Citei WEBHOOK_INVALID_URL como exemplo. · Fontes: [09:28] Bruno; [09:29] Larissa

**Diego (Eng. Sênior, Plataforma):** Não tratei disso na reunião.

**Sofia (Eng. Segurança):** URL com http é recusada com erro de validação. Só aceitamos https. · Fontes: [09:23] Sofia

### FR-02 Editar webhook

**Entrevistador:** Qual é o fluxo principal da edição?

**Marcos (PM):** Não tratei do detalhe. Concordei que o cliente escolhe por endpoint quais eventos recebe, e isso pode mudar. · Fontes: [09:33] Marcos

**Larissa (Tech Lead):** Não tratei disso na reunião.

**Bruno (Eng. Pleno, Pedidos):** Um PATCH para editar o cadastro. · Fontes: [09:33] Bruno

**Diego (Eng. Sênior, Plataforma):** Não tratei disso na reunião.

**Sofia (Eng. Segurança):** Não tratei disso na reunião.

**Entrevistador:** Em que condições a edição é bloqueada ou retorna erro?

**Marcos (PM):** Não tratei disso na reunião.

**Larissa (Tech Lead):** Exige autenticação, igual ao resto do CRUD. · Fontes: [09:48] Larissa

**Bruno (Eng. Pleno, Pedidos):** Se o webhook não existir, retorna WEBHOOK_NOT_FOUND, que foi um dos códigos que citei. · Fontes: [09:28] Bruno

**Diego (Eng. Sênior, Plataforma):** Não tratei disso na reunião.

**Sofia (Eng. Segurança):** A regra do https vale para qualquer URL cadastrada. Ela é validada no schema. · Fontes: [09:23] Sofia

### FR-03 Remover webhook

**Entrevistador:** Qual é o fluxo principal da remoção?

**Marcos (PM):** Não tratei disso na reunião.

**Larissa (Tech Lead):** Não tratei disso na reunião.

**Bruno (Eng. Pleno, Pedidos):** Um DELETE para remover o cadastro. · Fontes: [09:33] Bruno

**Diego (Eng. Sênior, Plataforma):** Não tratei disso na reunião.

**Sofia (Eng. Segurança):** Não tratei disso na reunião.

**Entrevistador:** Em que condições a remoção é bloqueada ou retorna erro?

**Marcos (PM):** Não tratei disso na reunião.

**Larissa (Tech Lead):** Exige autenticação. · Fontes: [09:48] Larissa

**Bruno (Eng. Pleno, Pedidos):** Webhook inexistente retorna WEBHOOK_NOT_FOUND. · Fontes: [09:28] Bruno

**Diego (Eng. Sênior, Plataforma):** Não tratei disso na reunião.

**Sofia (Eng. Segurança):** Não tratei disso na reunião.

### FR-04 Listar webhooks de um cliente

**Entrevistador:** Qual é o fluxo principal da listagem?

**Marcos (PM):** Não tratei do detalhe, mas está dentro do CRUD que o Bruno listou. · Fontes: [09:33] Bruno

**Larissa (Tech Lead):** Não tratei disso na reunião.

**Bruno (Eng. Pleno, Pedidos):** Um GET que lista os webhooks de um customer. · Fontes: [09:33] Bruno

**Diego (Eng. Sênior, Plataforma):** Não tratei disso na reunião.

**Sofia (Eng. Segurança):** Não tratei disso na reunião.

**Entrevistador:** Em que condições a listagem é bloqueada ou retorna erro?

**Marcos (PM):** Não tratei disso na reunião.

**Larissa (Tech Lead):** Exige autenticação. · Fontes: [09:48] Larissa

**Bruno (Eng. Pleno, Pedidos):** Não tratei disso na reunião.

**Diego (Eng. Sênior, Plataforma):** Não tratei disso na reunião.

**Sofia (Eng. Segurança):** Não tratei disso na reunião.

### FR-05 Filtrar eventos por status

**Entrevistador:** Qual é o fluxo principal do filtro?

**Marcos (PM):** Cada webhook tem uma lista dos status que quer ouvir, por exemplo só SHIPPED e DELIVERED. O filtro é aplicado na hora de registrar o evento. · Fontes: [09:33] Marcos

**Larissa (Tech Lead):** Não tratei disso na reunião.

**Bruno (Eng. Pleno, Pedidos):** Na mudança de status, só registramos o evento para os webhooks do customer que querem aquele status. · Fontes: [09:34] Bruno

**Diego (Eng. Sênior, Plataforma):** Perguntei se o filtro era no registro ou no envio. Ficou no registro, e concordei. · Fontes: [09:34] Diego; [09:34] Bruno

**Sofia (Eng. Segurança):** Não tratei disso na reunião.

**Entrevistador:** Em que condições o fluxo do filtro muda?

**Marcos (PM):** Não tratei disso na reunião.

**Larissa (Tech Lead):** Não tratei disso na reunião.

**Bruno (Eng. Pleno, Pedidos):** Se nenhum webhook do customer quer aquele status, nada é registrado, o que economiza linhas na tabela. · Fontes: [09:34] Bruno

**Diego (Eng. Sênior, Plataforma):** Concordo com o Bruno. · Fontes: [09:34] Diego

**Sofia (Eng. Segurança):** Não tratei disso na reunião.

### FR-06 Registrar o evento junto com a mudança de status

**Entrevistador:** Qual é o fluxo principal do registro do evento?

**Marcos (PM):** Não tratei disso na reunião.

**Larissa (Tech Lead):** O evento entra na outbox dentro da mesma transação da mudança de status. Ficou decidido às 09:08 e repetido no resumo. · Fontes: [09:08] Larissa; [09:48] Larissa

**Bruno (Eng. Pleno, Pedidos):** Dentro do changeStatus, que já atualiza o pedido, grava o histórico e mexe no estoque, o evento é inserido na mesma transação. Pensamos numa função que recebe a transação atual. Depois das 09:50 combinamos que o evento guarda a foto do pedido no momento da mudança. · Fontes: [09:40] Bruno; [09:41] Bruno; [09:52] Bruno; código `src/modules/orders/order.service.ts`

**Diego (Eng. Sênior, Plataforma):** Se a transação commitou, o evento está registrado. Se deu rollback, o evento some junto. Cada evento recebe um identificador único no momento em que entra na outbox. · Fontes: [09:06] Diego; [09:25] Diego

**Sofia (Eng. Segurança):** Não tratei disso na reunião.

**Entrevistador:** Em que condições o registro muda ou falha?

**Marcos (PM):** Não tratei disso na reunião.

**Larissa (Tech Lead):** Prefiro guardar a foto do evento na inserção. Se o pedido mudar depois, o evento ainda mostra o estado de quando o status mudou. · Fontes: [09:52] Larissa

**Bruno (Eng. Pleno, Pedidos):** Se a inserção do evento falhar, a transação inteira volta atrás e o status não muda. Se a transição de status for inválida, o changeStatus já recusa antes, e nesse caso não há evento. · Fontes: [09:40] Bruno; código `src/modules/orders/order.status.ts`

**Diego (Eng. Sênior, Plataforma):** É essencial ficar dentro da transação. Fora dela, a garantia se perde. · Fontes: [09:41] Diego

**Sofia (Eng. Segurança):** Não tratei disso na reunião.

### FR-07 Enviar o evento assinado ao cliente

**Entrevistador:** Qual é o fluxo principal do envio?

**Marcos (PM):** Perguntei o formato e os headers, e o Diego respondeu. O payload é enxuto e não leva os itens do pedido. Se o cliente quiser os detalhes, consulta o GET /orders/:id. · Fontes: [09:43] Marcos; [09:43] Diego

**Larissa (Tech Lead):** O worker separado busca os eventos pendentes a cada 2 segundos e envia. · Fontes: [09:10] Larissa

**Bruno (Eng. Pleno, Pedidos):** Concordei com o payload enxuto. · Fontes: [09:44] Bruno

**Diego (Eng. Sênior, Plataforma):** O fluxo é este: (1) a cada 2 segundos, o worker pega os pendentes mais antigos em lote pequeno; (2) envia cada um com o identificador do evento, a assinatura e o horário do envio; (3) marca o evento como entregue. Também incluímos o identificador do cadastro. Com um único worker, a ordem é mantida por pedido. · Fontes: [09:08] Diego; [09:09] Diego; [09:12] Diego; [09:44] Diego; [09:45] Diego

**Sofia (Eng. Segurança):** O corpo é assinado com HMAC-SHA256 usando a secret do endpoint, e o cliente verifica do lado dele. Pedi para incluir o identificador do cadastro, para quem tem vários endpoints saber qual recebeu o envio. · Fontes: [09:20] Sofia; [09:22] Sofia; [09:44] Sofia

**Entrevistador:** Em que condições o envio muda ou falha?

**Marcos (PM):** O cliente pode receber o mesmo evento duas vezes. Vou deixar isso destacado no portal. · Fontes: [09:26] Marcos

**Larissa (Tech Lead):** A garantia é at-least-once, com o identificador do evento para o cliente fazer a dedup. · Fontes: [09:26] Larissa

**Bruno (Eng. Pleno, Pedidos):** Não tratei disso na reunião.

**Diego (Eng. Sênior, Plataforma):** Se o cliente não responde em 10 segundos, tratamos como falha e marcamos para retry. Em caso de duplicidade, o cliente deduplica pelo identificador do evento. · Fontes: [09:42] Diego; [09:25] Diego

**Sofia (Eng. Segurança):** Se o evento passar de 64KB, não enviamos e geramos erro. Não truncamos. · Fontes: [09:23] Sofia; [09:24] Larissa

### FR-08 Retentar com backoff e mover para DLQ

**Entrevistador:** Qual é o fluxo principal do retry?

**Marcos (PM):** Aceitei a janela de cerca de 15 horas. · Fontes: [09:17] Marcos

**Larissa (Tech Lead):** São 5 tentativas, com backoff de 1m, 5m, 30m, 2h e 12h. Depois disso, o evento vai para a DLQ, que é persistida separada. · Fontes: [09:17] Larissa; [09:48] Larissa

**Bruno (Eng. Pleno, Pedidos):** Propus 3 tentativas e perdi. Ficaram 5. · Fontes: [09:16] Bruno; [09:17] Larissa

**Diego (Eng. Sênior, Plataforma):** Depois de uma falha, tentamos de novo com intervalos crescentes. Ao atingir o teto, a falha é considerada permanente e o evento vai para a DLQ, com o payload, o motivo e o horário da falha. · Fontes: [09:15] Diego; [09:18] Diego

**Sofia (Eng. Segurança):** Não tratei disso na reunião.

**Entrevistador:** Em que condições o retry muda de comportamento?

**Marcos (PM):** Se o cliente ficar 15 horas fora do ar, o problema já é dele. · Fontes: [09:17] Marcos

**Larissa (Tech Lead):** Não tratei disso na reunião.

**Bruno (Eng. Pleno, Pedidos):** Não tratei disso na reunião.

**Diego (Eng. Sênior, Plataforma):** Não há retry indefinido, para o evento não ficar pendurado para sempre. Timeout de 10 segundos conta como falha. · Fontes: [09:15] Diego; [09:42] Diego

**Sofia (Eng. Segurança):** Não tratei disso na reunião.

### FR-09 Replay manual da DLQ pelo admin

**Entrevistador:** Qual é o fluxo principal do replay?

**Marcos (PM):** Não tratei disso na reunião.

**Larissa (Tech Lead):** É um endpoint admin para replay manual, reaproveitando o requireRole que já existe. · Fontes: [09:19] Larissa; [09:36] Larissa; código `src/middlewares/auth.middleware.ts`

**Bruno (Eng. Pleno, Pedidos):** Perguntei quem reprocessa, e o Diego respondeu. · Fontes: [09:18] Bruno

**Diego (Eng. Sênior, Plataforma):** O admin chama o endpoint de replay de um item da DLQ, e o evento volta para a outbox como pendente. · Fontes: [09:18] Diego; [09:35] Diego

**Sofia (Eng. Segurança):** O replay registra quem fez, para auditoria. · Fontes: [09:36] Sofia

**Entrevistador:** Em que condições o replay é bloqueado ou retorna erro?

**Marcos (PM):** Não tratei disso na reunião.

**Larissa (Tech Lead):** Sem role ADMIN, o replay é bloqueado. O requireRole já retorna FORBIDDEN nesse caso. · Fontes: [09:36] Larissa; código `src/middlewares/auth.middleware.ts`

**Bruno (Eng. Pleno, Pedidos):** Não tratei disso na reunião.

**Diego (Eng. Sênior, Plataforma):** Não tratei disso na reunião.

**Sofia (Eng. Segurança):** Operador não pode fazer replay. Mexer na fila de entrega não é tarefa de operador. · Fontes: [09:36] Sofia

### FR-10 Consultar histórico de entregas

**Entrevistador:** Qual é o fluxo principal da consulta?

**Marcos (PM):** O cliente consulta o histórico de um webhook e vê as últimas 100 entregas, com sucesso ou falha, o payload, a resposta e o tempo de resposta. · Fontes: [09:34] Marcos

**Larissa (Tech Lead):** Anotei como requisito. · Fontes: [09:35] Larissa

**Bruno (Eng. Pleno, Pedidos):** Não tratei disso na reunião.

**Diego (Eng. Sênior, Plataforma):** Não tratei disso na reunião.

**Sofia (Eng. Segurança):** Não tratei disso na reunião.

**Entrevistador:** Em que condições a consulta é bloqueada ou retorna erro?

**Marcos (PM):** Não tratei disso na reunião.

**Larissa (Tech Lead):** Exige autenticação, como o resto do CRUD. · Fontes: [09:48] Larissa

**Bruno (Eng. Pleno, Pedidos):** Para webhook inexistente, o código previsto é WEBHOOK_NOT_FOUND. · Fontes: [09:28] Bruno

**Diego (Eng. Sênior, Plataforma):** Não tratei disso na reunião.

**Sofia (Eng. Segurança):** Não tratei disso na reunião.

### FR-11 Rotacionar a secret

**Entrevistador:** Qual é o fluxo principal da rotação?

**Marcos (PM):** Não tratei disso na reunião.

**Larissa (Tech Lead):** Está no resumo: rotação com 24 horas de transição. · Fontes: [09:48] Larissa

**Bruno (Eng. Pleno, Pedidos):** Não tratei disso na reunião.

**Diego (Eng. Sênior, Plataforma):** É importante. Já tivemos cliente que vazou a secret em log. · Fontes: [09:22] Diego

**Sofia (Eng. Segurança):** O cliente pede uma nova secret pela API. A antiga continua válida por 24 horas junto com a nova, e depois disso deixa de valer. · Fontes: [09:21] Sofia; [09:22] Sofia

**Entrevistador:** Em que condições a rotação muda ou falha?

**Marcos (PM):** Não tratei disso na reunião.

**Larissa (Tech Lead):** Não tratei disso na reunião.

**Bruno (Eng. Pleno, Pedidos):** Não tratei disso na reunião.

**Diego (Eng. Sênior, Plataforma):** Não tratei disso na reunião.

**Sofia (Eng. Segurança):** Depois das 24 horas, a secret antiga deixa de valer. Outros erros da rotação não discuti. · Fontes: [09:21] Sofia

### Prioridade

**Entrevistador:** Qual é a prioridade de cada requisito?

**Marcos (PM):** Os que listei como necessidade dos clientes são essenciais: cadastro com filtro de status, CRUD e histórico de entregas. · Fontes: [09:31] Marcos; [09:33] Marcos; [09:34] Marcos

**Larissa (Tech Lead):** Tudo o que entrou no meu resumo final faz parte da entrega: outbox na mesma transação; worker; retry e DLQ; HMAC com rotação; at-least-once; CRUD; replay só para ADMIN. · Fontes: [09:48] Larissa

**Bruno (Eng. Pleno, Pedidos):** Não classifiquei prioridades. Confirmei o resumo. · Fontes: [09:49] Bruno

**Diego (Eng. Sênior, Plataforma):** Não classifiquei prioridades. Confirmei o resumo. · Fontes: [09:49] Diego

**Sofia (Eng. Segurança):** Não classifiquei prioridades. Confirmei o resumo e pedi a revisão de segurança antes do deploy. · Fontes: [09:49] Sofia

### Resumo da etapa 5

- **Clientes (FR-01 a FR-05 e FR-10):** os usuários autenticados que representam os clientes gerenciam webhooks por CRUD, sempre com URL https e secret gerada por nós. O filtro de status é aplicado no registro do evento, e o histórico mostra as últimas 100 entregas.
- **Registro do evento (FR-06):** acontece na mesma transação da mudança de status e guarda a foto do pedido no momento da mudança. Se o registro falhar, a mudança de status não acontece.
- **Envio (FR-07 e FR-08):** o worker separado envia o evento assinado com HMAC-SHA256, com timeout de 10 s e no máximo 64KB. A entrega é at-least-once com identificador de evento. São 5 tentativas com backoff e, ao esgotar, o evento vai para a DLQ.
- **Admin e segurança (FR-09 e FR-11):** o replay da DLQ é só para ADMIN e fica auditado. A secret pode ser rotacionada, com 24 h de convivência entre a antiga e a nova.
- **Prioridade:** alta para todos os 11 requisitos. FR-01 a FR-05 e FR-10 com base nas necessidades listadas pelo Marcos; FR-06 a FR-09 e FR-11 com base no resumo da Larissa às 09:48.

### Divergências e como a reunião resolveu

- **customer_id.** Marcos disse às 09:31 que viria do JWT. Larissa corrigiu às 09:32: vem no body ou no path. No PRD fica como "identificado na requisição"; a escolha entre body e path fica para o FDD.
- **Interpretação (1).** Bruno citou WEBHOOK_SECRET_REQUIRED às 09:28, antes de o Marcos dizer às 09:31 que nós geramos a secret. Por isso esse código não é usado no PRD; só WEBHOOK_NOT_FOUND e WEBHOOK_INVALID_URL.
- **Interpretação (2).** Para evento acima de 64KB, a reunião decidiu "não envia e gera erro". Não ficou definido para onde esse evento vai, se para a DLQ ou só para o log. Registrado como "Não definido na reunião".
- **Lacuna.** Não ficou definido quais respostas do cliente contam como falha, além do timeout e do cliente fora do ar.

### Confirmação do usuário

> Correto

---

## Etapa 6: Requisitos não funcionais

**Entrevistador:** Qual é a performance esperada?

**Marcos (PM):** O aviso precisa chegar em menos de 10 segundos, e o polling de 2 segundos serve. · Fontes: [09:02] Marcos; [09:10] Marcos

**Larissa (Tech Lead):** O polling é de 2 segundos, e aceitamos esse tempo como espera máxima até o evento ser pego. A mudança de status não pode esperar chamada HTTP. Concordei com o Bruno nisso. · Fontes: [09:10] Larissa; [09:04] Larissa

**Bruno (Eng. Pleno, Pedidos):** A transação de mudança de status já é pesada. Com uma chamada HTTP dentro dela, um cliente lento travaria outros pedidos. Também perguntei se o worker ficaria lento com muito evento acumulado. · Fontes: [09:04] Bruno; [09:07] Bruno

**Diego (Eng. Sênior, Plataforma):** Respondi ao Bruno: o worker lê só os pendentes, em lote pequeno, com índice por estado e por data de criação. O timeout por chamada é de 10 segundos, e o payload é enxuto. · Fontes: [09:08] Diego; [09:42] Diego; [09:43] Diego

**Sofia (Eng. Segurança):** Concordei com o timeout de 10 segundos. · Fontes: [09:42] Sofia

**Entrevistador:** Qual é a disponibilidade esperada?

**Marcos (PM):** Não tratei disso na reunião.

**Larissa (Tech Lead):** Não tratei disso na reunião.

**Bruno (Eng. Pleno, Pedidos):** Não tratei de meta de disponibilidade. O que falei é que a indisponibilidade do cliente não pode desfazer a mudança de status. · Fontes: [09:04] Bruno

**Diego (Eng. Sênior, Plataforma):** Não dei número. O worker roda em processo separado para não cair junto quando a API reinicia. · Fontes: [09:11] Diego

**Sofia (Eng. Segurança):** Não tratei disso na reunião.

**Entrevistador:** Quais são os requisitos de segurança e controle de acesso?

**Marcos (PM):** Perguntei se o CRUD podia ser usado por qualquer role autenticada, e a Sofia disse que sim, por enquanto. · Fontes: [09:36] Marcos; [09:37] Sofia

**Larissa (Tech Lead):** O replay exige role ADMIN, usando o requireRole que já existe. · Fontes: [09:36] Larissa; código `src/middlewares/auth.middleware.ts`

**Bruno (Eng. Pleno, Pedidos):** Perguntei o algoritmo do HMAC e a Sofia respondeu SHA-256. · Fontes: [09:20] Bruno; [09:20] Sofia

**Diego (Eng. Sênior, Plataforma):** O horário do envio vai junto, para o cliente conseguir detectar replay attack, se quiser. · Fontes: [09:44] Diego

**Sofia (Eng. Segurança):** HMAC-SHA256 sobre o corpo, com secret única por endpoint. Rotação com 24 horas de convivência entre a secret antiga e a nova. Só https. Limite de 64KB. Replay só para ADMIN, com registro de quem fez. Pelo menos dois dias úteis para eu revisar o código de segurança antes do deploy. · Fontes: [09:22] Sofia; [09:23] Sofia; [09:24] Larissa; [09:36] Sofia; [09:46] Sofia

**Entrevistador:** Quais são os requisitos de observabilidade?

**Marcos (PM):** O cliente acompanha as entregas pelo histórico, que mostra sucesso ou falha, a resposta e o tempo de resposta. · Fontes: [09:34] Marcos

**Larissa (Tech Lead):** Reaproveitamos o logger Pino do projeto. · Fontes: [09:30] Larissa; código `src/shared/logger/index.ts`

**Bruno (Eng. Pleno, Pedidos):** O Pino já está no projeto inteiro e não vamos colocar nada novo. O error middleware já trata AppError, Zod e Prisma. · Fontes: [09:29] Bruno; código `src/middlewares/error.middleware.ts`

**Diego (Eng. Sênior, Plataforma):** A DLQ guarda o payload, o motivo e o horário da falha, e serve de evidência para debug. Rate limiting fica em observação. · Fontes: [09:18] Diego; [09:39] Diego

**Sofia (Eng. Segurança):** O replay precisa ficar registrado em log com quem fez. · Fontes: [09:36] Sofia

**Entrevistador:** Quais são os requisitos de confiabilidade e integridade de dados?

**Marcos (PM):** Aceitei que o cliente pode receber o mesmo evento duas vezes, e vou documentar isso. · Fontes: [09:26] Marcos

**Larissa (Tech Lead):** O registro do evento é atômico com a mudança de status, e a entrega é at-least-once. A ordem só é garantida por pedido e enquanto houver um único worker. Isso ficou como limitação conhecida. · Fontes: [09:48] Larissa; [09:13] Larissa

**Bruno (Eng. Pleno, Pedidos):** Se o registro do evento falhar, tudo volta atrás. Não pode existir status mudado sem evento. · Fontes: [09:40] Bruno

**Diego (Eng. Sênior, Plataforma):** O outbox garante que não haja inconsistência entre o pedido e o evento. São 5 tentativas antes da DLQ, e a ordem fica implícita por pedido com um worker só. · Fontes: [09:06] Diego; [09:12] Diego; [09:15] Diego

**Sofia (Eng. Segurança):** Apontei que o at-least-once joga responsabilidade para o cliente, mas a decisão ficou assim. · Fontes: [09:25] Sofia; [09:26] Larissa

**Entrevistador:** Quais são os requisitos de compatibilidade e portabilidade?

**Marcos (PM):** Os clientes integram pela nossa API, e eu documento no portal como integrar. · Fontes: [09:32] Marcos; [09:40] Marcos

**Larissa (Tech Lead):** O worker vira uma nova entry-point no mesmo projeto, ao lado do src/server.ts. A feature segue os padrões que já temos: módulos, schemas Zod e códigos de erro. · Fontes: [09:11] Larissa; [09:30] Larissa; código `src/server.ts`

**Bruno (Eng. Pleno, Pedidos):** Mesmo banco e mesma DATABASE_URL, mas com outro PrismaClient, porque é outro processo Node. · Fontes: [09:30] Bruno

**Diego (Eng. Sênior, Plataforma):** Mesmo banco e mesma stack. O payload é JSON, com horário em ISO 8601. · Fontes: [09:11] Diego; [09:43] Diego

**Sofia (Eng. Segurança):** HMAC-SHA256 é padrão de mercado e todo cliente sério tem biblioteca para isso. · Fontes: [09:20] Sofia

**Entrevistador:** Existe algum requisito de compliance?

**Marcos (PM):** Não tratei disso na reunião.

**Larissa (Tech Lead):** Não tratei disso na reunião.

**Bruno (Eng. Pleno, Pedidos):** Não tratei disso na reunião.

**Diego (Eng. Sênior, Plataforma):** Não tratei disso na reunião.

**Sofia (Eng. Segurança):** Só a auditoria do replay, que registra quem fez. Não discuti norma ou regulação. · Fontes: [09:36] Sofia

**Entrevistador:** Existe algum requisito de acessibilidade?

**Marcos (PM):** Não tratei disso na reunião.

**Larissa (Tech Lead):** Não tratei disso na reunião. A entrega é só de endpoints, e o painel visual ficou fora. · Fontes: [09:40] Larissa

**Bruno (Eng. Pleno, Pedidos):** Não tratei disso na reunião.

**Diego (Eng. Sênior, Plataforma):** Não tratei disso na reunião.

**Sofia (Eng. Segurança):** Não tratei disso na reunião.

### Resumo da etapa 6

- **Performance:** aviso em menos de 10 s, com polling de 2 s e timeout de 10 s por chamada. O envio fica fora da transação de mudança de status, e o payload é enxuto.
- **Segurança:** HMAC-SHA256 com secret por endpoint e rotação com 24 h de transição; horário do envio junto, para o cliente detectar replay; somente https e limite de 64KB; replay só para ADMIN, com auditoria; revisão de segurança de pelo menos 2 dias úteis antes do deploy.
- **Confiabilidade:** registro atômico do evento, entrega at-least-once com identificador, 5 tentativas antes da DLQ e ordem garantida só por pedido, com um único worker.
- **Observabilidade:** logger Pino e error middleware existentes, histórico de entregas, DLQ como evidência para debug e log do replay.
- **Compatibilidade:** mesmo projeto, banco e stack. O worker é uma nova entry-point em processo separado, e o payload é JSON com horário ISO 8601.
- **Não definido na reunião:** meta de disponibilidade, latência das APIs de configuração, métricas e tracing, compliance além da auditoria do replay e acessibilidade.

### Divergências e como a reunião resolveu

- **At-least-once.** Sofia questionou às 09:25 que o modelo joga responsabilidade para o cliente. Diego defendeu como padrão de mercado, Marcos se comprometeu a documentar, e Larissa fechou às 09:26.
- **Performance do worker.** Bruno perguntou às 09:07 se o worker ficaria lento com acúmulo de eventos. Diego respondeu às 09:08 com a leitura só dos pendentes, em lote pequeno.
- **Interpretação.** Larissa disse às 09:24 que o limite de 64KB "é só requisito não funcional". Por isso ele fica na seção de RNF de segurança, e não como decisão de arquitetura.

### Confirmação do usuário

> Correto

---

## Etapa 7: Arquitetura e abordagem

**Entrevistador:** Onde essa feature vai rodar: no monólito atual, num serviço novo ou em outro formato?

**Marcos (PM):** Não tratei disso na reunião.

**Larissa (Tech Lead):** No mesmo projeto. O webhook vira um módulo igual aos outros, e o worker ganha uma entry-point própria com um script para iniciar. · Fontes: [09:11] Larissa; [09:30] Larissa

**Bruno (Eng. Pleno, Pedidos):** O módulo de webhooks segue o padrão de src/modules, com controller, service, repository, routes e schemas. O worker tem uma entry separada, e a lógica de processamento fica dentro do módulo. · Fontes: [09:27] Bruno; [09:28] Bruno; código `src/modules/orders/order.service.ts`

**Diego (Eng. Sênior, Plataforma):** Mesmo banco e mesma stack, mas o worker não pode ser o mesmo processo da API. · Fontes: [09:11] Diego

**Sofia (Eng. Segurança):** Não tratei disso na reunião.

**Entrevistador:** A comunicação com o cliente será síncrona ou assíncrona?

**Marcos (PM):** Não tratei disso na reunião.

**Larissa (Tech Lead):** Assíncrona. Levantei a dúvida e concordei com o Bruno que síncrono não serve. · Fontes: [09:03] Larissa; [09:04] Larissa

**Bruno (Eng. Pleno, Pedidos):** Síncrono não serve. Um cliente lento travaria outras mudanças de status, e não dá para desfazer a mudança de status porque o cliente está fora do ar. · Fontes: [09:04] Bruno

**Diego (Eng. Sênior, Plataforma):** Síncrono está fora de questão. · Fontes: [09:06] Diego

**Sofia (Eng. Segurança):** Não tratei disso na reunião.

**Entrevistador:** Vocês vão usar fila, mensageria ou streaming?

**Marcos (PM):** Não tratei disso na reunião.

**Larissa (Tech Lead):** A alternativa seria Redis Streams ou algo parecido, mas exigiria mais infraestrutura. Fechamos outbox no MySQL. · Fontes: [09:07] Larissa; [09:08] Larissa

**Bruno (Eng. Pleno, Pedidos):** Sugeri trigger no banco para ser mais reativo, mas ficou o polling. · Fontes: [09:09] Bruno; [09:10] Larissa

**Diego (Eng. Sênior, Plataforma):** Eu nem chamaria de fila: é padrão outbox. Subir um Redis Cluster seria overengineering para um time pequeno. O MySQL não tem listener nativo, por isso o worker lê a tabela por polling. · Fontes: [09:06] Diego; [09:07] Diego; [09:09] Diego

**Sofia (Eng. Segurança):** Não tratei disso na reunião.

**Entrevistador:** Vocês vão usar cache?

**Marcos (PM):** Não tratei disso na reunião.

**Larissa (Tech Lead):** Não tratei disso na reunião.

**Bruno (Eng. Pleno, Pedidos):** Não tratei disso na reunião.

**Diego (Eng. Sênior, Plataforma):** Não tratei disso na reunião.

**Sofia (Eng. Segurança):** Não tratei disso na reunião.

**Entrevistador:** Quais integrações externas são necessárias?

**Marcos (PM):** Os endpoints dos clientes, que recebem os eventos. Além disso, o portal de desenvolvedor, onde documento a integração. Se o cliente quiser os detalhes do pedido, ele consulta o GET /orders/:id. · Fontes: [09:02] Marcos; [09:40] Marcos; [09:43] Diego

**Larissa (Tech Lead):** Não tratei disso na reunião.

**Bruno (Eng. Pleno, Pedidos):** Externa, não tratei. Internamente, a integração crítica é com o changeStatus do serviço de pedidos. · Fontes: [09:40] Bruno; código `src/modules/orders/order.service.ts`

**Diego (Eng. Sênior, Plataforma):** As chamadas HTTP do worker para o endpoint de cada cliente. · Fontes: [09:06] Diego

**Sofia (Eng. Segurança):** Os endpoints dos clientes ficam fora da nossa infraestrutura e precisam ser https. · Fontes: [09:19] Sofia; [09:23] Sofia

**Entrevistador:** Quais são os componentes principais?

**Marcos (PM):** Não tratei disso na reunião.

**Larissa (Tech Lead):** Outbox; worker; DLQ separada; endpoints de configuração; endpoint de replay. · Fontes: [09:48] Larissa

**Bruno (Eng. Pleno, Pedidos):** O módulo de webhooks; a entry do worker; o processador dentro do módulo; uma função de publicação do evento que recebe a transação atual e é chamada pelo serviço de pedidos; o cadastro de configuração. · Fontes: [09:27] Bruno; [09:28] Bruno; [09:41] Bruno; [09:21] Bruno

**Diego (Eng. Sênior, Plataforma):** A outbox; o worker; a DLQ; o endpoint admin de replay. · Fontes: [09:06] Diego; [09:18] Diego

**Sofia (Eng. Segurança):** O cadastro de configuração, com uma secret por endpoint. · Fontes: [09:21] Sofia

### Resumo da etapa 7

- **Onde roda:** dentro do sistema existente. O webhook é um novo módulo no padrão atual, e o worker roda como processo separado, no mesmo projeto, banco e stack.
- **Comunicação:** assíncrona. A mudança de status só registra o evento, e o worker entrega depois por HTTP.
- **Mecanismo:** padrão outbox no MySQL, lido por polling. Não haverá fila externa nem streaming.
- **Componentes:** cadastro de webhooks, outbox de eventos, worker de envio, DLQ, endpoints de configuração, histórico e replay.
- **Integrações:** externas, os endpoints https dos clientes e o portal de desenvolvedor. Interna, o fluxo de mudança de status do pedido.
- **Cache:** não definido na reunião.

### Divergências e como a reunião resolveu

- **Fila ou outbox.** Larissa falou em "fila/outbox" às 09:03, e Diego reformulou como padrão outbox às 09:06. A decisão ficou às 09:08.
- **Mesmo processo ou separado.** Diego exigiu processo separado às 09:11, e Larissa e Bruno concordaram.
- **Trigger ou polling.** Ficou o polling, como já registrado nas etapas anteriores.
- **Interpretação.** O portal de desenvolvedor entra como integração e dependência organizacional, e não como componente técnico. Ele aparece na reunião só como compromisso do Marcos de documentar.

### Confirmação do usuário

> Correto

---

## Etapa 8: Decisões e trade-offs

**Entrevistador:** Que decisões de arquitetura já foram assumidas?

**Marcos (PM):** As que eu acompanhei: polling de 2 segundos; 5 tentativas com uma janela de cerca de 15 horas; at-least-once com identificador de evento; e-mail de alerta fora desta fase; painel fora do escopo. · Fontes: [09:10] Marcos; [09:17] Marcos; [09:26] Marcos; [09:38] Marcos

**Larissa (Tech Lead):** Outbox no MySQL; worker em polling de 2 segundos; 5 tentativas com backoff de 1m, 5m, 30m, 2h e 12h; at-least-once com identificador de evento; reuso máximo dos padrões do projeto; replay só para ADMIN. Também registrei a ordem garantida só por pedido como limitação conhecida. Depois das 09:50, decidimos guardar a foto do pedido no evento. · Fontes: [09:08] Larissa; [09:10] Larissa; [09:17] Larissa; [09:26] Larissa; [09:30] Larissa; [09:36] Larissa; [09:13] Larissa; [09:52] Larissa

**Bruno (Eng. Pleno, Pedidos):** O evento é registrado dentro da transação do changeStatus; o filtro de status é aplicado no registro; o módulo segue o padrão de src/modules, com códigos de erro WEBHOOK_; o worker usa um PrismaClient próprio; o evento guarda a foto do pedido. · Fontes: [09:40] Bruno; [09:34] Bruno; [09:27] Bruno; [09:29] Larissa; [09:30] Bruno; [09:52] Bruno

**Diego (Eng. Sênior, Plataforma):** Outbox; worker em processo separado e em polling; 5 tentativas e DLQ em tabela separada; replay manual; at-least-once; timeout de 10 segundos. · Fontes: [09:06] Diego; [09:09] Diego; [09:11] Diego; [09:15] Diego; [09:18] Diego; [09:24] Diego; [09:42] Diego

**Sofia (Eng. Segurança):** HMAC-SHA256 sobre o corpo; secret por endpoint; rotação com 24 horas de transição; replay só para ADMIN, com auditoria. · Fontes: [09:22] Sofia; [09:36] Sofia

**Entrevistador:** Por que cada uma dessas decisões foi tomada?

**Marcos (PM):** O polling de 2 segundos atende o "abaixo de 10 segundos" dos clientes. Um cliente fora do ar por 15 horas já tem um problema sério dele. O at-least-once eu consigo documentar bem no portal. · Fontes: [09:10] Marcos; [09:17] Marcos; [09:26] Marcos

**Larissa (Tech Lead):** Outbox no MySQL porque Redis Streams exigiria subir mais infraestrutura. Reuso dos padrões para o webhook ficar igual aos outros módulos. Foto do pedido no evento para ele refletir o estado de quando o status mudou. · Fontes: [09:07] Larissa; [09:30] Larissa; [09:52] Larissa

**Bruno (Eng. Pleno, Pedidos):** Dentro da transação, para não existir status mudado sem evento. Filtro no registro, para economizar linhas. PrismaClient próprio porque ele é por processo. · Fontes: [09:40] Bruno; [09:34] Bruno; [09:30] Bruno

**Diego (Eng. Sênior, Plataforma):** Outbox: garante consistência com a transação principal. Polling: o MySQL não tem listener nativo. Processo separado: um reinício da API não derruba o worker. 5 tentativas: cobrem indisponibilidades longas, e já tivemos cliente duas horas fora. Teto de tentativas: evita evento pendurado para sempre. DLQ separada: deixa a leitura da outbox mais limpa e serve de evidência. At-least-once: é o padrão de mercado, porque exactly-once exige coordenação dos dois lados. · Fontes: [09:06] Diego; [09:09] Diego; [09:11] Diego; [09:15] Diego; [09:16] Diego; [09:18] Diego; [09:25] Diego

**Sofia (Eng. Segurança):** O HMAC permite ao cliente validar a origem e a integridade do envio. A secret por endpoint evita que o vazamento de uma exponha todas. A rotação com 24 horas dá tempo para o cliente migrar. Replay é coisa de admin, não de operador. · Fontes: [09:19] Sofia; [09:21] Sofia; [09:36] Sofia

**Entrevistador:** Qual é o trade-off de cada decisão?

**Marcos (PM):** Os clientes nunca pediram ordem global, então essa limitação não me preocupa. · Fontes: [09:14] Marcos

**Larissa (Tech Lead):** No polling, a espera chega a 2 segundos no pior caso, e aceitamos. A ordem só vale por pedido e com um único worker. Não registrei trade-off para o reuso de padrões nem para a foto do pedido. · Fontes: [09:10] Larissa; [09:13] Larissa

**Bruno (Eng. Pleno, Pedidos):** A integração obriga o serviço de pedidos a chamar uma função do módulo de webhooks, passando a transação. Escolhemos uma função em vez de injetar um repository inteiro. · Fontes: [09:41] Bruno; [09:41] Diego

**Diego (Eng. Sênior, Plataforma):** Com um worker só não dá para escalar em paralelo sem perder a ordem; no futuro, isso se resolve com partição por pedido ou lock. A tabela da outbox cresce, e o arquivamento ficou fora desta feature. Com o at-least-once, o cliente pode receber duplicado. · Fontes: [09:12] Diego; [09:13] Diego; [09:08] Diego; [09:24] Diego

**Sofia (Eng. Segurança):** O at-least-once joga responsabilidade para o cliente. O CRUD fica aberto a qualquer role autenticada por enquanto, e pode ser endurecido mais adiante. · Fontes: [09:25] Sofia; [09:37] Sofia

### Resumo da etapa 8

| Decisão | Justificativa | Trade-off |
| --- | --- | --- |
| **D1** Outbox no MySQL existente | Consistência atômica com a mudança de status, sem nova infraestrutura | A tabela cresce, e o arquivamento ficou fora de escopo |
| **D2** Worker em processo separado, em polling de 2 s | O MySQL não notifica processo externo; um reinício da API não para o envio | Até 2 s de espera; ordem só por pedido e com um único worker |
| **D3** 5 tentativas com backoff (1m/5m/30m/2h/12h) e DLQ separada | Cobre indisponibilidades longas (caso real de 2 h) sem pendurar evento | Depois de cerca de 15 h, o evento depende de replay manual |
| **D4** HMAC-SHA256 com secret por endpoint e rotação de 24 h | Origem e integridade verificáveis; vazamento isolado (caso real) | O cliente precisa implementar a verificação |
| **D5** At-least-once com identificador de evento | Padrão de mercado; exactly-once exige coordenação dos dois lados | O cliente precisa deduplicar |
| **D6** Reuso dos padrões do projeto | O webhook fica igual aos outros módulos | Não definido na reunião |
| **D7** Replay só para ADMIN, CRUD para qualquer role autenticada | Replay não é tarefa de operador | O CRUD fica menos restrito por enquanto |

### Divergências e como a reunião resolveu

- **At-least-once.** Sofia questionou a responsabilidade do cliente às 09:25, e Larissa fechou às 09:26. Sofia reconhece a decisão.
- **3 ou 5 tentativas.** Bruno perdeu para 5 às 09:17 e confirmou o resumo às 09:49.
- **Interpretação (1).** O filtro no registro e a foto do pedido ficam nos requisitos funcionais (FR-05 e FR-06), e não como decisões de produto separadas. A justificativa delas está na reunião, mas o trade-off não, e o nível de detalhe é de FDD.
- **Interpretação (2).** O trade-off da D3 ("depois de cerca de 15 h, depende de replay manual") é uma consequência direta do que Diego descreveu às 09:15 e 09:18. Ninguém disse isso com essas palavras.

### Confirmação do usuário

> Correto

---

## Etapa 9: Dependências

**Entrevistador:** Existe algo que precisa vir de outro time ou de outra área para essa feature funcionar?

**Marcos (PM):** Da minha parte, três coisas: documentar no portal de desenvolvedor como integrar, com destaque para o fato de que o mesmo evento pode chegar duas vezes; confirmar o prazo com a Atlas; atualizar os clientes. · Fontes: [09:26] Marcos; [09:40] Marcos; [09:47] Marcos; [09:49] Marcos

**Larissa (Tech Lead):** A revisão de segurança da Sofia entra no fim das três sprints. Antes de codar, abro o doc de design e marco uma sessão para o Bruno e o Diego revisarem comigo. O painel visual é do time de frontend, mas está fora desta entrega e não é dependência. · Fontes: [09:47] Larissa; [09:50] Larissa; [09:40] Larissa

**Bruno (Eng. Pleno, Pedidos):** Não tratei disso na reunião.

**Diego (Eng. Sênior, Plataforma):** Não tratei disso na reunião.

**Sofia (Eng. Segurança):** Pedi pelo menos dois dias úteis para revisar o código de segurança antes do deploy, principalmente o HMAC e a geração de secret. Precisam me agendar antes de subir. · Fontes: [09:46] Sofia; [09:49] Sofia

**Entrevistador:** Existe algo técnico que precisa estar pronto antes?

**Marcos (PM):** Não tratei disso na reunião.

**Larissa (Tech Lead):** Na minha estimativa, a ordem das etapas é: (1) modelagem da outbox e da DLQ, uma sprint; (2) worker e retry, uma sprint; (3) CRUD e histórico de entregas; (4) integração com o serviço de pedidos e testes ponta a ponta. O worker precisa de uma entry-point e de um script próprios para rodar. · Fontes: [09:46] Larissa; [09:11] Larissa

**Bruno (Eng. Pleno, Pedidos):** O changeStatus precisa ser estendido para registrar o evento dentro da transação atual. O worker usa o mesmo banco e a mesma DATABASE_URL, com uma instância própria do PrismaClient. · Fontes: [09:40] Bruno; [09:41] Bruno; [09:30] Bruno; código `src/modules/orders/order.service.ts`

**Diego (Eng. Sênior, Plataforma):** O worker precisa rodar como processo separado, conectado ao mesmo banco. · Fontes: [09:11] Diego

**Sofia (Eng. Segurança):** Não tratei disso na reunião.

**Entrevistador:** O que os clientes precisam fazer do lado deles para a feature funcionar?

**Marcos (PM):** Integrar pela nossa API, seguindo o que eu documentar no portal. · Fontes: [09:32] Marcos; [09:40] Marcos

**Larissa (Tech Lead):** Deduplicar os eventos pelo identificador, porque a garantia é at-least-once. · Fontes: [09:26] Larissa

**Bruno (Eng. Pleno, Pedidos):** Não tratei disso na reunião.

**Diego (Eng. Sênior, Plataforma):** Estar preparados para receber o mesmo evento duas vezes e deduplicar pelo identificador do evento. · Fontes: [09:24] Diego; [09:25] Diego

**Sofia (Eng. Segurança):** Expor um endpoint https e verificar a assinatura HMAC do lado deles. Quando rotacionarem a secret, migrar os sistemas dentro das 24 horas. · Fontes: [09:20] Sofia; [09:21] Sofia; [09:23] Sofia

### Resumo da etapa 9

Organizacionais:
- Revisão de segurança da Sofia, com pelo menos 2 dias úteis antes do deploy.
- Documentação no portal de desenvolvedor, feita pelo Marcos.
- Confirmação do prazo com a Atlas, feita pelo Marcos.
- Sessão de revisão do doc de design com Larissa, Bruno e Diego antes de começar a codar.

Técnicas:
- Extensão da transação do changeStatus.
- Worker com entry-point própria, rodando em processo separado e conectado ao mesmo banco com PrismaClient próprio.

Externas (clientes):
- Endpoint https.
- Verificação da assinatura HMAC.
- Deduplicação pelo identificador do evento.
- Migração para a nova secret dentro das 24 h após a rotação.

### Divergências e como a reunião resolveu

- Não houve divergência nesta etapa.
- **Interpretação (1).** A ordem das sprints que a Larissa deu às 09:46 é uma estimativa de esforço, não uma dependência formal. Dela entra só a parte técnica (a modelagem vem antes do worker), e o cronograma fica fora do PRD.
- **Interpretação (2).** O painel do time de frontend não entra como dependência, porque está fora de escopo.

### Confirmação do usuário

> Correto

---

## Etapa 10: Riscos e mitigação

**Entrevistador:** Quais são os principais riscos da feature?

**Marcos (PM):** O comercial: se não entregarmos no prazo, a Atlas pode migrar para o concorrente. · Fontes: [09:00] Marcos

**Larissa (Tech Lead):** Para mim, o risco é escalar para vários workers e perder a ordem. Por isso deixei isso registrado como limitação conhecida. · Fontes: [09:13] Larissa

**Bruno (Eng. Pleno, Pedidos):** Vejo dois: uma chamada síncrona travar a mudança de status de outros pedidos, mas isso a arquitetura já resolve; o status mudar e o evento não sair, se o registro ficar fora da transação. · Fontes: [09:04] Bruno; [09:40] Bruno

**Diego (Eng. Sênior, Plataforma):** Vejo quatro: cliente fora do ar por muito tempo; vazamento de secret pelo cliente; entrega duplicada por causa do at-least-once; bombardear o cliente com muitas chamadas quando vários pedidos mudam ao mesmo tempo. · Fontes: [09:16] Diego; [09:22] Diego; [09:24] Diego; [09:38] Diego

**Sofia (Eng. Segurança):** Vejo dois: adulteração ou falsificação do envio no caminho até o cliente; vazamento de secret, que é ainda pior se a secret for global. · Fontes: [09:19] Sofia; [09:21] Sofia

**Entrevistador:** Qual é a probabilidade de cada risco?

**Marcos (PM):** Não dei número. Mas a Atlas já sinalizou que pode migrar, e fiquei confiante com a estimativa de três sprints. Disse que a Atlas vai gostar. · Fontes: [09:00] Marcos; [09:47] Marcos

**Larissa (Tech Lead):** A perda de ordem só acontece se escalarmos, e o single-worker é a decisão atual. · Fontes: [09:12] Diego; [09:13] Larissa

**Bruno (Eng. Pleno, Pedidos):** Não tratei disso na reunião.

**Diego (Eng. Sênior, Plataforma):** Cliente fora do ar: já aconteceu, com um cliente que ficou duas horas fora numa manutenção planejada. Vazamento de secret: também já aconteceu, uma vez. Duplicidade: pode acontecer. Rajada de chamadas: ainda não sei e vamos observar. Vários workers: é problema do futuro, não de agora. · Fontes: [09:16] Diego; [09:22] Diego; [09:24] Diego; [09:39] Diego; [09:13] Diego

**Sofia (Eng. Segurança):** Não dei probabilidade. Usei o vazamento que o Diego contou como base para a decisão. · Fontes: [09:22] Diego; [09:22] Sofia

**Entrevistador:** Qual é o impacto esperado de cada risco?

**Marcos (PM):** Perder um cliente B2B para o concorrente. Um cliente 15 horas fora do ar já tem um problema sério dele. A falta de ordem global não afeta os clientes, porque eles só querem saber se cada pedido mudou. · Fontes: [09:00] Marcos; [09:17] Marcos; [09:14] Marcos

**Larissa (Tech Lead):** Não tratei disso na reunião.

**Bruno (Eng. Pleno, Pedidos):** Se o evento ficar fora da transação, o status muda e o cliente nunca é avisado. Uma chamada síncrona travaria a mudança de status de outros pedidos. · Fontes: [09:40] Bruno; [09:04] Bruno

**Diego (Eng. Sênior, Plataforma):** Fora da transação, perdemos a garantia toda. Na duplicidade, o cliente recebe o mesmo evento duas vezes. Na rajada, são 50 chamadas em um minuto para o mesmo cliente. Com vários workers, o cliente recebe status fora de ordem. · Fontes: [09:41] Diego; [09:24] Diego; [09:38] Diego; [09:12] Diego

**Sofia (Eng. Segurança):** Com uma secret global, se uma vaza, vaza tudo. Com secret por endpoint, o impacto fica restrito àquele endpoint. · Fontes: [09:21] Sofia

**Entrevistador:** Qual é a mitigação para cada risco?

**Marcos (PM):** Prazo: confirmo o prazo com a Atlas e atualizo os clientes. Duplicidade: documento com destaque no portal. · Fontes: [09:47] Marcos; [09:49] Marcos; [09:26] Marcos

**Larissa (Tech Lead):** Prazo: escopo enxuto, com e-mail, painel e rate limiting fora, e estimativa de três sprints com a revisão incluída. Ordem: single-worker documentado como limitação. · Fontes: [09:37] Larissa; [09:39] Larissa; [09:40] Larissa; [09:47] Larissa; [09:13] Larissa

**Bruno (Eng. Pleno, Pedidos):** O registro do evento fica dentro da transação, com rollback se falhar. Na minha estimativa, a integração com o serviço de pedidos entra junto com os testes ponta a ponta. · Fontes: [09:40] Bruno; [09:46] Larissa

**Diego (Eng. Sênior, Plataforma):** Cliente fora do ar: 5 tentativas com backoff ao longo de cerca de 15 horas, timeout de 10 segundos e DLQ com payload e motivo da falha. Duplicidade: identificador único do evento para dedup. Rajada: observar. · Fontes: [09:17] Diego; [09:42] Diego; [09:18] Diego; [09:25] Diego; [09:39] Diego

**Sofia (Eng. Segurança):** Adulteração: HMAC-SHA256 sobre o corpo e https obrigatório. Vazamento: secret por endpoint. Para tudo isso: revisão de segurança antes do deploy. · Fontes: [09:20] Sofia; [09:23] Sofia; [09:21] Sofia; [09:46] Sofia

**Entrevistador:** Qual é o plano de contingência se o risco acontecer?

**Marcos (PM):** Não tratei disso na reunião.

**Larissa (Tech Lead):** Não tratei disso na reunião.

**Bruno (Eng. Pleno, Pedidos):** Não tratei disso na reunião.

**Diego (Eng. Sênior, Plataforma):** Cliente fora do ar além da janela: replay manual da DLQ pelo admin. Necessidade de escalar: partição por pedido ou lock pessimista. Rajada: implementar rate limiting se virar problema. · Fontes: [09:18] Diego; [09:13] Diego; [09:39] Diego

**Sofia (Eng. Segurança):** Se a secret vazar, o cliente rotaciona pela API. A antiga vale por mais 24 horas e depois deixa de valer. · Fontes: [09:21] Sofia

### Resumo da etapa 10

| Risco | Probabilidade | Mitigação principal | Contingência |
| --- | --- | --- | --- |
| **R1** Cliente fora do ar por muito tempo | Alta (já aconteceu, 2 h) | 5 tentativas com backoff, timeout de 10 s, DLQ | Replay manual pelo ADMIN |
| **R2** Vazamento de secret pelo cliente | Média (já aconteceu uma vez) | Secret por endpoint, HMAC, revisão de segurança | Rotação com 24 h de transição |
| **R3** Status muda e o evento não sai | Não definida na reunião | Registro na mesma transação com rollback, testes ponta a ponta | Não definido na reunião |
| **R4** Entrega duplicada | Não definida na reunião | Identificador do evento, documentação no portal | Não definido na reunião |
| **R5** Rajada de chamadas para o mesmo cliente | Não definida na reunião | Observar | Implementar rate limiting se virar problema |
| **R6** Perda de ordem ao escalar workers | Baixa (single-worker é a decisão atual) | Single-worker documentado como limitação | Partição por pedido ou lock pessimista |
| **R7** Perda da Atlas por atraso | Não definida na reunião | Escopo enxuto, estimativa de 3 sprints, confirmação do prazo | Não definido na reunião |

### Divergências e como a reunião resolveu

- Não houve divergência nesta etapa.
- **Interpretação (1).** Para R2, "já aconteceu uma vez" virou probabilidade média, e não alta, porque foi um único caso relatado. Para R1, ficou alta, porque a indisponibilidade de cliente aparece como algo recorrente ("já tinha cliente nosso").
- **Interpretação (2).** O risco de adulteração que a Sofia levantou foi juntado ao R2, porque as mitigações são as mesmas. O risco de chamada síncrona do Bruno não virou item, porque a arquitetura já o elimina.
- **Interpretação (3).** Em R7, a confiança do Marcos às 09:47 não é uma avaliação de probabilidade. Por isso ficou como "Não definida".

### Confirmação do usuário

> Correto

---

## Etapa 11: Critérios de aceitação

**Entrevistador:** Que condições objetivas precisam ser verdadeiras para considerar a feature pronta?

**Marcos (PM):** O cliente cadastra o webhook, recebe a secret na criação e escolhe os status que quer ouvir. Ele consegue ver as últimas 100 entregas, com sucesso ou falha, payload, resposta e tempo de resposta. O aviso chega em menos de 10 segundos. A documentação no portal deixa claro que o evento pode chegar duas vezes. · Fontes: [09:31] Marcos; [09:33] Marcos; [09:34] Marcos; [09:02] Marcos; [09:26] Marcos

**Larissa (Tech Lead):** O worker pega os eventos a cada 2 segundos. São 5 tentativas com backoff de 1m, 5m, 30m, 2h e 12h, e depois DLQ. O replay só funciona com role ADMIN. Os erros do módulo têm prefixo WEBHOOK_. · Fontes: [09:10] Larissa; [09:17] Larissa; [09:36] Larissa; [09:29] Larissa

**Bruno (Eng. Pleno, Pedidos):** Toda mudança de status com webhook interessado registra o evento na mesma transação; se o registro falhar, o status não muda. Sem webhook interessado, nenhum evento é registrado. O webhook inexistente retorna WEBHOOK_NOT_FOUND. · Fontes: [09:40] Bruno; [09:34] Bruno; [09:28] Bruno

**Diego (Eng. Sênior, Plataforma):** O worker roda em processo separado, então um reinício da API não para o envio. Uma chamada sem resposta em 10 segundos conta como falha e vai para retry. Ao esgotar as tentativas, o evento vai para a DLQ com payload, motivo e horário, e o replay o devolve como pendente. Cada envio leva o identificador do evento, o identificador do cadastro e o horário do envio. · Fontes: [09:11] Diego; [09:42] Diego; [09:18] Diego; [09:25] Diego; [09:44] Diego; [09:45] Diego

**Sofia (Eng. Segurança):** Cada envio é assinado com HMAC-SHA256 sobre o corpo, usando a secret única do endpoint. Uma URL http é recusada com erro de validação. Um evento acima de 64KB não é enviado e gera erro. Depois de uma rotação, a secret antiga vale por 24 horas e depois deixa de valer. O replay fica registrado com quem fez. · Fontes: [09:22] Sofia; [09:23] Sofia; [09:24] Larissa; [09:21] Sofia; [09:36] Sofia

**Entrevistador:** Existe alguma condição que bloqueia o deploy mesmo com tudo funcionando?

**Marcos (PM):** Não tratei disso na reunião.

**Larissa (Tech Lead):** A revisão da Sofia está incluída no fim das três sprints. · Fontes: [09:47] Larissa

**Bruno (Eng. Pleno, Pedidos):** Não tratei disso na reunião.

**Diego (Eng. Sênior, Plataforma):** Não tratei disso na reunião.

**Sofia (Eng. Segurança):** Sim: a revisão de segurança, com pelo menos dois dias úteis, antes do deploy. Não esqueçam de me agendar. · Fontes: [09:46] Sofia; [09:49] Sofia

### Resumo da etapa 11

Checklist proposto, todos com fonte:

1. Mudança de status com webhook interessado registra o evento na mesma transação. Se o registro falhar, a mudança é desfeita.
2. Status sem webhook interessado não gera evento.
3. O worker roda em processo separado e pega os pendentes a cada 2 s. A primeira tentativa acontece em menos de 10 s.
4. Todo envio é assinado com HMAC-SHA256 com a secret do endpoint e leva o identificador do evento, o identificador do cadastro e o horário do envio.
5. URL http é recusada com erro de validação.
6. Evento acima de 64KB não é enviado e gera erro.
7. Sem resposta em 10 s, a tentativa conta como falha e o evento vai para retry.
8. Depois de 5 tentativas (1m/5m/30m/2h/12h), o evento vai para a DLQ com payload, motivo e horário.
9. O replay só aceita ADMIN, registra quem fez e devolve o evento como pendente.
10. Depois da rotação, a secret antiga vale por 24 h e depois deixa de valer.
11. A secret é gerada pelo sistema, devolvida na criação e única por endpoint.
12. O histórico mostra as últimas 100 entregas, com sucesso ou falha, payload, resposta e tempo de resposta.
13. Os erros do módulo usam o prefixo WEBHOOK_, e webhook inexistente retorna WEBHOOK_NOT_FOUND.
14. A revisão de segurança, de pelo menos 2 dias úteis, foi concluída antes do deploy.
15. A documentação no portal destaca a entrega at-least-once e a dedup pelo identificador do evento.

### Divergências e como a reunião resolveu

- Não houve divergência nesta etapa.
- **Interpretação (1).** O critério 3 junta duas metas da reunião: o polling de 2 s (Larissa, 09:10) e o "abaixo de 10 s" (Marcos, 09:02). Ele é medido até a primeira tentativa, como combinado na etapa 3.
- **Interpretação (2).** O critério 15 é uma entrega do Marcos, e não do código. Mantido porque ele se comprometeu com isso na reunião.

### Confirmação do usuário

> Correto

---

## Etapa 12: Testes e validação

**Entrevistador:** Quais tipos de teste são obrigatórios?

**Marcos (PM):** Não tratei disso na reunião.

**Larissa (Tech Lead):** Na estimativa, a integração com o serviço de pedidos entra junto com os testes ponta a ponta. A revisão de segurança da Sofia fica no fim. · Fontes: [09:46] Larissa; [09:47] Larissa

**Bruno (Eng. Pleno, Pedidos):** Não tratei disso na reunião.

**Diego (Eng. Sênior, Plataforma):** Não tratei disso na reunião.

**Sofia (Eng. Segurança):** Revisão do código de segurança, com atenção ao HMAC e à geração de secret. · Fontes: [09:46] Sofia

**Entrevistador:** Qual será a abordagem de validação?

**Marcos (PM):** Não tratei disso na reunião.

**Larissa (Tech Lead):** Antes de codar, abro o doc de design e faço uma sessão de revisão com o Bruno e o Diego. Antes do deploy, entra a revisão da Sofia. · Fontes: [09:50] Larissa; [09:47] Larissa

**Bruno (Eng. Pleno, Pedidos):** Não tratei disso na reunião.

**Diego (Eng. Sênior, Plataforma):** Não tratei disso na reunião.

**Sofia (Eng. Segurança):** Revisão manual de pelo menos dois dias úteis antes do deploy. · Fontes: [09:46] Sofia

**Nota do entrevistador sobre o código.** O projeto já tem uma suíte de testes de integração com Vitest e Supertest, rodando contra o banco. Ela cobre as transições de status, o débito e a reposição de estoque e as transições inválidas (`tests/orders.test.ts`, `package.json`). Como o changeStatus vai ser estendido, essa suíte é a referência natural para checar que nada quebrou.

### Resumo da etapa 12

- **Tipos obrigatórios com fonte:** testes ponta a ponta cobrindo a integração com o serviço de pedidos (Larissa, 09:46); revisão de código de segurança, com foco em HMAC e geração de secret (Sofia, 09:46).
- **Abordagem de validação:** revisão do doc de design com Larissa, Bruno e Diego antes de codar (09:50); revisão de segurança de pelo menos 2 dias úteis antes do deploy, já incluída nas três sprints (09:46 e 09:47).
- **Base no código:** a suíte de integração existente de pedidos (`tests/orders.test.ts`) serve de regressão para o changeStatus estendido.
- **Não definido na reunião:** testes unitários, teste de carga e uso de TDD ou de QA manual guiado por roteiro.

### Divergências e como a reunião resolveu

- Não houve divergência nesta etapa.
- **Interpretação.** A regressão da suíte existente não foi dita na reunião. Ela vem do código, junto com a fala do Bruno às 09:40 de que a transação atual continua fazendo o update do pedido, o histórico e o estoque. No PRD, entra com fonte `CODIGO`.

### Confirmação do usuário

> Correto
