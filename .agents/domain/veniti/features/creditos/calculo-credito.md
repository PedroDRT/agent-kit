# Cálculo de Crédito

## Description

Sistema responsável por calcular o valor de crédito ou débito gerado por um atendimento finalizado. O cálculo determina quanto o cliente (seguradora) deve ao Veniti pelo serviço prestado, ou quanto deve ser reembolsado em casos de cancelamento. Usa padrão Strategy com múltiplos métodos de cálculo configuráveis por plano.

## Inputs

| Parâmetro | Origem | Descrição |
|---|---|---|
| `id_atendimento` | Sistema | Atendimento finalizado |
| `id_acionamento` | Sistema | Acionamento correspondente |
| `tipo_acionamento` | Acionamento | WEB, automático, etc. |
| `km_total` | Acionamento | Quilometragem total percorrida |
| `km_acionamento` | Acionamento | Km de deslocamento da base |
| `tariffs` / `tarifas` | Acionamento/Plano | Tabela de tarifas aplicáveis |
| `excessPlanFees` | Acionamento | Taxas de excedente do plano |
| `tipo_servico` | Atendimento | Tipo de serviço prestado |
| `id_plano` | Atendimento | Plano do beneficiário |
| `metodo_calculo` | Plano | Estratégia de cálculo configurada |

## Outputs

- Registro de crédito/débito criado na tabela `clientes_contratos_creditos`
- Valor disponível no extrato de créditos do cliente (`/cliente/creditos/`)
- Relatório de métricas atualizado (`CalcularMetricas`)
- Disponível na listagem interna de créditos (`/assistencia/creditos/`)

## Calculation Methods (Strategy Pattern)

### ValorAtendimento (por atendimento)
- Valor fixo por tipo de serviço, independente da distância
- Configurado no cadastro do plano por tipo de serviço

### ValorTabela (por tabela tarifária)
- Valor calculado com base em `km_total` e tabela de tarifas
- Considera faixas de distância com preços escalonados
- Suporta taxa adicional por km excedente

### ValorManual (override manual)
- Valor definido manualmente pela operação
- Usado em casos excepcionais ou negociações especiais

## Business Rules

- O método de cálculo é configurado por plano (`id_plano`) e pode variar por tipo de serviço
- Valores podem ser **positivos** (crédito — cliente deve pagar) ou **negativos** (débito — Veniti deve reembolsar)
- `CalcularValorCreditoSocorreAe.php` é uma variante especializada para a marca "Socorre A&E"
- `MetodoCalculoResolver.php` determina qual estratégia usar com base na configuração do plano
- `MetodoCalculoRegister.php` registra todas as estratégias disponíveis (padrão Registry Pattern)
- Excedente de km (`excessPlanFees`) é calculado separadamente e somado ao crédito
- O cálculo é disparado automaticamente após o acionamento entrar em status `FINALIZADO`
- Créditos podem ser listados e filtrados via `ListarCreditos.php`
- Métricas de faturamento consolidadas via `CalcularMetricas.php`

## Use Cases

| Arquivo | Responsabilidade |
|---|---|
| `src/UseCases/CalcularValorCredito.php` | Orquestrador principal do cálculo |
| `src/UseCases/CalcularValorCreditoSocorreAe.php` | Variante para Socorre A&E |
| `src/UseCases/CalcularMetricas.php` | Cálculo de métricas de faturamento |
| `src/UseCases/ListarCreditos.php` | Listagem e filtro de créditos |
| `src/UseCases/MetodoCalculo/ValorAtendimento.php` | Estratégia por atendimento |
| `src/UseCases/MetodoCalculo/ValorTabela.php` | Estratégia por tabela |
| `src/UseCases/MetodoCalculo/ValorManual.php` | Estratégia manual |

## Edge Cases

- Atendimento finalizado sem acionamento vinculado (cálculo não disparado)
- Plano sem método de cálculo configurado para o tipo de serviço
- `km_total` zerado ou negativo (erro de geolocalização no campo)
- Tarifa desatualizada no momento do cálculo (plano alterado após acionamento criado)
- Crédito gerado duplicado por chamada dupla do trigger de finalização
- Excedente de km para beneficiário que não autorizou pagamento

## Dependencies

- **Use Cases**: `src/UseCases/` (todos os arquivos de cálculo)
- **Modelos**: `src/Models/Acionamento.php`, `src/Models/Creditos.php`
- **Portais**: `html/assistencia/creditos/`, `html/cliente/creditos/`
- **Banco**: `clientes_contratos_creditos`, `clientes_planos`

## Related Flows

- [[fluxo-calculo-credito]]
- [[fluxo-atendimento-completo]]

## Related Features

- [[ciclo-vida-acionamento]]
- [[processamento-pagamento-reembolso]]

---

## Impacto em Outros Módulos

### Arquitetura Financeira: Dois Fluxos Independentes

O módulo de Créditos é **independente** dos módulos de Fechamento e Faturamento. O sistema possui dois modelos de cobrança paralelos:

| Modelo | Tabela central | Para quê serve |
|---|---|---|
| **Créditos** | `clientes_contratos_creditos` | Controle de saldo do cliente (seguradora) com o Veniti |
| **Fechamento Cliente** | `fechamento_clientes` + `fechamento_clientes_produtos` | Faturamento mensal por número de beneficiários/veículos ativos |

