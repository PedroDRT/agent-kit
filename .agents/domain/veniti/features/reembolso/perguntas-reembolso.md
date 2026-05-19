---
type: feature
module: reembolso
layer: feature
related:
  - solicitacao-reembolso
  - ocorrencias-reembolso
---

# Perguntas de Reembolso

> Novo módulo que permite configurar perguntas exibidas no fluxo de reembolso, associadas a contextos específicos do atendimento.

## Descrição

Novo módulo que permite configurar perguntas configuráveis exibidas no fluxo de reembolso. As perguntas são associadas a contextos específicos (cliente, tipo de evento, tipo de serviço, motivo de reembolso) e aparecem no modal de cadastro do reembolso quando o contexto do atendimento corresponde. Após a criação do reembolso, as perguntas ficam acessíveis em uma aba dedicada.

**Caminho de acesso:** `Configurações → Financeiro → Perguntas de reembolso`

---

## Entradas

### Permissões

| Permissão | Descrição |
|---|---|
| Cadastrar Perguntas de reembolso | Permite criar novas perguntas |
| Consultar Perguntas de reembolso | Define se o usuário visualiza o módulo |
| Editar Perguntas de reembolso | Permite editar perguntas existentes |
| Excluir Perguntas de reembolso | Permite excluir perguntas |

### Cadastro de Configuração de Perguntas

O cadastro é feito em **duas etapas**:

#### Etapa 1 — Configuração de Contexto

| Campo | Tipo |
|---|---|
| Cliente | Select |
| Tipo de evento | Select |
| Tipo de serviço | Select |
| Motivo do reembolso | Select |

Regras dos campos de configuração:
- Todos possuem a opção **Todos**
- Nenhum permite seleção múltipla
- Cada campo aceita apenas uma opção por vez
- O sistema impede cadastro de perguntas com **configurações idênticas** (duplicidade)

#### Etapa 2 — Cadastro das Perguntas

Segue o mesmo padrão do **módulo de perguntas do atendimento** já existente, com as seguintes diferenças:
- **Não é possível** definir a pergunta como obrigatória
- **Todas as perguntas são opcionais**
- **Não é possível** definir exibição para prestador
- Lógicas disponíveis: apenas **Exibir pergunta** e **Ocultar pergunta**

### Filtros de Busca

| Campo | Tipo |
|---|---|
| Cliente | Select |
| Tipo de evento | Select |
| Tipo de serviço | Select |
| Motivo do reembolso | Select |

## Saídas

- Configuração de perguntas salva e disponível para uso
- Perguntas exibidas no modal de cadastro do reembolso quando contexto corresponde
- Aba "Perguntas" disponível dentro do reembolso após criação
- Respostas registradas nas ocorrências do reembolso

---

## Regras de Negócio

### Regras de Exibição no Reembolso

- Ao iniciar um reembolso, o sistema verifica se a configuração de alguma pergunta corresponde ao contexto do atendimento
- Quando há correspondência, a pergunta é exibida no modal de cadastro
- O sistema exibe sempre a pergunta **mais específica** para o contexto
- Se uma pergunta possui **Descrição/Orientação**, o sistema exibe o modal de orientação ao responder
- Após o reembolso ser criado, as perguntas aparecem em uma **nova aba** dentro do reembolso (aba "Perguntas")
- Todas as respostas são registradas nas **ocorrências do reembolso**
- As respostas podem ser **alteradas posteriormente**, mesmo após o reembolso já ter sido criado

## Estrutura da Tela (Listagem)

O módulo exibe um **datatable** com as colunas:
- Cliente
- Tipo de evento
- Tipo de serviço
- Motivo do reembolso
- Status

### Barra de Navegação

| Aba | Função |
|---|---|
| Visualizar | Exibe datatable com perguntas cadastradas |
| Cadastrar | Abre fluxo de cadastro de perguntas |
| Procurar | Abre filtro de busca (colapsado por padrão em Visualizar) |

## Critérios de Aceitação

1. Módulo acessível em `Configurações → Financeiro → Perguntas de reembolso`
2. Datatable exibe colunas: Cliente, Tipo de evento, Tipo de serviço, Motivo do reembolso, Status
3. Campos de configuração aceitam apenas uma seleção por vez
4. Opção "Todos" disponível em todos os campos de configuração
5. Sistema impede cadastro de configurações idênticas
6. Perguntas cadastradas não permitem definir como obrigatória
7. Perguntas não possuem opção "exibir para prestador"
8. Lógicas disponíveis: apenas "Exibir pergunta" e "Ocultar pergunta"
9. Perguntas correspondentes ao atendimento aparecem no modal de cadastro do reembolso
10. Após criação do reembolso, perguntas disponíveis na aba "Perguntas"
11. Respostas registradas nas ocorrências do reembolso
12. Respostas podem ser editadas após criação do reembolso
13. Modal de orientação exibido quando a pergunta possui Descrição/Orientação
14. Sistema exibe a pergunta mais específica quando houver múltiplas configurações

---

## Dependências

- Módulo de Perguntas do Atendimento (mesmo padrão de comportamento)
- Ocorrências do reembolso (registro de respostas)
- [[solicitacao-reembolso]] — perguntas são exibidas no modal de cadastro
- [[ocorrencias-reembolso]] — respostas registradas como ocorrências

## Features Relacionadas

- [[solicitacao-reembolso]]
- [[ocorrencias-reembolso]]
