# ADR-004: Assinatura HMAC-SHA256 com secret por endpoint e rotação com 24 horas de convivência

- **Status:** Aceito
- **Decidido em:** reunião técnica registrada em `TRANSCRICAO.md`, [09:22] Sofia
- **Decisores:** Sofia (Eng. Segurança), Bruno (Eng. Pleno, Pedidos), Diego (Eng. Sênior, Plataforma), Larissa (Tech Lead)
- **Relacionado a:** [PRD-DEC-04](../PRD.md#prd-dec-04-decisão-hmac-sha256-com-secret-por-webhook-e-rotação-de-24-horas), [PRD-FR-07](../PRD.md#prd-fr-07-enviar-o-evento-assinado-ao-cliente), [PRD-FR-11](../PRD.md#prd-fr-11-rotacionar-a-secret), [ADR-005](ADR-005-entrega-at-least-once-com-x-event-id.md)

## Status

Aceito. A Sofia fechou a decisão às 09:22: "HMAC-SHA256 sobre o corpo do request, secret por endpoint, suporte a rotação com grace period de 24h".

## Contexto

- `ADR-004-CTX-01` Os eventos levam dados de pedidos para endpoints fora da infraestrutura da plataforma. O cliente precisa conseguir validar que a requisição veio da plataforma e que ninguém adulterou o conteúdo no caminho.
- `ADR-004-CTX-02` Já houve cliente que vazou a secret no log da própria aplicação.
- `ADR-004-CTX-03` O TLS é obrigatório (URL https, recusada com erro de validação no schema Zod). A Sofia disse que isso "nem é decisão arquitetural", por isso fica fora deste ADR.

## Decisão

`ADR-004` Vamos assinar o corpo de cada request com HMAC-SHA256 e enviar a assinatura em um header (`X-Signature`), para o cliente verificar do lado dele. Cada endpoint de webhook tem uma secret única, nunca uma secret global da plataforma. O cliente pode rotacionar a secret pela API. Depois da rotação, a secret antiga continua válida por 24 horas em paralelo com a nova e depois deixa de valer.

## Alternativas Consideradas

### `ADR-004-ALT-01` Secret global da plataforma

A Sofia mesma a descartou às 09:21, ao explicar por que a secret tem que ser por endpoint: "Senão se vaza uma, vaza tudo". Com uma secret global, um único vazamento exporia os envios de todos os clientes.

## Consequências

### Positivas

- `ADR-004-POS-01` O cliente consegue validar a origem e a integridade de cada envio.
- `ADR-004-POS-02` HMAC-SHA256 é o padrão de mercado, e todo cliente sério tem biblioteca para ele.
- `ADR-004-POS-03` Um vazamento atinge só o endpoint daquela secret.
- `ADR-004-POS-04` A convivência de 24 horas dá tempo para o cliente migrar os sistemas dele sem perder envios.

### Negativas

- `ADR-004-NEG-01` O cliente precisa implementar a verificação da assinatura do lado dele.
- `ADR-004-NEG-02` O HMAC e a geração de secret exigem revisão de segurança de pelo menos dois dias úteis antes do deploy.
- `ADR-004-NEG-03` Depois de rotacionar, o cliente tem só 24 horas para migrar. Depois disso, a secret antiga deixa de valer.

### Trade-off

Ganhamos autenticidade e integridade verificáveis pelo cliente e um impacto de vazamento limitado a um endpoint. Em troca, o cliente passa a ter trabalho de integração, e a entrega depende de uma revisão de segurança dedicada.
