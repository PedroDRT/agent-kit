---
type: feature
module: relatorios
layer: feature
related: []
---

# Feature: Relatório de Contratos por Cliente

> Relatório no portal Assistência que consolida contratos dos clientes com datas de vigência e status, com suporte a filtros combinados e exportação.

## Descrição

Relatório no portal Assistência que consolida contratos dos clientes com datas de vigência e status. Permite filtrar por cliente, código, situação e status, combinar critérios e exportar dados — apoiando gestão contratual com identificação clara de contratos ativos, vencidos e por situação.

**Portal:** Assistência (`/assistencia/relatorios/contratoxcliente/`)
**Módulo:** Relatórios
**Status:** Implementado e validado — ciclo QA completo em 2026-04-10 (33 casos, Run 10)

---

## Entradas

### Filtros Disponíveis

| Filtro | Tipo | Valores |
|---|---|---|
| Cliente | Select2 multi-select | Todos os clientes cadastrados |
| Código do contrato | input texto livre | Suporta busca parcial |
| Situação | Select2 | Em vigência / Vencido |
| Status | Select2 | Ativo / Inativo |

Filtros combináveis em AND lógico.

## Saídas

- DataTable com colunas: **Código do contrato**, **Vigência de**, **Vigência até**, **Status**
- Linha de agrupamento por cliente (row group): Razão Social + CNPJ antes dos contratos daquele cliente
- Exportação em XLS e CSV (PDF: desejável, não obrigatório)
- Botão de gerenciamento de colunas (Altere as colunas)

---

## Regras de Negócio

- Situação calculada automaticamente: `vigencia_ate < data_atual` = Vencido; `vigencia_ate >= data_atual` = Em vigência
- Filtros combináveis em AND lógico — retorna apenas registros que atendem a todos os critérios
- Permissão **"Consultar relatório de contratos"** criada e funcional
- Usuários sem permissão não visualizam o relatório (menu oculto e URL direta bloqueada no backend)
- Perfil **ADMIN**: permissão habilitada por padrão, não removível (botão Salvar sempre desabilitado na tela de perfil ADMIN)
- Outros perfis: adição e remoção manual da permissão possível e persistente

## Critérios de Aceitação

1. Relatório **"Contratos"** acessível no menu lateral (módulo Relatórios), item "Contrato por clientes"
2. Lista todos os contratos cadastrados no sistema
3. Colunas: **Código do contrato**, **Vigência de**, **Vigência até**, **Status**
4. DataTable com **linha de agrupamento por cliente** (row group): Razão Social + CNPJ antes dos contratos daquele cliente
5. Filtro por **Cliente** (Select2 multi-select com todos os clientes)
6. Filtro por **Código do contrato** (input texto livre — suporta busca parcial)
7. Filtro por **Situação** (Select2: Em vigência / Vencido)
8. Filtro por **Status** (Select2: Ativo / Inativo — status do contrato, não do cliente)
9. Filtros **combináveis** (AND lógico)
10. Situação calculada automaticamente: `vigencia_ate < data_atual` = Vencido; `vigencia_ate >= data_atual` = Em vigência
11. Exportação em **XLS e CSV** (obrigatório). PDF: desejável, não obrigatório
12. Botão de **gerenciamento de colunas** (Altere as colunas)
13. Permissão **"Consultar relatório de contratos"** criada e funcional
14. Usuários **sem permissão** não visualizam o relatório (menu oculto e URL direta bloqueada no backend)
15. Perfil **ADMIN**: permissão habilitada por padrão, não removível (botão Salvar sempre desabilitado na tela de perfil ADMIN)
16. **Outros perfis**: adição e remoção manual da permissão possível e persistente

## Comportamentos Confirmados (QA Run 10)

| Comportamento | Confirmado | Observação |
|---|---|---|
| Situação `>=` inclui `vigencia_ate = hoje` em "Em vigência" | ✅ Sim | Boundary confirmado por CT001.5 |
| Bloqueio de acesso via URL direta (backend, não só menu) | ✅ Sim | Retorna mensagem "não possui autorização" |
| Perfil ADMIN: botão Salvar sempre disabled | ✅ Sim | Independente de interações no checkbox |
| Permissão persiste após reload | ✅ Sim | Requer `page.reload()` após salvar para refletir sessão PHP |
| Menu requer nova navegação para refletir remoção de permissão | ✅ Sim | Comportamento esperado — menu renderizado a cada carregamento |
| Select2 requer jQuery API para interação automática | ✅ Sim | `selectOption()` do Playwright não funciona — ver Seção de Automação |
| DataTables row grouping é assíncrono | ✅ Sim | `tr.dtrg-group` renderizado após DOM inicial — aguardar `.first()` visible |
| Acesso sem autenticação redireciona para `/assistencia/login/` | ✅ Sim | Redirect HTTP 3xx confirmado via request fixture |

## Escopo de Teste

- Controle de acesso (permissão por perfil, menu, URL direta, sem autenticação)
- Exibição e estrutura do relatório (colunas, row grouping, breadcrumb)
- Filtros individuais (Cliente, Código, Situação, Status)
- Combinação de filtros (AND lógico)
- Estado vazio (mensagem "Nenhum resultado encontrado!")
- Exportação XLS e CSV
- Gerenciamento de colunas
- Persistência de permissão (add/remove) em perfil não-ADMIN

## Fora do Escopo

- Criação, edição ou exclusão de contratos
- Exportação em PDF (desejável mas não obrigatória)
- Outros portais (Cliente, Prestador, Beneficiário, Reembolso)
- Gestão completa de perfis além da nova permissão

## Notas de Automação

### Select2 — Interação Obrigatória via jQuery

Os filtros de Cliente, Situação e Status usam Select2. `page.locator().selectOption()` **não funciona** — o `<select>` está oculto.

