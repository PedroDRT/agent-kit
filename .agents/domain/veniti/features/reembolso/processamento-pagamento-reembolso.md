---
type: feature
module: reembolso
layer: feature
related:
  - fluxo-reembolso-completo
  - solicitacao-reembolso
  - calculo-credito
---

# Processamento de Pagamento de Reembolso

> Feature responsável pela análise, aprovação e execução do pagamento de reembolsos no portal interno de Assistência.

## Descrição

Feature responsável pela análise, aprovação e execução do pagamento de reembolsos. Ocorre integralmente no portal interno de Assistência (`/assistencia/reembolso/`). Envolve revisão da documentação recebida, definição do valor final aprovado, agendamento do pagamento e confirmação do crédito.

---

## Entradas

| Campo | Origem | Descrição |
|---|---|---|
| `id_reembolso` | Operação | Identificador do reembolso a processar |
| `valor_final` | Operação | Valor aprovado (pode diferir do solicitado) |
| `valor_excedente` | Sistema/Operação | Calculado automaticamente; editável pelo operador |
| `data_agendamento` | Operação | Data prevista para pagamento |
| `observacao` | Operação | Justificativa de aprovação/recusa |
| `situacao` | Sistema | Transição de status |
| `numero_documento` | Operação | Número do documento financeiro gerado |

### Modal de Aprovação

Ao clicar em aprovar o reembolso, o sistema exibe um modal com os campos:

- **Valor Solicitado (R$)** — valor informado no cadastro
- **Valor Excedente (R$)** — calculado automaticamente quando há limite de reembolso; editável manualmente
- **Valor Aprovado (R$)** — definido pelo operador
- **Diferença (R$)**

**Cálculo automático do valor excedente:**
```
Valor excedente = Valor solicitado − Limite de reembolso
```
O campo Limite de reembolso não é exibido no modal de aprovação, mas seu valor é usado no cálculo. O operador pode editar manualmente o valor excedente resultante.

## Saídas

- Status do reembolso atualizado
- Lançamento em contas a pagar (`contas_pagar`) criado ou atualizado
- Notificação enviada ao beneficiário sobre aprovação, agendamento ou recusa
- Ocorrência registrada em caso de recusa
- Registro de auditoria no log do sistema
- Status final `PAGO` marca o encerramento do ciclo financeiro

---

## Regras de Negócio

- Somente usuários com permissão de `reembolso` no portal Assistência podem processar pagamentos
- `valor_final` não pode ser superior ao `valor` originalmente solicitado
- A recusa gera uma ocorrência obrigatória (`html/assistencia/reembolso/ocorrencias.php`)
- Um reembolso `RECUSADO` encerra o ciclo — não pode ser reativado sem novo processo
- `PAGO` é estado terminal — não admite reversão
- O agendamento requer `data_agendamento` válida e no futuro
- Arquivos de nota podem ser gerenciados em `html/assistencia/reembolso/arquivos.php`
- Quando o valor excedente é editado manualmente, a alteração deve ser registrada nas ocorrências do reembolso

## Passos de Processamento

### 1. Análise da Documentação (NOTA_RECEBIDA)
- Operação acessa `/assistencia/reembolso/editar.php`
- Visualiza nota fiscal enviada (link S3 assinado, 10 min de validade)
- Confere dados bancários/PIX do recebedor
- Decide: Aprovar, Agendar ou Recusar

### 2. Aprovação (→ APROVADO)
- Operação abre modal de aprovação
- Se existir limite de reembolso cadastrado, o sistema calcula e preenche automaticamente o valor excedente
- Operação define `valor_final` (igual ou inferior ao solicitado)
- Sistema cria lançamento em contas a pagar

### 3. Agendamento (→ AGENDADO)
- Define `data_agendamento` para pagamento futuro
- Beneficiário é notificado com a data prevista

### 4. Geração do Pagamento (→ PAGAMENTO_GERADO)
- Integração com sistema financeiro gera `numero_documento`
- Pagamento enfileirado para processamento bancário

### 5. Confirmação (→ PAGO)
- Confirmação do débito bancário recebida
- Ciclo encerrado, reembolso arquivado

### 6. Recusa (→ RECUSADO)
- Operação preenche justificativa
- Sistema cria ocorrência vinculada ao reembolso
- Beneficiário notificado sobre a recusa e motivo

## Casos de Borda

- Valor final definido como zero (reembolso de cortesia negado financeiramente)
- Pagamento agendado para data de feriado bancário
- Reembolso aprovado mas pagamento devolvido pelo banco (chave PIX inválida)
- Dois operadores processando o mesmo reembolso simultaneamente
- Nota fiscal com CNPJ de empresa não cadastrada
- Agendamento de data muito distante sem comunicação ao beneficiário

---

## Dependências

- **Portal**: `html/assistencia/reembolso/` (editar.php, arquivos.php, ocorrencias.php)
- **Contas a pagar**: `html/assistencia/contas_pagar/`
- **Modelos**: `src/Models/Reembolso.php`
- **Storage**: AWS S3 para arquivos de nota fiscal
- **Notificações**: SMS, e-mail via PHPMailer
- **Banco**: tabelas `reembolsos`, `contas_pagar`, `ocorrencias`, `arquivos`

## Flows Relacionados

- [[fluxo-reembolso-completo]]

## Features Relacionadas

- [[solicitacao-reembolso]]
- [[calculo-credito]]
