# Fluxos — Status Personalizados de Reembolso

## Visão Geral

Feature nova que permite criar **status personalizados** para reembolsos além dos status padrão do sistema. Cada status é configurável pelos operadores da Assistência e pode ser aplicado a qualquer reembolso durante seu ciclo de vida.

---

## Fluxo 1: Criar Status Personalizado

**Tela inicial:** Configurações → Financeiro → Status Reembolso
**Pré-condição:** Usuário autenticado com permissão `configuracao_escrita`
**Ator:** Operador de Assistência

### Passos Detalhados

1. Navegue até módulo Status Reembolso (`/assistencia/status-reembolso`)
2. Clique em botão **"Cadastrar"** (ou **"Novo"**)
3. **Modal abre** com campos de formulário vazios
4. Preencha campo **Nome**: `"Em Negociação"` (obrigatório, máx 100 caracteres)
   - Validação inline ao sair do campo (blur event)
   - Se vazio: exibe erro `"Nome é obrigatório"`
5. Preencha campo **Descrição** (opcional, máx 500 caracteres): `"Aguardando análise com cliente"`
6. Clique em botão **"Salvar"**
   - Botão está **desabilitado** se Nome vazio
   - **Spinner aparece** dentro do botão durante submissão
7. **Backend processa** validação adicional:
   - Verifica duplicação de nome (exibe erro se duplicado)
   - Valida comprimento dos campos
   - Registra timestamp de criação
   - Registra usuário criador
8. **Sucesso:** Toast exibido no canto superior direito com mensagem `"Status criado com sucesso"`
9. **Modal fecha automaticamente** após 1-2 segundos
10. **Datatable atualiza** exibindo novo status na lista com novo ID

### Estado Final Esperado

- Status **"Em Negociação"** aparece na datatable com:
  - ID gerado (ex: 5)
  - Nome completo
  - Descrição
  - Data de criação (timestamp)
- Novo status disponível para seleção em qualquer reembolso

### Validações Observadas

| Validação | Tipo | Quando Ocorre |
|---|---|---|
| Nome obrigatório | Inline | Ao blur do input |
| Comprimento máximo (100 chars) | Inline | Em tempo real |
| Duplicação de nome | Backend | Ao submeter formulário |
| Formato de descrição | Aceita texto livre | Sem validação especial |

### Riscos Identificados (QA Notes)

- **RN-001:** Validação de duplicação: apenas no backend ou também no frontend?
  - Risco: timeout de submissão se usuário não for notificado imediatamente
- **RN-002:** Timestamp de criação: UTC ou fuso horário do cliente?
  - Risco: divergência em reembolsos em diferentes fusos
- **RN-003:** Usuário criador: registrado corretamente?
  - Risco: auditoria incompleta se não registrar
- **RN-004:** Limite de status personalizados: existe?
  - Risco: crescimento indefinido de options

---

## Fluxo 2: Editar Status Personalizado

**Tela inicial:** Listagem de Status Reembolso
**Pré-condição:** Status já existe, não está em uso (ou permite edição mesmo em uso?)
**Ator:** Operador de Assistência

### Passos Detalhados

1. Na listagem, localize status a editar na **datatable**
2. Clique em **ícone "Editar"** (lápis) ou linha do status
3. **Modal abre** com dados atuais pré-preenchidos:
   - Nome: `"Em Negociação"`
   - Descrição: `"Aguardando análise com cliente"`
4. Altere o Nome: `"Em Negociação" → "Em Análise Interna"`
   - Validação inline ao blur
5. Clique em **"Salvar"**
   - Spinner aparece no botão
   - Backend processa validações (duplicação, formato)
6. **Sucesso:** Toast `"Status atualizado com sucesso"`
7. **Modal fecha**
8. **Datatable atualiza** imediatamente com novo nome

### Estado Final Esperado

- Status renomeado para **"Em Análise Interna"** em toda a aplicação
- Reembolsos que usam esse status refletem novo nome imediatamente
- Histórico de alteração registrado (assumindo auditoria)

### Comportamento Especial

- Botão **Salvar pode estar desabilitado** se nenhuma mudança foi feita (optimization)
- Se status está em uso (reembolsos vinculados):
  - Permite edição? (Ambiente de teste vai informar)
  - Ou bloqueia com mensagem: `"Status em uso, não pode ser alterado"`

---

## Fluxo 3: Excluir Status Personalizado

