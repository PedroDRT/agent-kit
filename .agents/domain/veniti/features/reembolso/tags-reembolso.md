# Tags no Reembolso

## Resumo

O módulo de Tags foi atualizado para suportar o fluxo de reembolso. Tags podem ser configuradas para exibição no reembolso e, quando vinculadas ao atendimento, podem ser herdadas automaticamente no momento da criação do reembolso.

> **Nota de implementação:** O PDF original (item 3.5) descreve a adição de um **checkbox "Exibir no reembolso"** no cadastro de tags. A implementação real utilizou uma abordagem diferente: a opção **'Reembolso'** foi adicionada ao select de **Tipo de Evento** do cadastro/edição de tags. Ver seção "Implementação Real" abaixo.

---

## Alteração no Cadastro/Edição de Tags

### Especificação do PDF (Documentação Original)

Seria criado um novo checkbox no cadastro/edição de tags:
- **"Exibir no reembolso"**

Somente tags com esse checkbox marcado poderiam ser exibidas e vinculadas no reembolso.

### Implementação Real (Conforme Verificado)

Em vez de um checkbox, foi adicionada a opção **'Reembolso'** no select de **Tipo de Evento** do cadastro/edição de tags.

Caminho: `Configurações → Operação → Tags`

**Tipos de Evento disponíveis (após a alteração):**

| Tipo de Evento | Exibido no Reembolso? | Exibido nos Atendimentos? |
|---|---|---|
| `TODOS` | Sim | Sim |
| `Reembolso` | Sim | **Não** |
| Demais opções | Não | Sim |

---

## Regras de Exibição no Reembolso

| Tipo de Evento da Tag | Vinculada ao Atendimento? | Disponível no Reembolso? | Pré-selecionada (Herança)? |
|---|---|---|---|
| `Reembolso` | Sim | Sim | **Não** |
| `Reembolso` | Não | Sim | Não |
| `TODOS` | Sim | Sim | **Sim** |
| `TODOS` | Não | Sim | Não |
| Outros | — | **Não** | Não |

---

## Herança de Tags do Atendimento

A herança ocorre **apenas** quando:
- O Tipo de Evento da tag é `TODOS` **E**
- A tag está vinculada ao atendimento

Nesse caso, ao criar um reembolso para esse atendimento, a tag vem **automaticamente selecionada**.

Tags com Tipo de Evento = `Reembolso` vinculadas ao atendimento **não** herdam automaticamente.

---

## Comportamento Visual e Funcional

- O campo de tags no reembolso fica posicionado no card **"INFORMAÇÕES DO REEMBOLSO"**
- É possível **adicionar/remover** tags manualmente no reembolso após a criação
- Inserção e remoção de tags no reembolso **devem ser registradas nas ocorrências** do reembolso
- Tags com Tipo de Evento = `Reembolso` **não aparecem** na listagem de tags de atendimentos

---

## Critérios de Aceitação

1. Tag com Tipo de Evento = `Reembolso` deve aparecer disponível no seletor de tags do reembolso
2. Tag com Tipo de Evento = `Reembolso` **não** deve aparecer na listagem de tags de atendimentos
3. Tag com Tipo de Evento = outros (exceto `TODOS` e `Reembolso`) **não** deve aparecer no seletor de tags do reembolso
4. Tag com Tipo de Evento = `TODOS` vinculada ao atendimento deve vir pré-selecionada ao criar o reembolso (herança)
5. Tag com Tipo de Evento = `TODOS` **não** vinculada ao atendimento deve aparecer disponível, mas **não** pré-selecionada
6. Tag com Tipo de Evento = `Reembolso` vinculada ao atendimento **não** deve vir pré-selecionada (sem herança)
7. Inserção manual de tag no reembolso deve ser registrada nas ocorrências
8. Remoção manual de tag no reembolso deve ser registrada nas ocorrências
9. O campo de tags deve estar posicionado no card "INFORMAÇÕES DO REEMBOLSO"
10. As opções existentes de Tipo de Evento (além de `Reembolso`) devem continuar funcionando normalmente (regressivo)

---

## Dependências

- `Configurações → Operação → Tags` — cadastro/edição de tags
- [[ocorrencias-reembolso]] — registro de inserção/remoção de tags
- [[solicitacao-reembolso]] — campo de tags no card INFORMAÇÕES DO REEMBOLSO

## Pontos de Atenção

- Tags com Tipo de Evento = `TODOS` não vinculadas ao atendimento aparecem disponíveis no seletor do reembolso, mas sem pré-seleção (confirmado)
- Tags com Tipo de Evento = `Reembolso` vinculadas ao atendimento NÃO herdam automaticamente (confirmado)
- É possível adicionar/remover tags no reembolso após criação (confirmado)
