# Agrupamento em Lote (Faturamento)

## Description

O **Lote** (também referenciado como `fechamento_prestador` no banco) é o agrupamento de acionamentos finalizados em uma unidade faturável. Representa a fatura que o Veniti emite ao prestador de serviço, detalhando todos os serviços executados em um período e o valor total a ser pago. O lote é o ponto de entrada para o módulo de Contas a Pagar.

**Analogia contábil:** O Lote para o prestador é equivalente a uma Nota de Serviço recebida — é o instrumento que autoriza o pagamento.

## Inputs

| Campo | Tipo | Descrição |
|---|---|---|
| `id_prestador` | integer | Prestador que executou os serviços |
| `numero` | string | Número da fatura/lote |
| `serie` | string | Série da nota (controle fiscal) |
| `time_emissao` | datetime | Data e hora de emissão |
| `time_pagamento` | datetime | Data prevista de pagamento |
| `time_recebimento` | datetime | Data de confirmação de recebimento |
| `time_mes_referente` | date | Competência (mês de referência) |
| `forma_pagamento` | string | PIX, TED, DOC, DEPÓSITO, BOLETO, CHEQUE |
| `id_conta_bancaria` | integer | Conta do prestador para recebimento |
| `itens` | array | Lista de `fechamento_prestador_itens` (acionamentos incluídos) |

### Item do Lote (`fechamento_prestador_itens`)

| Campo | Tipo | Descrição |
|---|---|---|
| `id_acionamento` | integer | Acionamento finalizado incluído |
| `valor` | decimal | Valor do acionamento neste lote |
| `km_total` | decimal | KM percorrido (base para cálculo tarifário) |
| `tarifa_aplicada` | decimal | Tarifa acordada no fechamento |

## Outputs

- Lote criado em `fechamento_prestador` com itens em `fechamento_prestador_itens`
- Lançamento gerado em `contas_pagar` com valor total do lote
- Lote visível ao prestador em `/prestador/faturamento/`
- Lote visível à operação em `/assistencia/faturamento/prestador/`
- Histórico de alterações registrado no log do sistema ("alterou as informações do lote")

## Status Lifecycle

```
ABERTO
  └──→ APROVADO (Assistência aprova os valores)
         └──→ AGENDADO (data de pagamento definida → gera lançamento em Contas a Pagar)
                └──→ REALIZADO (pagamento confirmado manualmente pela Assistência)
  └──→ RECUSADO (negociação não acordada)
  └──→ CANCELADO (lote cancelado)
```

> **Distinção importante:** `REALIZADO` é o status do **lote** (módulo Faturamento). O lançamento correspondente em **Contas a Pagar** tem seu próprio status `PAGO`, separado.

| Status (Assistência) | Status (Prestador) | Significado |
|---|---|---|
| ABERTO | ABERTO | Lote criado, aguardando ação |
| APROVADO | APROVADO | Valores aprovados pela Assistência |
| AGENDADO | AGENDADO | Pagamento agendado; lançamento gerado em Contas a Pagar |
| REALIZADO | REALIZADO | Pagamento confirmado manualmente |
| RECUSADO | RECUSADO | Lote recusado |
| CANCELADO | CANCELADO | Lote cancelado |

## Business Rules

- Somente acionamentos com status `FINALIZADO` são elegíveis para inclusão em lote
- Um acionamento só pode aparecer em **um único lote** — não pode ser faturado duas vezes
- O lote agrupa acionamentos do **mesmo prestador** no mesmo período de referência
- O campo `numero` do lote é o identificador contábil usado na Contas a Pagar
- A forma de pagamento e conta bancária são definidas no lote — devem corresponder ao cadastro do prestador
- O status `REALIZADO` registra `time_recebimento` — confirmação manual de que o pagamento foi efetuado; sem integração bancária automática
- O lançamento em `contas_pagar` gerado pelo lote tem status próprio (`PAGO`) — distinto do status `REALIZADO` do lote
- Lotes `CANCELADO` liberam os acionamentos para serem incluídos em novo lote
- O lote `APROVADO` sem data de pagamento pode ficar "travado" sem avançar para `AGENDADO`
- A coluna `Data Pagamento` no portal do prestador corresponde ao `time_pagamento` agendado
- Filtros disponíveis no portal do prestador: Data emissão, Assistência, Situação, Número, Protocolo

## Edge Cases

- Acionamento finalizado após o fechamento do período — próximo lote ou retroativo?
- Dois lotes criados para o mesmo prestador no mesmo mês (duplicata)
- Lote `AGENDADO` com forma de pagamento PIX e chave PIX inválida
- Prestador muda dados bancários após lote `APROVADO`
- Lote `REALIZADO` mas prestador não confirma recebimento (divergência)
- Acionamento contestado pelo prestador após inclusão em lote `APROVADO`
- Lote com zero itens criado por erro

## QA Notes

- **Risco crítico:** Não há evidência de validação que impeça o mesmo acionamento de aparecer em dois lotes — verificar constraint na tabela `fechamento_prestador_itens`
- **Risco:** `REALIZADO` registrado manualmente — sem integração bancária, a confirmação de pagamento depende de ação humana
- **Validation gap:** A forma de pagamento e dados bancários são validados no momento da criação do lote ou somente no pagamento?
- **Comportamento unclear:** Lote `CANCELADO` — os acionamentos voltam automaticamente para o pool ou requerem ação manual?
- **Edge case:** O que acontece com o lançamento em `contas_pagar` quando um lote é cancelado após ter sido registrado?
- **Auditoria:** O log registra "alterou as informações do lote" — mas a granularidade do log (qual campo mudou, de qual valor para qual) não está confirmada

## Dependencies

- **Portais**: `html/assistencia/faturamento/prestador/` (gestão de lotes), `html/prestador/faturamento/` (visão do prestador)
- **Contas a Pagar**: `html/assistencia/contas_pagar/` (lançamentos gerados)
- **Modelos**: `src/Models/Faturamento.php`
- **Fechamento**: `html/assistencia/fechamento/` (contexto financeiro)
- **Banco**: `fechamento_prestador`, `fechamento_prestador_itens`, `contas_pagar`

## Related Flows

- [[faturamento-lote]]
- [[fechamento-financeiro]]

## Related Features

- [[fechamento-financeiro]]
- [[calculo-credito]]
- [[execucao-servico]]
- [[ciclo-vida-acionamento]]
