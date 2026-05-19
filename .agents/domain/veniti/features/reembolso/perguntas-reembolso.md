# Perguntas de Reembolso

## Resumo

Novo módulo que permite configurar perguntas configuráveis exibidas no fluxo de reembolso. As perguntas são associadas a contextos específicos (cliente, tipo de evento, tipo de serviço, motivo de reembolso) e aparecem no modal de cadastro do reembolso quando o contexto do atendimento corresponde. Após a criação do reembolso, as perguntas ficam acessíveis em uma aba dedicada.

---

## Acesso ao Módulo

**Caminho:** `Configurações → Financeiro → Perguntas de reembolso`

---

## Permissões

Quatro novas permissões são necessárias:

| Permissão | Descrição |
|---|---|
| Cadastrar Perguntas de reembolso | Permite criar novas perguntas |
| Consultar Perguntas de reembolso | Define se o usuário visualiza o módulo |
| Editar Perguntas de reembolso | Permite editar perguntas existentes |
| Excluir Perguntas de reembolso | Permite excluir perguntas |

---

## Estrutura da Tela (Listagem)

O módulo exibe um **datatable** com as colunas:
- Cliente
- Tipo de evento
- Tipo de serviço
- Motivo do reembolso
- Status

O datatable suporta:
- Opções padrão de exportação do sistema
- Personalização de colunas

### Barra de Navegação

| Aba | Função |
|---|---|
| Visualizar | Exibe datatable com perguntas cadastradas |
| Cadastrar | Abre fluxo de cadastro de perguntas |
| Procurar | Abre filtro de busca (colapsado por padrão em Visualizar) |

---

## Filtros de Busca

| Campo | Tipo |
|---|---|
| Cliente | Select |
| Tipo de evento | Select |
| Tipo de serviço | Select |
| Motivo do reembolso | Select |

---

## Cadastro de Configuração de Perguntas

O cadastro é feito em **duas etapas**:

### Etapa 1 — Configuração de Contexto

Campos:
- **Cliente**
- **Tipo de evento**
- **Tipo de serviço**
- **Motivo do reembolso**

Regras dos campos de configuração:
- Todos possuem a opção **Todos**
- Nenhum permite seleção múltipla
- Cada campo aceita apenas uma opção por vez
- O sistema impede cadastro de perguntas com **configurações idênticas** (duplicidade)

Após preencher e clicar em **Salvar**, o sistema abre a Etapa 2.

### Etapa 2 — Cadastro das Perguntas

O comportamento segue o mesmo padrão do **módulo de perguntas do atendimento** já existente na plataforma, com as seguintes diferenças:

- **Não é possível** definir a pergunta como obrigatória
- **Todas as perguntas são opcionais**
- **Não é possível** definir exibição para prestador
- Lógicas disponíveis: apenas **Exibir pergunta** e **Ocultar pergunta**

---

## Regras de Exibição no Reembolso

### No modal de cadastro do reembolso

- Ao iniciar um reembolso, o sistema verifica se a configuração de alguma pergunta corresponde ao contexto do atendimento
- Quando há correspondência, a pergunta é exibida no modal de cadastro
- O sistema exibe sempre a pergunta **mais específica** para o contexto

### Após o reembolso ser criado

- As perguntas aparecem em uma **nova aba** dentro do reembolso (aba "Perguntas")

### Modal de orientação

- Se uma pergunta possui **Descrição/Orientação**, o sistema exibe o modal de orientação ao responder, seguindo o comportamento do módulo de perguntas do atendimento

---

## Registro das Respostas

- Todas as respostas são registradas nas **ocorrências do reembolso**
- As respostas podem ser **alteradas posteriormente**, mesmo após o reembolso já ter sido criado

---

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

## Pontos de Atenção

- Regra de "mais específica": quando há múltiplas configurações que correspondem ao contexto (ex: uma com cliente específico e outra com "TODOS"), o sistema deve exibir a mais específica
- Comportamento exato da duplicidade de configurações deve ser validado na UI
