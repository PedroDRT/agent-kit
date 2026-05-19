# Feature: Relatório de Atendimentos

> **Portal:** Assistência (`/assistencia/relatorios/atendimentos/`)
> **Módulo:** Relatórios
> **Mapeado em:** 2026-04-16 — via Playwright MCP (exploração direta)
> **Contexto de origem:** Teste de urgência — problema de desempenho nos filtros

---

## Resumo

Relatório no portal Assistência que consolida atendimentos com suporte a 29 filtros combinados. Permite pesquisar atendimentos por período, status, cliente, prestador, tipo de serviço, beneficiário, localização, plano, veículo, atributos financeiros e operacionais. Resultados exportáveis em XLS, CSV, PDF e XLS por colunas visíveis.

---

## Critérios de Aceitação

1. Relatório **"Atendimentos"** acessível no menu lateral (módulo Relatórios).
2. Todos os 29 filtros renderizados na abertura da página.
3. Campo **Período** preenchido automaticamente com os últimos **30 dias** como valor padrão.
4. Tabela de resultados (**DataTable**) exibe colunas padrão: Data, Protocolo, CNPJ Cliente, Plano, Tipo de Serviço, Beneficiário, CPF/CNPJ Beneficiário, Observação, Tipo de evento, Segmentação.
5. Botões de exportação (.XLS, .CSV, .XLS colunas visíveis, .PDF, Imprimir) e gestão de colunas (Altere as colunas, Salvar alteração) **desabilitados** antes da pesquisa e **habilitados** após resultados.
6. Filtros combináveis em AND lógico — retorna apenas registros que atendem a todos os critérios.
7. Estado vazio exibe `"Nenhum resultado encontrado!"` na tabela.
8. Botão de limpar filtros restaura o formulário ao estado padrão (mantém Período = últimos 30 dias).
9. Acesso sem autenticação redireciona para `/assistencia/login/` (RN-GLOBAL-001).

---

## Filtros Disponíveis

### Linha 1 — Temporais e Status
| Filtro | ID HTML | Tipo | Valores / Comportamento |
|---|---|---|---|
| Período (de) | `#atendimento_data_de` | input date | Default: hoje − 30 dias |
| Período (até) | `#atendimento_data_ate` | input date | Default: hoje |
| Tipo de serviço | `#rel_atendimento_tipo_servico` | Select2 multi | TODOS + 130+ tipos |
| Status | `#rel_atendimento_status` | Select2 single | TODOS, ABERTO, AGUARDANDO ACEITE, SERVIÇO ACEITO, NA ORIGEM, NO DESTINO, NEGADO, RECUPERADO, NÃO RECUPERADO, EM BUSCA, CANCELADO, FINALIZADO, NEGATIVA, NPS PENDENTE |

### Linha 2 — Entidades Principais
| Filtro | ID HTML | Tipo | Valores |
|---|---|---|---|
| Cliente | `#rel_atendimento_id_cliente` | Listbox multi-select | TODOS + clientes cadastrados |
| Prestadores | `#rel_atendimento_id_prestador` | Select2 single | TODOS + prestadores |
| Tipo de atendimento | `#rel_atendimento_tipo_atendimento` | Select2 single | Assistência, Aviso de colisão, Dúvida, Roubo / Furto |

### Linha 3 — Plano e Localização
| Filtro | ID HTML | Tipo | Valores |
|---|---|---|---|
| Plano | `#rel_atendimento_plano` | Select2 single (filtrado por cliente) | TODOS + planos |
| Cidade | `#rel_atendimento_cidades` | Select2 single (filtrado por estado) | Cidades dinâmicas |
| Estado | `#rel_atendimento_estados` | Select2 single | 27 estados brasileiros |

