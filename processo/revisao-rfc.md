# Revisão da RFC

Registro da rodada de revisão simulada da `docs/RFC.md` pelos participantes da reunião e das rodadas do verificador independente.

## Decisões de planejamento

- Alternativas de arquitetura incluídas: síncrono, Redis Streams, trigger, worker na API, exactly-once e payload montado no envio. As alternativas de parâmetro (retry indefinido, 3 tentativas, failed na própria outbox, secret global, `customer_id` do JWT, id autoincremental, truncar payload, filtro no envio) ficaram só nos ADRs ou no PRD.
- Worker na API e exactly-once foram mantidos, mesmo levantados e descartados pela mesma pessoa, porque foram postos na mesa na reunião e mudariam a forma da solução.
- Arquivamento e endurecimento do CRUD entraram como questões em aberto, com "Para fechar: Não definido na reunião", porque foram explicitamente adiados.
- A lacuna sobre respostas que contam como falha entrou, porque o assunto aparece em [09:42] Diego.

## Rodada de revisão da equipe (versão 1)

Primeira versão com 2.186 palavras, reescrita para 1.792 antes da revisão.

### Marcos (Product Manager)

1. **Omissão.** O TL;DR não diz a meta prometida aos clientes, "abaixo de 10 segundos" ([09:02] Marcos). Quem só ler o resumo não sabe se a proposta atende o que foi prometido.
   - **Decisão:** aplicado. O TL;DR passou a citar a meta, e a frase repetida em `RFC-CTX-01` saiu para não duplicar.

### Bruno (Engenheiro Pleno, time de Pedidos)

1. **Divergência com a transcrição.** O risco `RFC-RISCO-04` falava só em "crescimento da outbox", mas a preocupação levantada foi a performance do worker com a tabela cheia ([09:07] Bruno), respondida em [09:08] Diego.
   - **Decisão:** aplicado. O risco virou "Outbox grande deixa o worker lento ([09:07] Bruno)".
2. **Omissão.** A RFC não cita os arquivos que propus para o worker ([09:28] Bruno).
   - **Decisão:** não aplicado. Nomes de arquivos novos são detalhe de FDD, e esses arquivos ainda não existem no repositório.

### Diego (Engenheiro Sênior, time de Plataforma)

1. **Omissão.** O texto não diz o que o replay faz: recolocar o evento na outbox como pendente ([09:18] Diego). Só o diagrama mostra isso.
   - **Decisão:** aplicado em `RFC-PROP-03`.
2. **Omissão.** A progressão do backoff (1m/5m/30m/2h/12h, [09:17] Diego) não aparece.
   - **Decisão:** não aplicado. É parâmetro, já registrado no ADR-003. A RFC fica com "5 tentativas, cerca de 15 horas".

### Sofia (Engenheira de Segurança)

1. **Omissão.** A mitigação de `RFC-RISCO-07` (vazamento de secret) não cita a rotação com 24 horas de convivência ([09:21] Sofia), que é a resposta prevista para um vazamento.
   - **Decisão:** aplicado.

A Sofia e o Marcos não comentaram o snapshot do payload (`RFC-PROP-05`, `RFC-ALT-06`), porque a decisão foi tomada em [09:52] Larissa, depois que os dois saíram da call.

### Ajustes de tamanho depois da revisão

Para ficar abaixo de 1.800 palavras:
- o TL;DR foi encurtado;
- `RFC-PROP-03` passou a apontar para `RFC-IMP-04` a `RFC-IMP-06` em vez de repetir a lista de padrões;
- frases de `RFC-IMP-02`, `RFC-IMP-07` e `RFC-PROP-01` foram enxugadas.

Resultado: 1.758 palavras.

**Resumo:** 7 comentários, 5 aplicados e 2 recusados.

## Verificador independente

O verificador foi um subagente do Claude Code criado só com os caminhos dos arquivos e o prompt adversarial da tarefa, sem resumo da RFC nem explicação das escolhas.

### Rodada 1: NEEDS_FIX

