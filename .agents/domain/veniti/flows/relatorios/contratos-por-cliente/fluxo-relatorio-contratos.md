---
type: flow
module: relatorios
layer: flow
related:
  - contratos-por-cliente
---

# Fluxos — Relatório de Contratos por Cliente

> Descreve os fluxos de acesso, filtragem e gerenciamento de permissão no Relatório de Contratos por Cliente do portal Assistência.

## Descrição

Descreve os fluxos de interação para acessar e operar o Relatório de Contratos por Cliente, incluindo aplicação de filtros via jQuery API (Select2), acesso sem autenticação, bloqueio por permissão e gerenciamento de permissão por perfil.

**URL:** `/assistencia/relatorios/contratoxcliente/`
**Validado em:** 2026-04-10 (Run 10 — 32/32 passed)

---

## Fluxo

### Fluxo 1 — Acesso via menu lateral (usuário com permissão)

**Pré-condição:** Usuário autenticado, perfil AGENTS com permissão "Consultar relatório de contratos"

1. Clicar em "Relatórios" no menu lateral
2. Aguardar expansão do submenu
3. Clicar em `a[href="/assistencia/relatorios/contratoxcliente/"]` ("Contrato por clientes")

**Estado final:** URL `/assistencia/relatorios/contratoxcliente/`, tabela `#datatable_contratos_clientes` visível, breadcrumb "Relatórios / Contratos por Cliente".

### Fluxo 2 — Acesso direto por URL (autenticado com permissão)

1. Navegar diretamente para `/assistencia/relatorios/contratoxcliente/`

**Estado final:** `#btn_contratos_clientes_filtrar` visível, tabela carregada, sem mensagem de acesso negado.

### Fluxo 3 — Acesso sem autenticação (RN-GLOBAL-001)

1. Acessar diretamente `/assistencia/relatorios/contratoxcliente/`

**Estado final:** Redirect HTTP 3xx para `/assistencia/login/`.

> Automação: Usar `request` fixture do Playwright (não `browser.newContext()` — causa `ERR_CONNECTION_REFUSED` no ambiente local).

### Fluxo 4 — Acesso sem permissão bloqueado (usuário autenticado)

**Pré-condição:** Usuário autenticado com perfil AGENTS com permissão **removida**

**Setup (remover permissão):**
1. Navegar para `/assistencia/usuario_perfil/editar.php?id=284`
2. Aguardar `#preloading` hidden
3. Clicar em `label[for="edit_profile_view_report_contractxclients"]` (não clicar no checkbox diretamente)
4. Clicar em `#bt_editar_perfil`
5. Aguardar `domcontentloaded` + `#preloading` hidden
6. **Executar `page.reload()`** + aguardar carregamento completo ← obrigatório para sessão PHP refletir

**Teste:**
7. Navegar para `/assistencia/dashboard/` (garantir commit do DB)
8. Expandir menu Relatórios → verificar ausência de `a[href="/assistencia/relatorios/contratoxcliente/"]`
9. Navegar para `/assistencia/relatorios/contratoxcliente/`
10. Verificar que `#datatable_contratos_clientes` NÃO está visível
11. Verificar que `main` contém "não possui autorização"

**Teardown obrigatório:** Restaurar permissão repetindo os passos com checkbox desmarcado → marcar e salvar.

> Lição aprendida (H013): O `page.reload()` no passo 6 é obrigatório. Sem ele, a sessão PHP mantém o estado anterior e o menu/acesso não reflete a alteração salva no DB.

### Fluxo 5 — Filtro por cliente

1. Setar valor do Select2 via jQuery:
   ```js
   await page.evaluate(() => { $('#contratos_clientes_id_cliente').val('4143').trigger('change'); });
   await page.waitForFunction(() => window.$('#contratos_clientes_id_cliente').val() == '4143');
   ```
2. Clicar em `#btn_contratos_clientes_filtrar`
3. Aguardar `#datatable_contratos_clientes_processing` desaparecer + tbody visível

**Estado final:** Apenas 1 grupo `tr.dtrg-group` com o cliente selecionado e seus contratos.

### Fluxo 6 — Filtro por código de contrato

1. `page.locator('#contratos_clientes_id_contrato').fill('AUTO_CLI_COD')`
2. Clicar em `#btn_contratos_clientes_filtrar`

**Estado final:** Tabela exibe apenas contratos cujo código contém "AUTO_CLI_COD".

### Fluxo 7 — Filtro por situação

**Valores Select2:** `'1'` = Em vigência | `'0'` = Vencido | `''` = Todos

