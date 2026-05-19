# Ocorrências do Reembolso

## Resumo

O reembolso passou a suportar **cadastro manual de ocorrências**, seguindo o mesmo padrão das ocorrências de atendimento. Além disso, diversas ações do reembolso são **registradas automaticamente** como ocorrências.

---

## Cadastro Manual de Ocorrências

Dentro de um reembolso aberto, foi disponibilizado o botão **"+ Cadastrar Ocorrência"**, que abre um modal com o campo:
- **Descrição** (textarea, obrigatório)

Ao clicar em **Cadastrar**, a ocorrência é registrada no histórico do reembolso.

O comportamento é idêntico ao módulo de ocorrências dos atendimentos.

---

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

---

## Aba Ocorrências no Reembolso

A aba **"Ocorrências"** no reembolso exibe o histórico completo de ocorrências (manuais e automáticas), na mesma estrutura do histórico de atendimentos.

---

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
