# Fechamento Financeiro — Por Prestador

## Description

O **Fechamento com o Prestador** é a perspectiva do fechamento financeiro em que a Assistência (Veniti) agrupa os acionamentos finalizados pelo prestador em um período, negocia os valores bilateralmente e gera o lote de pagamento que alimenta Contas a Pagar.

- **Quem inicia:** Assistência ou Prestador
- **O que fecha:** Serviços executados pelo prestador no período (acionamentos finalizados)
- **Resultado:** Lote (batch) de pagamentos ao prestador → alimenta Contas a Pagar
- **Portais:** `/assistencia/faturamento/prestador/` e `/prestador/fechamento/`

## Inputs

| Campo | Tipo | Descrição |
|---|---|---|
| `id_prestador` | integer | Prestador a ser fechado |
| `periodo_referencia` | date | Mês de referência |
| `itens` | array | Lista de acionamentos finalizados incluídos |
| `forma_pagamento` | string | Método de pagamento (PIX, TED, etc.) |
| `id_conta_bancaria` | integer | Conta bancária de débito |
| `data_agendamento_pagamento` | date | Data prevista para pagamento |

## Outputs

- Lote criado em `fechamento_prestador` com seus itens em `fechamento_prestador_itens`
- Lançamento criado em `contas_pagar`
- Prestador visualiza o lote em `/prestador/faturamento/`
- Status do lote: `ABERTO → APROVADO → AGENDADO → REALIZADO`

## Status States (Lote)

```
ABERTO → EM NEGOCIAÇÃO → SOLICITAÇÃO → ANÁLISE → ACEITO → AGENDADO → REALIZADO
                       ↘ CONTRAPROPOSTA (prestador sugere valores diferentes)
                       ↘ RECUSADO
```

| Status | Ator | Descrição |
|---|---|---|
| `ABERTO` | Sistema | Lote criado, negociação não iniciada |
| `EM NEGOCIAÇÃO` | Assistência | Assistência propõe valores |
| `SOLICITAÇÃO` | Prestador | Prestador solicita ajuste |
| `ANÁLISE` | Assistência | Assistência analisa contraproposta |
| `CONTRAPROPOSTA` | Prestador | Prestador sugere valores alternativos |
| `ACEITO` | Ambos | Valores acordados |
| `RECUSADO` | Qualquer | Negociação encerrada sem acordo |
| `AGENDADO` | Assistência | Pagamento agendado para data definida |
| `REALIZADO` | Sistema | Pagamento confirmado |
| `CANCELADO` | Assistência | Lote cancelado |

## Business Rules

- O fechamento consolida acionamentos `FINALIZADO` no período
- Um acionamento só entra no fechamento se estiver em status `FINALIZADO`
- A negociação de tarifas permite ajuste bilateral — Prestador pode apresentar contraproposta
- Após `ACEITO`, os valores são fixados — não podem ser alterados sem cancelar e recriar o lote
- O fechamento com Prestador gera lançamento em `contas_pagar` — é a origem dos pagamentos ao prestador
- Prestadores visualizam seus lotes em `/prestador/faturamento/` com filtros por período, situação e número
- A coluna `Data Recebimento` no lote do prestador registra quando o pagamento foi efetivamente recebido (confirmação bancária)
- O período de referência (`time_mes_referente`) determina em qual competência o pagamento é lançado

## Edge Cases

- Acionamento finalizado fora do período de fechamento — incluído no próximo período ou retroativo?
- Prestador contesta valor de KM calculado durante a negociação
- Fechamento duplicado para o mesmo período e prestador
- Lote cancelado após pagamento agendado (reversão bancária)
- Conta bancária inválida no momento do agendamento do pagamento
- Múltiplos lotes abertos simultaneamente para o mesmo prestador

## QA Notes

- **Risco crítico:** O fluxo de negociação bilateral (Assistência ↔ Prestador) tem múltiplas transições de status — o que acontece se ambos tentarem mudar o status simultaneamente?
- **Risco:** Acionamento finalizado durante o processamento do fechamento — é incluído ou excluído do lote corrente?
- **Validation gap:** Não está claro se há validação de que todos os acionamentos no lote pertencem ao mesmo prestador e período
- **Comportamento unclear:** Se o prestador tem lote `RECUSADO`, os acionamentos são incluídos em novo lote automaticamente ou requerem ação manual?
- **Edge case crítico:** `REALIZADO` sem confirmação bancária real — o sistema confia na marcação manual ou há integração bancária?
- **Risco de auditoria:** Mudanças de status no lote devem ser rastreadas com usuário e timestamp — verificar se o log está implementado

## Dependencies

- **Portal:** `html/assistencia/faturamento/prestador/` (lotes prestador)
- **Portal Prestador:** `html/prestador/fechamento/`, `html/prestador/faturamento/`
- **Contas a Pagar:** `html/assistencia/contas_pagar/`
- **Modelos:** `src/Models/Faturamento.php`
- **Use Cases:** `src/UseCases/CalcularMetricas.php`
- **Banco:** `fechamento_prestador`, `fechamento_prestador_itens`, `contas_pagar`

## Related Flows

- [[fechamento-financeiro]] (por-prestador)
- [[faturamento-lote]]

## Related Features

- [[agrupamento-lote]]
- [[execucao-servico]]
