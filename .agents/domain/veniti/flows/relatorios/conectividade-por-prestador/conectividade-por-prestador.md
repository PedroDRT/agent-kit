# Flow — Relatório de Conectividade de Prestadores

**Feature:** melhorias-no-relatorio-de-conectividade  
**Portal:** Assistência  
**URL:** `/assistencia/relatorios/provider_connectivity/`  
**Seletores:** `knowledge/system/selectors/melhorias-no-relatorio-de-conectividade.json`

---

## Observações Gerais da Exploração

| Item | Observado |
|---|---|
| Label do novo filtro | `"Prestadores com horário configurado:"` |
| Opção padrão do novo filtro | *(vazio)* — comportamento "Todos" |
| Opção 1 | `"APENAS COM HORÁRIO CONFIGURADO"` (value: `with`) |
| Opção 2 | `"APENAS SEM HORÁRIO CONFIGURADO"` (value: `without`) |
| Parâmetro HTTP do novo filtro | `config=with` / `config=without` / `config=` |
| Datatable endpoint | `__acoes.php?datatable_provider_connectivity` → **200 OK** |
| Gráfico endpoint | `__acoes.php?graph_provider_connectivity` → **200 OK** |
| Alert dialog no carregamento | Pode aparecer em ambientes sem dados no período — aceitar via `page.on('dialog', d => d.accept())` |
| Tooltip do ícone informativo | **Não observável** — datatable retorna 400; seletor pendente de confirmação |

---

## Flow 1 — Login no Portal Assistência

```
1. Acessar http://[baseUrl]/assistencia/
   → Redireciona para /assistencia/login/

2. Preencher:
   - CPF/CNPJ: [de environment.json → users.agent1.cnpj]
   - Usuário:  [de environment.json → users.agent1.user]
   - Senha:    [de environment.json → users.agent1.password]

3. Clicar em "Logar como Assistência"
   → Redireciona para /assistencia/dashboard/

✅ Verificação: URL contém "/assistencia/dashboard/"
```

---

## Flow 2 — Acesso ao Relatório de Conectividade

### Via URL Direta (recomendado para testes)

```
1. Autenticar (Flow 1)
2. Navegar para: http://[baseUrl]/assistencia/relatorios/provider_connectivity/
3. Aceitar dialog "DataTables Ajax error" se aparecer
   → page.on('dialog', d => d.accept())

✅ Verificação:
   - URL = /assistencia/relatorios/provider_connectivity/
   - Heading visível: "Relatórios / Conectividade de Prestadores"
   - Filtro "Prestadores com horário configurado:" presente na página
```

### Via Menu de Navegação

```
1. Autenticar (Flow 1)
2. Clicar no menu "Relatórios" na sidebar esquerda
   → Expande submenu
3. Clicar em "Conectividade de Prestadores NOVO"
   → Navega para /assistencia/relatorios/provider_connectivity/
4. Aceitar dialog se aparecer

✅ Verificação: idêntica ao acesso por URL direta
```

### Acesso Sem Autenticação (RN-GLOBAL-001)

```
1. Sem sessão ativa, acessar diretamente a URL do relatório
2. Aguardar redirecionamento

✅ Verificação: URL redireciona para /assistencia/login/
```

---

## Flow 3 — Aplicar Filtro Padrão (Período)

```
Pré-condição: Relatório carregado (Flow 2)

1. Limpar campo #filter_date_from e preencher com data inicial (DD/MM/YYYY)
2. Limpar campo #filter_date_to e preencher com data final (DD/MM/YYYY)
3. Clicar em #btn_provider_connectivity_filter ("Filtrar...")
4. Aceitar dialog de erro do DataTables se aparecer (bug ativo)

✅ Verificação esperada (após correção do bug):
   - Datatable exibe linhas com colunas: Nome | CNPJ | Status | Conectividade | % Conexão
   - Gráfico (#graph_provider_connectivity / #div_grafico) é atualizado
   - Endpoint GET __acoes.php?datatable_provider_connectivity retorna 200

⚠️ Estado atual: endpoint retorna 400 — todos os filtros falham no datatable
```

---

## Flow 4 — Aplicar Novo Filtro: Prestadores com Horário Configurado

