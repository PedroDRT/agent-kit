# Fluxos — Relatório de Contratos por Cliente

> **Mapeado em:** 2026-04-10 | **Validado em:** 2026-04-10 (Run 10 — 32/32 passed)
> **Arquivo anterior:** `knowledge/system/flows/relatorio-contratos-por-cliente.md` (supersedido)
> **URL:** `/assistencia/relatorios/contratoxcliente/`

---

## Fluxo 1 — Acesso via menu lateral (usuário com permissão)

**Tela inicial:** `/assistencia/dashboard/` (após login)
**Pré-condição:** Usuário autenticado, perfil AGENTS com permissão "Consultar relatório de contratos"

1. Clicar em "Relatórios" no menu lateral
2. Aguardar expansão do submenu
3. Clicar em `a[href="/assistencia/relatorios/contratoxcliente/"]` ("Contrato por clientes")

**Estado final:** URL `/assistencia/relatorios/contratoxcliente/`, tabela `#datatable_contratos_clientes` visível, breadcrumb "Relatórios / Contratos por Cliente".

---

## Fluxo 2 — Acesso direto por URL (autenticado com permissão)

**Pré-condição:** Usuário autenticado com permissão

1. Navegar diretamente para `/assistencia/relatorios/contratoxcliente/`

**Estado final:** `#btn_contratos_clientes_filtrar` visível, tabela carregada, sem mensagem de acesso negado.

---

## Fluxo 3 — Acesso sem autenticação (RN-GLOBAL-001)

**Pré-condição:** Sem sessão ativa

1. Acessar diretamente `/assistencia/relatorios/contratoxcliente/`

**Estado final:** Redirect HTTP 3xx para `/assistencia/login/`.

> ⚠️ **Automação:** Usar `request` fixture do Playwright (não `browser.newContext()` — causa `ERR_CONNECTION_REFUSED` no ambiente local). Ver `contratos-por-cliente.md` → Seção Automação.

---

## Fluxo 4 — Acesso sem permissão bloqueado (usuário autenticado)

**Pré-condição:** Usuário autenticado com perfil AGENTS com permissão **removida**

**Setup (remover permissão via setAgentsPermission):**
1. Navegar para `/assistencia/usuario_perfil/editar.php?id=284`
2. Aguardar `#preloading` hidden
3. Clicar em `label[for="edit_profile_view_report_contractxclients"]` (não clicar no checkbox diretamente)
4. Clicar em `#bt_editar_perfil`
5. Aguardar `domcontentloaded` + `#preloading` hidden
6. **Executar `page.reload()`** + aguardar `domcontentloaded` + `#preloading` hidden ← obrigatório para sessão PHP refletir

**Teste:**
7. Navegar para `/assistencia/dashboard/` (garantir commit do DB)
8. Expandir menu Relatórios → verificar ausência de `a[href="/assistencia/relatorios/contratoxcliente/"]`
9. Navegar para `/assistencia/relatorios/contratoxcliente/`
10. Verificar que `#datatable_contratos_clientes` NÃO está visível
11. Verificar que `main` contém "não possui autorização"

**Teardown obrigatório (restaurar permissão):**
12. Repetir passos 1–6 com checkbox desmarcado → marcar e salvar

> **Lição aprendida (H013):** O `page.reload()` no passo 6 é obrigatório. Sem ele, a sessão PHP mantém o estado anterior e o menu/acesso não reflete a alteração salva no DB.

---

## Fluxo 5 — Filtro por cliente

**Pré-condição:** Relatório carregado com permissão

1. Setar valor do Select2 via jQuery:
   ```js
   await page.evaluate(() => { $('#contratos_clientes_id_cliente').val('4143').trigger('change'); });
   ```
2. Aguardar commit do jQuery (race condition):
   ```js
   await page.waitForFunction(() => window.$('#contratos_clientes_id_cliente').val() == '4143');
   ```
3. Clicar em `#btn_contratos_clientes_filtrar`
4. Aguardar `#datatable_contratos_clientes_processing` desaparecer + tbody visível

**Estado final:** Apenas 1 grupo `tr.dtrg-group` com "LUCASCLI (46.902.443/0001-79)", contratos AUTO_CLI_COD, PET_CLI_COD, RESI_CLI_COD visíveis.

---

## Fluxo 6 — Filtro por código de contrato

**Pré-condição:** Relatório carregado

1. `page.locator('#contratos_clientes_id_contrato').fill('AUTO_CLI_COD')`
2. Clicar em `#btn_contratos_clientes_filtrar`
3. Aguardar recarregamento

**Estado final:** Tabela exibe apenas contratos cujo código contém "AUTO_CLI_COD".