**Tela inicial:** Listagem de Status Reembolso
**Pré-condição:** Status existe
**Ator:** Operador de Assistência

### Passos Detalhados

1. Localize status na **datatable**
2. Clique em **ícone "Excluir"** (lixo/delete) na coluna de ações
3. **Modal de confirmação** exibe:
   - Ícone de aviso (triângulo com exclamação)
   - Mensagem: `"Deseja excluir o status 'Em Negociação'? Esta ação é irreversível."`
   - Botão **"Confirmar"** (vermelho/danger)
   - Botão **"Cancelar"** (neutro)
4. Clique em **"Confirmar"**
   - Spinner aparece no botão
   - Backend processa exclusão
5. **Sucesso:** Toast `"Status removido com sucesso"`
6. **Modal fecha**
7. **Datatable atualiza** (remove linha)

### Estado Final Esperado

- Status removido da listagem
- Status **não aparece mais** em nenhum dropdown de reembolso
- Reembolsos que usavam esse status: como se comportam?
  - Campo fica vazio?
  - Status padrão é aplicado?
  - Erro de referência?

### Ambiguidade Crítica (QA Notes)

- **RN-002:** Exclusão com reembolsos vinculados:
  - ❓ Bloqueia com erro: `"Não é possível excluir, status em uso"`?
  - ❓ Ou força exclusão em cascata?
  - ❓ Ou marca como `deleted_at` (soft delete)?
  - **Risco:** Data integrity se não implementado corretamente

---

## Fluxo 4: Aplicar Status em Reembolso

**Tela inicial:** Detalhes de um reembolso (`/assistencia/reembolsos/{id}`)
**Pré-condição:** Status personalizado criado, reembolso aberto
**Ator:** Operador de Assistência

### Passos Detalhados

1. Abra qualquer reembolso (clique na linha na listagem)
2. Localize card **"INFORMAÇÕES DO REEMBOLSO"**
3. Dentro do card, encontre field **Select "Status Adicional"** (ou "Status Extra", "Status Customizado")
4. Clique no select para abrir dropdown
5. **Dropdown carrega** com opções:
   - Opção vazia (placeholder): `"Selecione um status"`
   - Primeira opção: `"Em Negociação"` (do fluxo anterior)
   - Segunda opção: `"Aguardando Aprovação"` (se existir)
   - etc.
   - **Timing:** Carregamento pode levar 500ms (API call)
6. Selecione **"Em Negociação"**
7. Select agora exibe: `"Em Negociação"` como valor selecionado
8. Clique em botão **"Salvar"** (do formulário de reembolso, não do select)
   - Spinner aparece no botão
   - Backend persiste status adicional no reembolso
9. **Sucesso:** Toast `"Reembolso atualizado com sucesso"` ou similar
10. **Verificação:** Status refletido em:
    - Datatable de reembolsos (coluna "Status Adicional" se existir)
    - Próximas vezes que abrir esse reembolso
    - Relatórios e filtros

### Estado Final Esperado

- Reembolso agora tem **status base** (ex: NOTA_RECEBIDA, APROVADO) + **status adicional** (ex: "Em Negociação")
- Status adicional persiste em banco de dados
- Status adicionais podem ser:
  - Alterados (clicando no select novamente)
  - Removidos (selecionando opção vazia)

### Regras de Negócio (QA Notes)

- **RN-003:** Status adicional é **obrigatório**?
  - Fluxo esperado: não é obrigatório (pode ser deixado vazio)
- **RN-004:** Status base vs Status Adicional:
  - Status base: NOTA_RECEBIDA, APROVADO, NEGADO, CANCELADO (do sistema)
  - Status adicional: "Em Negociação", "Aguardando Aprovação" (customizado)
  - **Risco:** Confundindo usuário com dois status?
- **RN-005:** Campo desabilitado em qual status?
  - ❓ Reembolso em CANCELADO: select fica desabilitado?
  - ❓ Reembolso em FECHADO: select fica desabilitado?

---

## Fluxo 5: Filtrar Reembolsos por Status Adicional

**Tela inicial:** Listagem de reembolsos (`/assistencia/reembolsos`)
**Pré-condição:** Reembolsos com status adicionais aplicados
**Ator:** Operador de Assistência

### Passos Detalhados

