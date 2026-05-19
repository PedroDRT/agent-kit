---
type: feature
module: financeiro
layer: feature
related:
  - fluxo-edicao-em-massa
  - faturamento-lote
---

# Edição em Massa — Contas a Pagar

> Adiciona ações em lote à listagem de títulos de Contas a Pagar, permitindo Liquidar e Agendar múltiplos registros sem abrir cada item individualmente.

## Descrição

A **Edição em Massa** no módulo Contas a Pagar adiciona ações em lote à listagem de títulos, permitindo **Liquidar** e **Agendar** múltiplos registros sem abrir cada item individualmente. A funcionalidade opera por meio de uma coluna de seleção (checkbox) inserida como primeira coluna do datatable e de uma **barra de ação flutuante** exibida quando há ao menos um título selecionado.

- **Módulo:** Contas a Pagar — Portal Assistência
- **Quem usa:** Operação financeira com permissão de edição de contas a pagar

**Contexto:** Antes desta feature, agendar ou liquidar títulos era uma ação unitária — exigia abrir cada título individualmente, navegar até a tela de edição e salvar. Com alto volume de títulos, o processo se tornava operacionalmente lento.

---

## Entradas

| Elemento | Tipo | Obrigatório | Descrição |
|---|---|---|---|
| Seleção de itens | checkbox (datatable) | — | Um ou múltiplos títulos selecionados na listagem |
| Data de pagamento | date (dd/mm/aaaa) | Sim | Usada nos modais Liquidar (1 item e lote) |
| Valor pago | decimal | Sim (1 item) | Ausente no modal de lote — valor integral de cada item é usado automaticamente |
| Juros | decimal | Não | Calculado automaticamente; campo bloqueado para edição manual |
| Desconto | decimal | Não | Calculado automaticamente; campo bloqueado para edição manual |
| Data de agendamento | date (dd/mm/aaaa) | Sim | Usada no modal Agendar |

## Saídas

- **Liquidar (1 item):** título liquidado com a data e valor informados; juros/desconto calculados automaticamente com base na diferença
- **Liquidar (lote):** todos os títulos selecionados liquidados com a data informada e o valor integral original de cada um
- **Agendar (1 ou mais itens):** todos os títulos selecionados recebem a mesma data de agendamento informada
- Após qualquer ação bem-sucedida: toast de sucesso exibido, datatable recarregado, seleção e barra flutuante limpas

---

## Regras de Negócio

### Seleção e Barra Flutuante
- A coluna de checkbox é a **primeira coluna** do datatable
- A barra flutuante aparece ao selecionar ao menos 1 item e desaparece quando todos são desmarcados
- A barra exibe: quantidade de itens selecionados e soma total dos valores (precisão financeira)
- A seleção **não persiste** ao paginar, filtrar ou ordenar — barra e checkboxes são limpos automaticamente

### Liquidar — 1 item
- Modal exibe: Data de pagamento (obrigatório, pré-preenchida com a data atual), Valor pago (obrigatório), Desconto e Juros (bloqueados)
- `valor pago < valor original` → desconto = diferença; juros = 0,00
- `valor pago > valor original` → juros = diferença; desconto = 0,00
- `valor pago = valor original` → juros = 0,00; desconto = 0,00
- Campos Juros e Desconto são **bloqueados para edição manual** no front-end
- Ao alterar o Valor Pago novamente, o cálculo é refeito

### Liquidar — múltiplos itens
- Modal exibe apenas: Data de pagamento (obrigatório) e resumo (quantidade + valor total)
- Campos Valor Pago, Juros e Desconto **não aparecem** no modal de lote
- O valor de cada título é automaticamente considerado como seu valor integral original
- Todos os títulos são liquidados com a mesma data informada

### Agendar — 1 ou múltiplos itens
- Modal único com campo: Data de agendamento (obrigatório) e resumo (quantidade + valor total)
- A mesma data é aplicada a todos os itens selecionados

### Permissões
- Respeita a permissão de **edição de contas a pagar** já existente no perfil do usuário
- **Sem permissão de edição:** coluna de checkbox e barra flutuante não são exibidas; ações em massa não podem ser executadas
- **Com permissão de edição:** acesso completo à feature

## Casos de Borda

- Seleção mista de títulos com status `pago`, `agendado` e `vencido` — qualquer combinação é aceita; não há bloqueio por status
- Fechar modal (botão X ou clique externo) sem confirmar: nenhuma alteração é aplicada; seleção é preservada
- Modal de lote não exibe campos de valor — impossível editar valor individualmente por item via ação em massa
- O valor exibido na listagem e somado na barra flutuante já contempla juros e descontos existentes no título

---

## Notas de QA

- **Verificar bloqueio de campos:** Juros e Desconto devem estar disabled no front-end; verificar também se o back-end rejeita/ignora envio manual desses valores
- **Precisão financeira:** Somar múltiplos valores decimais pode gerar erros de ponto flutuante — verificar CT003.2
- **Recálculo de juros/desconto:** Alterar o Valor Pago uma segunda vez deve recalcular corretamente — verificar CT004.7
- **Regressão crítica:** Clicar em célula da linha (fora do checkbox) deve abrir o detalhe do título — verificar CT002.1
- **Permissão:** Cenário sem permissão (CT001.2) deve ser validado manualmente; automação cobre apenas perfil com permissão
- **Não persistência de seleção:** Paginar e ordenar devem limpar seleção e barra flutuante — verificar CT003.5 e CT003.6

## Dependências

- **Portal:** Listagem de Contas a Pagar — Portal Assistência
- **Permissão:** permissão de edição de contas a pagar vinculada ao perfil do usuário
- **Massa de dados:** títulos em diferentes status com valores conhecidos (ver `knowledge/test-data/test-assets.md`)

## Flows Relacionados

- [[fluxo-edicao-em-massa]]
- [[faturamento-lote]]

## Features Relacionadas

- [[faturamento-lote]]