---

## Fluxo 7 — Filtro por situação

**Valores Select2:** `'1'` = Em vigência | `'0'` = Vencido | `''` = Todos

1. Setar via jQuery:
   ```js
   await page.evaluate(() => { $('#contratos_clientes_situacao').val('0').trigger('change'); });
   await page.waitForFunction(() => window.$('#contratos_clientes_situacao').val() == '0');
   ```
2. Clicar em `#btn_contratos_clientes_filtrar`

**Estado final:** Apenas contratos com `vigencia_ate < data_atual` visíveis (ex: CONTRATO VENCIDO).
**Boundary:** Contrato com `vigencia_ate = hoje` deve aparecer em "Em vigência" (`'1'`), não em "Vencido".

---

## Fluxo 8 — Filtro por status

**Valores Select2:** `'1'` = Ativo | `'0'` = Inativo | `''` = Todos

1. Setar via jQuery:
   ```js
   await page.evaluate(() => { $('#contratos_clientes_status').val('0').trigger('change'); });
   await page.waitForFunction(() => window.$('#contratos_clientes_status').val() == '0');
   ```
2. Clicar em `#btn_contratos_clientes_filtrar`

**Estado final:** Apenas contratos com status Inativo (ex: CONTRATO INATIVO — cliente AGENTECLI).

---

## Fluxo 9 — Estado vazio

1. `page.locator('#contratos_clientes_id_contrato').fill('XXXCODIGOINEXISTENTE')`
2. Clicar em `#btn_contratos_clientes_filtrar`

**Estado final:** `td.dataTables_empty` visível com texto exato "Nenhum resultado encontrado!".

---

## Fluxo 10 — Limpar filtros

**Pré-condição:** Um ou mais filtros aplicados

1. Clicar em `#btn_contratos_clientes_limpar_filtro`
2. Aguardar recarregamento

**Estado final:** Todos os filtros resetados (inputs vazios, selects em branco), todos os contratos de todos os clientes exibidos (≥ 3 grupos de clientes).

---

## Fluxo 11 — Gerenciamento de permissão (perfil não-ADMIN)

**Adicionar permissão:**
1. Navegar para `/assistencia/usuario_perfil/editar.php?id=284`
2. Aguardar `#preloading` hidden
3. Verificar que `#edit_profile_view_report_contractxclients` está desmarcado
4. Clicar em `label[for="edit_profile_view_report_contractxclients"]`
5. Clicar em `#bt_editar_perfil`
6. `page.reload()` + aguardar `#preloading` hidden
7. Verificar que checkbox está marcado (persistência confirmada)

**Remover permissão:**
1–2. Mesmos passos
3. Verificar que checkbox está marcado
4. Clicar em label para desmarcar
5–7. Salvar, recarregar, confirmar desmarcado
8. Navegar para `/assistencia/relatorios/contratoxcliente/` e confirmar acesso negado

---

## Fluxo 12 — Tentativa de remover permissão do perfil ADMIN

1. Navegar para `/assistencia/usuario_perfil/editar.php?id=0`
2. Aguardar `#preloading` hidden
3. Verificar `#bt_editar_perfil` está `disabled`
4. Clicar em `label[for="edit_profile_view_report_contractxclients"]`
5. Verificar que `#bt_editar_perfil` permanece `disabled`
6. `page.reload()` → verificar que checkbox permanece marcado

**Estado final:** Impossível salvar qualquer alteração no perfil ADMIN.

> **Atenção:** O checkbox `#edit_profile_view_report_contractxclients` NÃO está disabled no DOM do perfil ADMIN — apenas o botão Salvar está. Verificar `#bt_editar_perfil[disabled]`, não `input[disabled]`.

---

## Resumo de Seletores

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
| Paginação | `#datatable_contratos_clientes_paginate` | |
| XLS | `.dt-button.buttons-excel` | |
| CSV | `.dt-button.buttons-csv` | |
| Gerenciar colunas | `.dt-button.buttons-colvis` | |
| Mensagem acesso negado | `main` | Contém "não possui autorização" |
| Checkbox permissão | `#edit_profile_view_report_contractxclients` | Label: `label[for="..."]` |
| Label checkbox | `label[for="edit_profile_view_report_contractxclients"]` | Clicar aqui, não no input |
| Btn Salvar perfil | `#bt_editar_perfil` | disabled=true para ADMIN |
| Overlay carregamento | `#preloading` | Aguardar hidden antes de interagir |
| Título página | `h6` | `.filter({ hasText: 'Relatório de Contratos por Cliente' })` — há múltiplos h6 |
