# Feature: Melhorias no Relatório de Conectividade

**Módulo:** Relatórios — Assistência  
**URL:** `/assistencia/relatorios/provider_connectivity/`  
**Portal:** Assistência  
**Versão do documento:** V1 — 2025

---

## Objetivo

Ajustar o relatório de conectividade de prestadores para que os indicadores respeitem a configuração personalizada de horário de trabalho quando existente, garantindo maior precisão e aderência à realidade operacional.

---

## Processo Atual (Before)

- O relatório considera **24 horas por dia** para todos os prestadores, independentemente de possuírem configuração de horário.
- Filtros disponíveis: Período, Prestadores, Cidade, Estado, Categoria de Serviço, Segmento de Serviço, Tipo de Serviço, Conexão.
- Datatable exibe: Nome, CNPJ, Status, Conectividade (tempo conectado), % Conexão.
- A configuração de horário de trabalho (definida via Prestador > Configuração > Geral ou Assistência > Rede > Prestadores > Editar) **não impactava** o relatório.

---

## Novo Processo (After)

### 3.1 — Regra de Cálculo da Conectividade

Para cada prestador no relatório, o sistema verifica se existe configuração personalizada de horário de trabalho.

| Cenário | Regra de Cálculo |
|---|---|
| **Sem horário configurado** | Mantém regra atual: 24 horas por dia para todos os dias do período filtrado |
| **Com horário configurado** | Considera exclusivamente os intervalos configurados por dia da semana |
| **Dia da semana não configurado** | Dia completamente desconsiderado do cálculo — não impacta negativamente o percentual |

**Definição de "configuração ativa":** Prestador com ao menos 1 dia da semana configurado é considerado "com horário configurado".

**Exemplo:** Segunda-feira configurada das 08:00 às 18:00 → considera apenas 10 horas para todas as segundas-feiras dentro do período selecionado.

**Edge case — período inteiramente em dias não configurados:** Exibir 0% de conexão e 0h de conectividade, sem erro.

### 3.2 — Novo Filtro no Relatório

| Campo | Detalhe |
|---|---|
| **Nome (spec)** | Prestadores com horário de trabalho |
| **Nome (UI real)** | **Prestadores com horário configurado:** |
| **Tipo** | Select (Select2) |
| **Opções (UI real)** | *(vazio — Todos)* / `APENAS COM HORÁRIO CONFIGURADO` / `APENAS SEM HORÁRIO CONFIGURADO` |
| **Valor padrão** | Vazio (comportamento "Todos") |
| **Parâmetro na API** | `config=` |

**Comportamento:**
- **Todos:** sem filtro adicional (comportamento atual).
- **Apenas com horário configurado:** lista somente prestadores com configuração ativa de horário.
- **Apenas sem horário configurado:** lista somente prestadores sem configuração de horário.

### 3.3 — Tooltip Informativo na Listagem

Para prestadores com configuração personalizada de horário:
- Exibir **ícone informativo** ao lado do nome no datatable.
- Ao passar o mouse, exibir tooltip com o seguinte texto (dinâmico):

```
Este prestador possui configuração personalizada de horário de trabalho.

Os indicadores não são calculados com base nas 24 horas do dia, mas sim conforme o horário configurado:

Domingo: [HH:MM às HH:MM | Não configurado]
Segunda-feira: [HH:MM às HH:MM | Não configurado]
Terça-feira: [HH:MM às HH:MM | Não configurado]
Quarta-feira: [HH:MM às HH:MM | Não configurado]
Quinta-feira: [HH:MM às HH:MM | Não configurado]
Sexta-feira: [HH:MM às HH:MM | Não configurado]
Sábado: [HH:MM às HH:MM | Não configurado]

Os indicadores respeitam exclusivamente esses períodos configurados.
```

**Regras do tooltip:**
- Horário reflete exatamente o que estiver configurado.
- Dia não configurado → exibir nome do dia + "Não configurado".
- Todos os 7 dias sempre listados (nenhum omitido).
- **Não exibir** tooltip para prestadores sem horário configurado.

