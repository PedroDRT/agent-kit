# Status Personalizados de Reembolso

## Resumo

Módulo que permite configurar **status complementares personalizados** para reembolsos, sem alterar os status padrão do sistema. Possibilita que a equipe de assistência classifique o andamento dos reembolsos com status adicionais para controle interno.

---

## Critérios de Aceitação

### Módulo de Configuração

- **CA-001:** Módulo acessível em `Configurações → Financeiro → Status reembolso`
- **CA-002:** Deve exibir datatable com colunas: **Cliente, Tipo de evento, Tipo de serviço, Motivo do reembolso, Status**
- **CA-003:** Deve possuir opções padrão de exportação do sistema
- **CA-004:** Deve permitir personalização de colunas

### Permissões

- **CA-005:** Sistema deve criar 4 novas permissões:
  - Cadastrar Status personalizados de reembolso
  - Consultar Status personalizados de reembolso (controla visibilidade do módulo)
  - Editar Status personalizados de reembolso
  - Excluir Status personalizados de reembolso

### Cadastro de Status

- **CA-006:** Dado que o usuário acessa o módulo, quando clicar em "Cadastrar", então deve permitir criar um novo status personalizado
- **CA-007:** Dado que um status foi criado, quando o reembolso for aberto, então deve aparecer em um campo select dentro do card "INFORMAÇÕES DO REEMBOLSO"
- **CA-008:** Dado que o usuário selecionou um status, quando salvar, então o sistema deve armazenar o status complementar
- **CA-009:** Dado que múltiplos status foram cadastrados, quando listar reembolsos, então deve exibir nova coluna "Status adicional"
- **CA-010:** Dado que um reembolso não possui status adicional, quando exibido na lista, então a coluna deve mostrar "--"

### Filtro de Busca

- **CA-011:** Dado que o usuário acessa o filtro da listagem de reembolsos, quando selecionar um status personalizado, então deve retornar apenas reembolsos com esse status

### Regras Gerais

- **CA-012:** Os status padrão do sistema **não devem ser alterados**
- **CA-013:** Novos status funcionam como **status complementar** (adicional)
- **CA-014:** Status cadastrados podem ser utilizados para **controle interno da assistência**

---

## Escopo de Teste

✅ **Será testado:**

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

---

## Fora do Escopo

❌ **Não será testado neste ciclo:**

- Lógica condicional baseada em status adicional (comportamentos condicionais posteriores)
- Integração com relatórios específicos que possam usar status adicional
- Sincronização com sistemas externos de status personalizado
- Notificações baseadas em mudança de status adicional
- Histórico de alterações de status adicional (apenas register em ocorrências — será validado em escopo de ocorrências)

---

## Dependências e Premissas

### Dependências Externas

- **Módulo de Reembolso:** Feature completa de reembolso deve estar funcional
- **Módulo de Atendimento:** Reembolsos são vinculados a atendimentos
- **Sistema de Permissões:** 4 novas permissões devem ser criadas no sistema
- **Datatable:** Componente de lista com coluna adicional

### Premissas

- ✅ Usuário possui credenciais válidas com permissão `Consultar Status personalizados de reembolso`
- ✅ Reembolsos já existem na base de dados para filtro
- ✅ Sistema usa padrão `Todos` para campos genéricos (já validado em outros módulos)
- ✅ Campo select será posicionado no card "INFORMAÇÕES DO REEMBOLSO" (localização fixa definida)

---

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

## Pontos de Atenção (QA Notes)

### Ambiguidades Resolvidas pelos Testes

1. **Unicidade de nome — CONFIRMADO:** Sistema rejeita nomes duplicados com erro "Status com este nome já existe" (CT003.2). Nomes de status padrão (RASCUNHO, ANÁLISE, APROVADO, PENDENTE, FINALIZADO) são rejeitados com "Nome reservado" (CT010.2).

2. **Persistência do status — CONFIRMADO:** Status persiste por reembolso após salvar, sobrevive a logout/login e refresh (CT012.1, CT012.3). Estrutura de dados: campo `status_adicional` na tabela `reembolsos` (confirmado via comportamento).

3. **Exclusão com reembolsos vinculados — CONFIRMADO:** Exclusão bloqueada com mensagem "Status possui reembolsos vinculados" (CT005.3). Status sem vínculos pode ser excluído normalmente.

4. **Status padrão protegidos — CONFIRMADO:** RASCUNHO, ANÁLISE, APROVADO, PENDENTE e FINALIZADO não aparecem na listagem do módulo CRUD (CT010.1).

5. **Herança em templates — NÃO testado:** Reembolsos criados de templates — comportamento de herança do status adicional fora do escopo deste ciclo.

6. **Ordem de exibição no select — NÃO documentado:** Ordem dos status no select do reembolso não foi especificada nem testada formalmente.

### Ambiguidade Aberta

- **Filtro case-insensitive:** Comportamento de busca por status adicional (LIKE case-insensitive ou exact match) não foi documentado formalmente.

### Riscos Resolvidos

| Risco | Status | Observação |
|-------|--------|-----------|
| Usuário sem permissão vê módulo | ✅ Resolvido | Permissão `Consultar` bloqueia acesso — confirmado CT001.2 |
| Status não persiste após salvar | ✅ Resolvido | Persistência confirmada inclusive após logout/login |
| Coluna não aparece em datatable | ✅ Resolvido | Coluna "Status adicional" confirmada, exibe "--" quando vazio |
| Status padrão alterado acidentalmente | ✅ Resolvido | Status padrão protegidos — não aparecem no módulo CRUD |

---

## Fluxos Relacionados

- **Fluxo de Reembolso Completo:** Status adicional é complemento, não substitui status padrão
- **Gestão de Reembolso:** Seleção de status acontece dentro do reembolso já aberto
- **Listagem de Reembolsos:** Novo filtro e coluna para status adicional

---

## Estrutura de Dados Esperada

### Campo no Reembolso

```
reembolso.status_adicional: string (nullable)
  - Armazena o ID ou nome do status personalizado selecionado
  - Pode ser NULL se não foi selecionado
```

### Entidade Status Personalizado

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

---


