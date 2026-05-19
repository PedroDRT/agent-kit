---
type: flow
module: reembolso
layer: flow
related:
  - status-personalizados-reembolso
  - fluxo-reembolso-completo
---

# Fluxos — Status Personalizados de Reembolso

> Descreve os fluxos para criar, editar, excluir e aplicar status personalizados em reembolsos.

## Descrição

Descreve os fluxos de interação para criar, editar, excluir e aplicar status personalizados em reembolsos. Feature que permite criar **status personalizados** além dos status padrão do sistema, configuráveis pelos operadores da Assistência e aplicáveis a qualquer reembolso durante seu ciclo de vida.

---

## Fluxo

### Fluxo 1: Criar Status Personalizado

**Tela inicial:** Configurações → Financeiro → Status Reembolso
**Pré-condição:** Usuário autenticado com permissão `configuracao_escrita`

1. Navegue até módulo Status Reembolso (`/assistencia/status-reembolso`)
2. Clique em botão **"Cadastrar"**
3. **Modal abre** com campos de formulário vazios
4. Preencha campo **Nome** (obrigatório, máx 100 caracteres) — validação inline ao blur
5. Preencha campo **Descrição** (opcional, máx 500 caracteres)
6. Clique em **"Salvar"** — botão desabilitado se Nome vazio; spinner aparece durante submissão
7. Backend processa: verifica duplicação, valida comprimento, registra timestamp e usuário criador
8. Toast de sucesso exibido; modal fecha; datatable atualiza com novo status

**Estado final esperado:** Novo status aparece na datatable e disponível para seleção em qualquer reembolso.

#### Validações Observadas

| Validação | Tipo | Quando Ocorre |
|---|---|---|
| Nome obrigatório | Inline | Ao blur do input |
| Comprimento máximo (100 chars) | Inline | Em tempo real |
| Duplicação de nome | Backend | Ao submeter formulário |
| Formato de descrição | Aceita texto livre | Sem validação especial |

### Fluxo 2: Editar Status Personalizado

**Tela inicial:** Listagem de Status Reembolso

1. Na listagem, localize status a editar na datatable
2. Clique em **ícone "Editar"** (lápis) ou linha do status
3. Modal abre com dados atuais pré-preenchidos
4. Altere o Nome e/ou Descrição — validação inline ao blur
5. Clique em **"Salvar"** — spinner aparece; backend processa validações
6. Toast de sucesso; modal fecha; datatable atualiza com novo nome

**Estado final esperado:** Status renomeado em toda a aplicação; reembolsos que usam esse status refletem novo nome imediatamente.

### Fluxo 3: Excluir Status Personalizado

**Tela inicial:** Listagem de Status Reembolso

1. Localize status na datatable
2. Clique em **ícone "Excluir"** na coluna de ações
3. **Modal de confirmação** exibe mensagem de aviso e botões Confirmar / Cancelar
4. Clique em **"Confirmar"** — spinner aparece; backend processa exclusão
5. Toast de sucesso; modal fecha; datatable atualiza (remove linha)

**Estado final esperado:** Status removido da listagem e de todos os dropdowns de reembolso.

### Fluxo 4: Aplicar Status em Reembolso

**Tela inicial:** Detalhes de um reembolso (`/assistencia/reembolsos/{id}`)

1. Abra qualquer reembolso
2. Localize card **"INFORMAÇÕES DO REEMBOLSO"**
3. Encontre field **Select "Status Adicional"**
4. Clique no select para abrir dropdown com as opções de status personalizados
5. Selecione o status desejado
6. Clique em botão **"Salvar"** do formulário de reembolso
7. Toast de sucesso; status persiste no reembolso

**Estado final esperado:** Reembolso com status base + status adicional; visível em datatable, detalhes e filtros.

### Fluxo 5: Filtrar Reembolsos por Status Adicional

**Tela inicial:** Listagem de reembolsos (`/assistencia/reembolsos`)

1. Clique em **"Procurar"** para expandir filtros
2. Localize campo **Select "Status Adicional"** nos filtros
3. Selecione o status desejado
4. Clique em **"Pesquisar"**
5. Datatable atualiza exibindo apenas reembolsos com aquele status adicional

**Estado final esperado:** Filtro aplicado; datatable mostra apenas reembolsos relevantes; botão "Limpar" disponível.

---

## Pontos de Entrada

- `Configurações → Financeiro → Status Reembolso` (CRUD de status)
- Reembolso → Card "INFORMAÇÕES DO REEMBOLSO" → Select Status Adicional (aplicação)
- Listagem de reembolsos → Filtro Status Adicional (consulta)

## Pontos de Saída

- Status criado e disponível para seleção em todos os reembolsos
- Status aplicado persistido no reembolso
- Lista filtrada por status adicional

## Variações

### Padrões de UI

- **Modal de Formulário:** Abertura fade-in 300ms; campos obrigatórios marcados com `*`; validação inline ao blur; spinner durante submissão; fechamento via X, ESC ou Cancelar
- **DataTable:** Hover destaca linha; ações em coluna fixa à direita; paginação para 10+ registros
- **Toast:** Posição fixa canto superior direito; duração 3-5 segundos; verde (sucesso), vermelho (erro), azul (informação)
- **Select:** Inicia vazio com placeholder; navegação por setas ↑↓

---

## Notas de QA

### Riscos Identificados

| Anti-Padrão | Risco | Ação em QA |
|---|---|---|
| Validação de duplicação só no backend | Timeout se não notificado imediatamente | Testar submit com nome duplicado, verificar timing |
| Timestamp de criação: UTC ou fuso? | Divergência em diferentes fusos | Verificar fuso com cliente em outro timezone |
| Exclusão com reembolsos vinculados | Bloqueia ou força exclusão em cascata? | Testar: criar reembolso, aplicar status, tentar deletar |
| Campo desabilitado em qual status? | Em CANCELADO? FECHADO? | Testar select em cada status do reembolso |
| Limite de status personalizados | Performance: quantos status no select? | Testar com 100+ status |

### Regras de Negócio Extraídas

| ID | Regra |
|---|---|
| RN-ST-001 | Nome de status é obrigatório (máx 100 caracteres) |
| RN-ST-002 | Status com mesmo nome não pode ser duplicado |
| RN-ST-003 | Descrição é opcional (máx 500 caracteres) |
| RN-ST-004 | Status adicional em reembolso é **opcional** |
| RN-ST-005 | Usuário deve ter permissão `configuracao_escrita` para criar/editar/deletar |
| RN-ST-009 | Edição de status reflete em todos os reembolsos que o usam |

### Referências

- Seletores mapeados: `/knowledge/system/selectors/status-personalizados-reembolso.json`

## Features Relacionadas

- [[status-personalizados-reembolso]]
- [[fluxo-reembolso-completo]]
