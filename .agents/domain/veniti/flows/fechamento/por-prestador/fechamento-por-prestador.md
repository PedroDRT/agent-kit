---
type: flow
module: fechamento
layer: flow
related:
  - fechamento-por-prestador
  - agrupamento-lote
  - execucao-servico
  - faturamento-lote
  - fluxo-calculo-credito
  - atendimento-lifecycle
---

# Flow — Fechamento Financeiro por Prestador (Negociação Bilateral)

> Descreve o processo de encerramento financeiro entre o Veniti e o Prestador, com agrupamento em lote, negociação bilateral e agendamento de pagamento.

## Descrição

Descreve o processo de encerramento financeiro entre o Veniti (Assistência) e o Prestador de serviço. A Assistência agrupa os acionamentos finalizados em um lote, propõe valores, negocia bilateralmente com o prestador e agenda o pagamento.

**Portais:** `/assistencia/faturamento/prestador/` e `/prestador/fechamento/`
**Atores:** Operação financeira (Assistência) + Prestador

---

## Fluxo

### B1. Identificação de Acionamentos Faturáveis

- **Ator:** Sistema
- **Critério:** Acionamentos com status `FINALIZADO` no período de referência, não incluídos em lote anterior
- **Portal:** `/assistencia/faturamento/prestador/`

### B2. Criação do Lote

- **Ator:** Assistência (operação financeira)
- **Ação:** Agrupa acionamentos finalizados do prestador em um lote (`fechamento_prestador`)
- **Dados:** Número do lote, série, data emissão, mês de referência, itens (acionamentos)
- **Status:** `ABERTO`
- **Feature:** [[agrupamento-lote]]

### B3. Proposta de Valores pela Assistência

- **Ator:** Assistência
- **Ação:** Define valores tarifários para cada item do lote
- **Status:** `EM NEGOCIAÇÃO`

### B4. Revisão pelo Prestador

- **Ator:** Prestador
- **Portal:** `/prestador/fechamento/`
- **Opção A — Aceita:** Concorda com os valores → Status → `ACEITO`
- **Opção B — Contraproposta:** Sugere valores diferentes → Status → `CONTRAPROPOSTA`
- **Opção C — Solicitação de ajuste:** Solicita análise → Status → `SOLICITAÇÃO`

### B5. Análise de Contraproposta (Se aplicável)

- **Ator:** Assistência
- **Portal:** `/assistencia/faturamento/prestador/`
- **Status:** `ANÁLISE`
- **Decisão:** Aceita a contraproposta → `ACEITO` | Recusa → `RECUSADO` | Nova proposta → volta ao B3

### B6. Acordo Firmado (`ACEITO`)

- **Status:** `ACEITO`
- **Resultado:** Valores do lote fixados, lote pronto para pagamento

### B7. Agendamento do Pagamento

- **Ator:** Assistência (financeiro)
- **Ação:** Define data de pagamento e forma de pagamento (PIX, TED, etc.)
- **Sistema:** Status → `AGENDADO`; lançamento em `contas_pagar` criado/atualizado
- **Portal:** `/assistencia/contas_pagar/`

### B8. Realização do Pagamento

- **Ator:** Operação financeira / Sistema bancário
- **Ação:** Pagamento efetuado; confirmação registrada
- **Sistema:** Status → `REALIZADO`; `time_recebimento` gravado
- **Visível ao prestador em:** `/prestador/faturamento/` com coluna "Data Recebimento"

---

## Pontos de Entrada

| Portal | Ator | Trigger |
|---|---|---|
| `/assistencia/faturamento/prestador/` | Operação financeira | Criação de lote para prestador |
| `/prestador/fechamento/` | Prestador | Revisão de proposta recebida |

## Pontos de Saída

| Saída | Condição |
|---|---|
| Lote prestador `REALIZADO` | Pagamento ao prestador confirmado |
| Lote `RECUSADO` | Negociação sem acordo — acionamentos retornam ao pool |
| Lote `CANCELADO` | Cancelamento manual pela operação |

## Variações

### Múltiplos Lotes por Prestador no Período

- Possível quando há grande volume de acionamentos
- Cada lote é numerado independentemente
- Todos alimentam `contas_pagar` separadamente

### Fechamento sem Negociação

- Prestadores com tarifa fixa preestabelecida
- Status vai direto de `ABERTO` para `ACEITO` sem negociação bilateral

---

## Notas de QA

- **Risco crítico:** O processo de negociação bilateral (B3→B5) pode entrar em loop — há limite de rounds de negociação?
- **Risco:** Acionamento `FINALIZADO` durante o processamento do lote — incluído ou excluído? Lock de concorrência necessário
- **Risco de consistência:** Lote `ACEITO` mas pagamento não agendado → lote fica "esquecido" em status intermediário sem SLA
- **Validation gap:** `REALIZADO` marcado manualmente sem integração bancária real — alto risco de divergência entre sistema e extrato real
- **Comportamento unclear:** Lote `RECUSADO` → os acionamentos voltam automaticamente para novo lote ou requerem ação manual?
- **Risco de auditoria:** Todas as transições de status na negociação bilateral devem ter registro de usuário + timestamp — verificar completude do log
- **Edge case:** Prestador sem conta bancária cadastrada no momento do agendamento de pagamento

## Features Relacionadas

- [[fechamento-por-prestador]]
- [[agrupamento-lote]]
- [[execucao-servico]]
