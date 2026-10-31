### PRD: Order Management System | Sistema de Webhooks de Notificação de Pedidos

Versão: 1.0
Data: 2026-10-31
Responsável: Marcos (Product Manager)

---

### Resumo e contexto

- `PRD-CTX-01` A feature avisa os clientes B2B, em menos de 10 segundos, sempre que o status de um pedido deles muda na plataforma. Assim eles não precisam mais consultar a API de tempos em tempos para descobrir mudanças.
- `PRD-CTX-02` A notificação é só de saída: a plataforma envia para o cliente. Os clientes não enviam webhooks para a plataforma.

Onde essa feature será implantada
- `PRD-CTX-03` No sistema existente (Order Management System), como um novo módulo que segue o mesmo padrão dos módulos atuais.
- `PRD-CTX-04` Usa o mesmo banco MySQL e a mesma stack da API. O envio das notificações roda em um processo separado da API.
- `PRD-CTX-05` O ponto de partida é a mudança de status do pedido, que hoje já atualiza o pedido, grava o histórico de status e ajusta o estoque numa única transação.
- `PRD-CTX-06` O ciclo de vida do pedido segue a máquina de estados atual: PENDING, PAID, PROCESSING, SHIPPED, DELIVERED e CANCELLED.

---

### Problema e motivação

Problemas priorizados
- `PRD-PROB-01` Os clientes descobrem mudanças de status consultando o GET /orders de tempos em tempos. Impacto: a integração fica lenta e cara para eles, e três clientes (Atlas Comercial, MaxDistribuição e Nova Cargo) pediram notificação formalmente. Prioridade: alta.
- `PRD-PROB-02` Há risco de perder cliente. Impacto: a Atlas avisou que pode migrar para o concorrente se a entrega não sair no prazo. Prioridade: alta.
- `PRD-PROB-03` Enviar dados de pedidos para endpoints fora da infraestrutura da plataforma exige que o cliente consiga confirmar a origem e a integridade de cada envio. Impacto: sem isso, um envio falso ou adulterado não seria detectado pelo cliente. Prioridade: alta.

---

### Público-alvo e cenários de uso

Público-alvo
- `PRD-PUB-01` Clientes B2B que integram com a API da plataforma, começando por Atlas Comercial, MaxDistribuição e Nova Cargo.
- `PRD-PUB-02` Usuários do sistema que representam o cliente e acessam a API autenticados com o JWT da plataforma.
- `PRD-PUB-03` Desenvolvedores dos clientes, que vão integrar seguindo a documentação do portal de desenvolvedor.
- `PRD-PUB-04` Administradores da plataforma, que reprocessam eventos com falha permanente.

Cenários de uso chave
- `PRD-PUB-05` O cliente cadastra um webhook e escolhe quais status quer receber, por exemplo só SHIPPED e DELIVERED.
- `PRD-PUB-06` O cliente recebe a notificação de mudança de status e confere a assinatura para ter certeza de que ela veio da plataforma.
- `PRD-PUB-07` O cliente consulta o histórico de entregas de um webhook para ver o que deu certo e o que falhou.
- `PRD-PUB-08` O cliente pede uma secret nova para trocar a atual, por exemplo depois de um vazamento.
- `PRD-PUB-09` O administrador reprocessa um evento que esgotou as tentativas de entrega.
- `PRD-PUB-10` O cliente que tem vários webhooks cadastrados identifica de qual cadastro veio cada envio.

---

### Objetivos e métricas de sucesso

| Objetivo | Métrica | Meta |
| --- | --- | --- |
| `PRD-OBJ-01` Notificar o cliente em tempo real | Tempo entre a mudança de status e a primeira tentativa de envio | Menos de 10 segundos, com eventos pendentes lidos a cada 2 segundos |
| `PRD-OBJ-02` Não perder eventos de mudança de status | Mudanças de status sem evento registrado | 0 |
| `PRD-OBJ-03` Absorver indisponibilidades do cliente | Tentativas e janela de tempo antes de considerar falha permanente | 5 tentativas, cobrindo cerca de 15 horas |
| `PRD-OBJ-04` Entregar no prazo pedido pela Atlas | Sprints até a entrega, incluindo a revisão de segurança | 3 sprints (prazo da Atlas: fim de novembro) |