---

## Critérios de Aceitação

| ID | Dado | Quando | Então |
|---|---|---|---|
| CA1 | Prestadores sem horário configurado existem no período filtrado | Usuário filtra o relatório | Sistema calcula conectividade com 24h/dia |
| CA2 | Prestador possui horário configurado | Relatório é processado | Sistema calcula conectividade apenas nos intervalos configurados por dia da semana |
| CA3 | Dia da semana não está configurado para o prestador | Sistema calcula a conectividade | Dia é completamente desconsiderado do cálculo |
| CA4 | Usuário seleciona "Apenas com horário configurado" no filtro | Relatório é filtrado | Lista apenas prestadores com configuração ativa |
| CA5 | Usuário seleciona "Apenas sem horário configurado" no filtro | Relatório é filtrado | Lista apenas prestadores sem configuração de horário |
| CA6 | Prestador possui horário configurado | Prestador aparece na listagem | Ícone informativo exibido ao lado do nome com tooltip detalhando horários por dia da semana |
| CA7 | Dia da semana não está configurado | Tooltip é exibido | Nome do dia seguido de "Não configurado" |

---

## Dependências e Pré-condições

- Configuração de horário de trabalho pode ser definida em dois locais:
  - Portal Prestador: Configuração > Geral
  - Portal Assistência: Rede > Prestadores > Editar
- **Dados de teste necessários:**
  - Prestador A: horário configurado em todos os 7 dias
  - Prestador B: horário configurado apenas em alguns dias (sábado e domingo sem configuração)
  - Prestador C: sem nenhum horário configurado

---

## Fora do Escopo

- Cadastro/edição de horário de trabalho do prestador (funcionalidade já existente)
- Outros relatórios além do de conectividade
- Impacto no cálculo de outros módulos

---

## Descobertas da Exploração Real

### 2026-04-09 / 2026-04-10
- **Label divergente:** O filtro está implementado como "Prestadores com horário configurado:" — diferente do spec ("Prestadores com horário de trabalho")
- **Opções em uppercase:** UI exibe "APENAS COM HORÁRIO CONFIGURADO" e "APENAS SEM HORÁRIO CONFIGURADO" (spec usava minúsculas)
- **Endpoint API:** `__acoes.php?datatable_provider_connectivity&date_from=...&date_to=...&config=...`
- **Parâmetro do novo filtro:** `config=with` / `config=without` / `config=` (vazio = Todos)

---

## Comportamentos Confirmados

| Comportamento | Confirmado | Observação |
|---|---|---|
| Período inteiramente em dias não configurados → 0% sem erro | ✅ Sim | Sem divisão por zero; exibe 0% e 0h corretamente |
| Prestador com apenas 1 dia configurado incluído no filtro "com horário" | ✅ Sim | Definição "configuração ativa" = ao menos 1 dia configurado |
| Tooltip exibe todos os 7 dias sempre | ✅ Sim | Dias sem configuração exibem "Não configurado" — nunca omitidos |
| Sábado exibido no tooltip como "Não configurado" | ✅ Sim | Truncação no mockup era apenas visual |
| Ícone informativo ausente para prestadores sem horário | ✅ Sim | Ícone e tooltip aparecem apenas para prestadores com horário personalizado |
| Configuração via portal Prestador e via Assistência impacta igualmente | ✅ Sim | Ambos os locais de configuração afetam o cálculo do relatório |

---

## QA Notes

- **Divisão por zero resolvida:** Período inteiramente em dias não configurados exibe 0% e 0h sem erro.
- **Filtro "com horário" confirmado:** Inclui prestadores com configuração em apenas 1 dia da semana.
- **Tooltip 7 dias confirmado:** Todos os 7 dias sempre listados; mockup truncado era representação visual incompleta.
- **Parâmetro API:** `config=with` (apenas com horário) / `config=without` (apenas sem) / `config=` vazio = Todos. Endpoint: `__acoes.php?datatable_provider_connectivity&date_from=...&date_to=...&config=...`.
