---
type: flow
module: financeiro
layer: flow
related:
  - edicao-em-massa
  - faturamento-lote
---

# Fluxos — Edição em Massa de Contas a Pagar

> Descreve os fluxos de interação para execução de ações em lote na listagem de Contas a Pagar: Liquidar e Agendar títulos individualmente ou em grupo.

## Descrição

Descreve os fluxos de interação para execução de ações em lote na listagem de Contas a Pagar: Liquidar (título único e lote) e Agendar (1 ou múltiplos títulos). Todos os fluxos partem da listagem de Contas a Pagar no Portal Assistência e dependem de permissão de edição no perfil do usuário.

---

## Fluxo

### Fluxo 1 — Liquidar Título Único

**Pré-condição:** Usuário autenticado com permissão de edição de contas a pagar; ao menos 1 título disponível na listagem

1. Usuário acessa a listagem de Contas a Pagar no Portal Assistência
2. Usuário clica no **checkbox** da primeira coluna de exatamente **1 título** — a barra flutuante aparece na base da tela exibindo quantidade (1) e valor do item
3. Usuário clica no botão **"Liquidar"** na barra flutuante
4. Sistema abre o modal **"Liquidar"** com: Quantidade (1), Valor (R$) do item, campo Data de pagamento (pré-preenchida com a data atual), campo Valor pago, e campos Desconto e Juros (bloqueados para edição)
5. Usuário preenche o campo **Valor pago** — ao sair do campo, o sistema calcula automaticamente Desconto (se valor < original) ou Juros (se valor > original)
6. Usuário confirma ou altera a **Data de pagamento**
7. Usuário clica no botão **"Liquidar"** no modal
8. Sistema processa a liquidação, exibe toast de sucesso, recarrega o datatable e limpa seleção e barra flutuante

**Estado final:** Título liquidado com data e valores registrados; barra flutuante ocultada; datatable atualizado

### Fluxo 2 — Liquidar Múltiplos Títulos (Lote)

**Pré-condição:** Usuário com permissão de edição; ao menos 2 títulos disponíveis na listagem

1. Usuário seleciona **2 ou mais títulos** via checkboxes — barra flutuante exibe quantidade (N) e soma total dos valores
2. Usuário clica em **"Liquidar"** na barra flutuante
3. Sistema abre o modal **"Liquidar"** de lote com: Quantidade (N), Valor total (R$) e apenas o campo **Data de pagamento** — campos Valor pago, Juros e Desconto não aparecem
4. Modal exibe que todos os títulos serão liquidados pelo valor integral original de cada um
5. Usuário informa ou confirma a **Data de pagamento**
6. Usuário clica no botão **"Liquidar"** no modal
7. Sistema liquida todos os títulos selecionados com a data informada e os valores integrais originais; exibe toast de sucesso; recarrega datatable; limpa seleção e barra flutuante

**Estado final:** Todos os títulos selecionados liquidados com a mesma data; barra flutuante ocultada

### Fluxo 3 — Agendar Pagamento (1 ou Múltiplos Títulos)

**Pré-condição:** Usuário com permissão de edição; ao menos 1 título disponível na listagem

1. Usuário seleciona **1 ou mais títulos** via checkboxes — barra flutuante aparece com quantidade e soma total
2. Usuário clica em **"Agendar"** na barra flutuante
3. Sistema abre o modal **"Agendar"** com: Quantidade, Valor total (R$) e campo **Data de agendamento** (pré-preenchida com a data atual)
4. Usuário informa a **Data de agendamento**
5. Usuário clica no botão **"Agendar"** no modal
6. Sistema aplica a data informada a todos os itens selecionados; exibe toast de sucesso; recarrega datatable; limpa seleção e barra flutuante

**Estado final:** Todos os títulos selecionados com a mesma data de agendamento; barra flutuante ocultada

### Fluxo 4 — Cancelamento de Modal

**Pré-condição:** Modal "Liquidar" ou "Agendar" aberto com 1 ou mais itens selecionados

1. Usuário clica no botão **X** do modal ou clica fora da área do modal
2. Modal fecha sem executar nenhuma ação
3. A seleção de checkboxes e a barra flutuante são **preservadas** exatamente como estavam

**Estado final:** Nenhuma alteração nos títulos; seleção mantida; barra flutuante ainda visível

---

## Pontos de Entrada

| Ponto | Ator | Trigger |
|---|---|---|
| Seleção via checkbox individual | Operação financeira | Clique no checkbox da linha |
| Seleção via checkbox de cabeçalho | Operação financeira | Clique no "selecionar todos" do cabeçalho do datatable |

## Pontos de Saída

| Saída | Resultado |
|---|---|
| Liquidação confirmada (1 item) | Título liquidado; data, valor, juros e desconto registrados |
| Liquidação confirmada (lote) | Todos os títulos liquidados com data informada e valor integral de cada um |
| Agendamento confirmado | Todos os títulos recebem a data de agendamento informada |
| Modal cancelado | Nenhuma alteração; seleção preservada |
| Paginação / filtro / ordenação | Seleção e barra flutuante limpas automaticamente |

## Variações

### Regras de Negócio

- Barra flutuante aparece com ≥ 1 item selecionado e some quando nenhum está selecionado
- Juros e Desconto são calculados automaticamente com base no Valor Pago; bloqueados para edição manual
- Em lote, o valor de cada título é sempre o valor integral original — não editável
- Seleção não persiste ao paginar, filtrar ou ordenar
- Permissão de edição de contas a pagar controla visibilidade e execução da feature inteira

## Features Relacionadas

- [[edicao-em-massa]]
- [[faturamento-lote]]
