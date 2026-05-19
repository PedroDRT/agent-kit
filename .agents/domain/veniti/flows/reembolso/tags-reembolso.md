---
type: flow
module: reembolso
layer: flow
related:
  - tags-reembolso
  - fluxo-reembolso-completo
---

# Fluxos — Tags no Reembolso

> Descreve os fluxos para configurar tags para exibição no reembolso, herdar tags do atendimento e adicionar/remover tags manualmente em reembolsos.

## Descrição

Descreve os fluxos de interação para configurar tags para exibição no reembolso, para herdar tags do atendimento no momento da criação do reembolso e para adicionar ou remover tags manualmente em reembolsos existentes.

---

## Fluxo

### Fluxo 1: Configurar Tag para Exibição no Reembolso

**Tela inicial:** `Configurações → Operação → Tags`
**Pré-condição:** Usuário com permissão de edição de tags

1. Acessar `Configurações → Operação → Tags`
2. Criar nova tag ou editar existente
3. No campo **Tipo de Evento**, selecionar:
   - **`Reembolso`** — tag aparece APENAS no reembolso, não nos atendimentos
   - **`TODOS`** — tag aparece em atendimentos E no reembolso
   - Outras opções → tag NÃO aparece no reembolso
4. Preencher os demais campos (Cor de Destaque, Nome)
5. Clicar em **Salvar**

**Estado final esperado:** Tag configurada com o tipo de evento correto; exibição condicional aplicada conforme tipo selecionado.

### Fluxo 2: Herança de Tags do Atendimento no Reembolso

**Tela inicial:** Atendimento com tags vinculadas → criação de reembolso
**Pré-condição:** Atendimento possui tags com Tipo de Evento = 'TODOS' vinculadas

1. Acessar atendimento que possui tags com Tipo = 'TODOS' vinculadas
2. Editar → Ação → Reembolso
3. Confirmar abertura do reembolso
4. No modal de cadastro, observar o seletor de tags
5. Tags com Tipo = 'TODOS' vinculadas ao atendimento aparecem **pré-selecionadas**
6. Tags com Tipo = 'Reembolso' vinculadas ao atendimento aparecem disponíveis mas **não selecionadas**
7. Criar o reembolso

**Estado final esperado:** Reembolso criado com tags herdadas pré-selecionadas; campo de tags posicionado no card "INFORMAÇÕES DO REEMBOLSO".

### Fluxo 3: Adicionar/Remover Tags no Reembolso

**Tela inicial:** Reembolso aberto → card "INFORMAÇÕES DO REEMBOLSO"
**Pré-condição:** Reembolso criado

1. Abrir reembolso existente
2. Localizar campo **Tags** no card "INFORMAÇÕES DO REEMBOLSO"
3. Clicar no campo de tags para abrir o seletor
4. Selecionar/deselecionar tags (apenas tags com Tipo = 'Reembolso' ou 'TODOS' aparecem)
5. Salvar alterações

**Estado final esperado:** Tag adicionada ou removida do reembolso; ocorrência automática registrada na aba Ocorrências do reembolso.

---

## Pontos de Entrada

- `Configurações → Operação → Tags` (configuração de tipo de evento)
- Atendimento → Editar → Ação → Reembolso (herança no momento da criação)
- Reembolso → Card "INFORMAÇÕES DO REEMBOLSO" (adição/remoção manual)

## Pontos de Saída

- Tag configurada com visibilidade condicional aplicada
- Reembolso criado com tags herdadas pré-selecionadas
- Tags adicionadas/removidas com ocorrências registradas

## Variações

### Padrões de UI

- Campo de tags posicionado no card **"INFORMAÇÕES DO REEMBOLSO"** (Editar tab do reembolso)
- Seletor de tags segue mesmo padrão visual existente no sistema
- Tags filtradas automaticamente por tipo de evento (Reembolso/TODOS) — usuário não vê tags de outros tipos
- Inserção/remoção registradas automaticamente nas ocorrências

---

## Notas de QA

- [[RN-RBL-012]] — Tags filtradas por tipo de evento
- [[RN-RBL-013]] — Herança apenas para tags com Tipo = 'TODOS'
- [[RN-RBL-016]] — Inserção/remoção registradas nas ocorrências

## Features Relacionadas

- [[tags-reembolso]]
- [[fluxo-reembolso-completo]]
