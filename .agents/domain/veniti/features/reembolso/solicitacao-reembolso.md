---
type: feature
module: reembolso
layer: feature
related:
  - fluxo-reembolso-completo
  - processamento-pagamento-reembolso
  - perguntas-reembolso
  - tags-reembolso
  - ocorrencias-reembolso
  - status-personalizados-reembolso
  - criacao-atendimento
  - portal-self-service-beneficiario
---

# Solicitação de Reembolso

> Permite que um beneficiário solicite o reembolso de despesas realizadas com um serviço executado de forma autônoma, iniciado pela operação e concluído pelo beneficiário via portal self-service.

## Descrição

Permite que um beneficiário solicite o reembolso de despesas realizadas com um serviço de assistência executado de forma autônoma (sem acionamento de prestador Veniti). A solicitação é iniciada pela operação interna e concluída pelo próprio beneficiário via portal self-service (`/reembolso`).

---

## Entradas

**Dados iniciais (preenchidos pela operação):**

| Campo | Tipo | Descrição |
|---|---|---|
| `id_atendimento` | integer | Atendimento vinculado |
| `id_motivo_reembolso` | integer | Motivo do reembolso (cadastro em `/assistencia/motivo_reembolso/`) |
| `nome_solicitante` | string | Nome do solicitante (auto-preenchido do atendimento) |
| `telefone_solicitante` | string | Telefone do solicitante (auto-preenchido do atendimento) |
| `limite_reembolso` | decimal | Limite de reembolso (opcional, informativo; base do cálculo de excedente na aprovação) |
| `valor` | decimal | Valor solicitado pelo beneficiário |
| `observacao` | text | Observações internas |

**Dados financeiros (preenchidos pelo beneficiário via portal):**

| Campo | Tipo | Descrição |
|---|---|---|
| `recebedor_nome` | string | Nome do recebedor |
| `recebedor_documento` | string | CPF ou CNPJ |
| `recebedor_email` | string | E-mail para notificações |
| `recebedor_forma_pagamento` | string | PIX, TED, DOC |
| `recebedor_pix_tipo` | string | Tipo da chave PIX (CPF, CNPJ, email, telefone, aleatória) |
| `recebedor_pix_chave` | string | Chave PIX |
| `recebedor_agencia` | string | Agência bancária (para TED/DOC) |
| `recebedor_conta` | string | Número da conta (para TED/DOC) |
| `recebedor_tipo_conta` | string | Corrente ou Poupança |
| `id_arquivo_nota` | integer | ID do arquivo de nota fiscal/recibo enviado |

### Modal de Cadastro

Acesso: `Atendimento → Editar → Ação → Reembolso`. Após confirmar no modal de confirmação, o sistema exibe o modal de cadastro com os seguintes campos:

1. **Motivo do Reembolso** (obrigatório)
2. **Nome do Solicitante** (obrigatório, preenchido automaticamente das informações de solicitante do atendimento)
3. **Telefone do Solicitante** (obrigatório, preenchido automaticamente)
4. **Limite reembolso (R$)** (opcional, aceita apenas valores numéricos; valor armazenado para uso na aprovação)
5. **Valor solicitado (R$)**
6. **Observação** (opcional)
7. **Seção PERGUNTAS** — exibida quando existem perguntas configuradas em Configurações → Financeiro → Perguntas de reembolso que correspondem ao contexto do atendimento

## Saídas

- Registro criado na tabela `reembolsos` com status `NOTA_PENDENTE`
- Link de acesso gerado para o beneficiário (short URL via `html/__inc/funcoes.php`)
- E-mail enviado ao beneficiário com instruções e link
- Após envio de documentos: status avança para `NOTA_RECEBIDA`
- Após processamento: lançamento em contas a pagar (`contas_pagar`) gerado

---

## Regras de Negócio