Não definido na reunião: métrica de redução do polling no GET /orders e métrica de retenção dos clientes.

---

### Escopo

Incluso
- `PRD-ESC-01` Cadastro, edição, remoção e listagem de webhooks pelo cliente.
- `PRD-ESC-02` Escolha, por webhook, dos status de pedido que geram notificação.
- `PRD-ESC-03` Registro do evento na mesma operação da mudança de status, sem possibilidade de um acontecer sem o outro.
- `PRD-ESC-04` Envio assíncrono das notificações por um processo separado da API.
- `PRD-ESC-05` Novas tentativas com intervalo crescente e separação dos eventos com falha permanente.
- `PRD-ESC-06` Reprocessamento manual dos eventos com falha permanente, só por administrador.
- `PRD-ESC-07` Assinatura de cada envio com secret própria do webhook e troca de secret com período de transição.
- `PRD-ESC-08` Histórico de entregas por webhook.
- `PRD-ESC-09` Documentação da integração no portal de desenvolvedor, deixando claro que o mesmo evento pode chegar mais de uma vez.

Fora de escopo
- `PRD-FORA-01` E-mail avisando o cliente quando o webhook dele falha. Adiado para a próxima fase, depois de medir o impacto.
- `PRD-FORA-02` Painel visual para o cliente acompanhar os webhooks. Fica com o time de frontend, como projeto separado. Nesta entrega só há endpoints.
- `PRD-FORA-03` Limite de taxa de envio para cada cliente. Fica em observação, e será decidido depois se virar problema.
- `PRD-FORA-04` Arquivamento dos eventos já entregues. Está fora do escopo desta feature.
- `PRD-FORA-05` Webhooks de entrada, enviados pelo cliente para a plataforma. Descartado: os clientes só querem receber.
- `PRD-FORA-06` Garantia de ordem global entre eventos e envio por vários processos em paralelo. Registrado como limitação conhecida, para resolver no futuro.
- `PRD-FORA-07` Garantia de entrega exatamente uma vez (exactly-once). Descartada, porque exige coordenação dos dois lados.
- `PRD-FORA-08` Envio síncrono dentro da mudança de status. Descartado.
- `PRD-FORA-09` Fila ou streaming em infraestrutura nova, como Redis Streams ou Redis Cluster. Descartado como excesso para um time pequeno.
- `PRD-FORA-10` Trigger no banco para acordar o processo de envio. Descartado, porque o MySQL não notifica processo externo.
- `PRD-FORA-11` Novas tentativas sem limite. Descartado, para o evento não ficar pendurado para sempre.
- `PRD-FORA-12` Truncar eventos grandes. Descartado: acima do limite, o envio dá erro.
- `PRD-FORA-13` Restringir o gerenciamento de webhooks a papéis específicos. Fica para mais adiante. Por enquanto, qualquer usuário autenticado pode usar.

---

### Requisitos funcionais

#### PRD-FR-01 Cadastrar webhook
O cliente cadastra um endereço para receber notificações de mudança de status dos seus pedidos.

**Fluxo principal**
- O usuário autenticado que representa o cliente envia o endereço https e a lista de status que quer receber.
- O cliente é identificado nos dados da requisição, e não pelo token de acesso.
- O sistema gera uma secret única para esse webhook.
- O sistema salva o cadastro como ativo e devolve a secret na resposta da criação.

**Fluxos alternativos e exceções**
- Qualquer usuário autenticado pode cadastrar, sem exigência de papel específico nesta fase.

**Erros previstos**
- Requisição sem autenticação é recusada.
- Endereço com http, e não https, é recusado com erro de validação (WEBHOOK_INVALID_URL).

**Prioridade:** alta

---

#### PRD-FR-02 Editar webhook
O cliente altera um webhook já cadastrado.

**Fluxo principal**
- O usuário autenticado informa o webhook e os dados que quer mudar, como o endereço ou a lista de status.
- O sistema valida e salva a alteração.

**Fluxos alternativos e exceções**
- A regra de endereço https também vale na edição.

**Erros previstos**
- Webhook inexistente retorna WEBHOOK_NOT_FOUND.
- Endereço com http é recusado com erro de validação (WEBHOOK_INVALID_URL).
- Requisição sem autenticação é recusada.

**Prioridade:** alta

---

#### PRD-FR-03 Remover webhook
O cliente remove um webhook que não quer mais usar.

