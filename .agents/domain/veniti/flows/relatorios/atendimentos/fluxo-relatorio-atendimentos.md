# Fluxos — Relatório de Atendimentos

> **Mapeado em:** 2026-04-16 — via Playwright MCP (exploração direta)
> **URL:** `/assistencia/relatorios/atendimentos/`
> **Contexto:** Teste de urgência — problema de desempenho

---

## Fluxo 1 — Acesso via menu lateral

**Pré-condição:** Usuário autenticado no portal Assistência

1. No menu lateral, clicar em **"Relatórios"**
2. Aguardar expansão do submenu
3. Clicar em `a[href="/assistencia/relatorios/atendimentos/"]` — item **"Atendimentos"**

**Estado final:** URL `/assistencia/relatorios/atendimentos/`, formulário de filtros visível, campo Período preenchido com últimos 30 dias, tabela exibindo "Nenhum resultado encontrado!", botões de exportação desabilitados.

---

## Fluxo 2 — Acesso direto por URL

**Pré-condição:** Usuário autenticado

1. Navegar diretamente para `/assistencia/relatorios/atendimentos/`

**Estado final:** Idêntico ao Fluxo 1.

---

## Fluxo 3 — Acesso sem autenticação (RN-GLOBAL-001)

**Pré-condição:** Sem sessão ativa

1. Acessar diretamente `/assistencia/relatorios/atendimentos/`

**Estado final:** Redirect para `/assistencia/login/`.

> ⚠️ **Automação:** Usar `request` fixture do Playwright (evitar `browser.newContext()` — pode causar `ERR_CONNECTION_REFUSED` em ambiente local).

---

## Fluxo 4 — Pesquisa com filtro de Período

**Pré-condição:** Relatório carregado

1. Preencher `#atendimento_data_de` com a data de início desejada
2. Preencher `#atendimento_data_ate` com a data de fim
3. Clicar em `#btn_atendimentos_filtrar`
4. Aguardar atualização da tabela `#datatable_rel_atendimentos`

**Estado final:** Tabela exibe somente atendimentos dentro do intervalo. Botões de exportação habilitados se houver resultados.

> **Período amplo (12+ meses):** Retorna alto volume de dados — principal cenário de diagnóstico de performance.

---

## Fluxo 5 — Pesquisa com filtro de Status

**Pré-condição:** Relatório carregado

1. Setar valor do Select2 via jQuery:
   ```js
   await page.evaluate(() => {
     $('#rel_atendimento_status').val('FINALIZADO').trigger('change');
   });
   await page.waitForFunction(() =>
     window.$('#rel_atendimento_status').val() === 'FINALIZADO'
   );
   ```
2. Clicar em `#btn_atendimentos_filtrar`

**Estado final:** Somente atendimentos com `status = FINALIZADO`.

**Valores aceitos:** `ABERTO`, `AGUARDANDO ACEITE`, `SERVIÇO ACEITO`, `NA ORIGEM`, `NO DESTINO`, `NEGADO`, `RECUPERADO`, `NÃO RECUPERADO`, `EM BUSCA`, `CANCELADO`, `FINALIZADO`, `NEGATIVA`, `NPS PENDENTE`, `TODOS`

---

## Fluxo 6 — Pesquisa com filtro de Cliente (multi-select)

**Pré-condição:** Relatório carregado

1. Selecionar cliente(s) na listbox `#rel_atendimento_id_cliente`:
   ```js
   // Selecionar um cliente
   await page.evaluate(() => {
     $('#rel_atendimento_id_cliente').val(['<ID_CLIENTE>']).trigger('change');
   });
   ```
2. Clicar em `#btn_atendimentos_filtrar`

**Estado final:** Somente atendimentos do(s) cliente(s) selecionado(s).

> **Efeito colateral:** Ao selecionar um cliente, o campo `#rel_atendimento_plano` é filtrado dinamicamente para exibir apenas os planos daquele cliente.

---

## Fluxo 7 — Pesquisa com filtro de Tipo de Serviço (multi-select)

