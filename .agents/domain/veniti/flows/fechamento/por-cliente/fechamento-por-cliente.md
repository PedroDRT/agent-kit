# Flow — Fechamento Financeiro por Cliente

## Description

Descreve o processo de encerramento financeiro do período entre o Veniti (Assistência) e o Cliente (seguradora). A Assistência consolida os créditos do período, negocia descontos e emite a nota/fatura ao cliente.

**Portal:** `/assistencia/fechamento/cliente/`  
**Ator principal:** Operação financeira (Assistência)

---

## FLUXO A: Fechamento com o Cliente

### A1. Consolidação de Créditos do Período

- **Ator:** Sistema
- **Trigger:** Operação inicia fechamento do período em `/assistencia/fechamento/cliente/`
- **Dados consolidados:** Todos os créditos gerados por `CalcularValorCredito` no período para aquele cliente
- **Cálculo:** Inclui valor por atendimento, por KM, por tipo de veículo conforme configuração do plano
- **Resultado:** Lista de atendimentos com valores individuais e total do período

### A2. Definição de Descontos (Opcional)

- **Ator:** Operação
- **Ação:** Define desconto negociado com o cliente (se aplicável)
- **Resultado:** `Valor Total - Desconto = Valor Fatura`

### A3. Geração da Nota/Fatura

- **Sistema:** Gera número de nota (`numero_nota`) e data de emissão
- **Resultado:** Documento de cobrança com: Nota número, Cliente, Período, Valor inicial, Desconto, Total

### A4. Fechamento (`FECHADO`)

- **Sistema:** Status do fechamento → `FECHADO`
- **Resultado:** Valor fixado — cliente notificado sobre a fatura do período
- **Disponível em:** Portal Cliente (`/cliente/faturamento/`)

---

## Entry Points

| Portal | Ator | Trigger |
|---|---|---|
| `/assistencia/fechamento/cliente/` | Operação financeira | Início do período de fechamento com cliente |

## Exit Points

| Saída | Condição |
|---|---|
| Fechamento com cliente `FECHADO` | Fatura emitida ao cliente |

## Variations

### Contestação após Fechamento

- Cliente questiona valor na fatura após `FECHADO`
- Processo não mapeado no sistema — requer intervenção manual

---

## QA Notes

- **Risco:** Verificar se o desconto negociado é validado para não ultrapassar o valor total
- **Edge case:** Período sem atendimentos — fechamento gera nota com valor zero ou é bloqueado?
- **Risco fiscal:** `numero_nota` gerado sem integração com NF-e — conformidade fiscal não coberta pelo sistema

## Related Features

- [[fechamento-por-cliente]]
- [[calculo-credito]]

## Related Flows

- [[fluxo-calculo-credito]]
- [[atendimento-lifecycle]]