- Um reembolso só pode ser criado para atendimentos existentes e vinculados ao tenant (`id_veniti`)
- O `valor_final` pode diferir do `valor` solicitado (aprovação parcial)
- Somente reembolsos em status `NOTA_RECEBIDA` podem ser editados pelo beneficiário (`get_refund_edit`)
- A rejeição (`RECUSADO`) gera automaticamente uma ocorrência com justificativa
- O campo `pagamento_para` padrão é `BENEFICIARIO`; pode ser terceiro em casos especiais
- Links de acesso expiram — o portal exibe `link_expired.php` quando o link não é mais válido
- Arquivos de nota fiscal são armazenados no AWS S3 com URL assinada de 10 minutos
- O valor `valor_final` é preenchido pela operação no momento da aprovação
- O campo `limite_reembolso` é opcional; quando preenchido, é usado para calcular o valor excedente no modal de aprovação
- Nome e telefone do solicitante são preenchidos automaticamente do atendimento, mas podem ser editados manualmente

## Estados

```
NOTA_PENDENTE → NOTA_RECEBIDA → AGENDADO → APROVADO → PAGAMENTO_GERADO → PAGO
                              ↘ RECUSADO (em qualquer estado após NOTA_RECEBIDA)
```

| Status | Badge UI | Descrição |
|---|---|---|
| `NOTA_PENDENTE` | "AGUARDANDO NOTA" (warning) | Reembolso criado, aguardando envio de documentação pelo beneficiário |
| `NOTA_RECEBIDA` | "DOCUMENTAÇÃO RECEBIDA" | Documentos enviados, em análise |
| `AGENDADO` | "REEMBOLSO AGENDADO" | Pagamento agendado para data específica |
| `APROVADO` | "REEMBOLSO APROVADO" | Aprovado, aguardando geração de pagamento |
| `RECUSADO` | "REEMBOLSO RECUSADO" | Negado com registro de ocorrência |
| `PAGAMENTO_GERADO` | "PAGAMENTO GERADO" (info) | Pagamento gerado no sistema financeiro |
| `PAGO` | "REEMBOLSO PAGO" (success) | Pagamento confirmado |

### Header do Reembolso

O cabeçalho da tela do reembolso exibe: Número, Protocolo, Tipo, Data de Criação, **Nome do solicitante**, **Telefone do solicitante** e **Beneficiário**.

### Card "INFORMAÇÕES DO REEMBOLSO"

O card principal exibe e permite editar:
- Número do documento, CPF/CNPJ emissor, Motivo, Pagamento para
- Valor solicitado, Valor aprovado
- **Status adicional** — select com status personalizados configurados em Configurações → Financeiro → Status reembolso
- **Tags** — seletor filtrado por tipo de evento Reembolso e TODOS
- Observação

## Casos de Borda

- Beneficiário tenta acessar o link após expiração
- Nota fiscal enviada com valor diferente do solicitado
- Reembolso criado para atendimento que já possui reembolso ativo
- Dados bancários inválidos (agência/conta inexistente)
- Beneficiário altera dados bancários após agendamento do pagamento
- Upload de arquivo corrompido ou em formato inválido
- Chave PIX recusada pelo banco receptor

---

## Dependências

- **Modelos**: `src/Models/Reembolso.php`
- **Portais**: `html/reembolso/` (self-service), `html/assistencia/reembolso/` (gestão interna)
- **Armazenamento**: AWS S3 (`src/Models/S3.php`) para notas fiscais
- **Notificações**: SMS/e-mail via `src/Models/SMS.php`, PHPMailer
- **Contas a pagar**: `html/assistencia/contas_pagar/`
- **Banco**: tabelas `reembolsos`, `arquivos`, `motivo_reembolso`, `contas_pagar`

## Flows Relacionados

- [[fluxo-reembolso-completo]]

## Features Relacionadas

- [[processamento-pagamento-reembolso]]
- [[perguntas-reembolso]]
- [[tags-reembolso]]
- [[ocorrencias-reembolso]]
- [[status-personalizados-reembolso]]
- [[criacao-atendimento]]
- [[portal-self-service-beneficiario]]