1. Setar via jQuery (suporta múltiplos valores):
   ```js
   await page.evaluate(() => {
     $('#rel_atendimento_tipo_servico').val(['REBOQUE LEVE', 'REBOQUE PESADO']).trigger('change');
   });
   ```
2. Clicar em `#btn_atendimentos_filtrar`

**Estado final:** Somente atendimentos com tipo de serviço = REBOQUE LEVE ou REBOQUE PESADO.

---

## Fluxo 8 — Pesquisa com filtro de Prestador

1. Setar via jQuery:
   ```js
   await page.evaluate(() => {
     $('#rel_atendimento_id_prestador').val('<ID_PRESTADOR>').trigger('change');
   });
   ```
2. Clicar em `#btn_atendimentos_filtrar`

---

## Fluxo 9 — Pesquisa com filtros de texto livre (Beneficiário, Placa, Protocolo, Lote, Código Contrato)

```js
// Beneficiário
await page.locator('#rel_atendimento_beneficiario').fill('JOAO SILVA');

// Placa
await page.locator('#rel_atendimento_placa').fill('ABC1234');

// Protocolo
await page.locator('#rel_atendimento_protocolo').fill('20260001');

// Número do Lote
await page.locator('#rel_atendimento_num_faturado').fill('123');

// Código Contrato
await page.locator('#rel_atendimento_codigo_contrato').fill('AUTO_CLI_COD');
```

Clicar em `#btn_atendimentos_filtrar` após preencher.

> ⚠️ **Atenção:** O campo "Número do Lote" usa o ID `#rel_atendimento_num_faturado` — o ID não corresponde ao label exibido na UI.

---

## Fluxo 10 — Pesquisa com filtros booleanos (SIM/NÃO)

Filtros: `#rel_atendimento_reembolso`, `#rel_atendimento_faturado`, `#rel_atendimento_parceiro`, `#rel_atendimento_possui_arquivo`, `#rel_atendimento_a_vista`, `#rel_atendimento_acionamento_automatico`

```js
// Exemplo: Reembolso = SIM
await page.evaluate(() => {
  $('#rel_atendimento_reembolso').val('SIM').trigger('change');
});
await page.waitForFunction(() => window.$('#rel_atendimento_reembolso').val() === 'SIM');
```

> Esses filtros não possuem opção "TODOS" — estado vazio (sem valor selecionado) = sem filtro aplicado.

---

## Fluxo 11 — Pesquisa com checkboxes (Avulso / Caso 99)

```js
// Checkbox Avulso
await page.locator('#rel_atendimento_avulso').check();

// Checkbox Caso 99
await page.locator('#rel_atendimento_caso_99').check();
```

> ⚠️ **Verificar:** Os checkboxes podem ser estilizados (label cobrindo o input). Testar se clicar diretamente no input funciona — caso contrário, usar `label[for="rel_atendimento_avulso"]`.

---

## Fluxo 12 — Pesquisa com filtro de Estado → Cidade (dependência)

1. Setar Estado:
   ```js
   await page.evaluate(() => {
     $('#rel_atendimento_estados').val('SAO PAULO').trigger('change');
   });
   ```
2. Aguardar populate dinâmico do campo Cidade (`#rel_atendimento_cidades`)
3. Setar Cidade:
   ```js
   await page.evaluate(() => {
     $('#rel_atendimento_cidades').val('<NOME_CIDADE>').trigger('change');
   });
   ```
4. Clicar em `#btn_atendimentos_filtrar`

---

## Fluxo 13 — Pesquisa com filtro de Vigência do Plano

```js
await page.evaluate(() => {
  $('#rel_atendimento_vigencia').val('DENTRO DA VIGÊNCIA').trigger('change');
});
```

**Valores:** `FORA DA VIGÊNCIA`, `DENTRO DA VIGÊNCIA`, `VIGÊNCIA DECORRIDA`

---

## Fluxo 14 — Pesquisa com filtro de Tags (multi-select)

```js
await page.evaluate(() => {
  $('#rel_atendimento_tags').val(['TESTE', 'TAG REEMBOLSO']).trigger('change');
});
```

