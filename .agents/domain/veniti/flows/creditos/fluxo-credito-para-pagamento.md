# Fluxo Downstream: Crédito → Fechamento → Lote → Pagamento

## Description

Documenta como o valor de um atendimento finalizado percorre dois caminhos financeiros paralelos e independentes: o **débito de crédito** (cobrança ao cliente/seguradora) e o **lote de faturamento** (pagamento ao prestador). Ambos são disparados pelo mesmo acionamento `FINALIZADO`, mas seguem fluxos totalmente separados.

---

## Visão Geral da Arquitetura

```
Acionamento → FINALIZADO
    │
    ├─── [Automático] CalcularValorCredito.php
    │         └─→ DÉBITO em clientes_contratos_creditos
    │                  └─→ Atualiza saldo/balanço do contrato do cliente
    │                  └─→ Cronjob avisa cliente se utilização >= threshold
    │                  └─→ API /credit/verify bloqueia se utilização > 90%
    │
    └─── [Manual pela Assistência] Fechamento Prestador
              └─→ fechamento_prestador (Lote)
                       └─→ fechamento_prestador_itens (inclui acionamento)
                       └─→ contas_pagar (gerado ao status AGENDADO)
```

---

## Fluxo A: Crédito (Cliente → Veniti)

### Passo 1 — Geração do Débito Automático
- **Trigger**: Acionamento muda para `FINALIZADO`
- **Ator**: Sistema (automático)
- **Ação**: `CalcularValorCredito.php` cria registro tipo `DEBITO` em `clientes_contratos_creditos`
- **Dados**: `id_atendimento`, `id_dispatch`, `id_clientes_contrato`, valor calculado pela estratégia configurada

### Passo 2 — Atualização do Balanço do Contrato
- Balanço = (Créditos do período + Saldo anterior) − Débitos do período
- Visible no portal Assistência: `/assistencia/creditos/` (modal por contrato, navegação mensal)
- Visible no portal Cliente: `/cliente/creditos/contrato.php` (read-only)

### Passo 3 — Monitoramento de Utilização
- Cronjob `client_credit.php` executa diariamente
- Calcula `porcentagem_utilizacao` para todos os contratos com `ativar_credito = 1`
- Envia email de aviso se `utilização >= porcentagem_aviso` (config do plano)
- Throttle: email a cada 24h máximo (`email_enviado_em`)

### Passo 4 — Bloqueio Preventivo
- Antes de criar atendimento, o sistema chama `GET /assistencia/api/credit/verify/?id_plano=X&id_veniti=Y`
- Se `porcentagem_utilizacao > 90%` → retorna `is_blocked = 1`
- Exceções: `forma_credito = 'POS'` (pós-pago) ou `ativar_credito = 0` → nunca bloqueia

### Passo 5 — Reposição de Créditos (Manual)
- Operador da Assistência acessa `/assistencia/creditos/`
- Abre modal do contrato → "Cadastrar Crédito"
- Informa: Data, Valor, Descrição
- Cria registro tipo `CREDITO` em `clientes_contratos_creditos`
- Balanço atualizado imediatamente

---

## Fluxo B: Lote (Veniti → Prestador)

### Passo 1 — Elegibilidade para Lote
- Acionamentos com `situacao = 'FINALIZADO'` ficam elegíveis
- Acionamentos com `zerar_valores = 1` e `situacao = 'CANCELADO'` são excluídos
- Um acionamento só pode aparecer em **um único lote** (unicidade)

### Passo 2 — Criação do Lote
- Assistência acessa `/assistencia/faturamento/prestador/`
- Seleciona prestador + período de referência
- Sistema agrupa acionamentos finalizados no período
- Lote criado em `fechamento_prestador` com itens em `fechamento_prestador_itens`
- Status inicial: `ABERTO`

### Passo 3 — Negociação (Opcional)
- Assistência pode iniciar negociação de tarifas: status → `EM NEGOCIAÇÃO`
- Prestador pode apresentar contraproposta: status → `CONTRAPROPOSTA`
- Quando acordado: status → `ACEITO`

### Passo 4 — Agendamento de Pagamento
- Assistência define data e forma de pagamento: status → `AGENDADO`
- Sistema gera lançamento em `contas_pagar`
- Prestador visualiza em `/prestador/faturamento/`

### Passo 5 — Confirmação de Pagamento
- Assistência confirma pagamento manual: status → `REALIZADO`
- `time_recebimento` registrado — sem integração bancária automática

---

## Relação entre Fluxos A e B

| Aspecto | Fluxo A (Crédito) | Fluxo B (Lote) |
|---|---|---|
| **Direção** | Cliente → Veniti (cobrança) | Veniti → Prestador (pagamento) |
| **Tabela** | `clientes_contratos_creditos` | `fechamento_prestador` |
| **Trigger** | Automático no FINALIZADO | Manual pela Assistência |
| **Sincronismo** | Imediato | Periódico (mensal tipicamente) |
| **Status compartilhado** | Nenhum | Nenhum |
| **Dependência mútua** | Nenhuma | Nenhuma |

> O campo **"Situação Fechamento"** exibido na listagem de créditos indica o status da tarifa do **prestador** (`atendimentos_dispatch_tarifas.situacao`), não o fechamento com o cliente. É apenas uma referência cruzada para o operador.

---

## Fluxo Alternativo: Débito Manual

Quando a Assistência precisa registrar um custo não vinculado a atendimento automatizado:

1. Assistência acessa `/assistencia/creditos/` → abre modal do contrato
2. Clica em "Cadastrar Débito"
3. Informa: Data, Valor, Descrição
4. Cria registro `tipo = 'DEBITO'` sem `id_atendimento` nem `id_dispatch`
5. Balanço atualizado imediatamente

---

## Fluxo Alternativo: Cancelamento após Crédito Gerado

| Cenário | Comportamento |
|---|---|
| Acionamento cancelado com `zerar_valores = 0` | Débito **permanece** no saldo — cancelamento tem custo |
| Acionamento cancelado com `zerar_valores = 1` | Débito excluído via `excluido_em = NOW()` |
| Tarifa zerada (`qtd = 0`, `total = 0`) | Registro deletado fisicamente |

---

## Entry Points

- Acionamento entra em status `FINALIZADO` (automático)
- Operador cria crédito/débito manual (manual)

## Exit Points

- **Crédito**: Saldo do contrato atualizado, visível nos portais Assistência e Cliente
- **Lote**: Pagamento realizado ao prestador, lançamento `PAGO` em `contas_pagar`

## Related Features

- [[calculo-credito]]
- [[fluxo-calculo-credito]]
- [[fechamento-financeiro]]
- [[agrupamento-lote]]
- [[ciclo-vida-acionamento]]