```
Pré-condição: Relatório carregado (Flow 2)
              Prestadores A e B cadastrados no ambiente

1. Selecionar valor desejado em #filter_config:
   - Para "APENAS COM HORÁRIO CONFIGURADO": page.selectOption('#filter_config', 'with')
   - Para "APENAS SEM HORÁRIO CONFIGURADO": page.selectOption('#filter_config', 'without')
   - Para "Todos": page.selectOption('#filter_config', '')

2. Informar período em #filter_date_from e #filter_date_to
3. Clicar em #btn_provider_connectivity_filter ("Filtrar...")

✅ Parâmetro enviado no request: config=with | config=without | config=
✅ Verificação: apenas prestadores do tipo selecionado aparecem no datatable
```

---

## Flow 5 — Combinação de Filtros

```
Pré-condição: Relatório carregado

1. Preencher período
2. Selecionar prestador específico em #filter_prestador (opcional)
3. Selecionar valor em #filter_config (opcional)
4. Clicar em #btn_provider_connectivity_filter

✅ Todos os parâmetros são enviados juntos no mesmo request
✅ Filtros são cumulativos (AND)
```

---

## Flow 6 — Limpar Filtros

```
Pré-condição: Filtros aplicados

1. Clicar em #btn_provider_connectivity_clean_filters

✅ Comportamento: função resetFilters() é chamada
   - Limpa todos os campos com prefixo filter_ e filtro_
   - Redefine as datas para: (hoje - 1 mês) até hoje
   - Chama filter() automaticamente

✅ Verificação: campos retornam ao estado padrão; datatable é recarregado
```

---

## Flow 7 — Tooltip Informativo (PENDENTE — requer correção do bug 400)

```
Pré-condição: Datatable funcional + Prestador A ou B cadastrado

1. Aplicar filtro que inclua prestador com horário configurado (Prestador A ou B)
2. Localizar coluna "Nome" no datatable
3. Identificar ícone informativo ao lado do nome do prestador

✅ Verificação para prestador COM horário (CT006.1):
   - Ícone informativo presente na coluna Nome
   
4. Passar o mouse (hover) sobre o ícone informativo

✅ Verificação para conteúdo do tooltip (CT006.2):
   - Tooltip exibido com lista de dias da semana e horários configurados

✅ Verificação para dia não configurado (CT006.3 — Prestador B):
   - Dias sem configuração exibem "Não configurado"

✅ Verificação para prestador SEM horário (CT006.4 — Prestador C):
   - Ícone NÃO presente na coluna Nome

⚠️ Seletor do ícone: pendente de confirmação após correção do bug HTTP 400
   Seletor sugerido: '#datatable_provider_connectivity tbody tr td:first-child [title], i[class*="fa-"]'
```

---

## Estrutura do Formulário de Filtros

```
[FILTROS]
  Período:                  [#filter_date_from] Até [#filter_date_to]
  Prestadores:              [#filter_prestador] (Select2 com busca)
  Conexão:                  [#filtro_conexao] (ONLINE | OFFLINE)
  Cidade:                   [#filtro_cidade] (Select2 com busca)
  Estado:                   [#filtro_estado] (Select2 com busca)
  Categoria de Serviço:     [#filtro_categoria_servico] (Select2)
  Segmento de serviço:      [#filtro_segmento_servico] (Select2 — dependente de Categoria)
  Tipo de Serviço:          [#filtro_tipo_servico] (Select2 — dependente de Segmento)
  Prestadores com horário:  [#filter_config] (NOVO — with | without | vazio)

  [X Limpar — #btn_provider_connectivity_clean_filters]
  [Filtrar... — #btn_provider_connectivity_filter]
```

---

## Estrutura da Página de Resultados

```
[DATATABLE — #datatable_provider_connectivity]
  Botões: [.XLS] [.CSV] [Imprimir] [Altere as colunas]
  Colunas: Nome | CNPJ | Status | Conectividade | % Conexão

[GRÁFICO — canvas#graph_provider_connectivity (dentro de #div_grafico)]
  Tipo: Bar chart (Chart.js)
  Label: "Prestadores conectados"
```

---

## Notas de Ambiente

- Testes devem ser executados no ambiente Linux onde o banco remoto está acessível corretamente.
- O ambiente Windows apresentou problema de conectividade com o banco durante a exploração — desconsiderar observações de erro feitas nessa máquina.
