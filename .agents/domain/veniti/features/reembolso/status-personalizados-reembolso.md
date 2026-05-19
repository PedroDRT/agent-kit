---
type: feature
module: reembolso
layer: feature
related:
  - fluxo-reembolso-completo
---

# Status Personalizados de Reembolso

> Módulo que permite configurar status complementares personalizados para reembolsos, sem alterar os status padrão do sistema.

## Descrição

Módulo que permite configurar **status complementares personalizados** para reembolsos, sem alterar os status padrão do sistema. Possibilita que a equipe de assistência classifique o andamento dos reembolsos com status adicionais para controle interno.

**Caminho de acesso:** `Configurações → Financeiro → Status reembolso`

---

## Entradas

### Permissões

| Permissão | Descrição |
|---|---|
| Cadastrar Status personalizados de reembolso | Permite criar novos status |
| Consultar Status personalizados de reembolso | Controla visibilidade do módulo |
| Editar Status personalizados de reembolso | Permite editar status existentes |
| Excluir Status personalizados de reembolso | Permite excluir status |

### Cadastro de Status

Campos:
- **Nome** (obrigatório, máx 100–255 caracteres)
- **Descrição** (opcional)

## Saídas

- Novo status disponível para seleção no card "INFORMAÇÕES DO REEMBOLSO" de qualquer reembolso
- Nova coluna "Status adicional" na listagem de reembolsos
- Filtro de busca por status adicional disponível na listagem

---

## Regras de Negócio