### Linha 4 — Beneficiário e Responsável
| Filtro | ID HTML | Tipo | Valores |
|---|---|---|---|
| Beneficiário (Nome ou CPF/CNPJ) | `#rel_atendimento_beneficiario` | input text | Busca parcial |
| Responsável (tipo) | `#rel_atendimento_responsavel` | Select2 single | ACIONAMENTO, ACOMPANHAMENTO |
| Responsável (grupo) | `#rel_atendimento_grupos` | Select2 single (filtrado por tipo) | VIP e outros |
| Tipo de Veículo | `#rel_atendimento_tipo` | Select2 single | TODOS, AUTOMOVEL, BARCO, CAMINHAO, CARRETA, HIBRIDO, MOTO, ONIBUS, PICKUP, TRATOR, VAN |

### Linha 5 — Reembolso e Veículo
| Filtro | ID HTML | Tipo | Valores |
|---|---|---|---|
| Reembolso | `#rel_atendimento_reembolso` | Select2 single | SIM, NÃO |
| Veículo (Placa) | `#rel_atendimento_placa` | input text | Busca parcial |
| Número do Protocolo | `#rel_atendimento_protocolo` | input text | Busca exata |

### Linha 6 — Lote e Atributos Financeiros
| Filtro | ID HTML | Tipo | Valores |
|---|---|---|---|
| Número do Lote | `#rel_atendimento_num_faturado` | input text | Busca exata |
| Faturado | `#rel_atendimento_faturado` | Select2 single | SIM, NÃO |
| Parceiro do Cliente | `#rel_atendimento_parceiro` | Select2 single | SIM, NÃO |

### Linha 7 — Usuário e Vigência
| Filtro | ID HTML | Tipo | Valores |
|---|---|---|---|
| Usuários | `#rel_atendimento_usuario` | Select2 single | TODOS + usuários do sistema |
| Código Contrato | `#rel_atendimento_codigo_contrato` | input text | Busca exata |
| Vigência Plano | `#rel_atendimento_vigencia` | Select2 single | FORA DA VIGÊNCIA, DENTRO DA VIGÊNCIA, VIGÊNCIA DECORRIDA |

### Linha 8 — Tipo de Evento e Atributos
| Filtro | ID HTML | Tipo | Valores |
|---|---|---|---|
| Tipo de Evento | `#rel_atendimento_tipo_evento` | Select2 single | ACIDENTE, ANIMAL, CLIENTES ESPECIFICO, COLISAO, FUNERAL, FUNERAL PET, LIMITE, OUTRO CLIENTE, RESIDENCIAL, TODOS OS CLIENTES, VIDA |
| Atendimento com arquivo | `#rel_atendimento_possui_arquivo` | Select2 single | SIM, NÃO |
| Tipo de suporte | `#rel_atendimento_tipo_suporte` | Select2 single | Opções dinâmicas (configuráveis no sistema) |

### Linha 9 — Pagamento e Tags
| Filtro | ID HTML | Tipo | Valores |
|---|---|---|---|
| Atendimento com pagamento a vista | `#rel_atendimento_a_vista` | Select2 single | SIM, NÃO |
| Tags | `#rel_atendimento_tags` | Listbox multi-select | TESTE, TAG REEMBOLSO, REEMBOLSO 2 (e outras configuradas) |
| Acionamento Automático | `#rel_atendimento_acionamento_automatico` | Select2 single | SIM, NÃO |

### Checkboxes
| Filtro | ID HTML | Tipo |
|---|---|---|
| Avulso | `#rel_atendimento_avulso` | checkbox |
| Caso 99 | `#rel_atendimento_caso_99` | checkbox |

---

## Botões de Ação

| Botão | ID / Seletor | Comportamento |
|---|---|---|
| Limpar filtros | `#btn_atendimentos_limpar_filtro` | Sem texto visível (ícone). Reseta formulário. |
| Procurar | `#btn_atendimentos_filtrar` | Texto: "Procurar...". Executa a pesquisa. |
| Exportar .XLS | `.dt-button.buttons-excel` (1º) | Habilitado apenas após pesquisa com resultado |
| Exportar .CSV | `.dt-button.buttons-csv` | Habilitado apenas após pesquisa com resultado |
| Exportar .XLS colunas visíveis | `.dt-button.excelButtonVisible` | Exporta apenas colunas visíveis |
| Exportar .PDF | `.dt-button.buttons-pdf` | Habilitado apenas após pesquisa com resultado |
| Imprimir | `.dt-button.buttons-print` | Habilitado apenas após pesquisa com resultado |
| Altere as colunas | `.dt-button.buttons-colvis` | Habilitado após pesquisa |
| Salvar alteração | `.dt-button:has-text("Salvar alteração")` | Persiste configuração de colunas |