**Fluxo principal**
- O usuário autenticado informa o webhook que quer remover.
- O sistema remove o cadastro.

**Fluxos alternativos e exceções**
- Não definido na reunião.

**Erros previstos**
- Webhook inexistente retorna WEBHOOK_NOT_FOUND.
- Requisição sem autenticação é recusada.

**Prioridade:** alta

---

#### PRD-FR-04 Listar webhooks de um cliente
O cliente consulta os webhooks que tem cadastrados.

**Fluxo principal**
- O usuário autenticado informa o cliente.
- O sistema devolve os webhooks cadastrados para esse cliente.

**Fluxos alternativos e exceções**
- Não definido na reunião.

**Erros previstos**
- Requisição sem autenticação é recusada.

**Prioridade:** alta

---

#### PRD-FR-05 Filtrar eventos por status
Cada webhook recebe somente as mudanças para os status que escolheu.

**Fluxo principal**
- O pedido muda de status.
- O sistema verifica quais webhooks daquele cliente querem ouvir o novo status.
- O evento é registrado apenas para esses webhooks.

**Fluxos alternativos e exceções**
- Se nenhum webhook do cliente quer aquele status, nenhum evento é registrado.

**Erros previstos**
- Não definido na reunião.

**Prioridade:** alta

---

#### PRD-FR-06 Registrar o evento junto com a mudança de status
O evento de notificação é registrado na mesma operação que muda o status do pedido.

**Fluxo principal**
- O usuário muda o status de um pedido pelo fluxo que já existe.
- Na mesma transação que atualiza o pedido, o histórico e o estoque, o sistema registra o evento com um identificador único.
- O evento guarda os dados do pedido no momento da mudança.
- A mudança é confirmada, e o evento fica pendente de envio.

**Fluxos alternativos e exceções**
- Se o pedido mudar de novo depois, o evento continua mostrando o estado do momento em que o status mudou.
- Se a transição de status for inválida, o fluxo atual já a recusa e nenhum evento é criado.

**Erros previstos**
- Se o registro do evento falhar, a mudança de status inteira é desfeita.

**Prioridade:** alta

---

#### PRD-FR-07 Enviar o evento assinado ao cliente
Um processo separado entrega cada evento pendente ao endereço do cliente, com assinatura.

**Fluxo principal**
- A cada 2 segundos, o processo de envio busca os eventos pendentes mais antigos, em lotes pequenos.
- Para cada evento, monta um conteúdo enxuto, sem os itens do pedido, e assina com HMAC-SHA256 usando a secret do webhook.
- Envia junto o identificador do evento, o identificador do webhook e o horário do envio.
- Ao receber resposta de sucesso, marca o evento como entregue.

**Fluxos alternativos e exceções**
- O mesmo evento pode chegar mais de uma vez. O cliente usa o identificador do evento para descartar duplicados.
- Se o cliente quiser os detalhes do pedido, consulta o GET /orders/:id.
- A ordem de entrega só é garantida por pedido, enquanto houver um único processo de envio.

**Erros previstos**
- Cliente que não responde em 10 segundos: a tentativa conta como falha e o evento vai para nova tentativa.
- Evento acima de 64KB: não é enviado e gera erro. O destino desse evento não foi definido na reunião.

**Prioridade:** alta

---

#### PRD-FR-08 Retentar com intervalo crescente e separar falhas permanentes
Quando o envio falha, o sistema tenta de novo até um limite e depois separa o evento.

**Fluxo principal**
- Depois de uma falha, o sistema agenda nova tentativa com intervalos de 1 minuto, 5 minutos, 30 minutos, 2 horas e 12 horas.
- São 5 tentativas no total, cobrindo cerca de 15 horas.
- Ao esgotar as tentativas, o evento é considerado falha permanente e movido para a fila de falhas (DLQ), com o conteúdo, o motivo e o horário da falha.

**Fluxos alternativos e exceções**
- Se uma das tentativas tiver sucesso, o evento é marcado como entregue e o ciclo termina.

**Erros previstos**
- Falta de resposta em 10 segundos ou cliente fora do ar contam como falha.
- Não definido na reunião: quais outras respostas do cliente contam como falha.

**Prioridade:** alta

---

#### PRD-FR-09 Reprocessar manualmente eventos da DLQ
O administrador devolve um evento com falha permanente para nova entrega.