| # | Apontamento | Decisão | O que mudou |
| --- | --- | --- | --- |
| P1 | O TL;DR prometia "entregam em menos de 10 segundos", mas o PRD-OBJ-01 mede o tempo até a primeira tentativa | Aplicado | "Primeira tentativa de envio em menos de 10 segundos" |
| P2 | A mitigação de `RFC-RISCO-02` (acoplamento por função, testes) não mitiga o rollback, que é intencional | Aplicado | "Rollback intencional ([09:40] Bruno); mitigação: Não definido na reunião" |
| P3 | `RFC-CTX-05` atribuía a restrição a [09:07] Larissa, mas quem descartou infraestrutura nova foi o Diego | Aplicado | Reescrito e citado [09:07] Diego |
| P4 | "Polling contínuo" como trade-off da trigger não tem origem | Aplicado | Trocado por "Até 2 s de espera no pior caso" (ADR-002-NEG-01). A linha do síncrono passou a usar o trade-off do ADR-001 ("Novo ponto de falha na transação") |
| P5 | `RFC-IMP-01` e `RFC-IMP-07` faziam afirmações sem origem | Aplicado | `RFC-IMP-01` passou a citar [09:40] Bruno. `RFC-IMP-07` diz que nenhuma fala prevê mudança na máquina de estados, com o arquivo como origem |
| P6a | "Mesmo projeto" é da Larissa, e "lote pequeno" é de [09:08] Diego | Aplicado | Citações separadas em `RFC-PROP-02` |
| P6b | O ponteiro para `RFC-IMP-04` a `RFC-IMP-06` incluía `requireRole`, que não está na fala de [09:30], e omitia Zod | Aplicado | A lista de [09:30] Larissa voltou explícita: `AppError`, Pino, error middleware e Zod |
| P6c | "Problema do futuro" é do Diego, não da Larissa | Aplicado | `RFC-QA-02` cita [09:13] Diego e [09:13] Larissa |
| P7a | "Índice por status e criação" é detalhe de FDD | Aplicado | Virou "Leitura só dos pendentes, em lote pequeno" |
| P7b | O timeout de 10 s na proposta é decisão secundária | Aplicado | Removido de `RFC-PROP-02`; fica só em `RFC-RISCO-05` |
| P7c | "Garantias e limites" repete consequências dos ADRs | Recusado | A subseção é exigida pela tarefa. Cada item tem uma frase e linka o ADR, e juntar as garantias num lugar só é o valor próprio da RFC |
| P7d | "Decisões relacionadas" repete o índice dos ADRs | Recusado | A seção é obrigatória pelo README (requisito 2) |
| P7e | A tabela de síntese repete as alternativas | Recusado | A tabela é exigida pela tarefa e acrescenta a coluna "o que a proposta aceita em troca" |
| P7f | O replay (ADR-003) não estava nos links de `RFC-PROP-03` | Aplicado | Link para o ADR-003 adicionado |
| P7g | O TL;DR omitia "com um único worker" na garantia de ordem | Aplicado | Condição incluída |

Total: 15 apontamentos, 12 aplicados e 3 recusados. A RFC ficou com 1.776 palavras.

### Rodada 2: NEEDS_FIX

| # | Apontamento | Decisão | O que mudou |
| --- | --- | --- | --- |
| 1 | "Cerca de 15 horas" citava [09:17] Larissa, mas a fala é de [09:17] Diego | Aplicado | Citações separadas em `RFC-PROP-02` |
| 2 | A deduplicação pelo identificador citava [09:24] Diego, mas é de [09:25] Diego | Aplicado | `RFC-PROP-06` cita as duas falas |
| 3 | O replay manual citava [09:17] Diego, mas é de [09:18] Diego | Aplicado | `RFC-PROP-09` corrigido |
| 4 | Faltavam no impacto `prisma/schema.prisma`, `package.json` e `src/routes/index.ts` | Aplicado | Novos `RFC-IMP-07` (schema, com [09:06] Diego, [09:18] Diego e [09:21] Bruno) e `RFC-IMP-08` (roteador central). O `package.json` voltou a `RFC-IMP-02`. A máquina de estados passou a ser `RFC-IMP-09` |
| 5 | Em `RFC-RISCO-02`, o rollback estava como mitigação, mas é a causa | Aplicado | O rollback foi para a coluna de ângulo técnico, e a mitigação ficou "Não definido na reunião" |
| 6 | Tensão com o PRD-FR-07, que diz que o processo de envio "monta um conteúdo enxuto" | Não aplicado na RFC | A RFC segue o ADR-007, decidido em [09:52] Larissa. A redação do PRD vai como inconsistência no relatório final, sem alterar o PRD |
| 7a | Tabelas de síntese e de decisões repetem conteúdo | Recusado | Mesmo motivo da rodada 1 (P7d, P7e) |
| 7b | "Prisma Client próprio" é detalhe de FDD | Aplicado em parte | Saiu de `RFC-PROP-02`. Ficou só em `RFC-IMP-03`, onde é o impacto em `src/config/database.ts` |
| 7c | "Garantias e limites" está no limite da repetição | Recusado | O próprio verificador considera aceitável, e a subseção é exigida pela tarefa |
| 8 | Alternativas de retry, secret global e `customer_id` não entraram; o risco do ADR-006-NEG-03 também não | Recusado | As alternativas de parâmetro ficaram fora por decisão de planejamento. O ADR-006-NEG-03 é consequência inferida e não risco levantado na reunião. A `RFC-QA-05` cobre o ponto que foi falado |
| 9 | Autoria da Larissa é inferência | Recusado | A autoria foi definida pela usuária na tarefa, com base em [09:50] Larissa |
| 10 | "Até 2 s" interpreta "latência mínima 2 segundos no pior caso" | Recusado | É a mesma leitura do ADR-002-NEG-01 e do PRD, validada na entrevista do PRD |