1. Na listagem de reembolsos, clique em **"Procurar"** para expandir filtro
2. Aparecem campos de filtro (colapsados inicialmente)
3. Localize campo **Select "Status Adicional"** nos filtros
4. Clique no select de filtro
5. Dropdown com options:
   - Opção vazia (nenhum filtro)
   - "Em Negociação"
   - "Aguardando Aprovação"
   - etc.
6. Selecione **"Em Negociação"**
7. (Opcional) Clique em **"Pesquisar"** ou auto-apply ao selecionar
8. **Datatable atualiza** exibindo apenas reembolsos com status adicional = "Em Negociação"
9. Outros reembolsos são ocultados/removidos da visão

### Estado Final Esperado

- Filtro aplicado: status adicional = "Em Negociação"
- Datatable mostra apenas reembolsos relevantes
- Contador de registros atualiza (ex: "5 de 23 registros")
- Botão **"Limpar"** disponível para resetar filtro

### Timing

- Aplicação de filtro: 500ms a 1 segundo
- Auto-apply vs manual "Pesquisar": depende da implementação

---

## Padrões de UI Identificados

### Modal de Formulário

**Características observadas:**

- Abertura suave (fade-in 300ms)
- Campos obrigatórios marcados com asterisco `*` em vermelho
- Validação **inline** ao `blur` (deixar foco do campo)
- Mensagem de erro exibida logo **abaixo do campo** em cor vermelha
- Botão **"Salvar"** desabilitado até formulário ser 100% válido
- **Spinner** (animação de carregamento) dentro do botão durante submissão
- Fechar modal: clique em **X**, **ESC**, ou **Cancelar**
- Foco/focus order: Nome → Descrição → Botões (ordem lógica de tab)

### DataTable

**Características observadas:**

- Scroll horizontal para múltiplas colunas
- Hover na linha destaca com background claro
- Ações (editar, excluir) em **coluna fixa à direita**
- Paginação ativa para 10 ou mais registros
- Pode ter **busca inline** em cada coluna? (TBD)
- Colunas customizáveis em ordem? (TBD)

### Toast (Feedback Visual)

**Características observadas:**

- Posição fixa: **canto superior direito**
- Duração: **3 a 5 segundos** (desaparece automaticamente)
- Cores:
  - Verde: sucesso
  - Vermelho: erro
  - Azul: informação
- Mensagem clara, acionável
- Botão X para fechar manualmente (opcional)

### Select (Dropdown)

**Características observadas:**

- Inicia **vazio** com placeholder: `"Selecione um status"`
- Setas **↑ ↓** para navegação (acessibilidade)
- Pode ter **busca por digitação**? (combobox behavior, TBD)
- Options agrupadas por tipo? (TBD)
- Ícone de dropdown (seta para baixo)

---

## Anti-Padrões Observados (Riscos)

| Anti-Padrão | Risco | Ação em QA |
|---|---|---|
| Modal com ESC pode não funcionar | Usuário preso no modal | Testar ESC em cada modal |
| Validação só no frontend | Bypass possível via API direta | Testar validação via API |
| Seletores por `nth-child` | Quebram após reorder da tabela | Usar seletores mais robustos |
| Classes dinâmicas `.status-123` | ID muda com cada status | Evitar em automação |
| Sem atributos `data-testid` | Seletores CSS muito frágeis | Requisitar `data-testid` |
| Confirmação apenas visual | Exclusão acidental possível | Testar dupla confirmação |
| Reload completo após ação | Perde estado da página | Verificar se é SPA ou reload |
| Sem log de auditoria | Rastreabilidade perdida | Verificar se registra criador/editor/deletor |

---

## Ambiguidades Identificadas (Requerem Investigação)

| Pergunta | Impacto | Ação Recomendada |
|---|---|---|
| Validação de duplicação de nome ocorre no frontend ou backend? | UX: timeout esperado vs resposta rápida | Testar submit com nome duplicado, verificar timing |
| Status adicional é **obrigatório** em reembolso? | Fluxo: pode deixar vazio ou não? | Testar reembolso sem status adicional, verificar requisitos |
| Exclusão com reembolsos vinculados: bloqueia ou força? | Integridade: reembolsos órfãos? | Testar: criar reembolso, aplicar status, tentar deletar |
| Campo desabilitado em qual status do reembolso? | Fluxo: em FECHADO? CANCELADO? | Testar select em cada status do reembolso |
| Confirmação via Salvar ou auto-persist? | UX: usuário clica Salvar ou é automático? | Testar aplicação imediata vs via botão Salvar |
| Limite de status personalizados existe? | Performance: quantos status no select? | Testar com 100+ status, verificar paginação |
| Reembolsos em uso: permite edição/exclusão? | Data integrity | Testar editar/deletar status em uso |
| Timestamp: UTC ou fuso horário local? | Relatório: datas corretas? | Verificar fusohg com cliente em outro timezone |
| Usuário criador registrado? | Auditoria: rastreabilidade | Verificar logs de auditoria |