**Fluxo principal**
- O administrador escolhe um evento da DLQ e pede o reprocessamento.
- O sistema devolve o evento para a fila de envio como pendente.
- O sistema registra em log quem fez o reprocessamento, para auditoria.

**Fluxos alternativos e exceções**
- Não definido na reunião.

**Erros previstos**
- Usuário sem papel de administrador tem o acesso negado.

**Prioridade:** alta

---

#### PRD-FR-10 Consultar histórico de entregas
O cliente vê o resultado dos últimos envios feitos para um webhook.

**Fluxo principal**
- O usuário autenticado informa o webhook.
- O sistema devolve as últimas 100 entregas, com sucesso ou falha, o conteúdo enviado, a resposta recebida e o tempo de resposta.

**Fluxos alternativos e exceções**
- Não definido na reunião.

**Erros previstos**
- Webhook inexistente retorna WEBHOOK_NOT_FOUND.
- Requisição sem autenticação é recusada.

**Prioridade:** alta

---

#### PRD-FR-11 Rotacionar a secret
O cliente pede uma nova secret para um webhook sem interromper o recebimento.

**Fluxo principal**
- O cliente pede uma nova secret pela API.
- O sistema gera a nova secret e a devolve.
- Durante 24 horas, a secret antiga e a nova valem em paralelo, para o cliente migrar os sistemas dele.
- Depois das 24 horas, a secret antiga deixa de valer.

**Fluxos alternativos e exceções**
- Não definido na reunião.

**Erros previstos**
- Não definido na reunião.

**Prioridade:** alta

---

### Requisitos não funcionais

Performance
- `PRD-RNF-01` Os eventos pendentes são lidos a cada 2 segundos, para a primeira tentativa de envio acontecer em menos de 10 segundos.
- `PRD-RNF-02` A mudança de status do pedido não espera nenhuma chamada ao cliente. Um cliente lento não pode travar outros pedidos.
- `PRD-RNF-03` Cada chamada ao cliente tem timeout de 10 segundos.
- `PRD-RNF-04` O conteúdo enviado é enxuto e não inclui os itens do pedido.
- Latência das APIs de configuração: não definido na reunião.

Disponibilidade
- `PRD-RNF-05` O processo de envio roda separado da API, para continuar funcionando quando a API reinicia.
- Meta de disponibilidade: não definido na reunião.

Segurança e autorização
- `PRD-RNF-06` Todo envio é assinado com HMAC-SHA256 sobre o corpo da requisição.
- `PRD-RNF-07` Cada webhook tem uma secret única, gerada pela plataforma. Não existe secret global.
- `PRD-RNF-08` A secret pode ser rotacionada, com 24 horas de convivência entre a antiga e a nova.
- `PRD-RNF-09` Só são aceitos endereços https.
- `PRD-RNF-10` O tamanho máximo de um evento é 64KB. Acima disso, o envio dá erro.
- `PRD-RNF-11` Cada envio informa o horário em que foi feito, para o cliente conseguir detectar reenvios maliciosos (replay attack), se quiser.
- `PRD-RNF-12` O reprocessamento da DLQ exige papel de administrador e fica registrado com quem fez.
- `PRD-RNF-13` O gerenciamento de webhooks é liberado para qualquer usuário autenticado nesta fase.
- `PRD-RNF-14` A autenticação por JWT e a checagem de papel reaproveitam o controle de acesso que já existe na API.
- `PRD-RNF-15` Os papéis disponíveis hoje são administrador (ADMIN) e operador (OPERATOR).

Observabilidade
- `PRD-RNF-16` Os logs usam o logger estruturado que já existe no projeto (Pino).
- `PRD-RNF-17` Os erros da feature passam pelo tratamento centralizado de erros que já existe.
- `PRD-RNF-18` A DLQ guarda o conteúdo, o motivo e o horário de cada falha permanente, como evidência para investigação.
- Métricas e tracing: não definido na reunião.

Confiabilidade e integridade de dados
- `PRD-RNF-19` A entrega é pelo menos uma vez (at-least-once), com um identificador único por evento para o cliente descartar duplicados.
- `PRD-RNF-20` A ordem dos eventos só é garantida por pedido, e enquanto houver um único processo de envio.
- `PRD-RNF-21` O evento reflete o pedido no momento da mudança de status, mesmo que o pedido mude depois.
- `PRD-RNF-22` O registro do evento é atômico com a mudança de status: ou os dois acontecem, ou nenhum acontece.