Total: 12 apontamentos (contando 7a, 7b e 7c separadamente). 6 aplicados, 1 aplicado em parte, 1 tratado no relatório e 4 recusados. A RFC ficou com 1.796 palavras.

### Rodada 3: NEEDS_FIX, só com problemas menores (última rodada permitida)

| # | Apontamento | Decisão | O que mudou |
| --- | --- | --- | --- |
| 1 | `RFC-PROP-09`: as 15 horas citavam [09:18] Diego | Aplicado | Agora cita [09:17] Diego (15 horas) e [09:18] Diego (replay) |
| 2 | `RFC-IMP-05`: o `requireRole` não está em [09:32] Larissa | Aplicado | Acrescentado [09:36] Larissa |
| 3 | `RFC-IMP-08` não indicava a origem | Aplicado | O texto nomeia o `buildApiRouter` do próprio arquivo, e a origem é CODIGO |
| 4 | Criar uma lacuna para o armazenamento do histórico de entregas | Recusado na RFC | Pela regra 5, só entram como lacuna os pontos que o PRD marcou como "Não definido na reunião", e o PRD não marcou esse. O ponto vai no relatório final como lacuna para o FDD |
| 5 | Reduzir o Impacto a componentes, sem lista de arquivos | Recusado | A tarefa pede para citar os arquivos existentes afetados. A seção diz o que muda em cada um, sem descrever a implementação |
| 6a | A forma de integração ("função que recebe a transação") é detalhe de FDD | Aplicado | `RFC-PROP-01` diz só que o evento é gravado na transação, com rollback |
| 6b | O prefixo `WEBHOOK_` é detalhe de FDD | Recusado | É uma decisão registrada ([09:29] Larissa) e citada pelo README entre os padrões reaproveitados. A RFC não lista códigos específicos |
| 7 | Garantias e tabela-resumo repetem os ADRs | Recusado | Mesmo motivo das rodadas 1 e 2: as duas estruturas são exigidas pela tarefa |
| Obs. | "Até 2 s" interpreta "latência mínima" | Recusado | Mesmo motivo da rodada 2 (item 10) |

Total: 9 apontamentos. 4 aplicados, 1 levado ao relatório e 4 recusados. A RFC ficou com 1.794 palavras.

O veredito final continuou NEEDS_FIX, mas sem falha grave: o verificador declarou que os itens 3, 4, 5, 6 e 8 passam inteiros. Os apontamentos procedentes da rodada 3 foram aplicados. Pelo limite de três rodadas, o verificador não foi rodado de novo.

## Resumo das três rodadas

- Rodada 1: 15 apontamentos, 12 aplicados.
- Rodada 2: 12 apontamentos, 6 aplicados e 1 aplicado em parte.
- Rodada 3: 9 apontamentos, 4 aplicados.

As mudanças mais importantes foram:
- a meta do TL;DR ficou alinhada ao PRD-OBJ-01 (primeira tentativa, e não entrega);
- a falsa mitigação de `RFC-RISCO-02` saiu;
- seis citações foram corrigidas para a pessoa ou o minuto certos;
- o Impacto ganhou `prisma/schema.prisma` e `src/routes/index.ts`;
- detalhes de FDD saíram: índice, timeout, Prisma Client na proposta e forma da função de integração.

## Validação final

As 8 validações rodaram no terminal. Uma falhou na primeira execução:

- **Validação 3:** `src/modules`, citado em `RFC-PROP-03`, é uma pasta e não um arquivo. Corrigido para "módulo de domínio igual aos outros ([09:27] Bruno)".

Todas as outras passaram:
- 56 citações únicas, todas existentes na transcrição;
- nenhum formato fora do padrão;
- 50 IDs iguais entre a RFC e o tracker;
- links para o PRD e os ADRs válidos;
- 7 ADRs em "Decisões relacionadas";
- 1.794 palavras;
- nenhum travessão longo nos dois arquivos.
