# Faturamento em Lote

## Description

Descreve o processo de criação, gestão e pagamento de lotes de faturamento ao prestador. O lote é o instrumento financeiro que converte acionamentos finalizados em um pagamento estruturado. É a última etapa operacional antes da quitação financeira com o prestador.

**Posição no ciclo:** Acionamento `FINALIZADO` → Crédito calculado → **Lote criado** → Contas a Pagar → Pagamento

## Steps

### 1. Identificação de Acionamentos Elegíveis
- **Ator:** Sistema / Operação financeira
- **Critério de elegibilidade:**
  - Status do acionamento: `FINALIZADO`
  - Não incluído em nenhum lote anterior (sem `id_lote` associado)
  - Pertence ao prestador do lote sendo criado
  - Dentro do período de referência (`time_mes_referente`)
- **Portal:** `/assistencia/faturamento/prestador/` com filtros por prestador, período, situação

### 2. Seleção de Itens e Criação do Lote
- **Ator:** Operação financeira (Assistência)
- **Ação:** Seleciona acionamentos elegíveis e cria o lote
- **Dados definidos:**
  - `numero` — número da fatura (identificador contábil)
  - `serie` — série da nota
  - `time_emissao` — data de emissão
  - `time_mes_referente` — competência do pagamento
  - `forma_pagamento` — PIX, TED, DOC, etc.
  - `id_conta_bancaria` — conta bancária do prestador
- **Banco:** Criado em `fechamento_prestador`; itens em `fechamento_prestador_itens`
- **Status:** `ABERTO`
- **Log:** Entrada criada registrando a criação do lote

### 3. Negociação de Tarifas (Se necessário)
- **Ator:** Assistência + Prestador (bilateral)
- **Processo:** Ver [[fechamento-financeiro]] — Fluxo B (passos B3 a B5)
- **Resultado:** Valores dos itens do lote acordados

### 4. Aprovação do Lote
- **Ator:** Assistência (após acordo)
- **Status:** `ABERTO` → `APROVADO`
- **Efeito:** Valores fixados, lote pronto para agendamento de pagamento
- **Log:** "alterou as informações do lote" registrado

### 5. Agendamento do Pagamento
- **Ator:** Operação financeira
- **Ação:** Define `time_pagamento` e confirma `forma_pagamento` e conta bancária
- **Status:** `APROVADO` → `AGENDADO`
- **Sistema:** Lançamento criado/atualizado em `contas_pagar`
- **Filtros disponíveis (Contas a Pagar):**
  - Período
  - Nº Nota fiscal
  - Forma de pagamento: DEPOSITO, TRANSFERENCIA, DOC, TED, BOLETO, CHEQUE, PIX
  - Conta bancária de débito
  - Prestador / Fornecedor

### 6. Execução do Pagamento
- **Ator:** Operação financeira / banco
- **Ação:** Pagamento realizado via meio definido
- **Sistema:** Status → `REALIZADO`; `time_recebimento` gravado
- **Visível ao prestador:** `/prestador/faturamento/` — coluna "Data Recebimento" preenchida

### 7. Confirmação pelo Prestador
- **Ator:** Prestador (opcional)
- **Ação:** Visualiza lote em `/prestador/faturamento/` com status `REALIZADO`
- **Colunas visíveis:** Assistência, Número, Série, Data Emissão, Data Pagamento, Data Recebimento, Situação
- **Encerramento:** Ciclo financeiro do lote completo

---

## Entry Points

| Ponto | Ator | Portal |
|---|---|---|
| Criação manual de lote | Operação financeira | `/assistencia/faturamento/prestador/` |
| Revisão de lote recebido | Prestador | `/prestador/faturamento/` + `/prestador/fechamento/` |

## Exit Points

| Saída | Status | Consequência |
|---|---|---|
| Pagamento confirmado | `REALIZADO` | Ciclo financeiro encerrado |
| Lote recusado (negociação) | `RECUSADO` | Acionamentos retornam ao pool |
| Lote cancelado | `CANCELADO` | Lançamento em contas_pagar revertido |

## State Machine

```
ABERTO
  ├──→ EM NEGOCIAÇÃO → SOLICITAÇÃO → ANÁLISE
  │         └──────────────────────────────→ CONTRAPROPOSTA
  │                                                 ↓
  │                                              ANÁLISE
  │                                                 ↓
  ├──→ ACEITO ────────────────────────────→ AGENDADO → REALIZADO
  └──→ RECUSADO
  └──→ CANCELADO
```

## Business Rules

- Um acionamento `FINALIZADO` entra em **exatamente um lote** — constraint de unicidade
- Lotes agrupam acionamentos do **mesmo prestador** no mesmo período de referência
- O lote `APROVADO` tem valores imutáveis — alteração requer cancelamento e criação de novo lote
- `time_recebimento` só é preenchido quando o prestador efetivamente recebe — status `REALIZADO`
- Lotes `CANCELADO` devolvem os acionamentos ao pool para inclusão em novo lote
- O modal `modal_lote_contas_pagar.php` no portal de Assistência gerencia pagáveis vinculados ao lote
- Prestadores filtram seus lotes por: Data emissão, Assistência, Situação, Número, Protocolo
- A Assistência filtra lotes por: Data emissão/pagamento, Prestadores, CPF/CNPJ, Situação, Número, Anexo, Protocolo, Clientes

## Edge Cases

- Prestador com alto volume: dezenas de lotes simultâneos em negociação
- Lote `AGENDADO` com data de pagamento no passado (pagamento atrasado não executado)
- Mudança de forma de pagamento de PIX para TED após lote `APROVADO`
- Acionamento contestado pelo prestador após inclusão em lote `REALIZADO`
- Lote gerado para prestador descredenciado após a criação
- Conta bancária do prestador encerrada entre `AGENDADO` e `REALIZADO`
- `time_recebimento` preenchido incorretamente (data no futuro)

## QA Notes

- **Risco crítico:** Sem integração bancária real — `REALIZADO` e `time_recebimento` dependem de ação manual. Alto risco de divergência entre sistema e extrato real
- **Risco:** Acionamento incluído em lote `CANCELADO` — a liberação de volta ao pool é automática ou manual? Sem clareza no código
- **Validation gap:** A forma de pagamento e conta bancária são validadas no momento do agendamento? Um PIX com chave inválida só falha na execução bancária
- **Risco de duplicata:** Não há evidência de validação client-side + server-side que impeça duplo-clique criar dois lotes idênticos
- **Comportamento unclear:** Se o lote tem itens com valores zerados (acionamentos sem crédito calculado) — é permitido criar o lote?
- **Edge case de compliance:** `numero` e `serie` do lote têm formato definido para fins fiscais? Sistema não integra com NF-e
- **Risco de rastreabilidade:** O log registra "alterou as informações do lote" de forma genérica — não está claro qual campo foi alterado

## Related Features

- [[agrupamento-lote]]
- [[fechamento-financeiro]]
- [[calculo-credito]]
- [[execucao-servico]]

## Related Flows

- [[fechamento-financeiro]]
- [[fluxo-calculo-credito]]
- [[execucao-atendimento]]
- [[atendimento-lifecycle]]