Compatibilidade e portabilidade
- `PRD-RNF-23` O processo de envio é um novo ponto de entrada no mesmo projeto, usando o mesmo banco e a mesma stack.
- `PRD-RNF-24` O conteúdo enviado é JSON, com horários no formato ISO 8601.
- `PRD-RNF-25` A assinatura usa um padrão de mercado (HMAC-SHA256), para o qual os clientes já têm bibliotecas.
- `PRD-RNF-26` A validação dos dados de entrada segue o padrão de schemas Zod do projeto.
- `PRD-RNF-27` Os erros da feature reaproveitam a classe base de erros da aplicação.
- `PRD-RNF-28` Os códigos de erro da feature usam o prefixo WEBHOOK_.

Compliance
- Não definido na reunião, além da auditoria do reprocessamento (`PRD-RNF-12`).

Acessibilidade no frontend consumidor
- Não definido na reunião. A entrega é só de API, e o painel visual está fora de escopo (`PRD-FORA-02`).

---

### Arquitetura e abordagem

Abordagem
- `PRD-ARQ-01` Envio assíncrono com padrão outbox no banco que já existe. A mudança de status registra o evento, e um processo separado faz a entrega depois.

Componentes
- `PRD-ARQ-02` Módulo de webhooks dentro da aplicação, no mesmo padrão dos outros módulos.
- `PRD-ARQ-03` Padrão de módulos existente, usado como referência pelo módulo de pedidos.
- `PRD-ARQ-04` Registro de eventos pendentes (outbox), gravado junto com a mudança de status.
- `PRD-ARQ-05` Processo de envio separado da API, que lê os eventos pendentes por polling.
- `PRD-ARQ-06` Fila de falhas permanentes (DLQ), guardada separada dos eventos pendentes.
- `PRD-ARQ-07` Cadastro de webhooks, com endereço, secret própria, cliente e indicação de ativo.

Integrações
- `PRD-ARQ-08` Fluxo de mudança de status do pedido, que passa a registrar o evento dentro da sua transação.
- `PRD-ARQ-09` Endpoints https dos clientes, fora da infraestrutura da plataforma, que recebem os envios.
- `PRD-ARQ-10` Consulta de pedido que já existe na API, usada pelo cliente quando quer os detalhes que não vão no envio.

### Decisões e trade-offs principais

#### PRD-DEC-01 Decisão: outbox no MySQL existente
- **Justificativa:** garante que o evento só existe se a mudança de status foi confirmada, sem subir infraestrutura nova.
- **Trade-off:** a tabela de eventos cresce com o tempo, e o arquivamento ficou fora desta feature.

#### PRD-DEC-02 Decisão: processo de envio separado, com polling a cada 2 segundos
- **Justificativa:** o MySQL não avisa processos externos, e com um processo separado o envio não para quando a API reinicia.
- **Trade-off:** um evento pode esperar até 2 segundos para ser pego, e a ordem só é garantida por pedido, com um único processo de envio.

#### PRD-DEC-03 Decisão: 5 tentativas com intervalo crescente e DLQ separada
- **Justificativa:** cobre indisponibilidades longas do cliente, como um caso real de 2 horas, sem deixar eventos pendurados para sempre.
- **Trade-off:** depois de cerca de 15 horas, o evento só volta a ser enviado por reprocessamento manual.

#### PRD-DEC-04 Decisão: HMAC-SHA256 com secret por webhook e rotação de 24 horas
- **Justificativa:** o cliente consegue confirmar a origem e a integridade do envio, e o vazamento de uma secret afeta só um webhook.
- **Trade-off:** o cliente precisa implementar a verificação da assinatura e a troca de secret.

#### PRD-DEC-05 Decisão: entrega pelo menos uma vez, com identificador de evento
- **Justificativa:** é o padrão de mercado. Garantir exatamente uma vez exigiria coordenação dos dois lados.
- **Trade-off:** o cliente precisa descartar duplicados do lado dele.

#### PRD-DEC-06 Decisão: reuso máximo dos padrões do projeto
- **Justificativa:** o módulo de webhooks fica igual aos outros, com os mesmos padrões de erro, log, validação e organização.
- **Trade-off:** não definido na reunião.

