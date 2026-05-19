---
type: feature
module: relatorios
layer: feature
related: []
---

# Feature: Melhorias no Relatório de Conectividade

> Ajusta o relatório de conectividade de prestadores para respeitar a configuração personalizada de horário de trabalho nos cálculos de conectividade.

## Descrição

Ajustar o relatório de conectividade de prestadores para que os indicadores respeitem a configuração personalizada de horário de trabalho quando existente, garantindo maior precisão e aderência à realidade operacional.

**Módulo:** Relatórios — Assistência
**URL:** `/assistencia/relatorios/provider_connectivity/`
**Portal:** Assistência
**Versão do documento:** V1 — 2025

---

## Entradas

### Filtros Disponíveis

| Campo | Detalhe |
|---|---|
| Período | Datas de início e fim do relatório |
| Prestadores | Select2 com busca |
| Cidade | Select2 com busca |
| Estado | Select2 com busca |
| Categoria de Serviço | Select2 |
| Segmento de Serviço | Select2 |
| Tipo de Serviço | Select2 |
| Conexão | ONLINE / OFFLINE |
| **Prestadores com horário configurado (NOVO)** | Select2 — opções: vazio (Todos) / `APENAS COM HORÁRIO CONFIGURADO` / `APENAS SEM HORÁRIO CONFIGURADO` |

**Parâmetro API do novo filtro:** `config=with` / `config=without` / `config=` (vazio = Todos)

## Saídas

- Datatable com colunas: Nome, CNPJ, Status, Conectividade (tempo conectado), % Conexão
- Ícone informativo ao lado do nome para prestadores com horário configurado
- Tooltip com horários por dia da semana para prestadores com horário personalizado
- Gráfico de conectividade atualizado

---

## Regras de Negócio

### Processo Anterior (Before)

- O relatório considerava **24 horas por dia** para todos os prestadores, independentemente de possuírem configuração de horário.
- A configuração de horário de trabalho **não impactava** o relatório.

### Novo Processo (After)

#### Regra de Cálculo da Conectividade

Para cada prestador no relatório, o sistema verifica se existe configuração personalizada de horário de trabalho.

| Cenário | Regra de Cálculo |
|---|---|
| **Sem horário configurado** | Mantém regra atual: 24 horas por dia para todos os dias do período filtrado |
| **Com horário configurado** | Considera exclusivamente os intervalos configurados por dia da semana |
| **Dia da semana não configurado** | Dia completamente desconsiderado do cálculo — não impacta negativamente o percentual |

**Definição de "configuração ativa":** Prestador com ao menos 1 dia da semana configurado é considerado "com horário configurado".

**Edge case — período inteiramente em dias não configurados:** Exibir 0% de conexão e 0h de conectividade, sem erro.

#### Tooltip Informativo na Listagem

Para prestadores com configuração personalizada de horário:
- Exibir **ícone informativo** ao lado do nome no datatable
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
- Horário reflete exatamente o que estiver configurado
- Dia não configurado → exibir nome do dia + "Não configurado"
- Todos os 7 dias sempre listados (nenhum omitido)
- **Não exibir** tooltip para prestadores sem horário configurado

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

## Descobertas da Exploração Real

- **Label divergente:** O filtro está implementado como "Prestadores com horário configurado:" — diferente do spec ("Prestadores com horário de trabalho")
- **Opções em uppercase:** UI exibe "APENAS COM HORÁRIO CONFIGURADO" e "APENAS SEM HORÁRIO CONFIGURADO" (spec usava minúsculas)
- **Endpoint API:** `__acoes.php?datatable_provider_connectivity&date_from=...&date_to=...&config=...`
- **Parâmetro do novo filtro:** `config=with` / `config=without` / `config=` (vazio = Todos)

## Fora do Escopo

- Cadastro/edição de horário de trabalho do prestador (funcionalidade já existente)
- Outros relatórios além do de conectividade
- Impacto no cálculo de outros módulos

---

## Notas de QA

- **Divisão por zero resolvida:** Período inteiramente em dias não configurados exibe 0% e 0h sem erro.
- **Filtro "com horário" confirmado:** Inclui prestadores com configuração em apenas 1 dia da semana.
- **Tooltip 7 dias confirmado:** Todos os 7 dias sempre listados; mockup truncado era representação visual incompleta.
- **Parâmetro API:** `config=with` (apenas com horário) / `config=without` (apenas sem) / `config=` vazio = Todos.

## Dependências

- Configuração de horário de trabalho pode ser definida em dois locais:
  - Portal Prestador: Configuração > Geral
  - Portal Assistência: Rede > Prestadores > Editar
- **Dados de teste necessários:**
  - Prestador A: horário configurado em todos os 7 dias
  - Prestador B: horário configurado apenas em alguns dias (sábado e domingo sem configuração)
  - Prestador C: sem nenhum horário configurado