```js
// Método correto via jQuery API
await page.evaluate((value) => {
  $('#contratos_clientes_situacao').val(value).trigger('change');
}, '1'); // '1' = Em vigência, '0' = Vencido, '' = todos

// Aguardar commit ANTES de clicar em Filtrar (evitar race condition)
await page.waitForFunction((sel, expected) => {
  return window.$(sel).val() == expected;
}, ['#contratos_clientes_situacao', '1']);
```

### Styled Checkboxes — Permissões

O checkbox `#edit_profile_view_report_contractxclients` é estilizado (coberto por `label`). Clicar no checkbox diretamente não funciona.

```js
// Correto: clicar na label
await page.locator('label[for="edit_profile_view_report_contractxclients"]').click();
```

### Overlay #preloading

A página de edição de perfil exibe overlay de carregamento que intercepta cliques.

```js
await page.locator('#preloading').waitFor({ state: 'hidden', timeout: 15000 }).catch(() => {});
```

### Persistência de Permissão (Sessão PHP)

Após `#bt_editar_perfil.click()` + `waitForLoadState`, a sessão PHP pode ainda refletir o estado anterior. Obrigatório fazer `page.reload()`:

```js
await page.locator('#bt_editar_perfil').click();
await page.waitForLoadState('domcontentloaded');
await page.locator('#preloading').waitFor({ state: 'hidden', timeout: 15000 }).catch(() => {});
await page.reload();
await page.waitForLoadState('domcontentloaded');
await page.locator('#preloading').waitFor({ state: 'hidden', timeout: 15000 }).catch(() => {});
```

### Acesso Sem Autenticação — Usar request Fixture

Criar `browser.newContext()` sem cookies causa `ERR_CONNECTION_REFUSED` no ambiente local (limite de conexões TCP). Usar `request` fixture do Playwright:

```js
test('acesso sem autenticação', async ({ request }) => {
  const response = await request.get(RELATORIO_URL, {
    maxRedirects: 0,
    failOnStatusCode: false
  });
  expect(response.status()).toBeGreaterThanOrEqual(300);
  expect(response.status()).toBeLessThan(400);
  expect(response.headers()['location'] ?? '').toMatch(/login/);
});
```

### DataTables Row Grouping Assíncrono

```js
// Aguardar primeiro grupo antes de contar
const grupoRows = page.locator('#datatable_contratos_clientes tr.dtrg-group');
await expect(grupoRows.first()).toBeVisible();
const count = await grupoRows.count(); // seguro após .first() visible
```

---

## Notas de QA

- **Risco resolvido:** Acesso via URL sem permissão bloqueia no backend (não apenas frontend via menu) — confirmado por CT001.3.
- **Edge case confirmado:** `vigencia_ate = hoje` classificado como "Em vigência" — critério `>=` confirmado no Run 10.
- **Ambiguidade aberta:** Contrato sem data de vigência preenchida (`vigencia_ate = NULL`) — comportamento não documentado, sem dado de teste, sem cenário automatizado. Registrar CT_NEW_001.
- **UX observada (não é bug):** Após salvar permissão, a UI não exibe toast de confirmação. Requer `page.reload()` para confirmar persistência. Candidato para melhoria (RN-GLOBAL-002).
- **Ordenação de colunas:** Nenhum caso testa clicar no cabeçalho para reordenar (CT_NEW_002) — baixa prioridade.
- **Breadcrumb corrigido:** Bug de breadcrumb corrigido em 2026-04-10 (agora exibe "Relatórios / Contratos por Cliente").

## Dependências

**Dados criados no ambiente em 2026-04-10 (necessários para testes):**

| Dado | Cliente | Detalhes | Finalidade |
|---|---|---|---|
| CONTRATO VENCIDO | AGENTECLI | `vigencia_ate < 2026-04-10` | CT001.5 (filtro Vencido) |
| CONTRATO FINAL DATA ATUAL | AGENTECLI | `vigencia_ate = 2026-04-10` | CT001.5 boundary (vigência hoje) |
| CONTRATO INATIVO | AGENTECLI | `status = Inativo` | CT006.2 (filtro Status Inativo) |

> Atenção: `CONTRATO FINAL DATA ATUAL` tem `vigencia_ate = 2026-04-10`. A partir de **2026-04-11**, este contrato passa a ser "Vencido". Atualizar o dado ou criar estratégia de dado dinâmico para CT001.5.

**Clientes com contratos no ambiente (confirmados):**

| ID | Nome | Contratos conhecidos |
|---|---|---|
| 4143 | LUCASCLI (46.902.443/0001-79) | PET_CLI_COD, RESI_CLI_COD, AUTO_CLI_COD |
| 4144 | GEOVANNA CLIENTE | 008, 007 |
| 4439 | AUTOMACAOCLI | CONTRATO_RESI, CONTRATO_AUTO |
| 4445 | LIMITACAOCLI | BENEFICIARIO, AUTO |
| 4448 | CREDITOCLI | RESI, AUTO |
| 4453 | EZZE SEGUROS | 1 |
| 4470 | BRADESCO | BRAD |
| 4481 | ALFANUMERICO | nenhum (usar para estado vazio) |
| 4482 | NUMERICO | nenhum |
| 4483 | AGENTECLI | AUTO, CONTRATO VENCIDO, CONTRATO FINAL DATA ATUAL, CONTRATO INATIVO |

**Perfis:**

| Perfil | ID | Tem permissão | Salvar bloqueado |
|---|---|---|---|
| ADMIN | 0 | Sim (padrão) | Sempre |
| AGENTS | 284 | Sim | Não |

**Credenciais (de `/config/environment.json`):** `users.agent1`, `users.agent2`, `users.agent3` — todos no perfil AGENTS.