#### PRD-DEC-07 Decisão: reprocessamento só para administrador, gerenciamento de webhooks para qualquer usuário autenticado
- **Justificativa:** mexer na fila de entrega não é tarefa de operador.
- **Trade-off:** o gerenciamento de webhooks fica menos restrito nesta fase, e pode ser endurecido mais adiante.

---

### Dependências

#### PRD-DEP-01 Organizacional: revisão de segurança antes do deploy
A Sofia (Segurança) precisa de pelo menos 2 dias úteis para revisar o código de segurança, principalmente a assinatura HMAC e a geração de secret, antes do deploy. Essa revisão precisa ser agendada.

#### PRD-DEP-02 Organizacional: documentação no portal de desenvolvedor
O Marcos (PM) documenta no portal como integrar, com destaque para a entrega pelo menos uma vez e o descarte de duplicados pelo identificador do evento.

#### PRD-DEP-03 Organizacional: confirmação do prazo com a Atlas
O Marcos (PM) confirma com a Atlas o prazo de três sprints e atualiza os clientes.

#### PRD-DEP-04 Organizacional: revisão do desenho antes de codar
A Larissa (Tech Lead) abre o documento de design e faz uma sessão de revisão com o Bruno e o Diego antes do início da implementação.

#### PRD-DEP-05 Técnica: extensão da mudança de status do pedido
O fluxo de mudança de status do pedido precisa ser estendido para registrar o evento dentro da sua transação.

#### PRD-DEP-06 Técnica: processo de envio conectado ao mesmo banco
O processo de envio precisa rodar separado da API, conectado ao mesmo banco e com conexão própria.

#### PRD-DEP-07 Externa: preparo dos clientes
Os clientes precisam expor um endpoint https, verificar a assinatura HMAC, descartar eventos duplicados pelo identificador e migrar para a nova secret dentro das 24 horas depois de uma rotação.

---

### Riscos e mitigação

#### PRD-RISCO-01 Cliente fica fora do ar por muito tempo e deixa de receber eventos
- **Probabilidade:** alta. Já aconteceu: um cliente ficou duas horas fora do ar em manutenção planejada.
- **Impacto:** o cliente deixa de receber mudanças de status. Depois de cerca de 15 horas, os eventos dele vão para a DLQ.
- **Mitigação:**
  - 5 tentativas com intervalo crescente, cobrindo cerca de 15 horas.
  - Timeout de 10 segundos por chamada, para um cliente lento não segurar o envio.
  - DLQ com o conteúdo, o motivo e o horário da falha.
- **Plano de contingência:** reprocessamento manual da DLQ por administrador.

#### PRD-RISCO-02 Vazamento da secret de um cliente
- **Probabilidade:** média. Já aconteceu uma vez: um cliente vazou a secret no log da própria aplicação.
- **Impacto:** com uma secret global, o vazamento exporia todos os clientes. Com secret por webhook, o impacto fica restrito àquele webhook.
- **Mitigação:**
  - Secret única por webhook.
  - Assinatura HMAC-SHA256 sobre o corpo e uso obrigatório de https.
  - Revisão de segurança antes do deploy.
- **Plano de contingência:** o cliente rotaciona a secret pela API. A antiga vale por mais 24 horas e depois deixa de valer.

#### PRD-RISCO-03 Status do pedido muda e o evento não é registrado
- **Probabilidade:** não definida na reunião.
- **Impacto:** o cliente nunca fica sabendo da mudança, e a garantia da feature se perde.
- **Mitigação:**
  - Registro do evento dentro da mesma transação da mudança de status, desfazendo tudo se ele falhar.
  - Testes ponta a ponta da integração com a mudança de status.
- **Plano de contingência:** não definido na reunião.

#### PRD-RISCO-04 Cliente recebe o mesmo evento mais de uma vez
- **Probabilidade:** não definida na reunião.
- **Impacto:** o cliente processa a mesma mudança duas vezes, se não descartar duplicados.
- **Mitigação:**
  - Identificador único por evento, para o cliente descartar duplicados.
  - Documentação destacada no portal de desenvolvedor.
- **Plano de contingência:** não definido na reunião.