> Créditos e Fechamento Cliente são modelos **mutuamente exclusivos por contrato** — um contrato usa um ou outro.

### Relação com Fechamento Prestador / Lote / Contas a Pagar

Os créditos **não alimentam diretamente** `fechamento_prestador`, `fechamento_prestador_itens` nem `contas_pagar`. A relação é **indireta via acionamento**:

```
Acionamento FINALIZADO
    ├── gera DÉBITO em clientes_contratos_creditos  (cobrança ao cliente)
    └── é incluído em fechamento_prestador_itens    (pagamento ao prestador)
```

Esses dois processos ocorrem de forma independente e assíncrona — o crédito é gerado automaticamente, o lote requer ação manual da Assistência.

### Campo `situacao_fechamento` nos Créditos

A coluna "Situação Fechamento" exibida na listagem de créditos **não representa o fechamento com o cliente**. Ela indica o status da negociação de tarifa com o **prestador**:
- `FECHADO` → `atendimentos_dispatch_tarifas.situacao IN ('ACEITO', 'FECHADO')` — negociação encerrada
- `EM ABERTO` → negociação de tarifa com prestador ainda não fechada

### Bloqueio de Novos Atendimentos por Crédito

O endpoint `GET /assistencia/api/credit/verify/` é chamado antes de criar novos atendimentos:
- Calcula `porcentagem_utilizacao` do contrato no mês corrente
- Se `> 90%` → retorna `is_blocked = 1` (bloqueia novos atendimentos)
- Se `forma_credito = 'POS'` (pós-pago) → nunca bloqueia
- Se plano sem `ativar_credito` → nunca bloqueia

### Fórmula de Utilização

```
Modo padrão (porcentagem_calculo = 0):
  utilização = (débitos_mês + taxa_negociação) / (créditos + saldo_anterior) × 100

Modo valor fixo (porcentagem_calculo = 1):
  utilização = MAX(valor_credito - saldo_atual, 0) / valor_credito × 100
```

### Cronjob de Aviso de Crédito

`html/__cronjob/client_credit/client_credit.php` executa diariamente:
- Calcula utilização de todos os contratos com `ativar_credito = 1`
- Envia email quando utilização >= `porcentagem_aviso` configurado no plano
- Throttle de 24h via `clientes_planos.email_enviado_em`
- Email diferencia: crédito 100% esgotado vs. percentual atingido

### Gestão Manual pela Assistência

Via `/assistencia/creditos/`, a Assistência pode:
- **Adicionar crédito manual**: Data + Valor + Descrição → cria entrada `tipo=CREDITO`
- **Adicionar débito manual**: Data + Valor + Descrição → cria entrada `tipo=DEBITO`
- **Incluir atendimento**: vincula atendimento existente ao saldo do contrato
- **Editar crédito/débito manual**: altera data, valor e descrição
- **Excluir crédito/débito**: soft-delete via `excluido_em`

> Apenas créditos **manuais** (sem `id_atendimento`) são editáveis. Créditos automáticos são gerenciados pelo ciclo de vida do acionamento.

### Ciclo de Vida do Débito Automático

| Evento | Comportamento |
|---|---|
| Acionamento `FINALIZADO` | Débito criado automaticamente |
| Acionamento cancelado com `zerar_valores = 0` | Débito **permanece** (cancelamento com custo) |
| Acionamento cancelado com `zerar_valores = 1` | Débito excluído (`excluido_em = NOW()`) |
| Tarifa com `qtd = 0` e `total = 0` | Registro deletado fisicamente da tabela |

### Visão do Cliente no Portal

O cliente (seguradora) visualiza seus créditos em `/cliente/creditos/` (READ-ONLY):
- Listagem de contratos com Código, Filial, Data último crédito, Balanço, Valor fixo, Utilização, Status
- Detalhe por contrato com navegação mensal
- Métricas: BALANÇO, CREDITADO, DEBITADO, NO CONTRATO, PARA REPOSIÇÃO, VALOR FIXO
- **Não pode** contestar, editar ou solicitar ajuste de crédito pelo portal
- Acesso controlado por `$_SESSION['AUTH_CLIENTE_USER_VISUALIZAR_CREDITOS']`

## Schema da Tabela `clientes_contratos_creditos`

| Campo | Tipo | Descrição |
|---|---|---|
| `id` | INT PK | Identificador |
| `id_atendimento` | INT nullable | FK atendimentos — nulo para lançamentos manuais |
| `id_dispatch` | INT nullable | FK atendimentos_dispatch |
| `id_clientes_contrato` | INT NOT NULL | FK clientes_planos (contrato do cliente) |
| `descricao` | TEXT | Protocolo do atendimento ou descrição manual |
| `valor` | DECIMAL(10,2) | Valor do lançamento |
| `valor_fixo` | DECIMAL(10,2) | Valor de referência para cálculo por tabela |
| `tipo` | ENUM('DEBITO','CREDITO') | Tipo do lançamento |
| `date` | DATE | Data de competência do lançamento |
| `criado_em` | TIMESTAMP | Data de criação |
| `excluido_em` | TIMESTAMP nullable | Soft-delete |
| `id_dispatch_tarifa` | INT nullable | FK atendimentos_dispatch_tarifas |
| `tipo_tarifa` | INT | Tipo de tarifa aplicada |
| `km_saida` | INT | KM de saída da base |
| `qtd` | DECIMAL(10,2) | Quantidade para cálculo por tarifa |