```js
await page.evaluate(() => { $('#contratos_clientes_situacao').val('0').trigger('change'); });
await page.waitForFunction(() => window.$('#contratos_clientes_situacao').val() == '0');
```

**Boundary:** Contrato com `vigencia_ate = hoje` deve aparecer em "Em vigência" (`'1'`), não em "Vencido".

### Fluxo 8 — Filtro por status

**Valores Select2:** `'1'` = Ativo | `'0'` = Inativo | `''` = Todos

```js
await page.evaluate(() => { $('#contratos_clientes_status').val('0').trigger('change'); });
await page.waitForFunction(() => window.$('#contratos_clientes_status').val() == '0');
```

### Fluxo 9 — Estado vazio

1. `page.locator('#contratos_clientes_id_contrato').fill('XXXCODIGOINEXISTENTE')`
2. Clicar em `#btn_contratos_clientes_filtrar`

**Estado final:** `td.dataTables_empty` visível com texto exato "Nenhum resultado encontrado!".

### Fluxo 10 — Limpar filtros

1. Clicar em `#btn_contratos_clientes_limpar_filtro`

**Estado final:** Todos os filtros resetados; todos os contratos de todos os clientes exibidos (≥ 3 grupos de clientes).

### Fluxo 11 — Gerenciamento de permissão (perfil não-ADMIN)

**Adicionar permissão:**
1. Navegar para `/assistencia/usuario_perfil/editar.php?id=284`
2. Aguardar `#preloading` hidden
3. Verificar que `#edit_profile_view_report_contractxclients` está desmarcado
4. Clicar em `label[for="edit_profile_view_report_contractxclients"]`
5. Clicar em `#bt_editar_perfil`
6. `page.reload()` + aguardar `#preloading` hidden
7. Verificar que checkbox está marcado (persistência confirmada)

**Remover permissão:** Repetir passos verificando estado oposto.

### Fluxo 12 — Tentativa de remover permissão do perfil ADMIN

1. Navegar para `/assistencia/usuario_perfil/editar.php?id=0`
2. Aguardar `#preloading` hidden
3. Verificar `#bt_editar_perfil` está `disabled`
4. Clicar em `label[for="edit_profile_view_report_contractxclients"]`
5. Verificar que `#bt_editar_perfil` permanece `disabled`
6. `page.reload()` → verificar que checkbox permanece marcado

**Atenção:** O checkbox `#edit_profile_view_report_contractxclients` NÃO está disabled no DOM do perfil ADMIN — apenas o botão Salvar está.

---

## Pontos de Entrada

- Menu lateral: Relatórios → Contrato por clientes
- URL direta: `/assistencia/relatorios/contratoxcliente/`

## Pontos de Saída

- Tabela com contratos filtrados exibida (com row grouping por cliente)
- Redirect para login (sem autenticação)
- Mensagem "não possui autorização" (sem permissão)

## Variações

### Resumo de Seletores

| Elemento | Seletor | Notas |
|---|---|---|
| Menu item | `a[href="/assistencia/relatorios/contratoxcliente/"]` | |
| Filtro Cliente | `#contratos_clientes_id_cliente` | Select2 — jQuery API |
| Filtro Contrato | `#contratos_clientes_id_contrato` | Input texto |
| Filtro Situação | `#contratos_clientes_situacao` | Select2 — valores: 1/0/'' |
| Filtro Status | `#contratos_clientes_status` | Select2 — valores: 1/0/'' |
| Btn Filtrar | `#btn_contratos_clientes_filtrar` | |
| Btn Limpar | `#btn_contratos_clientes_limpar_filtro` | Sem texto visível |
| Tabela | `#datatable_contratos_clientes` | |
| Estado vazio | `td.dataTables_empty` | Texto: "Nenhum resultado encontrado!" |
| Linha grupo | `tr.dtrg-group` | Assíncrono — aguardar `.first()` visible |
| XLS | `.dt-button.buttons-excel` | |
| CSV | `.dt-button.buttons-csv` | |
| Mensagem acesso negado | `main` | Contém "não possui autorização" |
| Checkbox permissão | `#edit_profile_view_report_contractxclients` | Clicar na label, não no input |
| Label checkbox | `label[for="edit_profile_view_report_contractxclients"]` | |
| Btn Salvar perfil | `#bt_editar_perfil` | disabled=true para ADMIN |
| Overlay carregamento | `#preloading` | Aguardar hidden antes de interagir |

## Features Relacionadas

- [[contratos-por-cliente]]
