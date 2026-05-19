# Fechamento Financeiro — Por Cliente

## Description

O **Fechamento com o Cliente** é a perspectiva do fechamento financeiro em que a Assistência (Veniti) consolida os créditos consumidos pelo cliente (seguradora) em um determinado período e emite a nota/fatura correspondente. Opera em `/assistencia/fechamento/cliente/`.

- **Quem inicia:** Assistência (operação interna)
- **O que fecha:** Créditos consumidos pelo cliente no período (atendimentos realizados)
- **Resultado:** Nota/fatura emitida ao cliente com o valor a ser pago ao Veniti

## Inputs

| Campo | Tipo | Descrição |
|---|---|---|
| `id_cliente` | integer | Cliente a ser fechado |
| `periodo_inicio` | date | Data de início do período |
| `periodo_fim` | date | Data de fim do período |
| `desconto` | decimal | Desconto negociado (opcional) |
| `numero_nota` | string | Número da nota fiscal gerada |

## Outputs

- Nota de faturamento gerada (número + série)
- Exibe: Valor inicial, Desconto negociado, Total final
- Status progride para `FECHADO`
- Histórico de fechamento disponível no portal Cliente

## Business Rules

- O fechamento consolida créditos calculados por `CalcularValorCredito` no período para aquele cliente
- O `numero_nota` é o identificador contábil do fechamento com o Cliente
- Desconto negociado é opcional — `Valor Total - Desconto = Valor Fatura`
- Após `FECHADO`, os valores são fixados — contestação requer intervenção manual (processo não mapeado)

## Edge Cases

- Cliente questiona nota após `FECHADO` — processo de contestação não mapeado
- Fechamento duplicado para o mesmo período e cliente
- Créditos calculados incorretamente por `CalcularValorCredito` — impacto direto no valor da fatura

## QA Notes

- **Risco:** Verificar se o desconto negociado é validado para não ultrapassar o valor total (desconto negativo não deve ser permitido)
- **Edge case:** Período sem atendimentos — fechamento gera nota com valor zero? Ou é bloqueado?
- **Risco de auditoria:** Mudanças no valor do fechamento antes de `FECHADO` devem ser rastreadas com usuário e timestamp

## Dependencies

- **Portal:** `html/assistencia/fechamento/cliente/` (fechamento com cliente)
- **Portal Cliente:** `html/cliente/faturamento/` (visualização da fatura)
- **Use Cases:** `src/UseCases/CalcularValorCredito.php`, `src/UseCases/CalcularMetricas.php`
- **Modelos:** `src/Models/Faturamento.php`
- **Banco:** `clientes_contratos_creditos`

## Related Flows

- [[fechamento-financeiro]] (por-cliente)
- [[fluxo-calculo-credito]]

## Related Features

- [[calculo-credito]]
- [[execucao-servico]]