---

## Fluxo 15 — Estado vazio

1. Preencher qualquer filtro com valor inexistente (ex: protocolo = `"XXXINEXISTENTE"`)
2. Clicar em `#btn_atendimentos_filtrar`

**Estado final:** `td.dataTables_empty` visível com texto `"Nenhum resultado encontrado!"`.

---

## Fluxo 16 — Limpar filtros

**Pré-condição:** Um ou mais filtros preenchidos

1. Clicar em `#btn_atendimentos_limpar_filtro` (botão sem texto, apenas ícone)

**Estado final:** Formulário resetado para estado padrão. Campo Período mantido com os últimos 30 dias. Tabela volta ao estado vazio.

---

## Fluxo 17 — Exportar resultados

**Pré-condição:** Pesquisa executada com resultados

```js
// XLS
await page.locator('.dt-button.buttons-excel').first().click();

// CSV
await page.locator('.dt-button.buttons-csv').click();

// XLS colunas visíveis
await page.locator('.dt-button.excelButtonVisible').click();

// PDF
await page.locator('.dt-button.buttons-pdf').click();
```

> ⚠️ **`.dt-button.buttons-excel` resolve 2 elementos** (XLS completo e XLS colunas visíveis). Usar `.first()` para o XLS completo ou seletores mais específicos.

---

## Fluxo 18 — Configurar e salvar colunas

**Pré-condição:** Pesquisa executada com resultados

1. Clicar em `.dt-button.buttons-colvis` ("Altere as colunas")
2. Selecionar/deselecionar colunas desejadas
3. Clicar em `.dt-button:has-text("Salvar alteração")`

**Estado final:** Configuração de colunas persistida para o usuário.

---

## Fluxo 19 — Paginação

**Pré-condição:** Pesquisa retorna múltiplas páginas

```js
// Próxima página
await page.locator('#datatable_rel_atendimentos_paginate .next').click();

// Página anterior
await page.locator('#datatable_rel_atendimentos_paginate .previous').click();
```

---

## Resumo de IDs — Filtros

| Filtro | ID |
|---|---|
| Período de | `#atendimento_data_de` |
| Período até | `#atendimento_data_ate` |
| Tipo de serviço | `#rel_atendimento_tipo_servico` |
| Status | `#rel_atendimento_status` |
| Cliente | `#rel_atendimento_id_cliente` |
| Prestador | `#rel_atendimento_id_prestador` |
| Tipo de atendimento | `#rel_atendimento_tipo_atendimento` |
| Plano | `#rel_atendimento_plano` |
| Cidade | `#rel_atendimento_cidades` |
| Estado | `#rel_atendimento_estados` |
| Beneficiário | `#rel_atendimento_beneficiario` |
| Responsável (tipo) | `#rel_atendimento_responsavel` |
| Responsável (grupo) | `#rel_atendimento_grupos` |
| Tipo de Veículo | `#rel_atendimento_tipo` |
| Reembolso | `#rel_atendimento_reembolso` |
| Veículo (Placa) | `#rel_atendimento_placa` |
| Número do Protocolo | `#rel_atendimento_protocolo` |
| Número do Lote | `#rel_atendimento_num_faturado` ⚠️ |
| Faturado | `#rel_atendimento_faturado` |
| Parceiro do Cliente | `#rel_atendimento_parceiro` |
| Usuários | `#rel_atendimento_usuario` |
| Código Contrato | `#rel_atendimento_codigo_contrato` |
| Vigência Plano | `#rel_atendimento_vigencia` |
| Tipo de Evento | `#rel_atendimento_tipo_evento` |
| Atendimento com arquivo | `#rel_atendimento_possui_arquivo` |
| Tipo de suporte | `#rel_atendimento_tipo_suporte` |
| Pagamento a vista | `#rel_atendimento_a_vista` |
| Tags | `#rel_atendimento_tags` |
| Acionamento Automático | `#rel_atendimento_acionamento_automatico` |
| Avulso | `#rel_atendimento_avulso` |
| Caso 99 | `#rel_atendimento_caso_99` |