#### PRD-RISCO-05 Muitas chamadas seguidas para o mesmo cliente
- **Probabilidade:** não definida na reunião.
- **Impacto:** um cliente com 50 pedidos mudando de status em um minuto recebe 50 chamadas.
- **Mitigação:**
  - Observar o comportamento em produção.
- **Plano de contingência:** implementar limite de taxa de envio, se virar problema.

#### PRD-RISCO-06 Perda de ordem dos eventos ao escalar o envio
- **Probabilidade:** baixa. A decisão atual é ter um único processo de envio.
- **Impacto:** com vários processos em paralelo, o cliente pode receber os status de um pedido fora de ordem. Os clientes não pediram ordem global.
- **Mitigação:**
  - Um único processo de envio, com a limitação documentada.
- **Plano de contingência:** dividir o processamento por pedido ou usar lock pessimista, se for preciso escalar.

#### PRD-RISCO-07 Perder a Atlas por atraso na entrega
- **Probabilidade:** não definida na reunião.
- **Impacto:** a Atlas migra para o concorrente.
- **Mitigação:**
  - Escopo enxuto, com e-mail de alerta, painel e limite de taxa fora desta entrega.
  - Estimativa de três sprints, já incluindo a revisão de segurança.
  - Confirmação do prazo com a Atlas.
- **Plano de contingência:** não definido na reunião.

---

### Critérios de aceitação
Checklist objetivo que define se a feature está pronta.

- `PRD-CA-01` Uma mudança de status com webhook interessado registra o evento na mesma transação. Se o registro falhar, a mudança de status é desfeita.
- `PRD-CA-02` Uma mudança para um status que nenhum webhook do cliente escolheu não gera evento.
- `PRD-CA-03` O processo de envio roda separado da API e lê os eventos pendentes a cada 2 segundos. A primeira tentativa de envio acontece em menos de 10 segundos depois da mudança de status.
- `PRD-CA-04` Todo envio é assinado com HMAC-SHA256 usando a secret do webhook e informa o identificador do evento, o identificador do webhook e o horário do envio.
- `PRD-CA-05` O cadastro ou a edição com endereço http é recusado com erro de validação.
- `PRD-CA-06` Um evento acima de 64KB não é enviado e gera erro.
- `PRD-CA-07` Uma chamada sem resposta em 10 segundos conta como falha e o evento vai para nova tentativa.
- `PRD-CA-08` Depois de 5 tentativas com intervalos de 1m, 5m, 30m, 2h e 12h, o evento vai para a DLQ com o conteúdo, o motivo e o horário da falha.
- `PRD-CA-09` O reprocessamento da DLQ só é aceito para administrador, registra quem fez e devolve o evento para envio como pendente.
- `PRD-CA-10` Depois de uma rotação, a secret antiga continua válida por 24 horas e depois deixa de valer.
- `PRD-CA-11` A secret é gerada pela plataforma, devolvida na criação do webhook e é única por webhook.
- `PRD-CA-12` O histórico de um webhook mostra as últimas 100 entregas, com sucesso ou falha, o conteúdo, a resposta e o tempo de resposta.
- `PRD-CA-13` Os erros da feature usam o prefixo WEBHOOK_, e um webhook inexistente retorna WEBHOOK_NOT_FOUND.
- `PRD-CA-14` A revisão de segurança, de pelo menos 2 dias úteis, foi concluída antes do deploy.
- `PRD-CA-15` A documentação no portal de desenvolvedor destaca a entrega pelo menos uma vez e o descarte de duplicados pelo identificador do evento.

---

### Estratégia de testes e validação

Tipos de teste obrigatórios
- `PRD-TEST-01` Testes ponta a ponta da integração entre a mudança de status do pedido e o registro e envio do evento.
- `PRD-TEST-02` Revisão de código de segurança, com foco na assinatura HMAC e na geração de secret.
- `PRD-TEST-03` A suíte de testes de integração que já existe para pedidos (transições de status e estoque) continua passando depois da extensão da mudança de status.
- Testes unitários e teste de carga: não definido na reunião.

Estratégia de validação
- `PRD-TEST-04` O desenho da feature é revisado por Larissa, Bruno e Diego antes do início da implementação.
- `PRD-TEST-05` A revisão de segurança, de pelo menos 2 dias úteis, acontece antes do deploy e já está incluída nas três sprints.
- Uso de TDD ou de QA manual guiado por roteiro: não definido na reunião.