---

## Fluxos Críticos para Cobertura E2E

### Lifecycle Completo: Reembolso + Status Personalizado

1. **Setup:** Criar 2 status personalizados: "Em Negociação", "Aguardando Aprovação"
2. **Pré-requisito:** Reembolso em status NOTA_RECEBIDA (beneficiário enviou nota)
3. **Ação 1:** Abrir reembolso, aplicar status "Em Negociação", salvar
4. **Validação:** Status aparece em datatable e detalhes do reembolso
5. **Ação 2:** Mudar para status "Aguardando Aprovação"
6. **Validação:** Mudança refletida imediatamente
7. **Ação 3:** Filtrar reembolsos por status "Aguardando Aprovação"
8. **Validação:** Apenas reembolsos com esse status aparecem
9. **Ação 4:** Editar nome do status "Aguardando Aprovação" → "Em Aprovação"
10. **Validação:** Mudança refletida em reembolso aberto e em filtros
11. **Ação 5:** Tentar deletar status "Em Negociação"
12. **Validação:** Confirmar comportamento (bloqueia ou força?)

---

## Seletores CSS Mapeados

Para detalhes completos de seletores, consultar: `/knowledge/system/selectors/status-personalizados-reembolso.json`

**Resumo:**

- `button:has-text('Cadastrar')` — criar novo status
- `button:has-text('Procurar')` — expandir filtro
- `table` — datatable de status
- `div[role='dialog']` — modal de cadastro/edição
- `input[name='nome']` — campo nome
- `textarea[name='descricao']` — campo descrição
- `button:has-text('Salvar')` — salvar alterações
- `div[role='alertdialog']` — modal de confirmação
- `select[name='status_adicional']` — select em reembolso

---

## Regras de Negócio (RN) Extraídas

| ID | Regra | Prioridade |
|---|---|---|
| RN-ST-001 | Nome de status é obrigatório (máx 100 caracteres) | 🔴 Crítico |
| RN-ST-002 | Status com mesmo nome não pode ser duplicado | 🔴 Crítico |
| RN-ST-003 | Descrição é opcional (máx 500 caracteres) | 🟡 Médio |
| RN-ST-004 | Status adicional em reembolso é **opcional** | 🟡 Médio |
| RN-ST-005 | Usuário deve ter permissão `configuracao_escrita` para criar/editar/deletar | 🔴 Crítico |
| RN-ST-006 | Timestamp de criação é registrado automaticamente | 🟡 Médio |
| RN-ST-007 | Usuário criador é registrado para auditoria | 🟡 Médio |
| RN-ST-008 | Exclusão com reembolsos vinculados: [TBD no ambiente] | 🔴 Crítico |
| RN-ST-009 | Edição de status reflete em todos os reembolsos que o usam | 🔴 Crítico |
| RN-ST-010 | Status adicional persiste no reembolso mesmo após renovação de outros campos | 🟡 Médio |

---

## Casos de Teste Esperados

### Cobertura de Cenários

| Tipo | Quantidade | Exemplos |
|---|---|---|
| **Critérios de Aceitação** | 5+ | Nome obrigatório, duplicação, persisting |
| **Caminhos Felizes** | 3 | Create → Edit → Delete |
| **Cenários Negativos** | 8+ | Vazio, duplicado, caracteres especiais, bloqueios |
| **Transições de Status** | 5+ | Aplicar, alterar, remover em reembolso |
| **Integrações** | 2 | Reembolso + Status, Filtro + Status |
| **Limites / Edge Cases** | 5+ | Máx 100 chars, soft delete, cascade |

**Total esperado:** 25-30 casos de teste

---

**Fim do Fluxo**

---

## Referências

- Documentação do sistema: `/knowledge/system/00-index.md`
- Seletores mapeados: `/knowledge/system/selectors/status-personalizados-reembolso.json`
- Padrões de teste: `/knowledge/test-patterns/common-patterns.md`
- Regras de automação: `/knowledge/automation-rules/playwright-best-practices.md`