---

## Tabela de Resultados

| Elemento | Seletor |
|---|---|
| DataTable | `#datatable_rel_atendimentos` |
| Estado vazio | `td.dataTables_empty` — texto: `"Nenhum resultado encontrado!"` |
| Paginação | `#datatable_rel_atendimentos_paginate` |
| Próxima página | `#datatable_rel_atendimentos_paginate .next` |
| Página anterior | `#datatable_rel_atendimentos_paginate .previous` |

**Colunas padrão:** Data, Protocolo, CNPJ Cliente, Plano, Tipo de Serviço, Beneficiário, CPF/CNPJ Beneficiário, Observação, Tipo de evento, Segmentação.

**Colunas ordenáveis:** Data (ordenação padrão decrescente), Tipo de Serviço, Tipo de evento.

---

## Escopo de Teste

- Acesso ao módulo (autenticado, sem autenticação)
- Carregamento inicial e estado dos botões
- Todos os 29 filtros individualmente (happy path + negativo)
- Filtros com dependência (Cliente → Plano, Estado → Cidade, Responsável tipo → grupo)
- Combinações de filtros (AND lógico)
- Limpeza de filtros
- Exportação (XLS, CSV, PDF, colunas visíveis)
- Configuração e persistência de colunas
- Paginação e ordenação

---

## Fora do Escopo

- Abertura de atendimentos a partir do relatório
- Portal do Cliente, Prestador, Beneficiário
- Criação, edição ou exclusão de atendimentos

---

## Comportamentos Confirmados

| Comportamento | Confirmado | Caso de teste |
|---|---|---|
| Dependência Cliente → Plano (Select2 dinâmico) | ✅ Sim | CT005.4, CT009.3 |
| Dependência Estado → Cidade (Select2 dinâmico) | ✅ Sim | CT010.2 |
| Troca de Estado limpa campo Cidade | ✅ Sim | CT010.3 |
| Dependência Responsável tipo → grupo | ✅ Sim | CT013.3 |
| Tags lógica AND (múltiplas = deve ter todas) | ✅ Sim | CT028.3 |
| Protocolo ignora período (busca direta) | ✅ Sim | CT017.3 |
| Botões exportação desabilitados quando pesquisa sem resultado | ✅ Sim | CT033.6 |
| Persistência de colunas via "Salvar alteração" | ✅ Sim | CT034.3 — persiste para pesquisas futuras |
| Limpeza mantém Período = últimos 30 dias | ✅ Sim | CT031.2 |
| Botões desabilitados antes de qualquer pesquisa | ✅ Sim | CT002.4 |

---

## QA Notes

- **Problema de desempenho identificado:** Lentidão em produção com período > 6 meses — suspeita de query sem índice ou sem limitação obrigatória de período. Investigar com DBA.
- **Período não é obrigatório no backend:** Campo de período pode ser limpo — limite de resultados sem período definido não foi validado.
- **Select2 multi-select (Tipo de serviço, Tags, Cliente):** Interação via jQuery API necessária em automação — `selectOption()` do Playwright não funciona.
- **Filtros booleanos sem opção "TODOS":** Reembolso, Faturado, Parceiro, Atendimento com arquivo, Pagamento a vista e Acionamento Automático — campo vazio = sem filtro aplicado.
- **Tipo de atendimento sem "TODOS":** Campo `rel_atendimento_tipo_atendimento` não possui opção "TODOS" — campo vazio = sem filtro.
- **Número do Lote usa ID `rel_atendimento_num_faturado`** — nomenclatura divergente do label da UI.
- **Tags confirmado AND:** Múltiplas tags = AND lógico (deve ter todas), não OR.