- Os status padrão do sistema **não devem ser alterados**
- Novos status funcionam como **status complementar** (adicional)
- Status cadastrados podem ser utilizados para **controle interno da assistência**
- Status com mesmo nome não pode ser duplicado — erro "Status com este nome já existe"
- Nomes de status padrão (RASCUNHO, ANÁLISE, APROVADO, PENDENTE, FINALIZADO) são rejeitados com "Nome reservado"
- Exclusão bloqueada quando há reembolsos vinculados — mensagem "Status possui reembolsos vinculados"
- Status sem vínculos pode ser excluído normalmente
- Trim aplicado antes de salvar
- Caracteres especiais (@, #, &, !) aceitos no nome
- Alteração de nome reflete imediatamente no select do reembolso

### Estrutura de Dados

```
reembolso.status_adicional: string (nullable)
  - Armazena o ID ou nome do status personalizado selecionado
  - Pode ser NULL se não foi selecionado
```

```
ReembolsoStatus {
  id: int,
  nome: string,
  descricao: string (opcional),
  cliente_id: int (se aplicável),
  ativo: boolean,
  criado_em: timestamp,
  atualizado_em: timestamp
}
```

## Critérios de Aceitação

- **CA-001:** Módulo acessível em `Configurações → Financeiro → Status reembolso`
- **CA-002:** Deve exibir datatable com colunas: **Cliente, Tipo de evento, Tipo de serviço, Motivo do reembolso, Status**
- **CA-003:** Deve possuir opções padrão de exportação do sistema
- **CA-004:** Deve permitir personalização de colunas
- **CA-005:** Sistema deve criar 4 novas permissões (Cadastrar, Consultar, Editar, Excluir)
- **CA-006:** Dado que o usuário acessa o módulo, quando clicar em "Cadastrar", então deve permitir criar um novo status personalizado
- **CA-007:** Dado que um status foi criado, quando o reembolso for aberto, então deve aparecer em um campo select dentro do card "INFORMAÇÕES DO REEMBOLSO"
- **CA-008:** Dado que o usuário selecionou um status, quando salvar, então o sistema deve armazenar o status complementar
- **CA-009:** Dado que múltiplos status foram cadastrados, quando listar reembolsos, então deve exibir nova coluna "Status adicional"
- **CA-010:** Dado que um reembolso não possui status adicional, quando exibido na lista, então a coluna deve mostrar "--"
- **CA-011:** Dado que o usuário acessa o filtro da listagem de reembolsos, quando selecionar um status personalizado, então deve retornar apenas reembolsos com esse status
- **CA-012:** Os status padrão do sistema **não devem ser alterados**
- **CA-013:** Novos status funcionam como **status complementar** (adicional)
- **CA-014:** Status cadastrados podem ser utilizados para **controle interno da assistência**

## Comportamentos Confirmados

| Comportamento | Confirmado | Caso de teste |
|---|---|---|
| Unicidade de nome obrigatória — erro "Status com este nome já existe" | ✅ Sim | CT003.2, CT004.3 |
| Nome de status padrão rejeitado — erro "Nome reservado" | ✅ Sim | CT010.2 |
| Status padrão não aparecem no módulo CRUD | ✅ Sim | CT010.1 |
| Exclusão bloqueada quando há reembolsos vinculados | ✅ Sim | CT005.3 — mensagem "Status possui reembolsos vinculados" |
| Trim aplicado antes de salvar | ✅ Sim | CT011.1 |
| Campo Descrição é opcional | ✅ Sim | CT011.3 |
| Limite de nome: 100–255 caracteres | ✅ Sim | CT003.4 |
| Limite de descrição: ~1000 caracteres | ✅ Sim | CT011.2 |
| Caracteres especiais (@, #, &, !) aceitos no nome | ✅ Sim | CT003.5 |
| Alteração de nome reflete imediatamente no select do reembolso | ✅ Sim | CT012.2, CT012.3 |
| Persistência após logout/login | ✅ Sim | CT012.1 |
| Sem permissão Consultar → acesso negado ao módulo | ✅ Sim | CT001.2, CT009.3 |
| Apenas permissão Consultar → botões CRUD desabilitados | ✅ Sim | CT009.2 |
| Permissão Cadastrar ausente → botão "Cadastrar" desabilitado | ✅ Sim | CT003.6 |
| Filtro colapsado por padrão em Visualizar | ✅ Sim | CT008.5 — exibe apenas botão "Procurar" inicialmente |
| Status select vazio inicialmente (sem pré-seleção) | ✅ Sim | CT006.3 |
| Isolamento entre reembolsos — sem cross-contamination | ✅ Sim | CT006.5 |
| Reembolso sem status exibe "--" na coluna | ✅ Sim | CT007.2 |
| Personalização de colunas persiste após refresh | ✅ Sim | CT002.4 |

---

## Escopo de Teste

1. **Acesso ao módulo** — navegação até `Configurações → Financeiro → Status reembolso`
2. **Criação de status** — cadastro de novo status personalizado
3. **Exibição no reembolso** — campo select no card INFORMAÇÕES DO REEMBOLSO
4. **Seleção e persistência** — salvar status selecionado
5. **Coluna em datatable** — exibição de nova coluna "Status adicional"
6. **Filtro de busca** — filtrar reembolsos por status adicional
7. **Permissões** — validar 4 novas permissões controlam acesso
8. **Validação de status padrão** — status padrão do sistema não são alterados
9. **Exibição de "--"** — quando status não está preenchido
10. **Edição e exclusão** — modificar ou remover status cadastrados

## Fora do Escopo

- Lógica condicional baseada em status adicional (comportamentos condicionais posteriores)
- Integração com relatórios específicos que possam usar status adicional
- Sincronização com sistemas externos de status personalizado
- Notificações baseadas em mudança de status adicional
- Histórico de alterações de status adicional (apenas register em ocorrências — será validado em escopo de ocorrências)

---

## Notas de QA

- **Filtro case-insensitive:** Comportamento de busca por status adicional (LIKE case-insensitive ou exact match) não foi documentado formalmente.
- **Herança em templates — NÃO testado:** Reembolsos criados de templates — comportamento de herança do status adicional fora do escopo deste ciclo.
- **Ordem de exibição no select — NÃO documentado:** Ordem dos status no select do reembolso não foi especificada nem testada formalmente.

## Dependências

- **Módulo de Reembolso:** Feature completa de reembolso deve estar funcional
- **Módulo de Atendimento:** Reembolsos são vinculados a atendimentos
- **Sistema de Permissões:** 4 novas permissões devem ser criadas no sistema
- **Datatable:** Componente de lista com coluna adicional

## Features Relacionadas

- [[fluxo-reembolso-completo]]
