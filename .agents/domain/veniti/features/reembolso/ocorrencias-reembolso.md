---
type: feature
module: reembolso
layer: feature
related:
  - tags-reembolso
  - perguntas-reembolso
  - status-personalizados-reembolso
---

# Ocorrências do Reembolso

> O reembolso suporta cadastro manual de ocorrências e registro automático de ações relevantes, seguindo o mesmo padrão das ocorrências de atendimento.

## Descrição

O reembolso passou a suportar **cadastro manual de ocorrências**, seguindo o mesmo padrão das ocorrências de atendimento. Além disso, diversas ações do reembolso são **registradas automaticamente** como ocorrências.

---

## Entradas

### Cadastro Manual de Ocorrências

Dentro de um reembolso aberto, foi disponibilizado o botão **"+ Cadastrar Ocorrência"**, que abre um modal com o campo:
- **Descrição** (textarea, obrigatório)

## Saídas

- Ocorrência registrada no histórico do reembolso após cadastro manual
- Ocorrências automáticas geradas pelas ações listadas abaixo
- Aba **"Ocorrências"** exibe histórico completo (manuais e automáticas)

---

## Regras de Negócio

- O comportamento de cadastro manual é idêntico ao módulo de ocorrências dos atendimentos
- Ao clicar em **Cadastrar**, a ocorrência é registrada no histórico do reembolso

## Registro Automático de Ocorrências

As seguintes ações geram ocorrências automaticamente no reembolso:

| Ação | Registrado nas Ocorrências? |
|---|---|
| Inserção de Tag | ✅ Sim |
| Remoção de Tag | ✅ Sim |
| Alteração do Status Adicional | ✅ Sim |
| Edição/resposta de Perguntas | ✅ Sim |
| Inserção de Limite de reembolso | ✅ Sim |
| Inserção de Valor excedente | ✅ Sim |

## Critérios de Aceitação

1. Botão "Cadastrar Ocorrência" disponível dentro do reembolso
2. Ao clicar no botão, um modal é exibido com campo Descrição
3. Ao salvar, a ocorrência aparece no histórico do reembolso
4. Inserção/remoção de tags gera ocorrência automática
5. Alteração do status adicional gera ocorrência automática
6. Edição de respostas de perguntas gera ocorrência automática
7. Inserção do limite de reembolso gera ocorrência automática
8. Inserção/edição de valor excedente gera ocorrência automática

---

## Dependências

- [[tags-reembolso]] — registro de tags
- [[perguntas-reembolso]] — registro de respostas
- [[status-personalizados-reembolso]] — registro de status adicional
- Ocorrências do Atendimento (mesmo padrão de comportamento)
